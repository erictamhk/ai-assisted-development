# Extraction Patterns

This document provides patterns and techniques for extracting components from monolithic applications, enabling incremental migration to modular architectures.

---

## 1. Extraction Principles

### 1.1 Golden Rules of Extraction

1. **Never break working code** - Maintain functionality throughout extraction
2. **Extract by responsibility** - Single Responsibility Principle guides extraction
3. **Test at each step** - Ensure tests pass before and after each extraction
4. **Small batches** - Extract one concept at a time
5. **Define boundaries** - Clear interfaces between extracted and remaining code

### 1.2 Extraction Order

```
Recommended Extraction Order (Low to High Risk):

1. Value Objects (e.g., Email, Money, Address)
   → Simple, self-contained, no dependencies

2. Entities with simple behavior (e.g., Task, User)
   → Has identity, but limited dependencies

3. Domain Services (e.g., Calculator, Validator)
   → Stateless, focused logic

4. Aggregates (e.g., Order, ToDoList)
   → Complex invariants, multiple entities

5. Use Cases (e.g., CreateUser, ProcessOrder)
   → Application orchestration

6. Repositories
   → Data access abstraction

7. External Adapters (e.g., PaymentGateway, EmailService)
   → Integration points
```

---

## 2. Extraction Patterns

### 2.1 Extract Class Pattern

**When to use:** A class has multiple responsibilities or contains method groups that can be separated.

**Before:**
```typescript
class ToDoList {
  private tasks: Map<string, Task>;
  private projects: Map<string, Project>;
  
  addTask(task: Task): void { /* ... */ }
  removeTask(taskId: string): void { /* ... */ }
  createProject(name: string): Project { /* ... */ }
  deleteProject(projectId: string): void { /* ... */ }
  generateReport(): Report { /* ... */ }  // Different responsibility
  exportToPDF(): void { /* ... */ }  // Different responsibility
}
```

**After:**
```typescript
// Core domain
class ToDoList {
  private tasks: Map<string, Task>;
  private projects: Map<string, Project>;
  
  addTask(task: Task): void { /* ... */ }
  removeTask(taskId: string): void { /* ... */ }
  createProject(name: string): Project { /* ... */ }
  deleteProject(projectId: string): void { /* ... */ }
}

// Extracted - Reporting concern
class ReportGenerator {
  generateFor(toDoList: ToDoList): Report { /* ... */ }
  exportToPDF(report: Report): void { /* ... */ }
}
```

**Steps:**
1. Identify cohesive method groups
2. Create new class with extracted methods
3. Delegate from original class
4. Update callers
5. Remove delegation (if appropriate)

---

### 2.2 Extract Interface Pattern

**When to use:** A class needs multiple implementations or you want to define a contract.

**Before:**
```typescript
class InMemoryUserRepository {
  findById(id: string): User | null { /* ... */ }
  save(user: User): void { /* ... */ }
  findByEmail(email: string): User | null { /* ... */ }
}
```

**After:**
```typescript
// Domain layer - interface only
interface UserRepository {
  findById(id: UserId): Promise<User | null>;
  save(user: User): Promise<void>;
  findByEmail(email: Email): Promise<User | null>;
}

// Infrastructure layer - implementation
class InMemoryUserRepository implements UserRepository {
  findById(id: UserId): Promise<User | null> { /* ... */ }
  save(user: User): Promise<void> { /* ... */ }
  findByEmail(email: Email): Promise<User | null> { /* ... */ }
}

class PostgresUserRepository implements UserRepository {
  findById(id: UserId): Promise<User | null> { /* ... */ }
  save(user: User): Promise<void> { /* ... */ }
  findByEmail(email: Email): Promise<User | null> { /* ... */ }
}
```

---

### 2.3 Extract Domain Service Pattern

**When to use:** Business logic spans multiple aggregates or doesn't belong to any single entity.

**Before:**
```typescript
class OrderService {
  constructor(private db: Database) {}
  
  transferMoney(fromAccount: string, toAccount: string, amount: number): void {
    // Complex logic spanning multiple entities
    const from = this.db.findAccount(fromAccount);
    const to = this.db.findAccount(toAccount);
    
    if (from.balance < amount) {
      throw new InsufficientFundsError();
    }
    
    from.balance -= amount;
    to.balance += amount;
    
    this.db.save(from);
    this.db.save(to);
  }
}
```

**After:**
```typescript
// Domain layer - pure business logic
class MoneyTransferService {
  constructor(
    private accountRepository: AccountRepository,
    private eventPublisher: EventPublisher
  ) {}

  async transfer(fromId: AccountId, toId: AccountId, amount: Money): Promise<void> {
    const from = await this.accountRepository.findById(fromId);
    const to = await this.accountRepository.findById(toId);
    
    if (from.canWithdraw(amount)) {
      from.withdraw(amount);
      to.deposit(amount);
      
      await this.accountRepository.save(from);
      await this.accountRepository.save(to);
      
      this.eventPublisher.publish(new MoneyTransferredEvent(fromId, toId, amount));
    }
  }
}

// Application layer - orchestration
class TransferMoneyUseCase {
  constructor(private transferService: MoneyTransferService) {}
  
  async execute(input: TransferInput): Promise<TransferOutput> {
    const fromId = AccountId.of(input.fromAccount);
    const toId = AccountId.of(input.toAccount);
    const amount = Money.of(input.amount, input.currency);
    
    await this.transferService.transfer(fromId, toId, amount);
    return TransferOutput.success();
  }
}
```

---

### 2.4 Extract Aggregate Pattern

**When to use:** A large entity contains multiple nested concepts that should be separate aggregates.

**Before:**
```typescript
class Project {
  name: string;
  tasks: Task[];  // Could be separate aggregate
  teamMembers: TeamMember[];  // Could be separate aggregate
  budget: Budget;  // Could be value object
  timeline: Timeline;  // Could be value object
  
  addTask(task: Task): void { /* ... */ }
  assignTeamMember(member: TeamMember): void { /* ... */ }
  calculateBudgetRemaining(): number { /* ... */ }
  isOnSchedule(): boolean { /* ... */ }
}
```

**After:**
```typescript
// Project remains as Aggregate Root
class Project implements AggregateRoot<ProjectId> {
  private readonly id: ProjectId;
  private name: ProjectName;
  private _tasks: TaskCollection;
  private _team: TeamCollection;
  
  addTask(description: TaskDescription): Task {
    return this._tasks.add(Task.create(this.id, description));
  }
  
  assignTeamMember(memberInfo: MemberInfo): TeamMember {
    return this._team.add(TeamMember.create(this.id, memberInfo));
  }
}

// Separate aggregates (accessed through root)
class Task implements Entity<TaskId> {
  private readonly id: TaskId;
  private status: TaskStatus;
  
  complete(): void { /* ... */ }
}

class TeamMember implements Entity<MemberId> {
  private readonly id: MemberId;
  private role: TeamRole;
  
  changeRole(newRole: TeamRole): void { /* ... */ }
}

// Value objects
class Budget implements ValueObject {
  private readonly allocated: Money;
  private readonly spent: Money;
  
  get remaining(): Money { /* ... */ }
  isWithinLimits(): boolean { /* ... */ }
}
```

---

### 2.5 Extract Use Case Pattern

**When to use:** Application logic should be separated from domain logic and entry points.

**Before:**
```typescript
// Monolithic controller
class UserController {
  constructor(private db: Database) {}
  
  @Post('/users')
  async createUser(req: Request): Promise<Response> {
    const userData = req.body;
    
    // Validation logic
    if (!userData.email || !userData.email.includes('@')) {
      throw new ValidationError('Invalid email');
    }
    
    // Business logic
    const user = new User(userData.email, userData.name);
    await this.db.users.save(user);
    
    // Integration
    await this.emailService.sendWelcome(user.email);
    
    return Response.json({ id: user.id });
  }
}
```

**After:**
```typescript
// Domain - Entity
class User implements Entity<UserId> {
  private readonly id: UserId;
  private email: Email;
  private name: UserName;
  
  constructor(email: Email, name: UserName) {
    this.id = UserId.generate();
    this.email = email;
    this.name = name;
  }
}

// Use Case - Application layer
interface CreateUserInput {
  email: string;
  name: string;
}

interface CreateUserOutput {
  userId: string;
}

class CreateUserUseCase {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  async execute(input: CreateUserInput): Promise<CreateUserOutput> {
    // Validation
    const email = new Email(input.email);
    const name = new UserName(input.name);
    
    // Business logic
    const user = new User(email, name);
    await this.userRepository.save(user);
    
    // Integration
    await this.emailService.sendWelcome(email);
    
    return { userId: user.id.value };
  }
}

// Adapter - Controller
class UserController {
  constructor(private createUserUseCase: CreateUserUseCase) {}

  @Post('/users')
  async createUser(req: Request): Promise<Response> {
    const input: CreateUserInput = {
      email: req.body.email,
      name: req.body.name
    };
    
    const result = await this.createUserUseCase.execute(input);
    return Response.json(result);
  }
}
```

---

### 2.6 Extract Repository Pattern

**When to use:** Data access logic should be abstracted from business logic.

**Before:**
```typescript
class UserService {
  constructor(private db: Database) {}
  
  async findUserByEmail(email: string): Promise<User | null> {
    const row = await this.db.query(
      'SELECT * FROM users WHERE email = ?',
      [email]
    );
    return row ? this.mapToUser(row) : null;
  }
  
  async saveUser(user: User): Promise<void> {
    await this.db.query(
      'INSERT INTO users (id, email, name) VALUES (?, ?, ?)',
      [user.id, user.email, user.name]
    );
  }
  
  private mapToUser(row: any): User {
    // Mapping logic
  }
}
```

**After:**
```typescript
// Domain - Repository interface
interface UserRepository {
  findById(id: UserId): Promise<User | null>;
  findByEmail(email: Email): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: UserId): Promise<void>;
}

// Infrastructure - Repository implementation
class PostgresUserRepository implements UserRepository {
  constructor(
    private db: Database,
    private mapper: UserMapper
  ) {}

  async findByEmail(email: Email): Promise<User | null> {
    const row = await this.db.query(
      'SELECT * FROM users WHERE email = ?',
      [email.value]
    );
    return row ? this.mapper.toDomain(row) : null;
  }

  async save(user: User): Promise<void> {
    const data = this.mapper.toPersistence(user);
    await this.db.query(
      `INSERT INTO users (id, email, name) VALUES (?, ?, ?)
       ON CONFLICT (id) DO UPDATE SET email = ?, name = ?`,
      [data.id, data.email, data.name, data.email, data.name]
    );
  }
}
```

---

### 2.7 Strangler Facade Pattern

**When to use:** Gradually replace monolith functionality with microservices.

```typescript
// Strangler Facade
class ApiGateway {
  private router: Map<string, ServiceLocator>;
  
  async handleRequest(request: Request): Promise<Response> {
    const route = this.findRoute(request.path);
    
    if (this.shouldRouteToMicroservice(route)) {
      return this.routeToMicroservice(request, route);
    } else {
      return this.routeToMonolith(request, route);
    }
  }
  
  private shouldRouteToMicroservice(route: string): boolean {
    // Feature flag or routing rules
    return this.migratedRoutes.has(route);
  }
}

// Usage - gradually migrate routes
const gateway = new ApiGateway();
gateway.registerRoute('/api/users', { 
  microservice: 'user-service',
  migrated: true  // Route to new service
});
gateway.registerRoute('/api/products', { 
  microservice: null,  // Still in monolith
  migrated: false
});
```

---

### 2.8 Bridge Pattern for Repositories

**When to use:** Separate abstraction (domain interface) from implementation (infrastructure).

```typescript
// Abstraction - Domain interface
interface ToDoListRepository {
  findById(id: ToDoListId): Promise<ToDoList | null>;
  save(toDoList: ToDoList): Promise<void>;
}

// Implementor - Infrastructure
interface ToDoListRepositoryPeer {
  findByIdRaw(id: string): Promise<any>;
  saveRaw(data: any): Promise<void>;
}

// Concrete implementations
class InMemoryPeer implements ToDoListRepositoryPeer {
  private store: Map<string, any> = new Map();
  
  async findByIdRaw(id: string): Promise<any> {
    return this.store.get(id) || null;
  }
  
  async saveRaw(data: any): Promise<void> {
    this.store.set(data.id, data);
  }
}

class JpaPeer implements ToDoListRepositoryPeer {
  constructor(private entityManager: EntityManager) {}
  
  async findByIdRaw(id: string): Promise<any> {
    return this.entityManager.find(ToDoListEntity, id);
  }
  
  async saveRaw(data: any): Promise<void> {
    this.entityManager.persist(data);
  }
}

// Bridge - Connects abstraction to implementation
class ToDoListRepositoryBridge implements ToDoListRepository {
  constructor(private peer: ToDoListRepositoryPeer) {}
  
  async findById(id: ToDoListId): Promise<ToDoList | null> {
    const data = await this.peer.findByIdRaw(id.value);
    return data ? ToDoListMapper.toDomain(data) : null;
  }
  
  async save(toDoList: ToDoList): Promise<void> {
    const data = ToDoListMapper.toPersistence(toDoList);
    await this.peer.saveRaw(data);
  }
}
```

---

## 3. Extraction Checklist

### 3.1 Pre-Extraction

| Check | Status | Notes |
|-------|--------|-------|
| Code is tested | ☐ | |
| Dependencies identified | ☐ | |
| Interface defined | ☐ | |
| Target package structure created | ☐ | |
| Integration points mapped | ☐ | |

### 3.2 During Extraction

| Check | Status | Notes |
|-------|--------|-------|
| Single Responsibility maintained | ☐ | |
| Dependencies point inward | ☐ | |
| No framework dependencies in domain | ☐ | |
| Public interface documented | ☐ | |
| Tests still passing | ☐ | |

### 3.3 Post-Extraction

| Check | Status | Notes |
|-------|--------|-------|
| Original code updated to use new abstraction | ☐ | |
| Old code removed or deprecated | ☐ | |
| Documentation updated | ☐ | |
| Performance impact assessed | ☐ | |
| Rollback plan tested | ☐ | |

---

## 4. Common Extraction Issues

### 4.1 Circular Dependencies

**Problem:** A depends on B, B depends on A

**Solution:**
1. Identify the shared concept
2. Extract to third module (shared kernel)
3. Use dependency injection
4. Consider event-driven communication

### 4.2 Database Coupling

**Problem:** Tight coupling between domain and database schema

**Solution:**
1. Introduce repository abstraction
2. Map domain entities to persistence models
3. Use data mapper pattern
4. Delay database changes

### 4.3 Feature Envy

**Problem:** Method in class A uses more data from class B

**Solution:**
1. Move method to class B
2. Or move data to class A
3. Use parameter objects for related data

---

## 5. AI-Assisted Extraction

### 5.1 Extraction Planning Prompt

```markdown
# Plan extraction of [Component Name]

Given the code in [path/to/component], analyze and provide:

1. **Current Dependencies**
   - What does this component depend on?
   - What depends on this component?
   - What are the circular dependencies?

2. **Extraction Boundary**
   - What should be included in the extracted component?
   - What should stay in the monolith?

3. **Interface Design**
   - What methods/operations should be exposed?
   - What data types should be used?

4. **Migration Steps**
   - Step-by-step extraction plan
   - Estimated effort for each step
   - Risks and mitigations

5. **Test Strategy**
   - What tests are needed before extraction?
   - What tests are needed after extraction?
```

### 5.2 AI Extraction Best Practices

When working with AI to extract components:

1. **Provide context**: Share related code and patterns
2. **Specify boundaries**: Define what to extract and what to keep
3. **Request verification**: Ask AI to verify against patterns
4. **Iterate**: Small extractions, review, then continue
5. **Test at each step**: Run tests after each extraction

---

## 6. References

1. ref/refactor-to-ca-analysis.md - Real-world 14-step refactoring
2. doc/clean-architecture-refactoring-journey.md - Step-by-step guide
3. Martin Fowler. "Refactoring: Improving the Design of Existing Code"
4. Eric Evans. "Domain-Driven Design: Tackling Complexity in the Heart of Software"
5. "Migration From Monolith To Microservices Using Strangler Pattern" - Brainhub

---

## Appendix: 14-Step Clean Architecture Refactoring Journey

This appendix documents a proven refactoring journey from monolithic application to Clean Architecture with DDD. See refactor-to-ca-analysis.md for full details.

### Step 1: Foundation Setup

**Focus:** Test structure and error handling setup

**Changes:**
- Configure JDK, testing framework
- Establish test structure for commands
- Define error handling messages
- Set up dependencies

**Pattern:** Greenfield test setup

---

### Step 2: Establish Clean Architecture Layers

**Focus:** Layer creation and DDD foundation

**Changes:**
1. Create four-layer package structure:
   - `entity/` - Domain model
   - `usecase/` - Business logic
   - `adapter/` - Controllers, Presenters, Repositories
   - `io/` - Entry points

2. Apply DDD Tactical Design:
   - ToDoList → Aggregate Root
   - Project, Task → Entities
   - ProjectName, TaskId, ToDoListId → Value Objects

3. Remove Primitive Obsession:
   - Replace `Map<String, List<Task>>` with `Tasks` class
   - Replace `Map<ProjectName, List<Task>>` with `Project` class
   - Create `ProjectName` record

**Patterns Applied:**
- Layered Architecture
- Value Object
- Entity
- Aggregate Root

**Key Insight:** Aggregate boundaries and invariant enforcement are established early

---

### Step 3: Extract Classes from Monolith

**Focus:** Find more classes in entity package by extracting command/query logic

**Changes (using "Extract Class from Private Method" pattern):**
1. Extract `ToDoList::show` → Show class
2. Extract `ToDoList::add` → Add class
3. Extract `ToDoList::setDone` → SetDone class
4. Extract `ToDoList::help` → Help class
5. Extract `ToDoList::error` → Error class
6. Extract `ToDoList::execute` → Execute class

7. Clean entity package:
   - Move classes with external/IO references to usecase package

**Patterns Applied:**
- Extract Class
- Single Responsibility Principle
- Method Object

**Key Insight:** Each extracted class represents a potential Use Case

---

### Step 4: Form Commands and Queries

**Focus:** Implement Use Case pattern with Input/Output

**Command Use Cases:**
1. AddProjectUseCase:
   - AddProjectInput
   - AddProjectService
   - ToDoListRepository

2. AddTaskUseCase:
   - AddTaskInput
   - AddTaskService

3. SetDoneUseCase:
   - SetDoneInput
   - SetDoneService

**Query Use Cases:**
1. ShowUseCase:
   - ShowInput, ShowOutput
   - ShowService, ShowPresenter
   - DTOs and Mappers

2. HelpUseCase:
   - HelpInput, HelpOutput
   - HelpService, HelpPresenter

**Patterns Applied:**
- Command Query Separation
- Input/Output Port
- Use Case Pattern

---

### Step 5: Dependency Inversion

**Focus:** Make domain independent of infrastructure

**Changes:**
1. Define Repository interface in domain (port)
2. Create Repository implementation in infrastructure (adapter)
3. Inject repository via constructor
4. Use Dependency Injection container

**Patterns Applied:**
- Dependency Inversion Principle
- Dependency Injection
- Ports and Adapters

---

### Step 6: Extract Adapters

**Focus:** Move I/O handling to adapter layer

**Changes:**
1. Extract Console Controllers to `adapter/in/controller/console/`
2. Extract Web Controllers to `adapter/in/controller/web/`
3. Extract Presenters to `adapter/out/presenter/`
4. Extract Repositories to `adapter/out/repository/`

**Patterns Applied:**
- Adapter Pattern
- Hexagonal Architecture
- Interface Segregation

---

### Step 7: Entry Point Separation

**Focus:** Separate entry point technologies

**Changes:**
1. Create `io/standard/` for console entry
2. Create `io/springboot/` for Spring Boot entry
3. Create configuration classes for DI

**Patterns Applied:**
- Entry Point Pattern
- Strategy Pattern (for different entry types)

---

### Step 8: Polish Domain Model

**Focus:** Strengthen domain invariants and validation

**Changes:**
1. Add validation in constructors
2. Implement proper equals/hashCode
3. Add domain events where appropriate
4. Strengthen aggregate boundaries

**Patterns Applied:**
- Value Object Validation
- Domain Events
- Aggregate Pattern

---

### Step 9: Polish Use Cases

**Focus:** Ensure single responsibility and testability

**Changes:**
1. Verify each use case has one responsibility
2. Add comprehensive input validation
3. Ensure idempotency where needed
4. Add proper error handling

**Patterns Applied:**
- Single Responsibility Principle
- Fail Fast Pattern

---

### Step 10: Add Read-Only Interfaces

**Focus:** Expose domain data safely

**Changes:**
1. Create `ReadOnlyTask` interface
2. Create `ReadOnlyProject` interface
3. Use interfaces in query outputs
4. Prevent external mutation

**Patterns Applied:**
- Interface Segregation
- Immutability

---

### Step 11: Consolidate Repositories

**Focus:** Unify repository implementations

**Changes:**
1. Create repository peer pattern
2. Implement both CRUD and InMemory from same interface
3. Add configuration for repository selection

**Patterns Applied:**
- Strategy Pattern
- Repository Pattern

---

### Step 12: Spring Boot Integration

**Focus:** Complete Spring Boot configuration

**Changes:**
1. Configure use case injection
2. Configure repository injection
3. Set up data source configuration
4. Add Spring Boot application class

**Patterns Applied:**
- Spring Dependency Injection
- Configuration Pattern

---

### Step 13: Performance Optimization

**Focus:** Improve performance of refactored code

**Changes:**
1. Optimize database queries
2. Add caching where appropriate
3. Reduce unnecessary object creation
4. Profile and optimize bottlenecks

**Patterns Applied:**
- Caching Pattern
- Object Pooling (where appropriate)

---

### Step 14: Final Polish

**Focus:** Final review and documentation

**Changes:**
1. Review all code for consistency
2. Update documentation
3. Add comprehensive tests
4. Verify all functionality works

**Final Architecture:**
```
tw.teddysoft.tasks/
├── entity/                    # Domain Layer
├── usecase/                   # Application Layer
├── adapter/
│   ├── in/
│   │   ├── controller/
│   └── out/
│       ├── presenter/
│       └── repository/
└── io/
    ├── standard/
    └── springboot/
```

---

## Refactoring Journey Summary

### Patterns Applied in Order

| Step | Primary Patterns |
|------|-----------------|
| 1 | Test Setup |
| 2 | Layered Architecture, DDD Tactical |
| 3 | Extract Class, SRP |
| 4 | Command/Query, Use Case |
| 5 | Dependency Inversion |
| 6 | Adapter Pattern |
| 7 | Entry Point |
| 8 | Domain Model, Value Objects |
| 9 | SRP, Fail Fast |
| 10 | Interface Segregation |
| 11 | Strategy Pattern |
| 12 | Spring DI |
| 13 | Performance |
| 14 | Documentation |

### Key Principles Demonstrated

1. **Incremental Change**: Each step maintains working code
2. **Test First**: Tests guide the refactoring
3. **Single Responsibility**: Each component has one purpose
4. **Dependency Inversion**: Domain depends on abstractions
5. **Clear Boundaries**: Layers are clearly separated
6. **Ubiquitous Language**: Code expresses domain concepts
