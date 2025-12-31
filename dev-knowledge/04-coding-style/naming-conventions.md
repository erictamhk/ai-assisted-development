# Naming Conventions and Best Practices

## Overview

Consistent naming conventions are critical for code readability and maintainability. This document defines naming rules for all code components, with special focus on DDD patterns and AI-assisted development.

---

## Entities

### Rules

- **Singular nouns**: `Project`, `Task`, `ToDoList`
- **Business domain terms**: Use ubiquitous language from domain model
- **Noun phrase naming**: Clear business meaning
- **No abbreviations**: Except common ones (ID, URL, API, HTTP, JSON, JWT, SQL)

### Examples

```typescript
// GOOD - Clear, descriptive names
class User { }
class Product { }
class Order { }
class ShoppingCart { }
class PaymentMethod { }

// BAD - Abbreviations, unclear names
class Usr { }
class Prod { }
class Ord { }
class ShopCart { }
class PayMeth { }
```

---

## Value Objects

### Rules

- **Represent identity**: `ProjectName`, `TaskId`, `ToDoListId`
- **Use `record` keyword**: TypeScript records for immutability
- **Immutable fields**: All fields are `readonly`
- **Noun or noun phrase**: Describe what they represent

### Examples

```typescript
// GOOD - Identity value objects
type ProjectName = {
  readonly value: string;
};

type TaskId = {
  readonly value: string;
};

type UserId = {
  readonly value: string;
};

// GOOD - Value values
type Email = {
  readonly value: string;
};

type Money = {
  readonly amount: number;
  readonly currency: string;
};

type Percentage = {
  readonly value: number;
};

// GOOD - Complex value objects
type Address = {
  readonly street: string;
  readonly city: string;
  readonly state: string;
  readonly zipCode: string;
  readonly country: string;
};
```

---

## Read-Only Interfaces

### Rules

- **Read-only views**: `ReadOnlyProject`, `ReadOnlyTask`, `ReadOnlyUser`
- **Immutable**: All fields are `readonly`
- **Prefix**: Always start with `ReadOnly`
- **Purpose**: Prevent unauthorized modifications

### Examples

```typescript
// GOOD - Read-only interfaces
type ReadOnlyUser = {
  readonly id: UserId;
  readonly email: Email;
  readonly name: string;
  readonly createdAt: Date;
};

type ReadOnlyProduct = {
  readonly id: ProductId;
  readonly name: string;
  readonly description: string;
  readonly price: Money;
  readonly createdAt: Date;
};
```

---

## Use Cases (Input Ports)

### Rules

- **Suffix `UseCase`**: `AddTaskUseCase`, `ShowUseCase`, `SetDoneUseCase`
- **Verb + noun naming**: Describe intent, not implementation
- **Clear action**: What business operation is performed?

### Examples

```typescript
// GOOD - Clear use case interfaces
interface AddTaskUseCase extends Command<AddTaskInput, AddTaskOutput> { }
interface ShowUseCase extends Query<ShowInput, ShowOutput> { }
interface SetDoneUseCase extends Command<SetDoneInput, SetDoneOutput> { }
interface DeleteTaskUseCase extends Command<DeleteTaskInput, DeleteTaskOutput> { }

// GOOD - More complex use cases
interface RegisterUserUseCase extends Command<RegisterUserInput, RegisterUserOutput> { }
interface AuthenticateUserUseCase extends Command<AuthenticateUserInput, AuthenticateUserOutput> { }
interface UpdateUserProfileUseCase extends Command<UpdateProfileInput, UpdateProfileOutput> { }
interface ChangePasswordUseCase extends Command<ChangePasswordInput, ChangePasswordOutput> { }

// BAD - Describes implementation, not intent
interface UserCreator { } // Should be CreateUserUseCase
interface TaskAdder { } // Should be AddTaskUseCase
interface ProductUpdater { } // Should be UpdateProductUseCase
```

---

## Services (Use Case Implementations)

### Rules

- **Suffix `Service`**: `AddTaskService`, `ShowService`, `SetDoneService`
- **Implement interface**: Implements corresponding `UseCase` interface
- **Matches use case name**: `AddTaskService` implements `AddTaskUseCase`

### Examples

```typescript
// GOOD - Service implementations
class AddTaskService implements AddTaskUseCase {
  constructor(
    private taskRepository: ITaskRepository,
    private validator: TaskValidator
  ) {}

  async execute(input: AddTaskInput): Promise<AddTaskOutput> {
    // Implementation
  }
}

class ShowService implements ShowUseCase {
  constructor(
    private taskRepository: ITaskRepository,
    private presenter: ShowPresenter
  ) {}

  async execute(input: ShowInput): Promise<ShowOutput> {
    // Implementation
  }
}
```

---

## Controllers

### Rules

- **Suffix `Controller`**: `AddConsoleController`, `ShowController`, `WebController`
- **Prefix indicates medium**: `ConsoleController` vs `WebController`
- **Verb + noun**: Describe action
- **Matches use case**: `AddTaskController` uses `AddTaskService`

### Examples

```typescript
// GOOD - Console controllers
class AddConsoleController implements IConsoleController {
  constructor(private service: AddTaskService) { }

  async execute(command: string): Promise<void> {
    // Parse command and call service
  }
}

class ShowConsoleController implements IConsoleController {
  constructor(private service: ShowService) { }

  async execute(command: string): Promise<void> {
    // Parse command and call service
  }
}

// GOOD - Web controllers
class AddWebController {
  constructor(private service: AddTaskUseCase) { }

  @Post('/tasks')
  async addTask(@Body() input: AddTaskInput): Promise<AddTaskOutput> {
    return this.service.execute(input);
  }
}

class ShowWebController {
  constructor(private service: ShowUseCase) { }

  @Get('/tasks')
  async showTasks(@Query() input: ShowInput): Promise<ShowOutput> {
    return this.service.execute(input);
  }
}
```

---

## DTOs (Data Transfer Objects)

### Rules

- **Suffix `Dto`**: `TaskDto`, `ProjectDto`
- **Plain objects**: No behavior, only data
- **Public fields**: Simple POJOs

### Examples

```typescript
// GOOD - DTOs
class TaskDto {
  public id: string;
  public title: string;
  public description: string;
  public status: string;
  public createdAt: Date;
}

class ProjectDto {
  public id: string;
  public name: string;
  public description: string;
  public taskCount: number;
  public createdAt: Date;
}

// GOOD - Response DTOs
class CreateTaskResponseDto {
  public taskId: string;
  public message: string;
}
```

---

## PO (Persistent Objects)

### Rules

- **Suffix `Po`**: `TaskPo`, `ProjectPo`
- **Framework annotations**: For persistence mapping
- **Database representation**: Separate from domain entities

### Examples

```typescript
// GOOD - Persistent Objects
@Entity('tasks')
class TaskPo {
  @PrimaryGeneratedColumn()
  public id: number;

  @Column()
  public title: string;

  @Column('text')
  public description: string;

  @Column()
  public status: string;

  @CreateDateColumn()
  public createdAt: Date;
}

// GOOD - Plain POJO for simpler persistence
class TaskPo {
  public id: string;
  public title: string;
  public description: string;
  public status: string;
  public createdAt: Date;

  constructor(id: string, title: string) {
    this.id = id;
    this.title = title;
  }
}
```

---

## Mappers

### Rules

- **Suffix `Mapper`**: `TaskMapper`, `ProjectMapper`
- **Static methods**: `toDto()`, `toDomain()`, `toPo()`
- **Support single and collection conversions**: `toDtoList()`

### Examples

```typescript
// GOOD - Mapper implementations
class TaskMapper {
  static toDto(task: Task): TaskDto {
    return {
      id: task.id.value,
      title: task.title.value,
      description: task.description.value,
      status: task.status.value,
      createdAt: task.createdAt
    };
  }

  static toDomain(po: TaskPo): Task {
    return Task.reconstitute({
      id: TaskId.of(po.id),
      title: TaskTitle.of(po.title),
      description: TaskDescription.of(po.description),
      status: TaskStatus.of(po.status),
      createdAt: po.createdAt
    });
  }

  static toPo(task: Task): TaskPo {
    return {
      id: task.id.value,
      title: task.title.value,
      description: task.description.value,
      status: task.status.value,
      createdAt: task.createdAt
    };
  }

  static toDtoList(tasks: Task[]): TaskDto[] {
    return tasks.map(task => this.toDto(task));
  }
}
```

---

## Input/Output Objects

### Rules

- **Suffix `Input`**: `AddTaskInput`, `ShowInput`
- **Suffix `Output`**: `ShowOutput`, `TodayOutput`
- **Public fields**: For input objects
- **Implements `Input` interface**: For use case inputs

### Examples

```typescript
// GOOD - Input objects
interface AddTaskInput extends Input {
  public toDoListId: string;
  public projectName: string;
  public description: string;
}

interface ShowInput extends Input {
  public format: 'brief' | 'detailed';
  public sortBy: 'date' | 'name';
}

// GOOD - Output objects
interface ShowOutput extends Output {
  public tasks: ReadonlyArray<TaskDto>;
  public totalCount: number;
  public formattedOutput: string;
}

interface TodayOutput extends Output {
  public tasks: ReadonlyArray<TaskDto>;
  public message: string;
}
```

---

## Presenters

### Rules

- **Suffix `Presenter`**: `ShowPresenter`
- **Interface defined in output ports**: Presenter interface
- **Implementation in adapters**: `ShowConsolePresenter`

### Examples

```typescript
// GOOD - Presenter interface (in output ports)
interface ShowPresenter {
  present(output: ShowOutput): string;
}

// GOOD - Presenter implementation (in adapters)
class ShowConsolePresenter implements ShowPresenter {
  present(output: ShowOutput): string {
    if (output.tasks.length === 0) {
      return 'No tasks found.';
    }

    const lines = output.tasks.map(task =>
      `[${task.status}] ${task.title} (${task.description})`
    );

    return lines.join('\n');
  }
}

class ShowJsonPresenter implements ShowPresenter {
  present(output: ShowOutput): string {
    return JSON.stringify({
      tasks: output.tasks,
      totalCount: output.totalCount
    }, null, 2);
  }
}
```

---

## Domain Events

### Rules

- **Past tense naming**: `OrderCreated`, `TaskCompleted`
- **Suffix `Event`**: For event class names
- **Descriptive**: Capture what happened

### Examples

```typescript
// GOOD - Domain events
class TaskCreatedEvent {
  constructor(
    public readonly taskId: TaskId,
    public readonly projectId: ProjectId,
    public readonly createdAt: Date
  ) {}
}

class TaskCompletedEvent {
  constructor(
    public readonly taskId: TaskId,
    public readonly completedAt: Date,
    public readonly completedBy: UserId
  ) {}
}

class OrderShippedEvent {
  constructor(
    public readonly orderId: OrderId,
    public readonly trackingNumber: string,
    public readonly shippedAt: Date
  ) {}
}
```

---

## Domain Services

### Rules

- **Suffix `DomainService`**: For domain-level services
- **Suffix `Service`**: For application services
- **Clear purpose**: What business operation?

### Examples

```typescript
// GOOD - Domain services
interface PricingDomainService {
  calculatePrice(productId: ProductId, quantity: number, customerId: CustomerId): Money;
}

interface InventoryDomainService {
  checkAvailability(productId: ProductId, quantity: number): AvailabilityResult;
  reserve(productId: ProductId, quantity: number): ReservationResult;
}

// GOOD - Application services
class OrderApplicationService {
  constructor(
    private orderRepository: OrderRepository,
    private paymentService: PaymentService
  ) {}
}
```

---

## Method Naming

### Rules

| Category | Convention | Example |
|----------|------------|---------|
| Getters | `get` + Noun | `getName()`, `getId()` |
| Boolean Getters | `is` + Adjective | `isDone()`, `isActive()` |
| Setters | `set` + Noun | `setName()` (entities only) |
| Actions | Verb + noun | `addProject()`, `setDone()`, `deleteTask()` |
| Queries | Noun or verb | `getProjects()`, `findById()` |
| Factory | `of()` or `create()` | `ProjectName.of()`, `Task.create()` |
| Mappers | `to` + Target | `toDto()`, `toDomain()`, `toPo()` |

### Examples

```typescript
// GOOD - Method naming
class Task {
  getId(): TaskId { return this._id; }
  isDone(): boolean { return this._status === TaskStatus.DONE; }
  setDescription(description: TaskDescription): void { this._description = description; }
  addSubtask(subtask: Subtask): void { this._subtasks.push(subtask); }
  findSubtaskById(id: SubtaskId): Subtask | undefined { /* ... */ }
  static create(title: TaskTitle): Task { /* ... */ }
}

class TaskMapper {
  toDto(task: Task): TaskDto { /* ... */ }
  toDomain(po: TaskPo): Task { /* ... */ }
}
```

---

## Field Naming

### Rules

- **Descriptive names**: Clearly indicate purpose
- **Private fields**: Use `_` prefix or `#` for privacy
- **Collections**: Use plural or collection type names
- **Constants**: UPPER_SNAKE_CASE

### Examples

```typescript
// GOOD - Field naming
class Task {
  private readonly _id: TaskId;
  private _title: TaskTitle;
  private _description: TaskDescription | null;
  private _status: TaskStatus;
  private readonly _subtasks: Subtask[];
  private _createdAt: Date;

  private static readonly DEFAULT_TITLE = 'Untitled Task';
}

// GOOD - Constants
const MAX_RETRY_COUNT = 3;
const DEFAULT_TIMEOUT_MS = 5000;
const TASK_STATUS_OPEN = 'OPEN';

// BAD - Poor field naming
class Task {
  private id: string;          // Too generic
  private t: TaskTitle;         // Unclear abbreviation
  private stuff: Subtask[];     // Unclear purpose
}
```

---

## Common Mistakes to Avoid

```typescript
// BAD - Abbreviations
class UsrSrv {
  private rpos: Repository;
  prvdr: Provider;
}

// GOOD - Full words
class UserService {
  private repository: Repository;
  provider: Provider;
}

// BAD - Inconsistent naming
class UserService {
  createUser() { }
  makeUser() { }  // Inconsistent with createUser
  addNewUser() { } // Different from createUser
}

// GOOD - Consistent naming
class UserService {
  createUser() { }
  updateUser() { }
  deleteUser() { }
  getUser() { }
}

// BAD - Magic numbers
if (user.status === 2 && user.score > 75) {
  activatePremium(user);
}

// GOOD - Named constants
const PREMIUM_STATUS = 2;
const PREMIUM_ACTIVATION_SCORE_THRESHOLD = 75;

if (user.status === PREMIUM_STATUS && user.score > PREMIUM_ACTIVATION_SCORE_THRESHOLD) {
  activatePremium(user);
}
```

---

## AI-Specific Guidelines

When AI generates code, ensure these naming patterns are followed:

```typescript
// ✅ CORRECT: Use Case Interface Naming
interface AddTaskUseCase extends Command<AddTaskInput, CqrsOutput> { }

// ✅ CORRECT: Input as Inner Class
interface AddTaskUseCase {
    class AddTaskInput implements Input {
        public toDoListId: string;
        public projectName: string;
        public description: string;
    }
}

// ✅ CORRECT: Service Implementation Naming
class AddTaskService implements AddTaskUseCase {
    constructor(
        private repository: ToDoListRepository,
        private presenter: TaskPresenter
    ) {}
}

// ✅ CORRECT: Value Object with validation
class TaskId {
    private readonly _value: string;

    constructor(value: string) {
        if (!value || value.trim().length === 0) {
            throw new Error('Task ID cannot be empty');
        }
        this._value = value;
    }

    get value(): string { return this._value; }
}
```

---

## References

1. Martin, Robert C. "Clean Code: A Handbook of Agile Software Craftsmanship"
2. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software"
3. "TypeScript Deep Dive" by Basarat Ali Syed
4. Google TypeScript Style Guide
