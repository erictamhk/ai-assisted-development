# Comprehensive Security Guide

This document provides comprehensive security guidance for AI-assisted development, covering common vulnerabilities, secure coding practices, and security patterns.

---

## Part 1: Common Security Vulnerabilities

### 1.1 OWASP Top 10 Overview

| Rank | Vulnerability | Severity | Prevalence |
|------|---------------|----------|------------|
| 1 | Broken Access Control | Critical | High |
| 2 | Cryptographic Failures | Critical | Medium |
| 3 | Injection | Critical | High |
| 4 | Insecure Design | High | Medium |
| 5 | Security Misconfiguration | High | High |
| 6 | Vulnerable Components | Medium | High |
| 7 | Authentication Failures | High | High |
| 8 | Data Integrity Failures | Medium | Medium |
| 9 | Logging Failures | Low | Medium |
| 10 | SSRF | Medium | Low |

### 1.2 Injection Attacks

#### SQL Injection

```typescript
// ❌ BAD - SQL Injection vulnerable
async function findUser(email: string): Promise<User | null> {
  return db.query(`SELECT * FROM users WHERE email = '${email}'`);
}

// ✅ GOOD - Parameterized query
async function findUser(email: string): Promise<User | null> {
  return db.query('SELECT * FROM users WHERE email = $1', [email]);
}

// ✅ GOOD - ORM (Prisma)
async function findUser(email: string): Promise<User | null> {
  return prisma.user.findUnique({ where: { email } });
}

// ✅ GOOD - Query builder with parameters
async function findUser(email: string): Promise<User | null> {
  return queryBuilder
    .select('*')
    .from('users')
    .where('email = ?', email)
    .execute();
}
```

#### NoSQL Injection

```typescript
// ❌ BAD - NoSQL injection
async function findUser(username: string): Promise<User | null> {
  return mongoDb.collection('users').findOne({
    username: eval(`({ $ne: null })`)  // Never do this!
  });
}

// ✅ GOOD - Proper query
async function findUser(username: string): Promise<User | null> {
  return mongoDb.collection('users').findOne({ username });
}

// ✅ GOOD - Input sanitization
async function findUser(username: string): Promise<User | null> {
  const sanitized = username.replace(/[$]/g, '');
  return mongoDb.collection('users').findOne({ username: sanitized });
}
```

#### Command Injection

```typescript
// ❌ BAD - Command injection
async function processFile(filename: string): Promise<void> {
  return exec(`convert ${filename} output.png`);  // Dangerous!
}

// ✅ GOOD - Safe API usage
import { execFile } from 'child_process';
import { promisify } from 'util';

const execFileAsync = promisify(execFile);

async function processFile(filename: string): Promise<void> {
  // Validate filename
  if (!/^[a-zA-Z0-9_-]+\.(png|jpg|jpeg)$/.test(filename)) {
    throw new Error('Invalid filename');
  }
  
  // Use execFile with array of arguments
  await execFileAsync('convert', [filename, 'output.png']);
}

// ✅ GOOD - Use library instead of system command
import sharp from 'sharp';

async function processFile(filename: string): Promise<void> {
  await sharp(filename).toFile('output.png');
}
```

#### XSS (Cross-Site Scripting)

```typescript
// ❌ BAD - XSS vulnerable
function renderComment(comment: string): string {
  return `<div class="comment">${comment}</div>`;  // No escaping!
}

// ✅ GOOD - Proper escaping
import DOMPurify from 'dompurify';
import { JSDOM } from 'jsdom';

function renderComment(comment: string): string {
  const window = new JSDOM('').window;
  const purify = DOMPurify(window);
  return `<div class="comment">${purify.sanitize(comment)}</div>`;
}

// ✅ GOOD - React automatic escaping (in most cases)
function Comment({ text }: { text: string }) {
  return <div className="comment">{text}</div>;  // React escapes by default
}

// ✅ GOOD - Explicit encoding for URLs
function encodeForUrl(input: string): string {
  return encodeURIComponent(input);
}
```

#### CSRF (Cross-Site Request Forgery)

```typescript
// ❌ BAD - No CSRF protection
app.post('/transfer', (req, res) => {
  const { to, amount } = req.body;
  transferMoney(req.user.id, to, amount);  // Vulnerable!
});

// ✅ GOOD - CSRF token
import { csrf } from 'csurf';

const csrfProtection = csrf({ cookie: true });

app.post('/transfer', csrfProtection, (req, res) => {
  const { to, amount } = req.body;
  transferMoney(req.user.id, to, amount);
});

// ✅ GOOD - SameSite cookie
app.use(session({
  cookie: {
    secure: true,
    httpOnly: true,
    sameSite: 'strict'  // Prevents CSRF
  }
}));

// ✅ GOOD - Double Submit Cookie pattern
app.post('/transfer', async (req, res) => {
  const csrfToken = req.body.csrfToken;
  const cookieToken = req.cookies.csrf_token;
  
  if (csrfToken !== cookieToken) {
    throw new CsrfError('Invalid CSRF token');
  }
  
  transferMoney(req.user.id, req.body.to, req.body.amount);
});
```

### 1.3 Broken Access Control

#### IDOR (Insecure Direct Object Reference)

```typescript
// ❌ BAD - IDOR vulnerability
app.get('/api/documents/:id', async (req, res) => {
  const doc = await documentService.getById(req.params.id);
  return res.json(doc);  // No ownership check!
});

// ✅ GOOD - Verify ownership
app.get('/api/documents/:id', async (req, res) => {
  const doc = await documentService.getById(req.params.id);
  
  if (!doc) {
    return res.status(404).json({ error: 'Document not found' });
  }
  
  // Check ownership
  if (doc.ownerId !== req.user.id && !req.user.roles.includes('admin')) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  return res.json(doc);
});

// ✅ GOOD - Use indirect references
app.get('/api/documents/:encodedId', async (req, res) => {
  const docId = decodeDocumentId(req.params.encodedId);  // Indirect reference
  const doc = await documentService.getById(docId);
  
  // Verify access
  const access = await accessControl.canUserRead(req.user.id, docId);
  if (!access.allowed) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  return res.json(doc);
});
```

#### Path Traversal

```typescript
// ❌ BAD - Path traversal vulnerability
app.get('/files/:filename', (req, res) => {
  const filename = req.params.filename;
  const filepath = `/uploads/${filename}`;
  res.sendFile(filepath);  // Could access /etc/passwd!
});

// ✅ GOOD - Validate path
app.get('/files/:filename', (req, res) => {
  const filename = req.params.filename;
  
  // Validate filename
  if (!/^[a-zA-Z0-9_-]+\.[a-z0-9]+$/.test(filename)) {
    return res.status(400).json({ error: 'Invalid filename' });
  }
  
  const filepath = path.resolve('/uploads', filename);
  
  // Ensure filepath is within uploads directory
  if (!filepath.startsWith('/uploads/')) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  res.sendFile(filepath);
});

// ✅ GOOD - Use library for safe path handling
import { sanitize } from 'path-sanitizer';

app.get('/files/:filename', (req, res) => {
  const safePath = sanitize(req.params.filename, { allowPathTraversal: false });
  const filepath = path.join('/uploads', safePath);
  res.sendFile(filepath);
});
```

---

## Part 2: Authentication Security

### 2.1 Password Security

```typescript
// ✅ GOOD - Secure password hashing with Argon2
import * as argon2 from 'argon2';

class PasswordService {
  async hash(password: string): Promise<string> {
    return argon2.hash(password, {
      type: argon2.argon2id,
      memoryCost: 65536,  // 64 MB
      timeCost: 3,
      parallelism: 4
    });
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return argon2.verify(hash, password);
  }
}

// ✅ GOOD - Password complexity validation
interface PasswordRequirements {
  minLength: number;
  requireUppercase: boolean;
  requireLowercase: boolean;
  requireNumbers: boolean;
  requireSpecial: boolean;
  bannedPasswords: string[];
}

function validatePassword(password: string, requirements: PasswordRequirements): PasswordValidationResult {
  const errors: string[] = [];
  
  if (password.length < requirements.minLength) {
    errors.push(`Password must be at least ${requirements.minLength} characters`);
  }
  
  if (requirements.requireUppercase && !/[A-Z]/.test(password)) {
    errors.push('Password must contain at least one uppercase letter');
  }
  
  if (requirements.requireLowercase && !/[a-z]/.test(password)) {
    errors.push('Password must contain at least one lowercase letter');
  }
  
  if (requirements.requireNumbers && !/\d/.test(password)) {
    errors.push('Password must contain at least one number');
  }
  
  if (requirements.requireSpecial && !/[!@#$%^&*(),.?":{}|<>]/.test(password)) {
    errors.push('Password must contain at least one special character');
  }
  
  if (requirements.bannedPasswords.includes(password.toLowerCase())) {
    errors.push('This password is too common');
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
}
```

### 2.2 Session Security

```typescript
// ✅ GOOD - Secure session configuration
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

const redisClient = createClient({ url: process.env.REDIS_URL });

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET!,
  name: 'sessionId',  // Don't use default 'connect.sid'
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',  // HTTPS only in production
    httpOnly: true,  // Prevent XSS access to cookie
    sameSite: 'strict',  // CSRF protection
    maxAge: 30 * 60 * 1000  // 30 minutes
  }
}));

// ✅ GOOD - Session regeneration on login
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  const user = await authenticateUser(email, password);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Regenerate session to prevent session fixation
  req.session.regenerate((err) => {
    if (err) {
      return res.status(500).json({ error: 'Login failed' });
    }
    
    req.session.userId = user.id;
    req.session.loginTime = Date.now();
    
    res.json({ success: true });
  });
});
```

### 2.3 Token Security

```typescript
// ✅ GOOD - Secure JWT configuration
import jwt from 'jsonwebtoken';

interface TokenPayload {
  userId: string;
  roles: string[];
}

const ACCESS_TOKEN_SECRET = process.env.JWT_ACCESS_SECRET!;
const REFRESH_TOKEN_SECRET = process.env.JWT_REFRESH_SECRET!;

class TokenService {
  generateAccessToken(payload: TokenPayload): string {
    return jwt.sign(payload, ACCESS_TOKEN_SECRET, {
      algorithm: 'RS256',  // Asymmetric algorithm
      expiresIn: '15m',    // Short-lived
      issuer: 'my-app',
      audience: 'my-app-users'
    });
  }

  generateRefreshToken(payload: TokenPayload): string {
    return jwt.sign(payload, REFRESH_TOKEN_SECRET, {
      algorithm: 'RS256',
      expiresIn: '7d',
      issuer: 'my-app',
      audience: 'my-app-users',
      jwtid: crypto.randomUUID()  // For revocation
    });
  }

  async verifyAccessToken(token: string): Promise<TokenPayload | null> {
    try {
      const decoded = jwt.verify(token, ACCESS_TOKEN_SECRET, {
        algorithms: ['RS256'],
        issuer: 'my-app',
        audience: 'my-app-users'
      }) as TokenPayload;
      
      return decoded;
    } catch (error) {
      return null;
    }
  }
}

// ✅ GOOD - Token storage (client-side)
const tokenStorage = {
  accessToken: null as string | null,
  refreshToken: null as string | null,
  
  setTokens(access: string, refresh: string) {
    this.accessToken = access;
    this.refreshToken = refresh;
    
    // Store in memory only, never in localStorage
    // Access tokens are short-lived and can be refreshed
  },
  
  clearTokens() {
    this.accessToken = null;
    this.refreshToken = null;
  }
};
```

---

## Part 3: Data Protection

### 3.1 Encryption

```typescript
// ✅ GOOD - Encryption at rest
import crypto from 'crypto';

interface EncryptionConfig {
  algorithm: string;
  key: Buffer;
  ivLength: number;
}

class EncryptionService {
  private readonly config: EncryptionConfig;

  constructor() {
    this.config = {
      algorithm: 'aes-256-gcm',
      key: crypto.scryptSync(process.env.ENCRYPTION_KEY!, 'salt', 32),
      ivLength: 16
    };
  }

  encrypt(plaintext: string): string {
    const iv = crypto.randomBytes(this.config.ivLength);
    const cipher = crypto.createCipheriv(
      this.config.algorithm,
      this.config.key,
      iv
    );
    
    let encrypted = cipher.update(plaintext, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
  }

  decrypt(encryptedText: string): string {
    const [ivHex, authTagHex, encrypted] = encryptedText.split(':');
    
    const iv = Buffer.from(ivHex, 'hex');
    const authTag = Buffer.from(authTagHex, 'hex');
    
    const decipher = crypto.createDecipheriv(
      this.config.algorithm,
      this.config.key,
      iv
    );
    
    decipher.setAuthTag(authTag);
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}

// ✅ GOOD - Hashing (one-way)
import crypto from 'crypto';

function hashData(data: string): string {
  return crypto.createHash('sha256').update(data).digest('hex');
}

// For passwords, use bcrypt/Argon2 (see password security above)
```

### 3.2 Sensitive Data Handling

```typescript
// ✅ GOOD - Mask sensitive data in logs
interface User {
  id: string;
  email: string;
  passwordHash: string;
  ssn?: string;
  creditCard?: string;
}

function sanitizeForLogging(user: User): object {
  return {
    id: user.id,
    email: maskEmail(user.email),
    // Never log password hash
    // Never log SSN
    // Never log credit card
    hasCreditCard: !!user.creditCard
  };
}

function maskEmail(email: string): string {
  const [local, domain] = email.split('@');
  const maskedLocal = local.charAt(0) + '***@' + domain;
  return maskedLocal;
}

// ✅ GOOD - Secure API responses
app.get('/api/user/:id', async (req, res) => {
  const user = await userService.getById(req.params.id);
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  // Don't return sensitive fields
  res.json({
    id: user.id,
    name: user.name,
    email: user.email,
    // passwordHash: '***',  // Never
    // ssn: '***',           // Never
  });
});
```

### 3.3 Secure File Handling

```typescript
// ✅ GOOD - Secure file upload
import multer from 'multer';
import path from 'path';

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, '/uploads/');  // Outside web root
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = crypto.randomUUID();
    cb(null, `${uniqueSuffix}${path.extname(file.originalname)}`);
  }
});

const upload = multer({
  storage,
  limits: {
    fileSize: 5 * 1024 * 1024,  // 5MB limit
    files: 1
  },
  fileFilter: (req, file, cb) => {
    const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
    if (!allowedTypes.includes(file.mimetype)) {
      cb(new Error('Invalid file type'));
      return;
    }
    cb(null, true);
  }
});

// ✅ GOOD - Serve files with proper headers
app.get('/files/:id', async (req, res) => {
  const file = await fileService.getById(req.params.id);
  
  if (!file) {
    return res.status(404).json({ error: 'File not found' });
  }
  
  // Verify access
  const canAccess = await accessControl.canUserRead(req.user.id, file.id);
  if (!canAccess) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  // Set security headers
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('Content-Disposition', `inline; filename="${file.name}"`);
  
  // For sensitive files
  if (file.isPrivate) {
    res.setHeader('Cache-Control', 'no-store');
  }
  
  res.sendFile(file.path);
});
```

---

## Part 4: Security Headers

### 4.1 Essential Security Headers

```typescript
// Express.js security headers
import helmet from 'helmet';

app.use(helmet());

// Or manually configure
app.use((req, res, next) => {
  // Prevent MIME type sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');
  
  // XSS protection (legacy browsers)
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // HSTS - Force HTTPS
  res.setHeader('Strict-Transport-Security', 
    'max-age=31536000; includeSubDomains; preload');
  
  // Referrer policy
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  // Content Security Policy
  res.setHeader('Content-Security-Policy', 
    "default-src 'self'; " +
    "script-src 'self' 'unsafe-inline' 'unsafe-eval'; " +
    "style-src 'self' 'unsafe-inline'; " +
    "img-src 'self' data: https:; " +
    "font-src 'self'; " +
    "connect-src 'self' https://api.example.com");
  
  // Permissions policy
  res.setHeader('Permissions-Policy', 
    'geolocation=(), microphone=(), camera=()');
  
  next();
});
```

### 4.2 CORS Configuration

```typescript
import cors from 'cors';

app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || [],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400,  // 24 hours
  preflightContinue: false,
  optionsSuccessStatus: 204
}));
```

---

## Part 5: Security in Development

### 5.1 Environment Variables

```typescript
// ✅ GOOD - Secure environment variable handling
import dotenv from 'dotenv';

dotenv.config();

// Define required environment variables
const requiredEnvVars = [
  'NODE_ENV',
  'DATABASE_URL',
  'JWT_ACCESS_SECRET',
  'JWT_REFRESH_SECRET',
  'ENCRYPTION_KEY'
];

// Validate required variables
for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    throw new Error(`Missing required environment variable: ${envVar}`);
  }
}

// Use typed environment variables
interface Environment {
  nodeEnv: 'development' | 'production' | 'test';
  databaseUrl: string;
  jwtAccessSecret: string;
  jwtRefreshSecret: string;
  encryptionKey: string;
}

const env: Environment = {
  nodeEnv: (process.env.NODE_ENV as Environment['nodeEnv']) || 'development',
  databaseUrl: process.env.DATABASE_URL!,
  jwtAccessSecret: process.env.JWT_ACCESS_SECRET!,
  jwtRefreshSecret: process.env.JWT_REFRESH_SECRET!,
  encryptionKey: process.env.ENCRYPTION_KEY!
};
```

### 5.2 Security in Code Review

```markdown
## Security Review Checklist

### Authentication
- [ ] Passwords hashed (Argon2/bcrypt)
- [ ] No hardcoded credentials
- [ ] Token validation (signature, expiration)
- [ ] Authentication errors generic (no username enumeration)

### Authorization
- [ ] Authorization before business logic
- [ ] Resource access tied to authenticated user
- [ ] Role/permission checks explicit
- [ ] No IDOR vulnerabilities

### Data Protection
- [ ] Sensitive data encrypted
- [ ] TLS for all communications
- [ ] No sensitive data in logs
- [ ] PII masked

### Input Validation
- [ ] All inputs validated
- [ ] Parameterized queries (no SQL injection)
- [ ] Input sanitized (no XSS)
- [ ] File paths validated (no path traversal)

### Dependencies
- [ ] Dependencies up to date
- [ ] No known vulnerabilities
- [ ] Minimum necessary permissions
```

---

## Part 6: References

1. OWASP Top 10 - https://owasp.org/Top10/
2. OWASP Authentication Cheat Sheet
3. OWASP Session Management Cheat Sheet
4. OWASP Secure Coding Practices
5. ref/CONSTITUTION.md - Security conventions
6. dev-knowledge/12-security/auth-patterns.md - Auth patterns
7. dev-knowledge/12-security/domain-security.md - Domain security
8. RFC 7519 - JWT
9. RFC 6749 - OAuth 2.0

---

## Appendix: Security Quick Reference

### Security Headers

| Header | Purpose | Value |
|--------|---------|-------|
| X-Content-Type-Options | Prevent MIME sniffing | `nosniff` |
| X-Frame-Options | Prevent clickjacking | `DENY` |
| Strict-Transport-Security | Force HTTPS | `max-age=31536000` |
| Content-Security-Policy | Prevent XSS | Policy string |
| Referrer-Policy | Control referrer | `strict-origin...` |
| Permissions-Policy | Feature restrictions | Policy string |

### Common Attack Mitigations

| Attack | Prevention |
|--------|------------|
| SQL Injection | Parameterized queries |
| XSS | CSP, input sanitization |
| CSRF | SameSite cookies, CSRF tokens |
| IDOR | Authorization checks |
| Path traversal | Path validation |
| Brute force | Rate limiting, account lockout |
| Session hijacking | Secure cookies, short tokens |

### Password Requirements

| Requirement | Value |
|-------------|-------|
| Minimum length | 8 characters |
| Uppercase | Required |
| Lowercase | Required |
| Numbers | Required |
| Special characters | Required |
| Common passwords | Banned |