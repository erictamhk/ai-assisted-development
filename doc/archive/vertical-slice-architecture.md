# Vertical Slice Architecture

## Overview

Vertical Slice Architecture is an approach to organizing code by feature slices rather than by technical layers. Each slice contains all the components needed to implement a specific feature, from the user interface to the database access. This contrasts with traditional layered architectures where code is organized by technical type (controllers, services, repositories).

## Core Concept

```
Traditional Layered Architecture          Vertical Slice Architecture
┌──────────────────────────────────┐     ┌──────────────────────────────────┐
│          Presentation            │     │      Feature: User Auth          │
├──────────┬──────────┬──────────┤     ├──────────────────────────────────┤
│Controllers│Controllers│Controllers│     │  UserAuthController            │
├──────────┴──────────┴──────────┤     │  LoginUseCase                  │
│           Business Logic        │     │  AuthRepository                │
├──────────┬──────────┬──────────┤     │  AuthEvents                    │
│ Services │ Services │ Services │     │  AuthValidator                │
├──────────┴──────────┴──────────┤     ├──────────────────────────────────┤
│         Data Access             │     │     Feature: Order Management   │
├──────────┬──────────┬──────────┤     ├──────────────────────────────────┤
│  Repos  │  Repos  │  Repos   │     │  OrderController               │
└──────────┴──────────┴──────────┘     │  CreateOrderUseCase            │
                                       │  OrderRepository               │
                                       │  OrderEvents                   │
                                       └──────────────────────────────────┘
```

## Folder Structure

```
src/
├── features/
│   ├── authentication/
│   │   ├── api/                   # HTTP handlers
│   │   │   ├── login.controller.ts
│   │   │   ├── register.controller.ts
│   │   │   └── auth.middleware.ts
│   │   ├── application/           # Use cases
│   │   │   ├── use-cases/
│   │   │   │   ├── login/
│   │   │   │   │   ├── login.handler.ts
│   │   │   │   │   ├── login.request.ts
│   │   │   │   │   └── login.response.ts
│   │   │   │   └── register/
│   │   │   │       ├── register.handler.ts
│   │   │   │       ├── register.request.ts
│   │   │   │       └── register.response.ts
│   │   │   └── ports/            # Input/output ports
│   │   │       ├── inbound/
│   │   │       └── outbound/
│   │   ├── domain/               # Domain logic
│   │   │   ├── entities/
│   │   │   ├── value-objects/
│   │   │   ├── events/
│   │   │   └── services/
│   │   ├── infrastructure/       # External concerns
│   │   │   ├── repositories/
│   │   │   │   └── prisma-auth.repository.ts
│   │   │   ├── services/
│   │   │   │   ├── token.service.ts
│   │   │   │   └── email.service.ts
│   │   │   └── validators/
│   │   │       └── auth.validator.ts
│   │   └── tests/                # Feature tests
│   │       ├── login.e2e-spec.ts
│   │       └── login.mocks.ts
│   │
│   ├── orders/
│   │   ├── api/
│   │   ├── application/
│   │   ├── domain/
│   │   ├── infrastructure/
│   │   └── tests/
│   │
│   └── payments/
│       ├── api/
│       ├── application/
│       ├── domain/
│       ├── infrastructure/
│       └── tests/
│
├── shared/                         # Shared code
│   ├── kernel/                     # Core utilities
│   ├── utils/
│   ├── types/
│   ├── decorators/
│   └── validators/
│
├── core/                           # Framework agnostic
│   ├── bus/
│   ├── logger/
│   └── config/
│
└── main.ts                         # Entry point
```

## Implementation Example

### Feature: User Authentication

```typescript
// features/authentication/api/auth.controller.ts
import { Request, Response, Router } from 'express';
import { LoginHandler } from '../application/use-cases/login/login.handler';
import { RegisterHandler } from '../application/use-cases/register/register.handler';

export class AuthController {
  private router: Router;

  constructor(
    private readonly loginHandler: LoginHandler,
    private readonly registerHandler: RegisterHandler
  ) {
    this.router = Router();
    this.setupRoutes();
  }

  private setupRoutes(): void {
    this.router.post('/login', this.login.bind(this));
    this.router.post('/register', this.register.bind(this));
  }

  async login(req: Request, res: Response): Promise<void> {
    const result = await this.loginHandler.execute(req.body);
    
    if (result.success) {
      res.json({
        success: true,
        data: {
          user: result.value.user,
          accessToken: result.value.accessToken,
          refreshToken: result.value.refreshToken
        }
      });
    } else {
      res.status(401).json({
        success: false,
        error: result.error.message
      });
    }
  }

  async register(req: Request, res: Response): Promise<void> {
    const result = await this.registerHandler.execute(req.body);
    
    if (result.success) {
      res.status(201).json({
        success: true,
        data: {
          user: result.value.user,
          accessToken: result.value.accessToken
        }
      });
    } else {
      res.status(400).json({
        success: false,
        error: result.error.message
      });
    }
  }

  get router(): Router {
    return this.router;
  }
}
```

```typescript
// features/authentication/application/use-cases/login/login.handler.ts
import { Result, success, failure } from '../../../../shared/core/result';
import { UserRepository } from '../../ports/outbound/user.repository';
import { TokenService } from '../../ports/outbound/token.service';
import { AuthValidator } from '../../infrastructure/validators/auth.validator';

interface LoginRequest {
  email: string;
  password: string;
}

interface LoginResponse {
  user: {
    id: string;
    email: string;
    name: string;
  };
  accessToken: string;
  refreshToken: string;
}

type LoginResult = Result<LoginResponse, 'INVALID_CREDENTIALS' | 'VALIDATION_ERROR'>;

export class LoginHandler {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly tokenService: TokenService,
    private readonly validator: AuthValidator
  ) {}

  async execute(request: LoginRequest): Promise<LoginResult> {
    // 1. Validate input
    const validationResult = this.validator.validateLogin(request);
    if (!validationResult.isValid) {
      return failure('VALIDATION_ERROR');
    }

    // 2. Find user
    const user = await this.userRepository.findByEmail(request.email);
    if (!user) {
      return failure('INVALID_CREDENTIALS');
    }

    // 3. Verify password
    const isValid = await user.verifyPassword(request.password);
    if (!isValid) {
      return failure('INVALID_CREDENTIALS');
    }

    // 4. Generate tokens
    const accessToken = this.tokenService.generateAccessToken(user);
    const refreshToken = this.tokenService.generateRefreshToken(user);

    // 5. Return response
    return success({
      user: {
        id: user.id.value,
        email: user.email.value,
        name: user.name.value
      },
      accessToken,
      refreshToken
    });
  }
}
```

```typescript
// features/authentication/domain/entities/user.entity.ts
import { Entity } from '../../../../shared/domain/entity';
import { Email } from '../../../../shared/domain/value-objects/email';
import { Password } from '../../../../shared/domain/value-objects/password';
import { UserId } from './user-id';

interface UserProps {
  id: UserId;
  email: Email;
  name: string;
  password: Password;
  createdAt: Date;
  lastLoginAt?: Date;
}

export class User extends Entity<UserId, UserProps> {
  private constructor(props: UserProps) {
    super(props);
  }

  static create(input: { email: string; name: string; password: string }): User {
    const user = new User({
      id: UserId.generate(),
      email: Email.create(input.email),
      name: input.name,
      password: Password.create(input.password),
      createdAt: new Date()
    });
    return user;
  }

  async verifyPassword(plainPassword: string): Promise<boolean> {
    return this._props.password.compare(plainPassword);
  }

  updateLastLogin(): void {
    this._props.lastLoginAt = new Date();
  }

  get email(): Email {
    return this._props.email;
  }

  get name(): string {
    return this._props.name;
  }
}
```

```typescript
// features/authentication/infrastructure/repositories/prisma-user.repository.ts
import { UserRepository } from '../../ports/outbound/user.repository';
import { User } from '../../domain/entities/user.entity';
import { Email } from '../../../../shared/domain/value-objects/email';
import { Password } from '../../../../shared/domain/value-objects/password';
import { PrismaClient } from '@prisma/client';

export class PrismaUserRepository implements UserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    const user = await this.prisma.user.findUnique({ where: { id } });
    if (!user) return null;
    return this.toDomain(user);
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.prisma.user.findUnique({ where: { email } });
    if (!user) return null;
    return this.toDomain(user);
  }

  async save(user: User): Promise<void> {
    await this.prisma.user.upsert({
      where: { id: user.id.value },
      create: {
        id: user.id.value,
        email: user.email.value,
        name: user.name,
        password: user.password.value,
        createdAt: user.props.createdAt
      },
      update: {
        name: user.name,
        lastLoginAt: user.props.lastLoginAt
      }
    });
  }

  private toDomain(data: any): User {
    return User.create({
      email: data.email,
      name: data.name,
      password: data.password
    });
  }
}
```

## Feature Testing

```typescript
// features/authentication/tests/login.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import { AuthModule } from '../auth.module';
import { PrismaService } from '../../../shared/infrastructure/prisma.service';

describe('Authentication E2E', () => {
  let app: INestApplication;
  let prisma: PrismaService;

  beforeAll(async () => {
    const module: TestingModule = await Test.createTestingModule({
      imports: [AuthModule]
    }).compile();

    app = module.createNestApplication();
    await app.init();
    prisma = module.get(PrismaService);
  });

  beforeEach(async () => {
    // Clean up database
    await prisma.user.deleteMany();
  });

  afterAll(async () => {
    await app.close();
  });

  it('should register a new user', async () => {
    const response = await request(app.getHttpServer())
      .post('/auth/register')
      .send({
        email: 'test@example.com',
        password: 'SecurePass123!',
        name: 'Test User'
      })
      .expect(201);

    expect(response.body.success).toBe(true);
    expect(response.body.data.user.email).toBe('test@example.com');
    expect(response.body.data.accessToken).toBeDefined();
  });

  it('should login with valid credentials', async () => {
    // First register
    await request(app.getHttpServer())
      .post('/auth/register')
      .send({
        email: 'test@example.com',
        password: 'SecurePass123!',
        name: 'Test User'
      });

    // Then login
    const response = await request(app.getHttpServer())
      .post('/auth/login')
      .send({
        email: 'test@example.com',
        password: 'SecurePass123!'
      })
      .expect(200);

    expect(response.body.success).toBe(true);
    expect(response.body.data.accessToken).toBeDefined();
  });
});
```

## Cross-Cutting Concerns

```typescript
// features/authentication/api/auth.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { TokenService } from '../ports/outbound/token.service';

@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(private readonly tokenService: TokenService) {}

  async use(req: Request, res: Response, next: NextFunction): Promise<void> {
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      res.status(401).json({ error: 'No token provided' });
      return;
    }

    const token = authHeader.split(' ')[1];
    
    try {
      const payload = this.tokenService.verifyAccessToken(token);
      (req as any).user = payload;
      next();
    } catch (error) {
      res.status(401).json({ error: 'Invalid token' });
    }
  }
}

// Shared middleware registration
// shared/infrastructure/decorators/auth.decorator.ts
export const Auth = () => (target: any, key: string, descriptor: PropertyDescriptor) => {
  // Apply authentication to route handlers
};
```

## Benefits of Vertical Slice Architecture

| Benefit | Description |
|---------|-------------|
| **Feature Isolation** | Changes to one feature don't affect others |
| **Team Scalability** | Teams can work on different features independently |
| **Faster Development** | All code for a feature in one place |
| **Better Testing** | Tests are focused on single features |
| **Reduced Cognitive Load** | Don't need to understand entire codebase |
| **Easier Refactoring** | Feature boundaries make refactoring safer |

## When to Use

✅ **Use Vertical Slice Architecture when:**
- Building feature-rich applications
- Working in cross-functional teams
- Prioritizing feature delivery over technical layers
- Using frameworks that support feature-based organization (NestJS, Angular)

❌ **Consider alternatives when:**
- Building simple CRUD applications
- Team is small and highly cohesive
- Strict technical layering provides more value

## References

- "Vertical Slice Architecture" by Jimmy Bogard
- "Pragmatic Clean Architecture" series
- "Feature Folders" pattern
- NestJS Modular Architecture
