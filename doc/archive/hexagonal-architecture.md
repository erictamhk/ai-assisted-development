# Hexagonal Architecture (Ports and Adapters)

## Core Concept

Hexagonal Architecture, also known as Ports and Adapters, is an architectural pattern that structures applications around the business logic while keeping it independent of external concerns. The architecture creates a hexagonal boundary where the internal application (the domain) communicates with external systems (adapters) through well-defined interfaces (ports).

The fundamental insight is that business logic should not depend on databases, web frameworks, or any external system. Instead, external systems depend on abstractions defined by the business logic. This inversion of dependencies ensures that the core application can be tested independently of its delivery mechanisms and that external systems can be replaced without changing business code.

The pattern was formalized by Alistair Cockburn as a response to the limitations of layered architectures where business logic often became coupled to infrastructure concerns. By explicitly defining ports (interfaces) and placing adapters at the boundaries, the architecture creates clear separation between what the application does (core) and how it integrates with the outside world (adapters).

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HEXAGONAL ARCHITECTURE                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│                         ┌─────────────────────┐                         │
│                         │                     │                         │
│                         │     APPLICATION     │                         │
│                         │        CORE         │                         │
│                         │                     │                         │
│                         │  ┌───────────────┐  │                         │
│                         │  │   DOMAIN      │  │                         │
│                         │  │   LAYER       │  │                         │
│                         │  │               │  │                         │
│                         │  │  • Entities   │  │                         │
│                         │  │  • Value Obs  │  │                         │
│                         │  │  • Aggregates │  │                         │
│                         │  └───────────────┘  │                         │
│                         │           │          │                         │
│                         │           ▼          │                         │
│                         │  ┌───────────────┐  │                         │
│                         │  │   USE CASE    │  │                         │
│                         │  │   LAYER       │  │                         │
│                         │  │               │  │                         │
│                         │  │  • Services   │  │                         │
│                         │  │  • Use Cases  │  │                         │
│                         │  │  • Ports      │  │                         │
│                         │  └───────────────┘  │                         │
│                         │                     │                         │
│                         └──────────┬──────────┘                         │
│                                    │                                    │
│              ┌─────────────────────┼─────────────────────┐              │
│              │                     │                     │              │
│              ▼                     ▼                     ▼              │
│    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │
│    │     PORTS       │  │                 │  │     PORTS       │       │
│    │   (Interfaces)  │  │                 │  │   (Interfaces)  │       │
│    │                 │  │                 │  │                 │       │
│    │  • Input Ports  │  │                 │  │  • Output Ports │       │
│    │    (Use Cases)  │  │                 │  │  • Repositories │       │
│    │                 │  │                 │  │  • Presenters   │       │
│    └────────┬────────┘  │                 │  └────────┬────────┘       │
│             │           │                 │           │                │
│             │           │                 │           │                │
│             ▼           │                 │           ▼                │
│    ┌─────────────────┐  │                 │  ┌─────────────────┐       │
│    │   ADAPTERS      │  │                 │  │   ADAPTERS      │       │
│    │   (Inbound)     │  │                 │  │   (Outbound)    │       │
│    │                 │  │                 │  │                 │       │
│    │  • Web          │  │                 │  │  • Database     │       │
│    │  • Console      │  │                 │  │  • File System  │       │
│    │  • API          │  │                 │  │  • External APIs│       │
│    │  • CLI          │  │                 │  │  • Messaging    │       │
│    └─────────────────┘  │                 │  └─────────────────┘       │
│                         │                 │                            │
└─────────────────────────┼─────────────────┼────────────────────────────┘
                          │                 │
                          ▼                 ▼
              ┌───────────────────┐ ┌───────────────────┐
              │    EXTERNAL       │ │    EXTERNAL       │
              │    SYSTEMS        │ │    SYSTEMS        │
              │                   │ │                   │
              │  • Web Browsers   │ │  • Databases      │
              │  • Mobile Apps    │ │  • File Systems   │
              │  • CLI Users      │ │  • Web Services   │
              └───────────────────┘ └───────────────────┘
```

### The Three Core Layers

**Domain Layer (Core)**
The domain layer contains the business logic of the application. It consists of entities, value objects, aggregates, and domain services. This layer has NO dependencies on any other layer. It defines interfaces (ports) that describe what it needs from the outside world.

**Use Case Layer (Application)**
The use case layer contains application-specific business logic. It orchestrates operations by using domain objects and calling through ports to interact with external systems. This layer depends only on domain abstractions.

**Adapter Layer (External Integration)**
The adapter layer contains implementations of ports that integrate with external systems. Inbound adapters handle user input, while outbound adapters handle external system integration. Adapters depend on ports, never the reverse.

---

## Ports and Adapters Explained

### Ports (Interfaces)

Ports define the boundary between the application core and external systems. They are abstract contracts that specify what the application expects from the outside world.

**Input Ports (Use Case Interfaces)**
Input ports define operations that external systems can perform on the application. They represent the application's API for incoming operations.

```typescript
// Input Port - What the outside world can DO
interface AddTaskUseCase {
  execute(input: AddTaskInput): Promise<CqrsOutput>;
}

interface GetTaskUseCase {
  execute(input: GetTaskInput): Promise<GetTaskOutput>;
}

interface DeleteTaskUseCase {
  execute(input: DeleteTaskInput): Promise<CqrsOutput>;
}
```

**Output Ports (Repository and Presenter Interfaces)**
Output ports define operations that the application needs to perform on external systems. These are implemented by outbound adapters.

```typescript
// Output Port - What the application NEEDS
interface TaskRepository {
  save(task: Task): void;
  findById(id: TaskId): Promise<Task | null>;
  delete(id: TaskId): void;
}

interface TaskPresenter {
  present(dto: TaskDto): void;
}

interface NotificationService {
  sendNotification(message: string): Promise<void>;
}
```

### Adapters (Implementations)

Adapters implement ports and handle the specifics of interacting with external systems.

**Inbound Adapters (Delivery Mechanisms)**
Inbound adapters translate external input into calls to input ports.

```typescript
// Web Controller (Inbound Adapter)
class AddTaskWebController {
  constructor(private addTaskUseCase: AddTaskUseCase) {}

  async handleRequest(request: HttpRequest): Promise<HttpResponse> {
    const input = new AddTaskInput();
    input.description = request.body.description;
    input.projectId = request.params.projectId;
    
    const result = await this.addTaskUseCase.execute(input);
    
    return new HttpResponse(
      result.isSuccess ? 200 : 400,
      result.toDto()
    );
  }
}

// Console Controller (Inbound Adapter)
class AddTaskConsoleController {
  constructor(private addTaskUseCase: AddTaskUseCase) {}

  handleRequest(request: ConsoleRequest): Response {
    const input = new AddTaskInput();
    input.description = request.getArg('description');
    input.projectId = request.getArg('projectId');
    
    const result = this.addTaskUseCase.execute(input);
    return Response.of(result);
  }
}
```

**Outbound Adapters (External System Integrations)**
Outbound adapters implement output ports and handle communication with external systems.

```typescript
// Repository Implementation (Outbound Adapter)
class TaskCrudRepository implements TaskRepository {
  constructor(
    private entityManager: EntityManager,
    private taskPoClass: typeof TaskPo
  ) {}

  async save(task: Task): Promise<void> {
    const po = TaskMapper.toPo(task);
    await this.entityManager.save(po);
  }

  async findById(id: TaskId): Promise<Task | null> {
    const po = await this.entityManager.findOne(this.taskPoClass, {
      where: { id: id.value }
    });
    return po ? TaskMapper.toDomain(po) : null;
  }

  async delete(id: TaskId): Promise<void> {
    await this.entityManager.delete(this.taskPoClass, {
      where: { id: id.value }
    });
  }
}

// In-Memory Repository (Outbound Adapter for Testing)
class InMemoryTaskRepository implements TaskRepository {
  private tasks: Map<string, Task> = new Map();

  save(task: Task): void {
    this.tasks.set(task.getId().value, task);
  }

  findById(id: TaskId): Task | null {
    return this.tasks.get(id.value) || null;
  }

  delete(id: TaskId): void {
    this.tasks.delete(id.value);
  }
}

// Presenter Implementation (Outbound Adapter)
class TaskConsolePresenter implements TaskPresenter {
  constructor(private output: PrintWriter) {}

  present(dto: TaskDto): void {
    this.output.println(`Task: ${dto.description}`);
    this.output.println(`Status: ${dto.isDone ? 'Done' : 'Pending'}`);
    this.output.println(`ID: ${dto.id}`);
  }
}
```

---

## Dependency Direction

The most critical rule in Hexagonal Architecture is that **dependencies always point inward** toward the domain. This is the Dependency Inversion Principle applied at the architectural level.

```
Dependency Flow (CORRECT)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

External Input (Web/Console)
        │
        ▼
Inbound Adapter (Controller)
        │
        │ depends on
        ▼
Input Port (UseCase Interface)
        │
        │ depends on
        ▼
Use Case Service
        │
        │ depends on
        ▼
Domain Entities / Output Ports
        │
        │ ←─── implement
        ▼
Outbound Adapter (Repository/Presenter Implementation)
        │
        │ implements
        ▼
External Systems (DB, APIs, etc.)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Dependency Flow (INCORRECT - Violations)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✗ Domain depends on Infrastructure
✗ Controller directly calls Repository
✗ Use Case imports Framework classes
✗ Entity has JPA annotations
✗ Service uses HttpServletRequest
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## The Core Without Dependencies

The domain layer should have zero dependencies on any external library or framework. This makes it:

1. **Testable**: Core business logic can be tested without mocking infrastructure
2. **Maintainable**: Changes to external systems don't affect business logic
3. **Portable**: Core can be reused in different contexts

```typescript
// Domain Layer - Pure business logic, no dependencies
interface Entity<TId> {
  getId(): TId;
}

interface ValueObject {
  equals(other: ValueObject): boolean;
}

// Example: Entity with Value Object ID
class Project implements Entity<ProjectId> {
  constructor(
    private readonly id: ProjectId,
    private readonly name: ProjectName,
    private readonly tasks: TaskCollection
  ) {}

  getId(): ProjectId {
    return this.id;
  }

  addTask(task: Task): void {
    this.tasks.add(task);
  }

  removeTask(taskId: TaskId): void {
    this.tasks.remove(taskId);
  }
}

// Example: Value Object with validation
record ProjectName implements ValueObject {
  private readonly value: string;

  constructor(value: string) {
    if (value === null || value.trim().length === 0) {
      throw new DomainError('Project name cannot be empty');
    }
    if (value.length > 100) {
      throw new DomainError('Project name cannot exceed 100 characters');
    }
    this.value = value;
  }

  equals(other: ValueObject): boolean {
    return other instanceof ProjectName && this.value === other.value;
  }
}
```

---

## Use Case Implementation Pattern

Use cases encapsulate a single business operation. Each use case follows a consistent pattern:

```typescript
// Step 1: Input Port (Interface)
interface AddTaskUseCase extends Command<AddTaskInput, CqrsOutput> {
}

// Step 2: Input Object
class AddTaskInput implements Input {
  toDoListId: string;
  projectName: string;
  description: string;
}

// Step 3: Output Object (if custom needed)
class AddTaskOutput implements Output {
  taskId: string;
  createdAt: Date;
}

// Step 4: Use Case Service (Implementation)
class AddTaskService implements AddTaskUseCase {
  constructor(
    private toDoListRepository: ToDoListRepository,
    private taskPresenter: TaskPresenter
  ) {}

  async execute(input: AddTaskInput): Promise<CqrsOutput> {
    // 1. Validate input
    if (!input.description) {
      return CqrsOutput.create()
        .fail()
        .setMessage('Task description is required');
    }

    // 2. Load aggregate root
    const toDoList = await this.toDoListRepository.findById(
      ToDoListId.of(input.toDoListId)
    );

    if (!toDoList) {
      return CqrsOutput.create()
        .fail()
        .setMessage('ToDoList not found');
    }

    // 3. Perform business operation
    const project = toDoList.findProject(ProjectName.of(input.projectName));
    if (!project) {
      return CqrsOutput.create()
        .fail()
        .setMessage('Project not found');
    }

    const task = project.addTask(TaskDescription.of(input.description));

    // 4. Persist changes
    await this.toDoListRepository.save(toDoList);

    // 5. Return result
    return CqrsOutput.create()
      .succeed()
      .setData(TaskMapper.toDto(task));
  }
}
```

---

## CQRS Pattern in Hexagonal Architecture

Command Query Responsibility Segregation (CQRS) separates operations that change state (Commands) from operations that read data (Queries).

```typescript
// Command - Modifies state
interface Command<Input, Output> {
  execute(input: Input): Promise<Output>;
}

// Query - Reads state
interface Query<Input, Output> {
  execute(input: Input): Promise<Output>;
}

// Write Operation (Command)
interface CreateProjectCommand extends Command<CreateProjectInput, CqrsOutput> {
}

// Read Operation (Query)
interface GetProjectQuery extends Query<GetProjectInput, GetProjectOutput> {
}

// Different use cases for different operations
class CreateProjectService implements CreateProjectCommand {
  // Modifies state - returns success/failure
}

class GetProjectService implements GetProjectQuery {
  // Only reads - returns data
}
```

---

## Data Transfer Objects (DTOs)

DTOs transfer data between layers without behavior.

```typescript
// DTO - Simple data container
class TaskDto {
  id: string;
  description: string;
  isDone: boolean;
  projectName: string;
  createdAt: Date;
}

// PO - Database representation (in adapter layer)
class TaskPo {
  id: string;
  description: string;
  is_done: boolean;
  project_id: string;
  created_at: Date;
}

// Mapper - Transforms between layers
class TaskMapper {
  // Domain → DTO
  static toDto(task: Task): TaskDto {
    const dto = new TaskDto();
    dto.id = task.getId().value;
    dto.description = task.getDescription().value;
    dto.isDone = task.isDone();
    dto.projectName = task.getProjectName().value;
    dto.createdAt = task.getCreatedAt();
    return dto;
  }

  // PO → Domain
  static toDomain(po: TaskPo): Task {
    return new Task(
      TaskId.of(po.id),
      TaskDescription.of(po.description),
      po.is_done,
      ProjectName.of(po.project_name),
      po.created_at
    );
  }

  // Domain → PO
  static toPo(task: Task): TaskPo {
    const po = new TaskPo();
    po.id = task.getId().value;
    po.description = task.getDescription().value;
    po.is_done = task.isDone();
    po.project_id = task.getProjectName().value;
    po.created_at = task.getCreatedAt();
    return po;
  }

  // Support collections
  static toDtoList(tasks: Task[]): TaskDto[] {
    return tasks.map(TaskMapper.toDto);
  }
}
```

---

## Read-Only Interface Pattern

Prevent unintended modifications by exposing read-only views.

```typescript
// Read-Only Interface
interface ReadOnlyProject {
  getId(): ProjectId;
  getName(): ProjectName;
  getTasks(): ReadonlyArray<ReadOnlyTask>;
  containsTask(taskId: TaskId): boolean;
}

// Implementation
class ReadOnlyProjectImpl implements ReadOnlyProject {
  constructor(private project: Project) {}

  getId(): ProjectId {
    return this.project.getId();
  }

  getName(): ProjectName {
    return this.project.getName();
  }

  getTasks(): ReadonlyArray<ReadOnlyTask> {
    return this.project.getTasks()
      .map(task => new ReadOnlyTaskImpl(task));
  }

  containsTask(taskId: TaskId): boolean {
    return this.project.containsTask(taskId);
  }
}

// Throws on modification attempts
class ReadOnlyTaskImpl implements ReadOnlyTask {
  constructor(private task: Task) {}

  getId(): TaskId {
    return this.task.getId();
  }

  setDone(done: boolean): void {
    throw new UnsupportedOperationException('Read-only view');
  }
}
```

---

## Testing Strategy

Because of the clear separation, testing can be done at multiple levels:

```typescript
// 1. Domain Layer - Pure unit tests, no dependencies
describe('Project', () => {
  it('should create project with valid name', () => {
    const name = ProjectName.of('Test Project');
    const project = new Project(name);
    expect(project.getName()).toEqual(name);
  });

  it('should reject empty project name', () => {
    expect(() => new ProjectName.of(''))
      .toThrow(DomainError);
  });
});

// 2. Use Case Layer - Tests with in-memory repositories
describe('AddTaskService', () => {
  it('should add task to project', async () => {
    const repository = new InMemoryToDoListRepository();
    const service = new AddTaskService(repository);
    
    const input = new AddTaskInput();
    input.toDoListId = 'list-1';
    input.projectName = 'Project A';
    input.description = 'New Task';
    
    const result = await service.execute(input);
    
    expect(result.isSuccess).toBe(true);
  });
});

// 3. Integration Tests - Full stack with test adapters
describe('AddTaskFlow', () => {
  it('should handle end-to-end flow', async () => {
    const controller = new AddTaskConsoleController(
      new AddTaskService(TestRepository)
    );
    
    const request = new ConsoleRequest(['add', 'Project A', 'Task 1']);
    const response = controller.execute(request);
    
    expect(response.exitCode).toBe(ExitCode.SUCCESS);
  });
});
```

---

## Benefits of Hexagonal Architecture

1. **Testability**: Core business logic can be tested without external dependencies
2. **Flexibility**: External systems can be swapped without changing business code
3. **Maintainability**: Clear boundaries make code easier to understand and modify
4. **Adaptability**: Application can be delivered through multiple channels
5. **Parallel Development**: Teams can work on different parts independently
6. **Longevity**: Core business logic outlives specific technology choices

---

## Anti-Patterns to Avoid

### 1. Leaking Infrastructure into Domain
```typescript
// WRONG - Domain depends on JPA
@Entity
class Project {
  @Id
  private id: string;
  
  @Column
  private name: string;
}
```

### 2. Skipping Layers
```typescript
// WRONG - Controller directly calls repository
class Controller {
  async handle() {
    // Bypasses use case layer
    await this.entityManager.save(entity);
  }
}
```

### 3. Concrete Dependencies in Use Cases
```typescript
// WRONG - Use case depends on concrete implementation
class Service {
  constructor(
    private repository: CrudRepository // Should be TaskRepository interface
  ) {}
}
```

---

## References and Further Reading

1. Cockburn, Alistair. "Hexagonal Architecture." 2005.
2. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
3. Martin, Robert C. "The Dependency Inversion Principle." 1996.
4. "Implementing Domain-Driven Design." Vaughn Vernon. Addison-Wesley, 2013.
5. "Architecture Patterns with Python." Harry Percival and Bob Gregory. O'Reilly, 2021.
