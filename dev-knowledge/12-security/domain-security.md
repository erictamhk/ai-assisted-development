# Domain Security

This document defines domain security patterns for AI-assisted development, focusing on implementing authentication, authorization, and security boundaries within domain-driven design.

---

## 1. Security in Domain-Driven Design

### 1.1 Where Security Belongs

Security concerns should be integrated throughout the layered architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                       │
│         (Auth Middleware, Route Protection)                │
├─────────────────────────────────────────────────────────────┤
│                   Application Layer                         │
│    (Authorization in Use Cases, Permission Checks)         │
├─────────────────────────────────────────────────────────────┤
│                     Domain Layer                            │
│  (Security Domain Services, Access Control Policies)        │
├─────────────────────────────────────────────────────────────┤
│                  Infrastructure Layer                       │
│   (Authentication Providers, Token Validation, Encryption) │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Security Core Principle

**Authentication should be a cross-cutting concern handled at the edges, while authorization decisions should be made in the domain layer.**

```typescript
// ❌ BAD - Business logic mixed with authorization
class OrderService {
  async cancelOrder(orderId: string, userId: string): Promise<void> {
    // Authorization logic here - violates single responsibility
    const user = await this.userRepo.findById(userId);
    if (!user.hasRole('ADMIN') && !user.hasRole('ORDER_MANAGER')) {
      throw new UnauthorizedError('Insufficient permissions');
    }
    
    // Business logic
    const order = await this.orderRepo.findById(orderId);
    order.cancel();
    await this.orderRepo.save(order);
  }
}

// ✅ GOOD - Authorization delegated to domain service
class OrderService {
  constructor(
    private orderRepo: OrderRepository,
    private authorizationService: AuthorizationService
  ) {}

  async cancelOrder(input: CancelOrderInput): Promise<Result<void, OrderCancellationError>> {
    // Authorization as first step
    const authResult = await this.authorizationService.authorize({
      userId: input.userId,
      action: 'cancel',
      resource: { type: 'order', id: input.orderId }
    });

    if (authResult.isFailure) {
      return Result.failure(authResult.error);
    }

    // Business logic - pure domain
    const order = await this.orderRepo.findById(input.orderId);
    const result = order.cancel();
    
    if (result.isSuccess) {
      await this.orderRepo.save(order);
    }
    
    return result;
  }
}
```

---

## 2. Security Domain Models

### 2.1 User and Identity

```typescript
// Core identity entities
class UserId implements EntityId {
  constructor(private readonly value: string) {}
  toString(): string { return this.value; }
  equals(other: EntityId): boolean {
    return other instanceof UserId && this.value === other.value;
  }
}

class User implements Entity<UserId> {
  private readonly id: UserId;
  private email: Email;
  private passwordHash: PasswordHash;
  private roles: UserRole[];
  private permissions: Permission[];
  private isActive: boolean;
  private lastLoginAt: DateTime | null;
  private readonly createdAt: DateTime;

  constructor(props: UserProps) {
    this.id = props.id || UserId.generate();
    this.email = props.email;
    this.passwordHash = props.passwordHash;
    this.roles = props.roles || [];
    this.permissions = props.permissions || [];
    this.isActive = props.isActive !== false;
    this.lastLoginAt = null;
    this.createdAt = DateTime.now();
  }

  // Business methods
  hasRole(role: UserRole): boolean {
    return this.roles.some(r => r.equals(role));
  }

  hasPermission(permission: Permission): boolean {
    // Direct permission check
    if (this.permissions.some(p => p.equals(permission))) {
      return true;
    }
    // Role-based permission check
    return this.roles.some(role => role.hasPermission(permission));
  }

  activate(): void {
    this.isActive = true;
  }

  deactivate(): void {
    this.isActive = false;
  }

  recordLogin(): void {
    this.lastLoginAt = DateTime.now();
  }

  canPerform(action: string, resource: Resource): boolean {
    if (!this.isActive) return false;
    return this.roles.some(role => role.allows(action, resource));
  }
}

interface UserProps {
  id?: UserId;
  email: Email;
  passwordHash: PasswordHash;
  roles?: UserRole[];
  permissions?: Permission[];
  isActive?: boolean;
}
```

### 2.2 Roles and Permissions

```typescript
// Permission - atomic unit of authorization
class Permission implements ValueObject {
  constructor(
    private readonly resource: string,
    private readonly action: string
  ) {
    if (!resource || !action) {
      throw new InvalidPermissionError(resource, action);
    }
  }

  static create(resource: string, action: string): Permission {
    return new Permission(resource, action);
  }

  // Shorthand factory methods
  static read(entity: string): Permission {
    return new Permission(entity, 'read');
  }

  static write(entity: string): Permission {
    return new Permission(entity, 'write');
  }

  static delete(entity: string): Permission {
    return new Permission(entity, 'delete');
  }

  get resource(): string { return this.resource; }
  get action(): string { return this.action; }

  allows(other: Permission): boolean {
    // Wildcard permission
    if (this.resource === '*' || this.action === '*') {
      return this.resource === '*' || this.action === '*';
    }
    // Exact match
    return this.resource === other.resource && this.action === other.action;
  }

  toString(): string {
    return `${this.resource}:${this.action}`;
  }

  equals(other: ValueObject): boolean {
    return other instanceof Permission && 
           this.resource === other.resource && 
           this.action === other.action;
  }
}

// Role - collection of permissions with hierarchy
class Role implements Entity<RoleId> {
  private readonly id: RoleId;
  private readonly name: string;
  private permissions: Permission[];
  private parentRole: Role | null;

  constructor(props: RoleProps) {
    this.id = props.id || RoleId.generate();
    this.name = props.name;
    this.permissions = props.permissions || [];
    this.parentRole = props.parentRole || null;
  }

  hasPermission(permission: Permission): boolean {
    // Check own permissions
    if (this.permissions.some(p => p.allows(permission))) {
      return true;
    }
    // Check parent role permissions (inheritance)
    return this.parentRole?.hasPermission(permission) ?? false;
  }

  allows(action: string, resource: Resource): boolean {
    const permission = new Permission(resource.type, action);
    return this.hasPermission(permission);
  }

  addPermission(permission: Permission): void {
    if (!this.hasPermission(permission)) {
      this.permissions.push(permission);
    }
  }

  removePermission(permission: Permission): void {
    this.permissions = this.permissions.filter(
      p => !p.equals(permission)
    );
  }

  getAllPermissions(): Permission[] {
    const inherited = this.parentRole?.getAllPermissions() || [];
    return [...this.permissions, ...inherited];
  }
}

// Resource - entity being protected
class Resource implements ValueObject {
  constructor(
    private readonly type: string,
    private readonly id?: string
  ) {}

  static of(type: string, id?: string): Resource {
    return new Resource(type, id);
  }

  get type(): string { return this.type; }
  get id(): string | undefined { return this.id; }

  matches(other: Resource): boolean {
    // Type must match
    if (this.type !== other.type) return false;
    // If this resource has no specific ID, it represents the type
    if (!this.id) return true;
    // If other has no specific ID, it matches any
    if (!other.id) return true;
    // Both have IDs, must match
    return this.id === other.id;
  }

  isChildOf(parent: Resource): boolean {
    return parent.matches(this);
  }

  equals(other: ValueObject): boolean {
    if (!(other instanceof Resource)) return false;
    return this.type === other.type && this.id === other.id;
  }
}
```

### 2.3 Authentication Events

```typescript
// Domain events for security
class UserRegisteredEvent implements DomainEvent {
  readonly userId: UserId;
  readonly email: Email;
  readonly timestamp: DateTime;

  constructor(userId: UserId, email: Email) {
    this.userId = userId;
    this.email = email;
    this.timestamp = DateTime.now();
  }
}

class UserLoginSucceededEvent implements DomainEvent {
  readonly userId: UserId;
  readonly method: AuthenticationMethod;
  readonly ipAddress: string;
  readonly userAgent: string;
  readonly timestamp: DateTime;

  constructor(props: LoginSuccessProps) {
    this.userId = props.userId;
    this.method = props.method;
    this.ipAddress = props.ipAddress;
    this.userAgent = props.userAgent;
    this.timestamp = DateTime.now();
  }
}

class UserLoginFailedEvent implements DomainEvent {
  readonly email: Email;
  readonly reason: string;
  readonly ipAddress: string;
  readonly timestamp: DateTime;

  constructor(props: LoginFailedProps) {
    this.email = props.email;
    this.reason = props.reason;
    this.ipAddress = props.ipAddress;
    this.timestamp = DateTime.now();
  }
}

class PasswordChangedEvent implements DomainEvent {
  readonly userId: UserId;
  readonly timestamp: DateTime;
  readonly requiresReauth: boolean;

  constructor(userId: UserId, requiresReauth: boolean = true) {
    this.userId = userId;
    this.timestamp = DateTime.now();
    this.requiresReauth = requiresReauth;
  }
}
```

---

## 3. Authorization Patterns

### 3.1 Authorization Service

```typescript
// Domain authorization service
interface AuthorizationService {
  authorize(request: AuthorizationRequest): Promise<Result<AuthorizationDecision, AuthorizationError>>;
  checkPermission(userId: UserId, permission: Permission): Promise<boolean>;
  getAccessibleResources(userId: UserId, resourceType: string): Promise<Resource[]>;
}

class AuthorizationServiceImpl implements AuthorizationService {
  constructor(
    private userRepository: UserRepository,
    private roleRepository: RoleRepository,
    private auditLogger: SecurityAuditLogger
  ) {}

  async authorize(request: AuthorizationRequest): Promise<Result<AuthorizationDecision, AuthorizationError>> {
    const { userId, action, resource } = request;

    // Get user
    const userResult = await this.userRepository.findById(userId);
    if (userResult.isFailure) {
      return Result.failure(new UserNotFoundError(userId));
    }
    const user = userResult.value;

    // Check if user is active
    if (!user.isActive) {
      return Result.failure(new AccountDisabledError(userId));
    }

    // Check permission
    const permission = new Permission(resource.type, action);
    const hasPermission = user.hasPermission(permission);

    // Log authorization attempt
    await this.auditLogger.logAuthorization({
      userId: userId.toString(),
      action,
      resource,
      granted: hasPermission,
      timestamp: DateTime.now()
    });

    if (!hasPermission) {
      return Result.failure(new AccessDeniedError(userId, action, resource));
    }

    return Result.success(new AuthorizationDecision(
      userId,
      action,
      resource,
      DateTime.now().addMinutes(15) // Authorization valid for 15 min
    ));
  }

  async checkPermission(userId: UserId, permission: Permission): Promise<boolean> {
    const userResult = await this.userRepository.findById(userId);
    if (userResult.isFailure) return false;
    return userResult.value.hasPermission(permission);
  }

  async getAccessibleResources(userId: UserId, resourceType: string): Promise<Resource[]> {
    const userResult = await this.userRepository.findById(userId);
    if (userResult.isFailure) return [];
    
    const user = userResult.value;
    const resources: Resource[] = [];

    // Aggregate accessible resources from roles
    for (const role of user.roles) {
      for (const permission of role.getAllPermissions()) {
        if (permission.resource === resourceType || permission.resource === '*') {
          resources.push(Resource.of(resourceType));
        }
      }
    }

    return [...new Set(resources)]; // Remove duplicates
  }
}
```

### 3.2 Policy-Based Authorization

```typescript
// Authorization policy
interface AuthorizationPolicy {
  name: string;
  evaluate(context: PolicyContext): Promise<PolicyDecision>;
}

// Policy-based access control
class PolicyBasedAccessControl {
  private policies: Map<string, AuthorizationPolicy> = new Map();

  registerPolicy(policy: AuthorizationPolicy): void {
    this.policies.set(policy.name, policy);
  }

  async evaluate(policyName: string, context: PolicyContext): Promise<PolicyDecision> {
    const policy = this.policies.get(policyName);
    if (!policy) {
      throw new PolicyNotFoundError(policyName);
    }
    return policy.evaluate(context);
  }
}

// Example: Resource ownership policy
class OwnershipPolicy implements AuthorizationPolicy {
  name = 'ownership';

  async evaluate(context: PolicyContext): Promise<PolicyDecision> {
    const { userId, resource } = context;
    
    if (!resource.id) {
      return PolicyDecision.deny('Resource ID required for ownership check');
    }

    const ownership = await this.checkOwnership(userId, resource);
    if (!ownership) {
      return PolicyDecision.deny('You do not own this resource');
    }

    return PolicyDecision.allow();
  }

  private async checkOwnership(userId: UserId, resource: Resource): Promise<boolean> {
    // Implementation depends on resource type
    return false; // Placeholder
  }
}

// Usage
const policyEngine = new PolicyBasedAccessControl();
policyEngine.registerPolicy(new OwnershipPolicy());

const decision = await policyEngine.evaluate('ownership', {
  userId: currentUser.id,
  action: 'edit',
  resource: Resource.of('document', docId)
});
```

### 3.3 Attribute-Based Access Control (ABAC)

```typescript
// ABAC policy definition
class AttributeBasedPolicy {
  private rules: AccessRule[] = [];

  addRule(rule: AccessRule): void {
    this.rules.push(rule);
  }

  async evaluate(request: ABACRequest): Promise<ABACDecision> {
    for (const rule of this.rules) {
      if (this.matchesRule(request, rule)) {
        if (rule.effect === 'permit') {
          return ABACDecision.permit(rule.obligation);
        } else {
          return ABACDecision.deny(rule.obligation);
        }
      }
    }
    return ABACDecision.deny('No matching policy');
  }

  private matchesRule(request: ABACRequest, rule: AccessRule): boolean {
    // Check subject attributes
    for (const [attr, expected] of Object.entries(rule.subjectAttributes)) {
      if (request.subject[attr] !== expected) return false;
    }

    // Check resource attributes
    for (const [attr, expected] of Object.entries(rule.resourceAttributes)) {
      if (request.resource[attr] !== expected) return false;
    }

    // Check action matches
    if (rule.action && rule.action !== request.action) return false;

    // Check environment conditions
    for (const [attr, expected] of Object.entries(rule.environmentAttributes)) {
      if (request.environment[attr] !== expected) return false;
    }

    return true;
  }
}

interface AccessRule {
  name: string;
  effect: 'permit' | 'deny';
  subjectAttributes: Record<string, unknown>;
  resourceAttributes: Record<string, unknown>;
  action?: string;
  environmentAttributes?: Record<string, unknown>;
  obligation?: string;
}

// Example: Manager can edit documents in their department
const managerDocRule: AccessRule = {
  name: 'manager-edit-dept-docs',
  effect: 'permit',
  subjectAttributes: {
    role: 'MANAGER'
  },
  resourceAttributes: {
    type: 'document'
  },
  action: 'edit',
  environmentAttributes: {
    timeRange: 'business-hours'
  },
  obligation: 'Log access'
};
```

---

## 4. Domain Security Services

### 4.1 Authentication Service

```typescript
// Domain authentication service
interface AuthenticationService {
  authenticate(credentials: Credentials): Promise<Result<AuthenticationResult, AuthenticationError>>;
  verifyPassword(password: string, passwordHash: PasswordHash): Promise<boolean>;
  hashPassword(password: string): Promise<PasswordHash>;
  refreshSession(refreshToken: RefreshToken): Promise<Result<Session, SessionError>>;
}

class AuthenticationServiceImpl implements AuthenticationService {
  constructor(
    private userRepository: UserRepository,
    private tokenService: TokenService,
    private auditLogger: SecurityAuditLogger,
    private passwordHasher: PasswordHasher
  ) {}

  async authenticate(credentials: Credentials): Promise<Result<AuthenticationResult, AuthenticationError>> {
    const { email, password } = credentials;

    // Find user by email
    const emailVO = Email.create(email);
    if (emailVO.isFailure) {
      return Result.failure(new InvalidCredentialsError('Invalid email format'));
    }

    const userResult = await this.userRepository.findByEmail(emailVO.value);
    if (userResult.isFailure) {
      await this.auditLogger.logFailedLogin(email, 'user_not_found');
      return Result.failure(new InvalidCredentialsError('Invalid credentials'));
    }

    const user = userResult.value;

    // Check account status
    if (!user.isActive) {
      await this.auditLogger.logFailedLogin(email, 'account_disabled');
      return Result.failure(new AccountDisabledError());
    }

    // Verify password
    const passwordValid = await this.verifyPassword(password, user.passwordHash);
    if (!passwordValid) {
      await this.auditLogger.logFailedLogin(email, 'invalid_password');
      return Result.failure(new InvalidCredentialsError('Invalid credentials'));
    }

    // Generate tokens
    const accessToken = this.tokenService.generateAccessToken(user);
    const refreshToken = this.tokenService.generateRefreshToken(user);

    // Record successful login
    user.recordLogin();
    await this.userRepository.save(user);

    await this.auditLogger.logSuccessfulLogin(user.id, email);

    return Result.success(new AuthenticationResult(
      user,
      accessToken,
      refreshToken
    ));
  }

  async verifyPassword(password: string, passwordHash: PasswordHash): Promise<boolean> {
    return this.passwordHasher.verify(password, passwordHash);
  }

  async hashPassword(password: string): Promise<PasswordHash> {
    return this.passwordHasher.hash(password);
  }

  async refreshSession(refreshToken: RefreshToken): Promise<Result<Session, SessionError>> {
    const tokenData = this.tokenService.validateRefreshToken(refreshToken);
    if (tokenData.isFailure) {
      return Result.failure(tokenData.error);
    }

    const userResult = await this.userRepository.findById(tokenData.value.userId);
    if (userResult.isFailure) {
      return Result.failure(new SessionError('User not found'));
    }

    const user = userResult.value;
    if (!user.isActive) {
      return Result.failure(new AccountDisabledError());
    }

    const newAccessToken = this.tokenService.generateAccessToken(user);
    return Result.success(new Session(user, newAccessToken));
  }
}
```

### 4.2 Security Audit Service

```typescript
// Domain audit logging
interface SecurityAuditLogger {
  logEvent(event: SecurityEvent): Promise<void>;
  getEvents(filter: AuditFilter): Promise<SecurityEvent[]>;
  exportEvents(filter: AuditFilter, format: 'json' | 'csv'): Promise<string>;
}

class SecurityAuditLoggerImpl implements SecurityAuditLogger {
  constructor(private eventStore: SecurityEventStore) {}

  async logEvent(event: SecurityEvent): Promise<void> {
    // Validate event
    if (!event.isValid()) {
      throw new InvalidSecurityEventError();
    }

    // Store event
    await this.eventStore.save(event);

    // Real-time alerts for critical events
    if (event.requiresAlert()) {
      await this.sendSecurityAlert(event);
    }
  }

  async getEvents(filter: AuditFilter): Promise<SecurityEvent[]> {
    return this.eventStore.query(filter);
  }

  async exportEvents(filter: AuditFilter, format: 'json' | 'csv'): Promise<string> {
    const events = await this.getEvents(filter);
    
    if (format === 'json') {
      return JSON.stringify(events.map(e => e.toJSON()));
    } else {
      return this.toCSV(events);
    }
  }

  private async sendSecurityAlert(event: SecurityEvent): Promise<void> {
    // Send to security team (email, Slack, etc.)
    console.warn(`SECURITY ALERT: ${event.type} - ${event.description}`);
  }

  private toCSV(events: SecurityEvent[]): string {
    const headers = ['timestamp', 'type', 'userId', 'action', 'resource', 'result'];
    const rows = events.map(e => [
      e.timestamp.toISOString(),
      e.type,
      e.userId?.toString() || 'anonymous',
      e.action,
      e.resource?.toString() || '',
      e.result
    ]);
    return [headers.join(','), ...rows.map(r => r.join(','))].join('\n');
  }
}

// Security event types
class SecurityEvent {
  constructor(
    private readonly type: SecurityEventType,
    private readonly timestamp: DateTime,
    private readonly userId: UserId | null,
    private readonly action: string,
    private readonly resource: Resource | null,
    private readonly result: 'success' | 'failure',
    private readonly metadata: Record<string, unknown>
  ) {}

  isValid(): boolean {
    return this.type !== 'unknown' && this.timestamp !== null;
  }

  requiresAlert(): boolean {
    return [
      'UNAUTHORIZED_ACCESS_ATTEMPT',
      'SUSPICIOUS_LOGIN',
      'PASSWORD_BRUTE_FORCE',
      'UNAUTHORIZED_RESOURCE_ACCESS'
    ].includes(this.type);
  }

  toJSON(): object {
    return {
      type: this.type,
      timestamp: this.timestamp.toISOString(),
      userId: this.userId?.toString(),
      action: this.action,
      resource: this.resource?.toString(),
      result: this.result,
      metadata: this.metadata
    };
  }
}
```

---

## 5. Security Boundaries

### 5.1 Aggregate Security Boundaries

```typescript
// Security boundaries within aggregates
class AccountAggregate {
  private readonly id: AccountId;
  private owner: UserId;
  private members: AccountMember[];
  private isLocked: boolean;
  private securityEvents: SecurityEvent[];

  constructor(props: AccountProps) {
    this.id = props.id || AccountId.generate();
    this.owner = props.owner;
    this.members = props.members || [];
    this.isLocked = false;
    this.securityEvents = [];
  }

  // Authorization check before sensitive operations
  canModify(userId: UserId, action: AccountAction): boolean {
    if (this.isLocked) return false;

    if (this.owner.equals(userId)) return true;

    const member = this.members.find(m => m.userId.equals(userId));
    return member?.canPerform(action) ?? false;
  }

  // Record security event within aggregate
  recordSecurityEvent(event: SecurityEvent): void {
    this.securityEvents.push(event);
  }

  getSecurityEvents(): ReadonlyArray<SecurityEvent> {
    return [...this.securityEvents];
  }

  // Lock account - requires owner or admin
  lock(requestedBy: UserId): Result<void, AuthorizationError> {
    if (!this.canModify(requestedBy, 'lock')) {
      return Result.failure(new NotAuthorizedToLockAccountError(requestedBy));
    }

    this.isLocked = true;
    this.recordSecurityEvent(new SecurityEvent(
      'ACCOUNT_LOCKED',
      DateTime.now(),
      requestedBy,
      'lock',
      Resource.of('account', this.id.toString()),
      'success',
      {}
    ));

    return Result.success();
  }
}
```

### 5.2 Bounded Context Security

```typescript
// Security at bounded context boundaries
interface SecurityContext {
  getCurrentUser(): UserId | null;
  hasPermission(permission: Permission): boolean;
  isAuthenticated(): boolean;
}

// Anticorruption layer with security
class SecureAntiCorruptionLayer {
  constructor(
    private securityContext: SecurityContext,
    private externalService: ExternalServiceAdapter
  ) {}

  async getExternalData(request: ExternalRequest): Promise<Result<ExternalResponse, SecurityError>> {
    // Validate authentication first
    if (!this.securityContext.isAuthenticated()) {
      return Result.failure(new AuthenticationRequiredError());
    }

    // Check permission for this specific operation
    const permission = new Permission(request.resource, request.action);
    if (!this.securityContext.hasPermission(permission)) {
      return Result.failure(new AccessDeniedError(this.securityContext.getCurrentUser()!, permission));
    }

    // Proceed with external call
    return this.externalService.fetch(request);
  }
}
```

---

## 6. AI-Specific Guidelines

### 6.1 Security Pattern Generation

When AI generates security code:

1. **Never generate authentication bypass code**
2. **Always use parameterized queries for user lookups**
3. **Hash passwords with Argon2 or bcrypt**
4. **Validate all authorization requests**
5. **Log all security events**

### 6.2 Common Security Mistakes to Avoid

| Mistake | Correction |
|---------|------------|
| Storing passwords in plain text | Use bcrypt/Argon2 with salt |
| Checking permissions after logic | Check authorization first |
| Using user-provided IDs for authorization | Derive resource IDs from authenticated context |
| Missing audit logging | Log all security events |
| Hardcoding secrets | Use environment variables/vault |
| Not validating tokens | Always validate JWT signatures |
| Ignoring token expiration | Check `exp` claim |
| Exposing internal errors | Translate to generic error messages |

### 6.3 Security Review Checklist

```markdown
## Security Review Checklist for AI-Generated Code

### Authentication
- [ ] Passwords hashed with Argon2/bcrypt
- [ ] No hardcoded credentials
- [ ] Token validation includes signature, expiration, issuer
- [ ] Authentication errors don't reveal sensitive info

### Authorization
- [ ] Authorization checks before business logic
- [ ] Resource access tied to authenticated user
- [ ] Role/permission checks are explicit
- [ ] No IDOR vulnerabilities (Insecure Direct Object Reference)

### Data Protection
- [ ] Sensitive data encrypted at rest
- [ ] TLS for all communications
- [ ] No sensitive data in logs
- [ ] PII properly handled (masked, encrypted)

### Audit
- [ ] Security events logged
- [ ] Login attempts tracked
- [ ] Failed authorizations logged
- [ ] Audit logs immutable
```

---

## 7. References

1. doc/design-patterns.md - Security patterns
2. doc/vertical-slice-architecture-megred.md - Auth middleware examples
3. ref/CONSTITUTION.md - Security conventions
4. OWASP Security Guidelines
5. "Best Practices for User Authentication and Authorization" - Security Boulevard
6. "Practical Guide to OAuth2, JWT Authentication and RBAC" - Medium

---

## Appendix A: Input Validation Security

### A.1 Input Validation Layers

| Layer | Validation Type | Examples |
|-------|-----------------|----------|
| **Presentation** | Format validation | Email format, string length |
| **Application** | Business rules | Order limits, allowed values |
| **Domain** | Invariants | Entity constraints, business rules |
| **Infrastructure** | Sanitization | SQL parameters, XSS prevention |

### A.2 Validation Best Practices (from CONSTITUTION.md)

**Input Validation:**
- Validate all inputs
- Sanitize user data
- Validate domain constraints
- Type-safe where possible

**Data Protection:**
- No logging of sensitive data
- Secure storage of credentials
- Proper error messages (don't leak info)
- Use framework security features

### A.3 Common Validation Patterns

```typescript
// Input sanitization
function sanitizeInput(input: string): string {
  return input
    .replace(/[<>]/g, '')  // Remove HTML tags
    .trim()
    .substring(0, 1000);   // Limit length
}

// SQL injection prevention
async function findUserByEmail(email: string): Promise<User | null> {
  // ❌ BAD - SQL injection vulnerable
  // return db.query(`SELECT * FROM users WHERE email = '${email}'`);
  
  // ✅ GOOD - Parameterized query
  return db.query(
    'SELECT * FROM users WHERE email = $1',
    [email]
  );
}

// XSS prevention
function escapeHtml(text: string): string {
  const map: Record<string, string> = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#039;'
  };
  return text.replace(/[&<>"']/g, m => map[m]);
}
```

---

## Appendix B: Dependency Injection Security

### B.1 Constructor Injection (from CONSTITUTION.md)

**Constructor injection for dependencies:**
- Constructor injection preferred
- Field injection only for framework components
- Inject interfaces, not implementations
- Use configuration classes for wiring

### B.2 Security Benefits of DI

```typescript
// ✅ GOOD - Constructor injection with interfaces
class PaymentService {
  constructor(
    private readonly paymentRepository: PaymentRepository,
    private readonly authorizationService: AuthorizationService,
    private readonly auditLogger: AuditLogger
  ) {}
}

// ❌ BAD - Direct instantiation (hard to test, security risk)
class PaymentService {
  private readonly paymentRepository = new DatabasePaymentRepository();
  private readonly authorizationService = new AuthService();
}
```

### B.3 Preventing Unauthorized Modifications (from CONSTITUTION.md)

**Prevent unauthorized modifications:**
- Use readonly where possible
- Validate in setters
- Use private fields with accessors
- Implement proper encapsulation

---

## Appendix C: Secure Coding Guidelines

### C.1 Secure Coding Principles

| Principle | Description | Example |
|-----------|-------------|---------|
| **Fail Secure** | Default to secure state | Deny access on error |
| **Defense in Depth** | Multiple security layers | Auth + authorization + encryption |
| **Least Privilege** | Minimal permissions | Service accounts with limited access |
| **Secure Defaults** | Start with secure config | HTTPS required, CORS strict |
| **Fail Fast** | Detect errors early | Validate inputs before processing |

### C.2 Secure Error Handling

```typescript
// ❌ BAD - Leaking internal errors
try {
  await processPayment(order);
} catch (error) {
  // ❌ Never log sensitive data or return internal errors
  logger.error(error.stack);  // Stack trace leaks internal details
  throw error;  // Internal error exposed
}

// ✅ GOOD - Generic error to user, detailed log internally
try {
  await processPayment(order);
} catch (error) {
  // Log full details internally
  logger.error('Payment processing failed', { 
    orderId: order.id, 
    error: error.message,
    stack: error.stack 
  });
  
  // Return generic error to user
  throw new PaymentProcessingError('Unable to process payment');
}
```

### C.3 Security Headers

```typescript
// Express.js security headers
import helmet from 'helmet';

app.use(helmet());

// Or manually set headers
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

---

## Appendix D: Security Audit Checklist

### D.1 Code Review Security Checklist

- [ ] All inputs validated and sanitized
- [ ] No hardcoded credentials or secrets
- [ ] Authentication flows use secure patterns
- [ ] Authorization checks on every protected resource
- [ ] Sensitive data encrypted at rest and in transit
- [ ] No sensitive data in logs
- [ ] Proper error handling (no information leakage)
- [ ] Dependencies up to date
- [ ] Security headers configured
- [ ] CSRF protection implemented
- [ ] Rate limiting on sensitive endpoints

### D.2 Dependency Security

| Check | Action |
|-------|--------|
| Outdated dependencies | Regular updates, use Dependabot |
| Known vulnerabilities | Scan with npm audit, Snyk |
| Unnecessary dependencies | Remove unused packages |
| License compliance | Review dependencies' licenses |

### D.3 Security Testing

| Test Type | Purpose | Tools |
|-----------|---------|-------|
| Static Analysis | Find vulnerabilities in code | SonarQube, ESLint security plugin |
| Dynamic Testing | Test running application | OWASP ZAP, Burp Suite |
| Penetration Testing | Simulate real attacks | Professional pentest |
| Dependency Scanning | Check for vulnerable packages | npm audit, Snyk |
| Secret Scanning | Detect hardcoded secrets | TruffleHog, GitLeaks |
