# Auth Patterns

This document defines authentication and authorization patterns for AI-assisted development, providing concrete implementations for common authentication scenarios.

---

## 1. Authentication Architecture

### 1.1 Authentication Flow Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Authentication Flow                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌──────────┐     ┌────────────────┐     ┌──────────────────┐      │
│   │  Client  │────▶│ Auth Controller │────▶│ Auth Service     │      │
│   │          │◀────│                 │◀────│ (Domain Layer)   │      │
│   └──────────┘     └────────────────┘     └────────┬─────────┘      │
│         │                                              │              │
│         │                                              ▼              │
│         │                                    ┌──────────────────┐      │
│         │                                    │ Token Service    │      │
│         │                                    │ (Infrastructure) │      │
│         │                                    └────────┬─────────┘      │
│         │                                              │              │
│         ▼                                              ▼              │
│   ┌──────────┐                               ┌──────────────────┐      │
│   │  Token   │◀─────── Access Token ────────│ Validation       │      │
│   │ Storage  │                               │ (JWT/Paseto)     │      │
│   └──────────┘                               └──────────────────┘      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 Authentication Interface Definitions

```typescript
// Core authentication interfaces
interface AuthenticationProvider {
  authenticate(credentials: Credentials): Promise<Result<AuthenticationResult, AuthenticationError>>;
  refresh(refreshToken: RefreshToken): Promise<Result<Session, SessionError>>;
  revoke(token: Token): Promise<Result<void, RevocationError>>;
  validate(token: Token): Promise<TokenValidationResult>;
}

interface Credentials {
  email: string;
  password: string;
  // Optional: MFA token
  mfaToken?: string;
}

interface AuthenticationResult {
  user: User;
  accessToken: AccessToken;
  refreshToken: RefreshToken;
  expiresAt: DateTime;
  mfaRequired?: MFAMethod;
}

interface Session {
  userId: UserId;
  accessToken: AccessToken;
  refreshToken: RefreshToken;
  expiresAt: DateTime;
  createdAt: DateTime;
}

interface Token {
  readonly value: string;
  readonly type: TokenType;
  readonly expiresAt: DateTime;
}

type TokenType = 'access' | 'refresh' | 'id';
```

---

## 2. JWT Authentication Pattern

### 2.1 JWT Token Service

```typescript
// JWT token generation and validation
interface TokenService {
  generateAccessToken(user: User): AccessToken;
  generateRefreshToken(user: User): RefreshToken;
  generateIdToken(user: User): IdToken;
  validateAccessToken(token: string): Promise<Result<TokenClaims, TokenError>>;
  validateRefreshToken(token: string): Promise<Result<TokenClaims, TokenError>>;
}

class JwtTokenService implements TokenService {
  constructor(
    private readonly secret: string,
    private readonly config: JwtConfig,
    private readonly clock: Clock
  ) {}

  generateAccessToken(user: User): AccessToken {
    const now = this.clock.now();
    const expiresAt = now.add(this.config.accessTokenExpiresIn);

    const claims: TokenClaims = {
      sub: user.id.toString(),
      email: user.email.value,
      roles: user.roles.map(r => r.name),
      permissions: user.getAllPermissions().map(p => p.toString()),
      type: 'access',
      iat: now.toUnixTimestamp(),
      exp: expiresAt.toUnixTimestamp(),
      jti: this.generateJti()
    };

    const token = this.sign(claims);
    return new AccessToken(token, expiresAt);
  }

  generateRefreshToken(user: User): RefreshToken {
    const now = this.clock.now();
    const expiresAt = now.add(this.config.refreshTokenExpiresIn);

    const claims: TokenClaims = {
      sub: user.id.toString(),
      type: 'refresh',
      iat: now.toUnixTimestamp(),
      exp: expiresAt.toUnixTimestamp(),
      jti: this.generateJti()
    };

    const token = this.sign(claims);
    return new RefreshToken(token, expiresAt);
  }

  async validateAccessToken(token: string): Promise<Result<TokenClaims, TokenError>> {
    try {
      const decoded = this.decode(token);
      
      // Verify signature
      if (!this.verifySignature(token)) {
        return Result.failure(new InvalidTokenError('Invalid signature'));
      }

      // Check expiration
      if (decoded.exp && decoded.exp < this.clock.now().toUnixTimestamp()) {
        return Result.failure(new TokenExpiredError());
      }

      // Check token type
      if (decoded.type !== 'access') {
        return Result.failure(new InvalidTokenError('Invalid token type'));
      }

      return Result.success(decoded as TokenClaims);
    } catch (error) {
      return Result.failure(new InvalidTokenError('Token validation failed'));
    }
  }

  private sign(claims: TokenClaims): string {
    const header = { alg: 'HS256', typ: 'JWT' };
    const encodedHeader = base64UrlEncode(JSON.stringify(header));
    const encodedClaims = base64UrlEncode(JSON.stringify(claims));
    const signature = this.computeSignature(`${encodedHeader}.${encodedClaims}`);
    return `${encodedHeader}.${encodedClaims}.${signature}`;
  }

  private computeSignature(data: string): string {
    // HMAC-SHA256 signing
    return crypto.createHmac('sha256', this.secret)
      .update(data)
      .digest('base64url');
  }
}

interface JwtConfig {
  accessTokenExpiresIn: Duration;
  refreshTokenExpiresIn: Duration;
  issuer: string;
  audience: string;
}

interface TokenClaims {
  sub: string;
  email?: string;
  roles?: string[];
  permissions?: string[];
  type: string;
  iat: number;
  exp?: number;
  jti: string;
  iss?: string;
  aud?: string;
}
```

### 2.2 JWT Authentication Middleware

```typescript
// Express/Node.js authentication middleware
interface AuthMiddlewareOptions {
  excludePaths?: string[];
  tokenType?: 'Bearer' | 'JWT';
  allowExpired?: boolean;
}

class JwtAuthMiddleware {
  constructor(
    private readonly tokenService: TokenService,
    private readonly userRepository: UserRepository,
    private readonly options: AuthMiddlewareOptions = {}
  ) {}

  async handle(req: Request, res: Response, next: NextFunction): Promise<void> {
    // Check exclusion list
    if (this.shouldExclude(req.path)) {
      return next();
    }

    // Extract token from header
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      res.status(401).json({
        error: 'UNAUTHORIZED',
        message: 'Missing or invalid authorization header'
      });
      return;
    }

    const token = authHeader.substring(7);

    // Validate token
    const validationResult = await this.tokenService.validateAccessToken(token);
    if (validationResult.isFailure) {
      const error = validationResult.error;
      res.status(401).json({
        error: 'TOKEN_INVALID',
        message: error.message
      });
      return;
    }

    const claims = validationResult.value;

    // Fetch user from claims
    const userResult = await this.userRepository.findById(
      UserId.of(claims.sub)
    );

    if (userResult.isFailure) {
      res.status(401).json({
        error: 'USER_NOT_FOUND',
        message: 'User not found'
      });
      return;
    }

    const user = userResult.value;

    // Check if user is active
    if (!user.isActive) {
      res.status(403).json({
        error: 'ACCOUNT_DISABLED',
        message: 'Account is disabled'
      });
      return;
    }

    // Attach user to request
    req.user = user;
    req.tokenClaims = claims;

    next();
  }

  private shouldExclude(path: string): boolean {
    return this.options.excludePaths?.some(exclude => {
      if (exclude.endsWith('*')) {
        return path.startsWith(exclude.slice(0, -1));
      }
      return path === exclude;
    }) ?? false;
  }
}

// Express middleware setup
const authMiddleware = new JwtAuthMiddleware(
  tokenService,
  userRepository,
  {
    excludePaths: [
      '/health',
      '/api/public/*',
      '/auth/login',
      '/auth/register'
    ]
  }
);

app.use('/api', authMiddleware.middleware());
```

---

## 3. OAuth 2.0 Patterns

### 3.1 OAuth Authorization Flow

```typescript
// OAuth 2.0 Authorization Code Flow with PKCE
interface OAuthAuthorizationService {
  initiateAuthorization(request: AuthorizationRequest): Promise<AuthorizationUrl>;
  handleCallback(code: string, codeVerifier: string): Promise<Result<TokenSet, OAuthError>>;
  refreshAccessToken(refreshToken: string): Promise<Result<TokenSet, OAuthError>>;
  revokeToken(token: string): Promise<Result<void, OAuthError>>;
}

class OAuthAuthorizationServiceImpl implements OAuthAuthorizationService {
  constructor(
    private readonly config: OAuthConfig,
    private readonly httpClient: HttpClient,
    private readonly tokenService: TokenService,
    private readonly stateStore: StateStore
  ) {}

  async initiateAuthorization(request: AuthorizationRequest): Promise<AuthorizationUrl> {
    // Generate PKCE code verifier and challenge
    const codeVerifier = this.generateCodeVerifier();
    const codeChallenge = await this.generateCodeChallenge(codeVerifier);

    // Generate state for CSRF protection
    const state = this.generateState();

    // Store state and code verifier
    await this.stateStore.saveState(state, {
      codeVerifier,
      redirectUri: request.redirectUri,
      scope: request.scope,
      nonce: request.nonce
    });

    // Build authorization URL
    const params = new URLSearchParams({
      response_type: 'code',
      client_id: this.config.clientId,
      redirect_uri: request.redirectUri,
      scope: request.scope.join(' '),
      state: state,
      code_challenge: codeChallenge,
      code_challenge_method: 'S256',
      nonce: request.nonce
    });

    return new AuthorizationUrl(
      `${this.config.authorizationEndpoint}?${params.toString()}`
    );
  }

  async handleCallback(code: string, codeVerifier: string): Promise<Result<TokenSet, OAuthError>> {
    // Validate state
    const stateData = await this.stateStore.getState(codeVerifier);
    if (!stateData) {
      return Result.failure(new OAuthError('Invalid state', 'invalid_state'));
    }

    // Exchange code for tokens
    const tokenResponse = await this.httpClient.post<TokenResponse>(
      this.config.tokenEndpoint,
      {
        grant_type: 'authorization_code',
        code: code,
        client_id: this.config.clientId,
        code_verifier: codeVerifier,
        redirect_uri: stateData.redirectUri
      },
      {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': `Basic ${this.buildClientCredential()}`
        }
      }
    );

    if (tokenResponse.isFailure) {
      return Result.failure(tokenResponse.error);
    }

    // Convert OAuth tokens to internal tokens
    const tokens = tokenResponse.value;
    const userInfo = await this.fetchUserInfo(tokens.access_token);

    // Create or update user
    const user = await this.upsertUser(userInfo);

    // Generate internal tokens
    const accessToken = this.tokenService.generateAccessToken(user);
    const refreshToken = this.tokenService.generateRefreshToken(user);

    return Result.success(new TokenSet(
      accessToken,
      refreshToken,
      tokens.id_token,
      tokens.expires_in
    ));
  }

  private async generateCodeChallenge(verifier: string): Promise<string> {
    const encoder = new TextEncoder();
    const data = encoder.encode(verifier);
    const digest = await crypto.subtle.digest('SHA-256', data);
    return base64UrlEncode(Buffer.from(digest));
  }

  private generateState(): string {
    return base64UrlEncode(crypto.randomBytes(32));
  }

  private generateCodeVerifier(): string {
    return base64UrlEncode(crypto.randomBytes(32));
  }
}
```

### 3.2 OAuth Client Configuration

```typescript
interface OAuthConfig {
  clientId: string;
  clientSecret: string;
  authorizationEndpoint: string;
  tokenEndpoint: string;
  userInfoEndpoint: string;
  redirectUri: string;
  scopes: string[];
}

class OAuthClientConfig {
  private readonly config: OAuthConfig;

  constructor(config: OAuthConfig) {
    this.config = config;
  }

  get clientId(): string { return this.config.clientId; }
  
  get authorizationEndpoint(): string { return this.config.authorizationEndpoint; }
  
  get tokenEndpoint(): string { return this.config.tokenEndpoint; }

  get defaultScopes(): string[] {
    return [
      'openid',
      'profile',
      'email',
      ...this.config.scopes
    ];
  }

  validateRedirectUri(uri: string): boolean {
    return this.config.redirectUri === uri;
  }
}
```

---

## 4. Multi-Factor Authentication

### 4.1 MFA Service

```typescript
// Multi-factor authentication patterns
interface MFAService {
  setup(userId: UserId): Promise<Result<MFASetupResult, MFASetupError>>;
  verify(userId: UserId, code: string): Promise<Result<void, MFAVerificationError>>;
  disable(userId: UserId, password: string): Promise<Result<void, MFAError>>;
  getMethods(userId: UserId): Promise<MFAMethod[]>;
}

class MFAServiceImpl implements MFAService {
  constructor(
    private readonly totpService: TOTPService,
    private readonly userRepository: UserRepository,
    private readonly auditLogger: SecurityAuditLogger
  ) {}

  async setup(userId: UserId): Promise<Result<MFASetupResult, MFASetupError>> {
    const userResult = await this.userRepository.findById(userId);
    if (userResult.isFailure) {
      return Result.failure(new MFASetupError('User not found'));
    }
    const user = userResult.value;

    // Generate TOTP secret
    const totpSecret = await this.totpService.generateSecret();

    // Generate backup codes
    const backupCodes = this.generateBackupCodes();

    // Store MFA credentials (encrypted)
    const mfaCredentials = new MFACredentials({
      userId: userId,
      method: 'totp',
      secret: totpSecret.encrypted,
      backupCodes: backupCodes.hashed,
      enabled: false,
      verifiedAt: null
    });

    await this.userRepository.saveMFACredentials(mfaCredentials);

    await this.auditLogger.logEvent(new SecurityEvent(
      'MFA_SETUP_INITIATED',
      DateTime.now(),
      userId,
      'mfa_setup',
      Resource.of('user', userId.toString()),
      'success',
      { method: 'totp' }
    ));

    return Result.success(new MFASetupResult(
      totpSecret.uri,
      totpSecret.qrCode,
      backupCodes.plain
    ));
  }

  async verify(userId: UserId, code: string): Promise<Result<void, MFAVerificationError>> {
    const credentialsResult = await this.userRepository.getMFACredentials(userId);
    if (credentialsResult.isFailure) {
      return Result.failure(new MFAVerificationError('MFA not set up'));
    }
    const credentials = credentialsResult.value;

    // Check backup codes first
    if (this.isBackupCode(code, credentials.backupCodes)) {
      await this.markBackupCodeUsed(userId, code);
      await this.auditLogger.logEvent(new SecurityEvent(
        'MFA_VERIFIED_BACKUP',
        DateTime.now(),
        userId,
        'mfa_verify',
        Resource.of('user', userId.toString()),
        'success',
        { method: 'backup' }
      ));
      return Result.success();
    }

    // Verify TOTP code
    const isValid = await this.totpService.verify(credentials.secret, code);
    if (!isValid) {
      await this.auditLogger.logEvent(new SecurityEvent(
        'MFA_VERIFICATION_FAILED',
        DateTime.now(),
        userId,
        'mfa_verify',
        Resource.of('user', userId.toString()),
        'failure',
        { reason: 'invalid_code' }
      ));
      return Result.failure(new MFAVerificationError('Invalid code'));
    }

    // Mark MFA as verified
    credentials.verifiedAt = DateTime.now();
    credentials.enabled = true;
    await this.userRepository.saveMFACredentials(credentials);

    await this.auditLogger.logEvent(new SecurityEvent(
      'MFA_VERIFIED',
      DateTime.now(),
      userId,
      'mfa_verify',
      Resource.of('user', userId.toString()),
      'success',
      { method: 'totp' }
    ));

    return Result.success();
  }

  private generateBackupCodes(): { plain: string[]; hashed: string[] } {
    const codes = Array.from({ length: 10 }, () => 
      Math.random().toString(36).substring(2, 8).toUpperCase()
    );
    const hashed = codes.map(code => this.hashCode(code));
    return { plain: codes, hashed };
  }
}
```

### 4.2 TOTP Implementation

```typescript
// Time-based One-Time Password
interface TOTPService {
  generateSecret(): Promise<TOTPSecret>;
  verify(secret: string, code: string): Promise<boolean>;
  generateCode(secret: string): Promise<string>;
}

class TOTPServiceImpl implements TOTPService {
  private readonly algorithm = 'SHA1';
  private readonly digits = 6;
  private readonly period = 30;

  async generateSecret(): Promise<TOTPSecret> {
    const secret = base32Encode(crypto.randomBytes(20));
    const uri = this.generateUri('MyApp', secret);
    const qrCode = await this.generateQRCode(uri);
    
    return {
      secret,
      uri,
      qrCode,
      encrypted: this.encrypt(secret)
    };
  }

  async verify(secret: string, code: string): Promise<boolean> {
    const window = 1; // Allow 1 step tolerance
    const epoch = Math.floor(Date.now() / 1000 / this.period);

    for (let i = -window; i <= window; i++) {
      const expectedCode = this.generateCode(secret, epoch + i);
      if (this.constantTimeCompare(expectedCode, code)) {
        return true;
      }
    }
    return false;
  }

  private generateCode(secret: string, counter: number): string {
    const decodedSecret = base32Decode(secret);
    const counterBuffer = Buffer.alloc(8);
    counterBuffer.writeBigUInt64LE(BigInt(counter));

    const hmac = crypto.createHmac(this.algorithm, decodedSecret)
      .update(counterBuffer)
      .digest();

    const offset = hmac[hmac.length - 1] & 0x0f;
    const code = (
      ((hmac[offset] & 0x7f) << 24) |
      ((hmac[offset + 1] & 0xff) << 16) |
      ((hmac[offset + 2] & 0xff) << 8) |
      (hmac[offset + 3] & 0xff)
    ) % Math.pow(10, this.digits);

    return code.toString().padStart(this.digits, '0');
  }

  private generateUri(issuer: string, secret: string): string {
    const params = new URLSearchParams({
      secret: secret,
      issuer: issuer,
      algorithm: this.algorithm,
      digits: this.digits.toString(),
      period: this.period.toString()
    });
    return `otpauth://totp/${issuer}?${params.toString()}`;
  }

  private constantTimeCompare(a: string, b: string): boolean {
    if (a.length !== b.length) return false;
    let result = 0;
    for (let i = 0; i < a.length; i++) {
      result |= a.charCodeAt(i) ^ b.charCodeAt(i);
    }
    return result === 0;
  }
}
```

---

## 5. Session Management

### 5.1 Session Service

```typescript
// Session management patterns
interface SessionService {
  create(userId: UserId, metadata: SessionMetadata): Promise<Session>;
  validate(sessionId: SessionId): Promise<Result<Session, SessionError>>;
  refresh(sessionId: SessionId): Promise<Result<Session, SessionError>>;
  revoke(sessionId: SessionId, reason: RevocationReason): Promise<Result<void, SessionError>>;
  revokeAll(userId: UserId, excludeSessionId?: SessionId): Promise<Result<void, SessionError>>;
  getActiveSessions(userId: UserId): Promise<Session[]>;
}

class SessionServiceImpl implements SessionService {
  constructor(
    private readonly sessionStore: SessionStore,
    private readonly tokenService: TokenService,
    private readonly config: SessionConfig,
    private readonly auditLogger: SecurityAuditLogger
  ) {}

  async create(userId: UserId, metadata: SessionMetadata): Promise<Session> {
    // Check for concurrent session limit
    const activeSessions = await this.getActiveSessions(userId);
    if (activeSessions.length >= this.config.maxConcurrentSessions) {
      // Revoke oldest session
      const oldest = activeSessions
        .sort((a, b) => a.createdAt.toUnixTimestamp() - b.createdAt.toUnixTimestamp())[0];
      await this.revoke(oldest.id, 'CONCURRENT_SESSION_LIMIT');
    }

    // Create new session
    const session = new Session({
      id: SessionId.generate(),
      userId: userId,
      accessToken: this.tokenService.generateAccessTokenForSession(userId, metadata),
      refreshToken: this.tokenService.generateRefreshTokenForSession(userId, metadata),
      createdAt: DateTime.now(),
      expiresAt: DateTime.now().add(this.config.sessionExpiresIn),
      metadata: metadata,
      isRevoked: false
    });

    await this.sessionStore.save(session);

    await this.auditLogger.logEvent(new SecurityEvent(
      'SESSION_CREATED',
      DateTime.now(),
      userId,
      'session_create',
      Resource.of('session', session.id.toString()),
      'success',
      { ip: metadata.ipAddress, userAgent: metadata.userAgent }
    ));

    return session;
  }

  async validate(sessionId: SessionId): Promise<Result<Session, SessionError>> {
    const sessionResult = await this.sessionStore.findById(sessionId);
    if (sessionResult.isFailure) {
      return Result.failure(new SessionNotFoundError(sessionId));
    }

    const session = sessionResult.value;

    if (session.isRevoked) {
      return Result.failure(new SessionRevokedError(sessionId));
    }

    if (session.isExpired()) {
      return Result.failure(new SessionExpiredError(sessionId));
    }

    return Result.success(session);
  }

  async refresh(sessionId: SessionId): Promise<Result<Session, SessionError>> {
    const sessionResult = await this.validate(sessionId);
    if (sessionResult.isFailure) {
      return Result.failure(sessionResult.error);
    }

    const session = sessionResult.value;

    // Check for session rotation
    if (this.shouldRotateToken(session)) {
      const newAccessToken = this.tokenService.generateAccessTokenForSession(
        session.userId,
        session.metadata
      );
      const newRefreshToken = this.tokenService.generateRefreshTokenForSession(
        session.userId,
        session.metadata
      );

      session.rotateTokens(newAccessToken, newRefreshToken);
      await this.sessionStore.save(session);
    }

    // Extend session expiration
    session.extend(DateTime.now().add(this.config.sessionExpiresIn));
    await this.sessionStore.save(session);

    return Result.success(session);
  }

  async revoke(sessionId: SessionId, reason: RevocationReason): Promise<Result<void, SessionError>> {
    const sessionResult = await this.sessionStore.findById(sessionId);
    if (sessionResult.isFailure) {
      return Result.failure(new SessionNotFoundError(sessionId));
    }

    const session = sessionResult.value;
    session.revoke(reason);
    await this.sessionStore.save(session);

    await this.auditLogger.logEvent(new SecurityEvent(
      'SESSION_REVOKED',
      DateTime.now(),
      session.userId,
      'session_revoke',
      Resource.of('session', sessionId.toString()),
      'success',
      { reason: reason }
    ));

    return Result.success();
  }

  private shouldRotateToken(session: Session): boolean {
    const tokenAge = DateTime.now().toUnixTimestamp() - 
      session.accessToken.issuedAt.toUnixTimestamp();
    return tokenAge > this.config.tokenRotationThreshold;
  }
}
```

---

## 6. Password Security

### 6.1 Password Service

```typescript
// Secure password handling
interface PasswordService {
  hash(password: string): Promise<PasswordHash>;
  verify(password: string, hash: PasswordHash): Promise<boolean>;
  validate(password: string): Promise<PasswordValidationResult>;
  needsRehash(hash: PasswordHash): boolean;
}

class PasswordServiceImpl implements PasswordService {
  private readonly hasher: PasswordHasher;
  private readonly config: PasswordConfig;

  constructor(config: PasswordConfig) {
    this.hasher = new Argon2Hasher({
      memoryCost: 65536,  // 64 MB
      timeCost: 3,        // 3 iterations
      parallelism: 4      // 4 parallel threads
    });
    this.config = config;
  }

  async hash(password: string): Promise<PasswordHash> {
    // Validate password before hashing
    const validation = await this.validate(password);
    if (!validation.isValid) {
      throw new WeakPasswordError(validation.errors);
    }

    // Generate hash with salt
    const hash = await this.hasher.hash(password);
    return new PasswordHash(hash);
  }

  async verify(password: string, hash: PasswordHash): Promise<boolean> {
    return this.hasher.verify(hash.value, password);
  }

  async validate(password: string): Promise<PasswordValidationResult> {
    const errors: string[] = [];

    // Length check
    if (password.length < this.config.minLength) {
      errors.push(`Password must be at least ${this.config.minLength} characters`);
    }

    // Complexity checks (optional, based on policy)
    if (this.config.requireUppercase && !/[A-Z]/.test(password)) {
      errors.push('Password must contain at least one uppercase letter');
    }

    if (this.config.requireLowercase && !/[a-z]/.test(password)) {
      errors.push('Password must contain at least one lowercase letter');
    }

    if (this.config.requireNumber && !/\d/.test(password)) {
      errors.push('Password must contain at least one number');
    }

    if (this.config.requireSpecial && !/[!@#$%^&*]/.test(password)) {
      errors.push('Password must contain at least one special character');
    }

    // Check against common passwords
    if (this.isCommonPassword(password)) {
      errors.push('Password is too common');
    }

    // Check against breach database
    if (this.config.checkBreachedPasswords) {
      const isBreached = await this.checkBreachDatabase(password);
      if (isBreached) {
        errors.push('Password has appeared in a data breach');
      }
    }

    return {
      isValid: errors.length === 0,
      errors,
      strength: this.calculateStrength(password)
    };
  }

  needsRehash(hash: PasswordHash): boolean {
    return this.hasher.needsRehash(hash.value);
  }

  private isCommonPassword(password: string): boolean {
    const commonPasswords = ['password', '123456', 'qwerty', 'admin', 'letmein'];
    return commonPasswords.includes(password.toLowerCase());
  }

  private async checkBreachDatabase(password: string): Promise<boolean> {
    // Use HaveIBeenPwned k-anonymity API
    const hash = await crypto.subtle.digest('SHA-1', Buffer.from(password));
    const hashHex = Buffer.from(hash).toString('hex').toUpperCase();
    const prefix = hashHex.substring(0, 5);
    const suffix = hashHex.substring(5);

    const response = await fetch(`https://api.pwnedpasswords.com/range/${prefix}`);
    const text = await response.text();
    
    const hashes = text.split('\n').map(line => line.split(':')[0]);
    return hashes.includes(suffix);
  }

  private calculateStrength(password: string): PasswordStrength {
    let score = 0;
    
    // Length
    if (password.length >= 8) score += 1;
    if (password.length >= 12) score += 1;
    if (password.length >= 16) score += 1;
    
    // Complexity
    if (/[a-z]/.test(password)) score += 1;
    if (/[A-Z]/.test(password)) score += 1;
    if (/\d/.test(password)) score += 1;
    if (/[!@#$%^&*]/.test(password)) score += 1;
    
    // Entropy indicators
    if (password.length > 8 && score >= 4) score += 1;
    if (password.length > 12 && score >= 6) score += 1;

    if (score >= 8) return PasswordStrength.STRONG;
    if (score >= 5) return PasswordStrength.MODERATE;
    if (score >= 3) return PasswordStrength.WEAK;
    return PasswordStrength.VERY_WEAK;
  }
}

interface PasswordConfig {
  minLength: number;
  requireUppercase: boolean;
  requireLowercase: boolean;
  requireNumber: boolean;
  requireSpecial: boolean;
  checkBreachedPasswords: boolean;
}

enum PasswordStrength {
  VERY_WEAK = 0,
  WEAK = 1,
  MODERATE = 2,
  STRONG = 3
}
```

---

## 7. AI-Specific Guidelines

### 7.1 Authentication Pattern Generation

When AI generates authentication code:

1. **Always use secure token generation**
2. **Implement proper password hashing (Argon2/bcrypt)**
3. **Include rate limiting for login endpoints**
4. **Log all authentication events**
5. **Handle MFA gracefully**

### 7.2 Security Anti-Patterns to Avoid

| Anti-Pattern | Risk | Correct Approach |
|--------------|------|------------------|
| Storing passwords as MD5 | Easy to crack | Use Argon2/bcrypt |
| No rate limiting | Brute force attacks | Implement rate limits |
| Missing MFA | Account takeover | Offer MFA |
| Long-lived tokens | Token theft impact | Short-lived tokens |
| No token rotation | Session hijacking | Rotate tokens |
| Weak session IDs | Session prediction | Cryptographically random |
| Client-side auth checks | Bypassable | Server-side validation |
| Exposing user existence | Username enumeration | Generic error messages |

### 7.3 Authentication Code Template

```typescript
// AI-generated authentication use case template
class AuthenticateUserUseCase {
  constructor(
    private userRepository: UserRepository,
    private passwordService: PasswordService,
    private tokenService: TokenService,
    private auditLogger: SecurityAuditLogger,
    private rateLimiter: RateLimiter
  ) {}

  async execute(input: AuthenticateUserInput): Promise<Result<AuthResult, AuthError>> {
    // 1. Rate limiting check
    const rateLimitResult = await this.rateLimiter.check(input.email);
    if (rateLimitResult.isFailure) {
      await this.auditLogger.logFailedAttempt(input.email, 'rate_limited');
      return Result.failure(new TooManyAttemptsError());
    }

    // 2. Find user
    const userResult = await this.userRepository.findByEmail(input.email);
    if (userResult.isFailure) {
      await this.auditLogger.logFailedAttempt(input.email, 'user_not_found');
      // Generic error to avoid username enumeration
      return Result.failure(new InvalidCredentialsError());
    }
    const user = userResult.value;

    // 3. Check account status
    if (!user.isActive) {
      await this.auditLogger.logFailedAttempt(input.email, 'account_disabled');
      return Result.failure(new AccountDisabledError());
    }

    // 4. Verify password
    const passwordValid = await this.passwordService.verify(input.password, user.passwordHash);
    if (!passwordValid) {
      await this.auditLogger.logFailedAttempt(input.email, 'invalid_password');
      return Result.failure(new InvalidCredentialsError());
    }

    // 5. Generate tokens
    const accessToken = this.tokenService.generateAccessToken(user);
    const refreshToken = this.tokenService.generateRefreshToken(user);

    // 6. Record login
    user.recordLogin();
    await this.userRepository.save(user);

    await this.auditLogger.logSuccessfulLogin(user.id, input.email);

    return Result.success(new AuthResult(user, accessToken, refreshToken));
  }
}
```

---

## 8. References

1. doc/vertical-slice-architecture-megred.md - Auth feature examples
2. doc/on-demand-rule-loading.md - Auth rules
3. ref/CONSTITUTION.md - Security conventions
4. OWASP Authentication Cheat Sheet
5. OAuth 2.0 RFC 6749
6. OpenID Connect Core 1.0
7. RFC 6238 - TOTP
8. RFC 7519 - JWT

---

## Appendix A: Security Configuration Checklist

### A.1 Authentication Security Checklist

- [ ] Use strong password hashing (Argon2/bcrypt with proper parameters)
- [ ] Implement password complexity requirements
- [ ] Add multi-factor authentication support
- [ ] Implement account lockout after failed attempts
- [ ] Use secure, HttpOnly, SameSite cookies for sessions
- [ ] Implement proper session timeout
- [ ] Use secure token generation (cryptographically random)
- [ ] Implement token rotation on sensitive operations
- [ ] Validate all tokens (signature, expiration, issuer, audience)
- [ ] Log authentication events for audit

### A.2 Authorization Security Checklist

- [ ] Implement least privilege principle
- [ ] Use role-based access control (RBAC) or attribute-based access control (ABAC)
- [ ] Validate authorization on every request
- [ ] Check permissions before business logic
- [ ] Implement resource-level authorization
- [ ] Prevent IDOR vulnerabilities
- [ ] Use proper CORS configuration
- [ ] Implement rate limiting for sensitive endpoints

### A.3 Data Protection Checklist

- [ ] Use TLS 1.2+ for all communications
- [ ] Encrypt sensitive data at rest
- [ ] Hash passwords with proper algorithm
- [ ] Never log sensitive data (passwords, tokens, PII)
- [ ] Mask PII in logs and responses
- [ ] Use parameterized queries to prevent SQL injection
- [ ] Sanitize user inputs to prevent XSS
- [ ] Implement CSRF protection
- [ ] Use security headers (CSP, X-Frame-Options, HSTS)

### A.4 Audit and Monitoring Checklist

- [ ] Log all authentication events (success/failure)
- [ ] Log authorization failures
- [ ] Log privileged operations
- [ ] Implement tamper-evident logging
- [ ] Set up alerting for suspicious activity
- [ ] Regular security audits
- [ ] Penetration testing
- [ ] Vulnerability scanning

---

## Appendix B: Common Authentication Vulnerabilities

### B.1 OWASP Top 10 - Auth Related

| Vulnerability | Description | Mitigation |
|--------------|-------------|------------|
| Broken Authentication | Weak password policies, missing MFA | Implement strong auth policies |
| Broken Access Control | IDOR, missing authorization | Implement proper authorization |
| Sensitive Data Exposure | Unencrypted data, weak encryption | Encrypt data at rest and in transit |
| XML External Entities | Malicious XML input | Disable XXE, use safe parsers |
| Broken Access Control | Path traversal, IDOR | Validate and sanitize inputs |

### B.2 Authentication Attack Vectors

| Attack | Description | Prevention |
|--------|-------------|------------|
| Brute Force | Automated password guessing | Rate limiting, account lockout |
| Credential Stuffing | Using stolen credentials | Multi-factor authentication |
| Session Hijacking | Stealing session tokens | Secure cookies, token rotation |
| Session Fixation | Fixing session ID | Regenerate session on login |
| Phishing | Fake login pages | User education, HTTPS |
| Man-in-the-Middle | Intercepting traffic | TLS, certificate pinning |

### B.3 Password Security Guidelines

**Minimum Requirements:**
- Minimum 8 characters
- Mix of uppercase, lowercase, numbers, special characters
- No common passwords (check against list)
- No personal information

**Storage:**
- Always hash passwords (never encrypt)
- Use Argon2id (preferred) or bcrypt
- Generate unique salt per password
- Configure appropriate work factors

**Transmission:**
- Always use HTTPS
- Never send passwords in URL
- Implement CSP to prevent XSS

---

## Appendix C: Token Security Best Practices

### C.1 JWT Best Practices

| Practice | Description |
|----------|-------------|
| Short expiration | Access tokens: 5-15 minutes |
| Store securely | Not in localStorage (XSS vulnerable) |
| Validate claims | issuer, audience, expiration |
| Algorithm verification | Explicit algorithm, no 'none' |
| Key management | Use RS256/ES256, secure key storage |
| Refresh tokens | Longer-lived, stored securely, one-time use |

### C.2 Session Security

| Practice | Description |
|----------|-------------|
| Session ID length | Minimum 128 bits |
| Secure generation | Cryptographically random |
| Cookie attributes | HttpOnly, Secure, SameSite=Strict |
| Session timeout | Inactive: 15-30 minutes |
| Concurrent sessions | Limit, detect anomalies |
| Logout | Invalidate server-side, clear client |

### C.3 OAuth 2.0 Security

| Grant Type | Use Case | Security Notes |
|------------|----------|----------------|
| Authorization Code | Web apps, mobile | With PKCE for public clients |
| Client Credentials | Machine-to-machine | With proper scoping |
| Device Code | IoT, smart TVs | With user code binding |
| Refresh Token | Token refresh | Store securely, rotate on use |
