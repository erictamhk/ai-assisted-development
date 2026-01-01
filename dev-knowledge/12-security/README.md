# Security Patterns

This folder contains comprehensive knowledge about security patterns and best practices for building secure applications with AI-assisted development.

## Contents

### Authentication & Authorization

- **[auth-patterns.md](auth-patterns.md)**: Authentication and authorization patterns including OAuth 2.0, JWT, session management, and role-based access control
- **[comprehensive-security-guide.md](comprehensive-security-guide.md)**: Complete security guide covering common vulnerabilities (OWASP Top 10), mitigations, and security best practices
- **[domain-security.md](domain-security.md)**: Domain-driven security patterns and domain-specific security concerns including aggregate-level security

## Quick Reference

### Security Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                    SECURITY LAYERS                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    INPUT VALIDATION                       │   │
│   │         Sanitize and validate all inputs                 │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│   ┌───────────────────────────▼─────────────────────────────┐   │
│   │                 AUTHENTICATION & AUTHORIZATION            │   │
│   │            Verify identity and check permissions          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│   ┌───────────────────────────▼─────────────────────────────┐   │
│   │                 DATA SECURITY                             │   │
│   │        Encryption, hashing, and secure storage            │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│   ┌───────────────────────────▼─────────────────────────────┐   │
│   │                 INFRASTRUCTURE SECURITY                   │   │
│   │        Network security, logging, and monitoring          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Security Principles

| Principle | Description |
|-----------|-------------|
| **Defense in Depth** | Multiple layers of security |
| **Least Privilege** | Minimum necessary permissions |
| **Fail Secure** | Default to secure state |
| **Separation of Duties** | Divide responsibilities |
| **Zero Trust** | Verify everything |

## Common Vulnerabilities

### OWASP Top 10 (2021)

| Rank | Vulnerability | Mitigation |
|------|---------------|------------|
| A01:2021 | Broken Access Control | Proper authorization checks |
| A02:2021 | Cryptographic Failures | Use strong encryption |
| A03:2021 | Injection | Input validation, parameterized queries |
| A04:2021 | Insecure Design | Security by design |
| A05:2021 | Security Misconfiguration | Secure defaults, minimal surface |
| A06:2021 | Vulnerable Components | Keep dependencies updated |
| A07:2021 | Identification Failures | Strong authentication |
| A08:2021 | Software/Data Integrity | Integrity verification |
| A09:2021 | Security Logging Failures | Comprehensive logging |
| A10:2021 | Server-Side Request Forgery | Validate URLs, disable unused protocols |

## Security Patterns

### Authentication Patterns

```typescript
interface AuthService {
  authenticate(credentials: Credentials): Promise<AuthResult>;
  refreshToken(refreshToken: string): Promise<AuthResult>;
  revokeToken(token: string): Promise<void>;
}

class JWTAuthService implements AuthService {
  async authenticate(credentials: Credentials): Promise<AuthResult> {
    const user = await this.userRepository.findByEmail(credentials.email);
    if (!user || !this.verifyPassword(credentials.password, user.passwordHash)) {
      throw new AuthenticationError('Invalid credentials');
    }
    return this.generateTokens(user);
  }
}
```

### Authorization Patterns

```typescript
interface AuthorizationService {
  checkPermission(user: User, resource: Resource, action: string): Promise<boolean>;
  hasRole(user: User, role: string): boolean;
}

class RBACService implements AuthorizationService {
  private readonly permissions: Map<string, Set<string>> = new Map([
    ['admin', new Set(['read', 'write', 'delete', 'admin'])],
    ['user', new Set(['read', 'write'])],
    ['guest', new Set(['read'])]
  ]);
}
```

### Input Validation

```typescript
class InputValidator {
  validateEmail(email: string): Result<string, ValidationError> {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      return Result.failure(new ValidationError('Invalid email format'));
    }
    return Result.success(email.toLowerCase());
  }

  sanitizeHtml(html: string): string {
    return DOMPurify.sanitize(html);
  }
}
```

## Anti-Patterns to Avoid

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| **Storing Plaintext Passwords** | Never store passwords in plaintext | Use bcrypt/Argon2 |
| **Rolling Your Own Crypto** | Crypto is hard, use established libraries | Use standard libraries |
| **No Input Validation** | Trusting user input | Validate everything |
| **Verbose Error Messages** | Leaking information | Generic error messages |
| **No Rate Limiting** | Vulnerable to brute force | Implement rate limits |
| **Insecure Direct Object References** | Exposing internal IDs | Use indirect references |

## Best Practices

### 1. Authentication

- Use strong password policies
- Implement multi-factor authentication
- Use secure session management
- Implement proper logout
- Monitor for suspicious activity

### 2. Authorization

- Implement role-based access control
- Use the principle of least privilege
- Verify permissions on every request
- Log authorization failures

### 3. Data Protection

- Encrypt sensitive data at rest
- Use TLS for data in transit
- Hash passwords properly
- Manage keys securely

### 4. Secure Development

- Keep dependencies updated
- Use static analysis tools
- Conduct security reviews
- Follow secure coding standards

## Domain Security

### Aggregate-Level Security

```typescript
class Order implements AggregateRoot<OrderId> {
  private constructor(
    private readonly id: OrderId,
    private readonly owner: UserId,
    private status: OrderStatus
  ) {}

  static create(id: OrderId, owner: UserId): Order {
    return new Order(id, owner, OrderStatus.DRAFT);
  }

  updateShippingAddress(address: Address, requester: User): void {
    if (!requester.id.equals(this.owner)) {
      throw new AuthorizationError('Cannot modify order');
    }
    this.shippingAddress = address;
  }
}
```

## Related Topics

- **Architecture**: [03-architecture/](../03-architecture/) - Secure architecture patterns
- **DDD**: [02-ddd/](../02-ddd/) - Domain security patterns
- **Error Handling**: [11-error-handling/](../11-error-handling/) - Security error handling
- **Deployment**: [13-deployment/](../13-deployment/) - Security in deployment

## References

- OWASP Top 10: https://owasp.org/Top10/
- "Security Engineering" by Ross Anderson
- "The Web Application Hacker's Handbook"
- NIST Cybersecurity Framework
- OAuth 2.0 Specification (RFC 6749)
