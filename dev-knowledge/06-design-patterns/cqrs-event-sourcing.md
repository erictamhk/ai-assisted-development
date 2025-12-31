# CQRS - Command Query Responsibility Segregation

## Overview

CQRS (Command Query Responsibility Segregation) is an architectural pattern that separates read and write operations into different models. Commands modify data (write operations), while queries read data (read operations). This separation enables independent scaling, optimization, and evolution of read and write paths.

## Core Concept

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CQRS ARCHITECTURE                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   COMMANDS (Write)                  QUERIES (Read)                     │
│   ┌───────────────────┐             ┌───────────────────┐               │
│   │                   │             │                   │               │
│   │  CreateUser       │             │  GetUser          │               │
│   │  UpdateUser       │             │  GetUserList      │               │
│   │  DeleteUser       │             │  GetUserStats     │               │
│   │  ActivateUser     │             │  SearchUsers      │               │
│   │  DeactivateUser   │             │  GetUserActivity  │               │
│   │                   │             │                   │               │
│   └─────────┬─────────┘             └─────────┬─────────┘               │
│             │                                   │                        │
│             ▼                                   ▼                        │
│   ┌───────────────────┐             ┌───────────────────┐               │
│   │   Command         │             │   Query           │               │
│   │   Handlers        │             │   Handlers        │               │
│   │                   │             │                   │               │
│   └─────────┬─────────┘             └─────────┬─────────┘               │
│             │                                   │                        │
│             ▼                                   ▼                        │
│   ┌───────────────────┐             ┌───────────────────┐               │
│   │   Domain          │    Events   │   Read Database   │               │
│   │   Model          │────────────►│   (Projections)  │               │
│   │                   │             │                   │               │
│   └───────────────────┘             └───────────────────┘               │
│                                                                      │
│   WRITE MODEL                    READ MODEL                            │
│   - Optimized for writes       - Optimized for reads                 │
│   - Complex business logic     - Simple, denormalized                 │
│   - Event-sourced optional   - Can be simple SQL or NoSQL         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Command Side

```typescript
// Commands are intent-based operations
interface Command {
  readonly type: string;
  readonly payload: Record<string, unknown>;
  readonly metadata: CommandMetadata;
}

interface CommandMetadata {
  readonly commandId: string;
  readonly userId: string;
  readonly correlationId: string;
  readonly timestamp: DateTime;
}

// Example commands
class CreateUser implements Command {
  readonly type = 'CreateUser';
  
  constructor(
    readonly payload: {
      email: string;
      name: string;
      roles: string[];
    },
    readonly metadata: CommandMetadata
  ) {}
}

class UpdateUser implements Command {
  readonly type = 'UpdateUser';
  
  constructor(
    readonly payload: {
      userId: string;
      email?: string;
      name?: string;
      roles?: string[];
    },
    readonly metadata: CommandMetadata
  ) {}
}

class DeactivateUser implements Command {
  readonly type = 'DeactivateUser';
  
  constructor(
    readonly payload: {
      userId: string;
      reason: string;
    },
    readonly metadata: CommandMetadata
  ) {}
}

// Command handler
class UserCommandHandler {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly eventPublisher: EventPublisher,
    private readonly validator: UserValidator
  ) {}

  async handle(command: Command): Promise<Result<void, CommandError>> {
    switch (command.type) {
      case 'CreateUser':
        return this.createUser(command as CreateUser);
      case 'UpdateUser':
        return this.updateUser(command as UpdateUser);
      case 'DeactivateUser':
        return this.deactivateUser(command as DeactivateUser);
      default:
        return Result.failure(new UnknownCommandError(command.type));
    }
  }

  private async createUser(command: CreateUser): Promise<Result<void, CommandError>> {
    // Validate
    const validationResult = this.validator.validateCreate(command.payload);
    if (!validationResult.isValid) {
      return Result.failure(new ValidationError(validationResult.errors));
    }

    // Create aggregate
    const user = User.create({
      email: command.payload.email,
      name: command.payload.name,
      roles: command.payload.roles
    });

    // Save
    await this.userRepository.save(user);

    // Publish domain events
    for (const event of user.getUncommittedEvents()) {
      await this.eventPublisher.publish(event);
    }

    return Result.success();
  }

  private async updateUser(command: UpdateUser): Promise<Result<void, CommandError>> {
    const user = await this.userRepository.findById(command.payload.userId);
    if (!user) {
      return Result.failure(new NotFoundError('User not found'));
    }

    // Update aggregate
    if (command.payload.email) {
      user.changeEmail(command.payload.email);
    }
    if (command.payload.name) {
      user.changeName(command.payload.name);
    }
    if (command.payload.roles) {
      user.updateRoles(command.payload.roles);
    }

    await this.userRepository.save(user);

    for (const event of user.getUncommittedEvents()) {
      await this.eventPublisher.publish(event);
    }

    return Result.success();
  }
}
```

## Query Side

```typescript
// Queries are read-only operations
interface Query<T> {
  readonly type: string;
  readonly payload: Record<string, unknown>;
}

// Example queries
class GetUser implements Query<UserDTO> {
  readonly type = 'GetUser';
  
  constructor(
    readonly payload: { userId: string }
  ) {}
}

class GetUserList implements Query<UserListDTO[]> {
  readonly type = 'GetUserList';
  
  constructor(
    readonly payload: {
      page?: number;
      limit?: number;
      status?: string;
      search?: string;
    }
  ) {}
}

class SearchUsers implements Query<UserSearchResult[]> {
  readonly type = 'SearchUsers';
  
  constructor(
    readonly payload: {
      query: string;
      filters?: Record<string, unknown>;
    }
  ) {}
}

// Query handler - reads from optimized read database
class UserQueryHandler {
  constructor(
    private readonly readRepository: UserReadRepository,
    private readonly cache: Cache
  ) {}

  async handle<T>(query: Query<T>): Promise<T> {
    switch (query.type) {
      case 'GetUser':
        return this.getUser(query as GetUser) as T;
      case 'GetUserList':
        return this.getUserList(query as GetUserList) as T;
      case 'SearchUsers':
        return this.searchUsers(query as SearchUsers) as T;
      default:
        throw new UnknownQueryError(query.type);
    }
  }

  async getUser(query: GetUser): Promise<UserDTO> {
    const cacheKey = `user:${query.payload.userId}`;
    
    // Try cache first
    const cached = await this.cache.get<UserDTO>(cacheKey);
    if (cached) {
      return cached;
    }

    // Read from denormalized database
    const user = await this.readRepository.findById(query.payload.userId);
    
    if (!user) {
      throw new NotFoundError('User not found');
    }

    // Cache result
    await this.cache.set(cacheKey, user, '1 hour');

    return user;
  }

  async getUserList(query: GetUserList): Promise<UserListDTO[]> {
    const { page = 1, limit = 20, status, search } = query.payload;
    
    const users = await this.readRepository.find({
      status: status as UserStatus,
      search,
      pagination: { page, limit }
    });

    return users.map(user => ({
      id: user.id,
      name: user.name,
      email: user.email,
      status: user.status,
      lastActive: user.lastActive
    }));
  }

  async searchUsers(query: SearchUsers): Promise<UserSearchResult[]> {
    return this.readRepository.search({
      query: query.payload.query,
      filters: query.payload.filters
    });
  }
}
```

## Read Models (Projections)

```typescript
// Read model - denormalized for fast reads
interface UserReadModel {
  id: string;
  email: string;
  name: string;
  status: UserStatus;
  roles: string[];
  createdAt: DateTime;
  lastActive: DateTime;
  
  // Denormalized for display
  displayName: string;
  initials: string;
  roleSummary: string;
}

// Projection from domain events to read model
class UserReadModelProjection {
  handles = [
    'UserCreated',
    'UserEmailChanged',
    'UserNameChanged',
    'UserStatusChanged',
    'UserRolesUpdated',
    'UserActivityLogged'
  ];

  constructor(private readonly readRepository: UserReadRepository) {}

  async apply(event: DomainEvent): Promise<void> {
    switch (event.eventType) {
      case 'UserCreated':
        await this.onUserCreated(event as UserCreated);
        break;
      case 'UserEmailChanged':
        await this.onUserEmailChanged(event as UserEmailChanged);
        break;
      case 'UserNameChanged':
        await this.onUserNameChanged(event as UserNameChanged);
        break;
      case 'UserStatusChanged':
        await this.onUserStatusChanged(event as UserStatusChanged);
        break;
      case 'UserRolesUpdated':
        await this.onUserRolesUpdated(event as UserRolesUpdated);
        break;
      case 'UserActivityLogged':
        await this.onUserActivityLogged(event as UserActivityLogged);
        break;
    }
  }

  private async onUserCreated(event: UserCreated): Promise<void> {
    await this.readRepository.create({
      id: event.aggregateId,
      email: event.payload.email,
      name: event.payload.name,
      status: UserStatus.ACTIVE,
      roles: event.payload.roles,
      createdAt: event.occurredAt,
      lastActive: event.occurredAt,
      displayName: event.payload.name,
      initials: this.getInitials(event.payload.name),
      roleSummary: event.payload.roles.join(', ')
    });
  }

  private async onUserEmailChanged(event: UserEmailChanged): Promise<void> {
    await this.readRepository.update(event.aggregateId, {
      email: event.payload.email
    });
  }

  private async onUserNameChanged(event: UserNameChanged): Promise<void> {
    await this.readRepository.update(event.aggregateId, {
      name: event.payload.name,
      displayName: event.payload.name,
      initials: this.getInitials(event.payload.name)
    });
  }

  private async onUserStatusChanged(event: UserStatusChanged): Promise<void> {
    await this.readRepository.update(event.aggregateId, {
      status: event.payload.status
    });
  }

  private async onUserRolesUpdated(event: UserRolesUpdated): Promise<void> {
    await this.readRepository.update(event.aggregateId, {
      roles: event.payload.roles,
      roleSummary: event.payload.roles.join(', ')
    });
  }

  private async onUserActivityLogged(event: UserActivityLogged): Promise<void> {
    await this.readRepository.update(event.aggregateId, {
      lastActive: event.occurredAt
    });
  }

  private getInitials(name: string): string {
    return name
      .split(' ')
      .map(part => part[0])
      .join('')
      .toUpperCase()
      .slice(0, 2);
  }
}
```

## API Layer

```typescript
// Express/Controller with CQRS
class UserController {
  constructor(
    private readonly commandBus: CommandBus,
    private readonly queryBus: QueryBus
  ) {}

  // POST /users - Create user (command)
  async createUser(req: Request, res: Response): Promise<void> {
    const command = new CreateUser(
      {
        email: req.body.email,
        name: req.body.name,
        roles: req.body.roles || ['user']
      },
      {
        commandId: CommandId.generate().value,
        userId: req.user.id,
        correlationId: req.correlationId,
        timestamp: DateTime.now()
      }
    );

    const result = await this.commandBus.send(command);

    if (result.success) {
      res.status(201).json({
        success: true,
        data: { userId: result.value }
      });
    } else {
      res.status(400).json({
        success: false,
        error: result.error.message
      });
    }
  }

  // GET /users/:id - Get user (query)
  async getUser(req: Request, res: Response): Promise<void> {
    const query = new GetUser(
      { userId: req.params.id }
    );

    try {
      const user = await this.queryBus.execute(query);
      res.json({ success: true, data: user });
    } catch (error) {
      if (error instanceof NotFoundError) {
        res.status(404).json({ success: false, error: 'User not found' });
      } else {
        res.status(500).json({ success: false, error: 'Internal server error' });
      }
    }
  }

  // GET /users - List users (query)
  async listUsers(req: Request, res: Response): Promise<void> {
    const query = new GetUserList({
      page: parseInt(req.query.page) || 1,
      limit: parseInt(req.query.limit) || 20,
      status: req.query.status as string,
      search: req.query.search as string
    });

    const users = await this.queryBus.execute(query);
    res.json({ success: true, data: users });
  }

  // GET /users/search - Search users (query)
  async searchUsers(req: Request, res: Response): Promise<void> {
    const query = new SearchUsers({
      query: req.query.q as string,
      filters: {
        status: req.query.status,
        roles: req.query.roles
      }
    });

    const results = await this.queryBus.execute(query);
    res.json({ success: true, data: results });
  }
}
```

## Benefits

| Aspect | Benefit |
|---------|----------|
| **Scalability** | Read/write can scale independently |
| **Optimization** | Each model optimized for its purpose |
| **Flexibility** | Different databases for reads/writes |
| **Evolution** | Read and write models evolve separately |
| **Simplicity** | Queries are simple, commands are focused |

## When to Use CQRS

| Use CQRS | Don't Use CQRS |
|----------|----------------|
| Complex domains with rich behavior | Simple CRUD applications |
| High read scalability needs | Simple reporting needs |
| Event-sourced systems | Traditional transactional systems |
| Distributed teams | Small, co-located teams |
| Audit requirements | Simple access patterns |

## Anti-Patterns

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| **Over-CQRS** | Applying CQRS everywhere | Use where it adds value |
| **Eventual Consistency Issues** | Read model lags behind | Communicate expectations |
| **Dual Writes** | Writing to both models | Use event sourcing or transactions |
| **Complex Projections** | Overly complex projections | Keep projections focused |

## References

- "CQRS" by Martin Fowler
- "CQRS Journey" by Microsoft
- "Domain-Driven Design" by Eric Evans
- "Implementing Domain-Driven Design" by Vaughn Vernon
