# Vertical Slice Architecture

## Overview

Vertical Slice Architecture is an approach to organizing code by feature slices rather than by technical layers. Each slice contains all the components needed to implement a specific feature, from the user interface to the database access. This contrasts with traditional layered architectures where code is organized by technical type (controllers, services, repositories).

This approach combines the benefits of modular architecture with the simplicity of feature-focused development. Each slice maintains clean separation from others, but within a slice, all related code is co-located for easy navigation and understanding.

---

## Core Concept

Vertical Slice Architecture is an organizational pattern that structures code by business features (vertical slices) rather than by technical layers (horizontal layers). Each vertical slice contains all the code needed to implement a specific feature, from the user interface down to the database, while maintaining architectural boundaries within the slice.

The key insight is that features are the primary unit of change in software systems. When requirements change, they typically change within a single feature, not across all entities or all controllers. By organizing code around features, each slice becomes a self-contained unit that can be developed, tested, and deployed independently.

Vertical Slice Architecture is particularly valuable in large, complex applications where purely layered architectures can lead to code that is difficult to navigate and modify. By keeping feature-related code together, developers can work more efficiently and with better context.

---

## Vertical Slice vs Layer-Based Structure

```
TRADITIONAL LAYER-BASED STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

src/
├── domain/
│   ├── Order.java
│   ├── Customer.java
│   └── Item.java
├── usecase/
│   ├── CreateOrderUseCase.java
│   ├── UpdateOrderUseCase.java
│   └── ...
├── adapter/
│   ├── controller/
│   ├── presenter/
│   └── repository/
└── ...

Problem: Related code is scattered across the project
         Changes span multiple directories
         Hard to see feature boundaries
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

VERTICAL SLICE STRUCTURE
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

src/
├── order/              # Feature: Order management
│   ├── domain/         # Domain objects for this feature
│   ├── usecase/        # Use cases for this feature
│   └── adapter/        # Adapters for this feature
│
├── customer/           # Feature: Customer management
│   ├── domain/
│   ├── usecase/
│   └── adapter/
│
├── payment/            # Feature: Payment processing
│   ├── domain/
│   ├── usecase/
│   └── adapter/
│
└── shared/             # Cross-cutting concerns
    ├── adapter/
    └── io/

Benefit: All code for a feature is in one place
         Clear feature boundaries
         Easy to navigate and modify
```

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

---

## Folder Structure

### Java-Based Structure

```
src/main/java/.../tasks/
│
├── feature-name/                         # Vertical Slice
│   │
│   ├── domain/                          # Business logic
│   │   ├── Entity.java                  # Domain entity
│   │   ├── EntityId.java                # Value object ID
│   │   └── ReadOnlyEntity.java          # Read-only interface
│   │
│   ├── usecase/                         # Application logic
│   │   ├── port/
│   │   │   ├── in/
│   │   │   │   └── add/                 # Use case subdirectory
│   │   │   │       ├── AddUseCase.java  # Input port
│   │   │   │       ├── AddInput.java    # Input object
│   │   │   │       └── AddOutput.java   # Output object
│   │   │   │
│   │   │   └── port/
│   │   │       ├── EntityDto.java       # DTO
│   │   │       ├── EntityPo.java        # Persistent object
│   │   │       └── EntityMapper.java    # Mapper
│   │   │
│   │   └── service/
│   │       └── AddService.java          # Use case implementation
│   │
│   └── adapter/                         # External integration
│       ├── in/
│       │   └── controller/
│       │       ├── console/             # Console adapter
│       │       │   └── AddController.java
│       │       └── web/                 # Web adapter
│       │           └── AddController.java
│       │
│       └── out/
│           ├── presenter/
│           │   └── AddPresenter.java
│           └── repository/
│               └── EntityRepository.java
│
├── shared/                              # Cross-cutting concerns
│   ├── adapter/
│   │   └── out/
│   │       └── repository/              # Shared implementations
│   │           ├── CrudRepository.java
│   │           └── InMemoryRepository.java
│   │
│   └── io/
│       ├── springboot/                  # Framework integration
│       │   ├── Application.java
│       │   └── config/
│       │       ├── RepositoryConfig.java
│       │       └── UseCaseConfig.java
│       └── standard/
│           └── Main.java
│
└── resources/
    └── application.properties
```

### TypeScript/JavaScript Structure (NestJS)

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

---

## Feature Organization

Each feature should be self-contained with its own domain, use cases, and adapters. The aggregate root feature manages communication between features.

### Feature Independence

```
FEATURE INDEPENDENCE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✓ CORRECT: Each feature is self-contained
   project/ → domain + usecase + adapter
   task/ → domain + usecase + adapter
   order/ → domain + usecase + adapter

✓ CORRECT: Cross-feature communication through aggregate root
   task/ → order/ (aggregate root) → project/
   (task communicates with project through order)

✗ WRONG: Direct dependencies between features
   task/ → project/ (direct import, BAD)
   project/ → task/ (direct import, BAD)

✓ CORRECT: Shared infrastructure only
   shared/ → contains repository implementations
   shared/ → contains configuration classes
   shared/ → contains framework setup
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Aggregate Root Pattern

The aggregate root feature (order/) serves as the entry point for managing other features:

```
AGGREGATE ROOT COMMUNICATION
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

order/ (Aggregate Root)
    │
    ├── Manages customer/ entities
    │   └── Controls consistency boundaries
    │
    ├── Manages item/ entities
    │   └── Ensures business invariants
    │
    └── Provides repository interfaces
        → used by all features

item/ Feature
    ├── Depends on order/ repository interface
    ├── NO direct dependency on customer/
    └── Imports NO domain classes from customer/

customer/ Feature
    ├── Depends on order/ repository interface
    ├── NO direct dependency on item/
    └── Imports NO domain classes from item/

shared/ Infrastructure
    ├── Contains repository implementations
    ├── Contains configuration classes
    └── Used by all features
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Code Placement Reference

| Code Type | Location | Example |
|-----------|----------|---------|
| New Entity | `feature/domain/` | `Project.java` |
| New Value Object | `feature/domain/` | `ProjectName.java` |
| Read-Only Interface | `feature/domain/` | `ReadOnlyProject.java` |
| Use Case Interface | `feature/usecase/port/in/verb/` | `AddProjectUseCase.java` |
| Input Class | `feature/usecase/port/in/verb/` | `AddProjectInput.java` |
| Output Class | `feature/usecase/port/in/verb/` | `AddProjectOutput.java` |
| Use Case Service | `feature/usecase/service/` | `AddProjectService.java` |
| DTO | `feature/usecase/port/` | `ProjectDto.java` |
| PO | `feature/usecase/port/` | `ProjectPo.java` |
| Mapper | `feature/usecase/port/` | `ProjectMapper.java` |
| Controller (Console) | `feature/adapter/in/controller/console/` | `AddProjectConsoleController.java` |
| Controller (Web) | `feature/adapter/in/controller/web/` | `AddProjectWebController.java` |
| Presenter Interface | `feature/usecase/port/out/` | `AddProjectPresenter.java` |
| Presenter Implementation | `feature/adapter/out/presenter/` | `AddProjectConsolePresenter.java` |
| Repository Interface | `feature/adapter/out/repository/` | `ProjectRepository.java` |
| Repository Implementation | `shared/adapter/out/repository/` | `ProjectCrudRepository.java` |
| Configuration | `shared/io/framework/config/` | `UseCaseInjection.java` |

---

## Dependency Rules Within a Feature

```
INTRA-FEATURE DEPENDENCIES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

domain/ (no dependencies - pure business logic)
    ↑
usecase/service/ → depends on → domain/
    ↑
usecase/port/ (DTOs, POs, Mappers - no behavior)
    ↑
adapter/in/ → depends on → usecase/port/in/ (use case interfaces)
    ↑
adapter/out/ → depends on → usecase/port/out/ (output ports) + domain/
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

INTER-FEATURE DEPENDENCIES
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

order/ (aggregate root)
    ↓ manages
customer/ and item/
    → communicate through order/ ONLY
    → NO direct imports between features

item/ and customer/
    → depend ONLY on shared/ for infrastructure
    → NEVER depend on each other
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DEPENDENCY VIOLATIONS (NEVER DO)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✗ Adapter depends on another adapter directly
✗ Domain layer depends on use case layer
✗ Use case layer depends on adapter layer
✗ Feature A imports domain from Feature B
✗ Feature A calls use case from Feature B
✗ Shared contains feature-specific code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Implementation Example

### Java Example: Feature Structure

```java
// feature/domain/Project.java
public class Project {
    private final ProjectId id;
    private final ProjectName name;
    private final List<Task> tasks;
    
    public Project(ProjectId id, ProjectName name) {
        this.id = id;
        this.name = name;
        this.tasks = new ArrayList<>();
    }
    
    public void addTask(Task task) {
        this.tasks.add(task);
    }
}

// feature/usecase/port/in/add/AddProjectUseCase.java
public interface AddProjectUseCase {
    AddProjectOutput execute(AddProjectInput input);
}

// feature/usecase/service/AddProjectService.java
@Service
public class AddProjectService implements AddProjectUseCase {
    private final ProjectRepository repository;
    
    public AddProjectService(ProjectRepository repository) {
        this.repository = repository;
    }
    
    @Override
    public AddProjectOutput execute(AddProjectInput input) {
        Project project = new Project(ProjectId.generate(), new ProjectName(input.getName()));
        repository.save(project);
        return new AddProjectOutput(project);
    }
}

// feature/adapter/in/controller/web/AddProjectController.java
@RestController
@RequestMapping("/api/projects")
public class AddProjectController {
    private final AddProjectUseCase addProjectUseCase;
    
    public AddProjectController(AddProjectUseCase addProjectUseCase) {
        this.addProjectUseCase = addProjectUseCase;
    }
    
    @PostMapping
    public ResponseEntity<AddProjectOutput> addProject(@RequestBody AddProjectInput input) {
        AddProjectOutput output = addProjectUseCase.execute(input);
        return ResponseEntity.status(HttpStatus.CREATED).body(output);
    }
}
```

### TypeScript Example: Authentication Feature

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

---

## Creating a New Feature

```bash
# Step 1: Create feature directory structure
mkdir -p src/main/java/.../tasks/newfeature/

# Step 2: Create the structure
newfeature/
├── domain/
│   ├── NewFeature.java
│   └── NewFeatureId.java
├── usecase/
│   ├── port/
│   │   ├── in/
│   │   │   └── action/
│   │   │       ├── ActionUseCase.java
│   │   │       ├── ActionInput.java
│   │   │       └── ActionOutput.java
│   │   ├── NewFeatureDto.java
│   │   ├── NewFeaturePo.java
│   │   └── NewFeatureMapper.java
│   └── service/
│       └── ActionService.java
└── adapter/
    ├── in/
    │   └── controller/
    │       ├── console/
    │       │   └── ActionConsoleController.java
    │       └── web/
    │           └── ActionWebController.java
    └── out/
        ├── presenter/
        │   └── ActionPresenter.java
        └── repository/
            └── NewFeatureRepository.java

# Step 3: Register in shared infrastructure
# Add to shared/io/framework/config/Injection.java:
@Bean
public NewFeatureRepository newFeatureRepository() {
    return new NewFeatureCrudRepository();
}

@Bean
public ActionUseCase actionUseCase(NewFeatureRepository repo) {
    return new ActionService(repo);
}

# Step 4: Update aggregate root if needed
# In order/domain/Order.java:
public NewFeature getNewFeature() {
    // If new feature is managed by Order
}
```

---

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

---

## Cross-Cutting Concerns

### Java Implementation

Repository implementations and configuration classes go in `shared/`:

```bash
shared/
├── adapter/
│   └── out/
│       └── repository/
│           ├── OrderCrudRepository.ts
│           ├── CustomerCrudRepository.ts
│           ├── OrderInMemoryRepository.ts
│           └── CustomerInMemoryRepository.ts
│
└── io/
    ├── express/
    │   ├── Application.ts
    │   └── config/
    │       ├── RepositoryInjection.ts
    │       ├── UseCaseInjection.ts
    │       ├── DataSourceConfig.ts
    │       └── SecurityConfig.ts
    │
    └── standard/
        └── Main.ts
```

### TypeScript Implementation

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

---

## Benefits of Vertical Slice Architecture

| Benefit | Description |
|---------|-------------|
| **Feature Isolation** | Changes to one feature don't affect others |
| **Team Scalability** | Teams can work on different features independently |
| **Faster Development** | All code for a feature in one place |
| **Better Testing** | Tests are focused on single features |
| **Reduced Cognitive Load** | Don't need to understand entire codebase |
| **Easier Refactoring** | Feature boundaries make refactoring safer |

Additional benefits:

1. **Feature Independence**: Each feature can be developed, tested, and deployed independently
2. **Easy Navigation**: All code for a feature is in one directory
3. **Clear Boundaries**: Feature boundaries are explicit and enforced
4. **Better Scalability**: Adding new features doesn't clutter existing code
5. **Team Collaboration**: Different teams can work on different features with minimal conflicts
6. **Faster Development**: Context switching between related code is minimized
7. **Easier Refactoring**: Impact of changes is contained within feature boundaries
8. **Simpler Onboarding**: New developers can understand one feature at a time

---

## When to Use Vertical Slice Architecture

### Choose Vertical Slice When:

- Building medium to large applications
- Multiple teams working on different features
- Features have distinct business logic
- Clear domain boundaries exist
- Independent deployment is desired
- Building feature-rich applications
- Working in cross-functional teams
- Prioritizing feature delivery over technical layers
- Using frameworks that support feature-based organization (NestJS, Angular)

### Consider Layer-Based When:

- Building small, simple applications
- Single developer or small team
- All features are tightly coupled
- Simpler navigation is preferred
- Rapid prototyping
- Building simple CRUD applications
- Team is small and highly cohesive
- Strict technical layering provides more value

---

## Migration from Layer-Based to Vertical Slice

1. **Identify Features**: Group related entities, use cases, and controllers by business capability
2. **Create Feature Directories**: Create feature folders for each identified group
3. **Move Domain Code**: Move entities and value objects to `feature/domain/`
4. **Move Use Cases**: Move use case interfaces and services to `feature/usecase/`
5. **Move Adapters**: Move controllers and presenters to `feature/adapter/`
6. **Create Shared Folder**: Move repository implementations to `shared/adapter/out/repository/`
7. **Update Imports**: Fix all import statements after restructuring
8. **Update Configuration**: Update dependency injection in config classes
9. **Test Thoroughly**: Ensure all functionality still works

---

## Anti-Patterns to Avoid

### 1. Feature Creep
Putting unrelated code into a feature folder just because it seems convenient.

### 2. Shared Feature Code
Having feature-specific code in the `shared/` folder, defeating the purpose of isolation.

### 3. Direct Cross-Feature Dependencies
Importing domain classes or using services from another feature directly.

### 4. Skipping Layers Within Feature
Bypassing use case layer by having controllers directly call repositories.

### 5. Circular Dependencies
Creating circular dependencies between features or within a feature's layers.

### 6. Adapter-to-Adapter Dependencies
Having adapters depend directly on other adapters instead of going through the use case layer.

### 7. Domain Layer Dependencies
Allowing the domain layer to depend on the use case layer, breaking the dependency flow.

---

## References and Further Reading

1. "Vertical Slice Architecture" by Jimmy Bogard
2. "Feature Folders" by Rachel Appel
3. "Modular monoliths" by Simon Brown
4. "Domain-Driven Design" by Eric Evans. Addison-Wesley, 2004
5. "Implementing Domain-Driven Design" by Vaughn Vernon. Addison-Wesley, 2013
6. "Pragmatic Clean Architecture" series
7. NestJS Modular Architecture
8. "Vertical Slice Architecture" - https://www.jimmybogard.com/vertical-slice-architecture/
