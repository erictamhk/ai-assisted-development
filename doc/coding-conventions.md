# Coding Conventions and Best Practices

## Core Concept

Coding conventions are a set of guidelines and standards that define how code should be written within a project. These conventions ensure consistency, readability, and maintainability across the codebase. While specific syntax may vary by language, the principles of clear, consistent code apply universally.

This document captures universal conventions extracted from real-world projects, focusing on practices that have proven effective in large, complex codebases. These conventions support team collaboration, reduce cognitive load, and make code easier to understand and modify.

---

## Naming Conventions

### General Principles
- Use descriptive names that clearly indicate purpose
- Avoid abbreviations except for widely accepted ones (id, num, config)
- Be consistent with existing naming patterns in the codebase
- Choose clarity over brevity

### Type Naming

| Type | Convention | Example |
|------|------------|---------|
| Entity | Singular noun | `Project`, `Task`, `User` |
| Value Object | Descriptive noun/phrase | `ProjectName`, `TaskId`, `Money` |
| Read-Only Interface | `ReadOnly` + Entity name | `ReadOnlyProject` |
| Use Case Interface | Verb + noun + `UseCase` | `AddTaskUseCase`, `ShowUseCase` |
| Use Case Service | Noun + `Service` | `AddTaskService`, `ShowService` |
| DTO | Entity + `Dto` | `TaskDto`, `ProjectDto` |
| PO (Persistent Object) | Entity + `Po` | `TaskPo`, `ProjectPo` |
| Mapper | Entity + `Mapper` | `TaskMapper`, `ProjectMapper` |
| Input | Operation + `Input` | `AddTaskInput`, `ShowInput` |
| Output | Operation + `Output` | `ShowOutput`, `TodayOutput` |
| Controller | Operation + `XxxController` | `AddConsoleController` |
| Presenter Interface | Operation + `Presenter` | `ShowPresenter` |
| Presenter Impl | Operation + `XxxPresenter` | `ShowConsolePresenter` |

### Method Naming

| Category | Convention | Example |
|----------|------------|---------|
| Getters | `get` + Noun | `getName()`, `getId()` |
| Boolean Getters | `is` + Adjective | `isDone()`, `isActive()` |
| Setters | `set` + Noun | `setName()` (entities only) |
| Actions | Verb + noun | `addProject()`, `setDone()`, `deleteTask()` |
| Queries | Noun or verb | `getProjects()`, `findById()` |
| Factory | `of()` or `create()` | `ProjectName.of()`, `Task.create()` |
| Mappers | `to` + Target | `toDto()`, `toDomain()`, `toPo()` |

### Field Naming

```typescript
// Private fields
private name: string;
private readonly id: ProjectId;
private tasks: TaskCollection;

// Descriptive names
private completedTasks: Task[];     // ✓ Clear
private doneItems: Task[];          // ✓ Acceptable
private stuff: Task[];              // ✗ Unclear

// Collections
private tasks: Task[];             // ✓ Plural for arrays
private taskList: Task[];          // ✓ Acceptable
private t: Task[];                 // ✗ Unclear
```

---

## Code Organization

### Package/Directory Structure
Organize code by feature (vertical slice) while maintaining hexagonal boundaries:

```
feature-name/
├── domain/              # Entities, Value Objects
├── usecase/
│   ├── port/in/        # Input ports (use case interfaces)
│   ├── service/        # Use case implementations
│   └── port/           # DTOs, POs, Mappers
└── adapter/
    ├── in/             # Controllers
    └── out/            # Presenters, Repositories
```

### Class/Interface Organization
```typescript
// Standard ordering within a class
1. Static fields/constants
2. Instance fields (private first, then protected/public)
3. Constructors
4. Public methods
5. Protected methods
6. Private methods
7. Getters/Setters
```

---

## Immutability

### Value Objects
Always use immutable value objects:

```typescript
// ✓ CORRECT - Immutable
record Email implements ValueObject {
  private readonly value: string;

  constructor(value: string) {
    if (!this.isValid(value)) {
      throw new DomainError('Invalid email');
    }
    this.value = value;
  }

  getValue(): string {
    return this.value;
  }

  private isValid(email: string): boolean {
    return email.includes('@');
  }
}

// ✗ INCORRECT - Mutable
class Email {
  public value: string;  // Public and mutable
}
```

### Entities
Make fields private with final where possible:

```typescript
class Project implements Entity<ProjectId> {
  private readonly id: ProjectId;      // ✓ Final - cannot be changed
  private name: ProjectName;            // ✓ Private - encapsulated
  private tasks: TaskCollection;        // ✓ Private - encapsulated

  constructor(id: ProjectId, name: ProjectName) {
    this.id = id;
    this.name = name;
    this.tasks = new TaskCollection();
  }
}
```

### Return Defensive Copies
```typescript
// Return unmodifiable views
getTasks(): ReadonlyArray<Task> {
  return Object.freeze([...this.tasks.getAll()]);
}

// Or return copy
getTasks(): Task[] {
  return [...this.tasks.getAll()];
}
```

---

## Constructor Injection

Prefer constructor injection for dependencies:

```typescript
// ✓ CORRECT - Constructor injection
class AddTaskService implements AddTaskUseCase {
  private readonly repository: ToDoListRepository;
  private readonly presenter: TaskPresenter;

  constructor(
    repository: ToDoListRepository,
    presenter: TaskPresenter
  ) {
    this.repository = repository;
    this.presenter = presenter;
  }
}

// ✓ CORRECT - Interface injection
class AddTaskService implements AddTaskUseCase {
  constructor(
    private repository: ToDoListRepository
  ) {}
}
```

---

## Error Handling

### Use Specific Exception Types
```typescript
// ✓ CORRECT - Specific exceptions
class DomainError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'DomainError';
  }
}

class ValidationError extends DomainError {
  constructor(field: string, message: string) {
    super(`Validation failed for ${field}: ${message}`);
    this.name = 'ValidationError';
  }
}

// ✗ INCORRECT - Generic exceptions
throw new Error('Something went wrong');
```

### Use Result/Output Types for Operations
```typescript
// Return structured output
interface CqrsOutput {
  isSuccess(): boolean;
  getMessage(): string | null;
  getData(): unknown;
  fail(): this;
  succeed(): this;
  setMessage(msg: string): this;
  setData(data: unknown): this;
}

// Usage
return CqrsOutput.create()
  .fail()
  .setMessage('Task not found');

return CqrsOutput.create()
  .succeed()
  .setData(TaskMapper.toDto(task));
```

---

## Validation Strategy

### Value Objects
Validate in constructor:

```typescript
record ProjectName implements ValueObject {
  private readonly value: string;

  constructor(value: string) {
    if (value === null || value.trim().length === 0) {
      throw new ValidationError('name', 'cannot be empty');
    }
    if (value.length > 100) {
      throw new ValidationError('name', 'cannot exceed 100 characters');
    }
    this.value = value.trim();
  }
}
```

### Use Cases
Validate inputs before business logic:

```typescript
class AddTaskService implements AddTaskUseCase {
  async execute(input: AddTaskInput): Promise<CqrsOutput> {
    // 1. Validate inputs
    if (!input.description) {
      return CqrsOutput.create()
        .fail()
        .setMessage('Description is required');
    }

    // 2. Validate business rules
    const toDoList = await this.repository.findById(
      ToDoListId.of(input.toDoListId)
    );

    if (!toDoList) {
      return CqrsOutput.create()
        .fail()
        .setMessage('ToDoList not found');
    }

    // 3. Business operation
    const task = toDoList.addTask(TaskDescription.of(input.description));
    await this.repository.save(toDoList);

    return CqrsOutput.create().succeed();
  }
}
```

---

## Stream API Usage

### Preferred Patterns

```typescript
// Method chaining
const activeTasks = tasks
  .filter(t => !t.isDone())
  .map(TaskMapper.toDto)
  .toList();

// Method references
const names = projects
  .stream()
  .map(Project::getName)
  .filter(n => n.value.includes('Important'))
  .toList();

// findFirst for single results
const task = tasks
  .stream()
  .filter(t => t.getId().equals(taskId))
  .findFirst()
  .orElse(null);

// Avoid old-style loops
// ✗ for (let i = 0; i < tasks.length; i++)
```

---

## Optional Pattern

### Use Optional for Nullable Returns

```typescript
// ✓ CORRECT - Use Optional
async findById(id: TaskId): Promise<Task | null> {
  const po = await this.entityManager.findOne(TaskPo, { where: { id: id.value } });
  return po ? TaskMapper.toDomain(po) : null;
}

// ✓ CORRECT - Return Optional
async findById(id: TaskId): Promise<Optional<Task>> {
  const po = await this.entityManager.findOne(TaskPo, { where: { id: id.value } });
  return Optional.ofNullable(po).map(TaskMapper.toDomain);
}

// ✗ INCORRECT - Return null directly for complex operations
async findTasks(): Promise<Task[]> {
  return null;  // Confusing
}
```

### Optional Operations

```typescript
const task = repository.findById(id);

// Use map instead of get
const taskName = repository.findById(id)
  .map(task => task.getDescription().value)
  .orElse('Unknown');

// Use filter
const doneTask = repository.findById(id)
  .filter(task => task.isDone());

// Use orElse for defaults
const taskName = repository.findById(id)
  .map(task => task.getName().value)
  .orElse('Default Project');
```

---

## Testing Conventions

### Test Organization
```
src/test/java/.../feature/
├── domain/
│   └── XxxTest.java
├── usecase/
│   └── XxxServiceTest.java
└── adapter/
    └── XxxControllerTest.java
```

### Test Naming
```typescript
// Describe behavior with underscores
@Test
public void add_a_project_with_duplicated_name_has_no_effect() {
  // Arrange
  // Act
  // Assert
}

@Test
public void remove_task_from_project_decreases_task_count() {
  // ...
}
```

### Test Structure (Arrange-Act-Assert)

```typescript
@Test
public void add_task_increases_project_task_count() {
  // 1. Arrange
  const repository = new InMemoryToDoListRepository();
  const project = new ProjectBuilder()
    .withName('Test Project')
    .build();
  await repository.save(project);

  const service = new AddTaskService(repository);
  const input = new AddTaskInput();
  input.toDoListId = project.getId().value;
  input.projectName = 'Test Project';
  input.description = 'New Task';

  // 2. Act
  const result = await service.execute(input);

  // 3. Assert
  assertTrue(result.isSuccess());
  const updatedProject = await repository.findById(project.getId());
  assertEquals(1, updatedProject.getTaskCount());
}
```

---

## Code Quality Rules

### KISS (Keep It Simple, Stupid)
- Prefer simple, straightforward solutions
- Avoid over-engineering for hypothetical future needs
- Clear, readable code over clever code

### DRY (Don't Repeat Yourself)
- Extract common patterns into shared utilities
- Use inheritance and composition appropriately
- Avoid copy-paste code

### Single Responsibility
- Each class has one reason to change
- Each method does one thing well
- Extract complex logic into private methods

### Tell, Don't Ask
```typescript
// ✓ Tell objects what to do
project.addTask(taskDescription);

// ✗ Ask about state, then act
if (project.getTasks().length < project.getMaxTasks()) {
  project.getTasks().push(newTask);
}
```

---

## Documentation Standards

### Self-Documenting Code
```typescript
// ✓ GOOD - Clear names, obvious intent
const completedTasks = tasks.filter(t => t.isDone());

// ✗ BAD - Needs comment to explain
const filtered = items.filter(/* filter out done items */);
```

### Minimal Comments
- Comment why, not what
- Explain complex business rules
- Document non-obvious decisions
- Keep comments synchronized with code

### When to Document
- API contracts and interfaces
- Complex business logic
- Non-obvious workarounds
- Design decisions and rationale

---

## Common Anti-Patterns to Avoid

1. **Primitive Obsession**: Using primitives instead of value objects
2. **Feature Envy**: Method accesses data of another class excessively
3. **God Class**: Class with too many responsibilities
4. **Magic Numbers/Strings**: Hardcoded values without explanation
5. **Long Methods**: Methods doing too many things
6. **Deep Nesting**: Multiple levels of conditionals/loops
7. **Public Fields**: Exposing internal state
8. **Null Returns**: Returning null instead of Optional/empty collections
9. **Mutable Collections**: Exposing internal collections
10. **Unused Code**: Code that is never called

---

## References and Further Reading

1. Martin, Robert C. "Clean Code: A Handbook of Agile Software Craftsmanship." Prentice Hall, 2008.
2. Martin, Robert C. "Clean Architecture: A Craftsman's Guide to Software Structure and Design." Prentice Hall, 2017.
3. Effective Java. Joshua Bloch. Addison-Wesley, 2018.
4. "TypeScript Deep Dive." Basarat Ali Syed.
5. "Design Patterns: Elements of Reusable Object-Oriented Software." Gamma et al. Addison-Wesley, 1994.
