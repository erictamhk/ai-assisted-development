# Domain-Driven Design Patterns

## Core Concept

Domain-Driven Design (DDD) is an approach to software development that focuses on modeling the business domain accurately and completely. The key insight is that complex business domains should be reflected in the code structure, making the software a faithful representation of the business concepts it serves.

DDD provides a set of patterns for modeling domain logic, organizing code around business concepts, and managing the complexity of large systems. These patterns help bridge the gap between domain experts' mental models and the actual software implementation.

The patterns in this document have been extracted from real-world refactoring experience and represent universal, timeless principles that apply across different projects and technology stacks.

---

## Entity Pattern

### Definition
Entities are domain objects that have a distinct identity that persists through time and state changes. Unlike value objects, entities are compared by identity, not by their attributes.

### Key Characteristics
- Has a unique identity (ID) that distinguishes it from other entities
- Equality is based on identity, not attributes
- Has a lifecycle (creation, modification, deletion)
- Encapsulates business behavior and rules
- Can change state over time

### Implementation

```typescript
// Entity Interface
interface Entity<TId> {
  getId(): TId;
}

// Concrete Entity
class Project implements Entity<ProjectId> {
  constructor(
    private readonly id: ProjectId,
    private name: ProjectName,
    private tasks: TaskCollection,
    private createdAt: Date
  ) {}

  getId(): ProjectId {
    return this.id;
  }

  getName(): ProjectName {
    return this.name;
  }

  getTasks(): ReadonlyArray<Task> {
    return this.tasks.getAll();
  }

  getCreatedAt(): Date {
    return this.createdAt;
  }

  // Business behavior
  addTask(description: TaskDescription): Task {
    const task = new Task(
      TaskId.generate(),
      description,
      false,
      this.id
    );
    this.tasks.add(task);
    return task;
  }

  removeTask(taskId: TaskId): void {
    this.tasks.remove(taskId);
  }

  rename(newName: ProjectName): void {
    this.name = newName;
  }

  // Equality by identity
  equals(other: Entity<ProjectId>): boolean {
    if (other === null || other === undefined) return false;
    return this.id.equals(other.getId());
  }
}
```

### When to Use Entities
- When the object has a lifecycle (can be created, modified, deleted)
- When the object needs to be tracked and referenced
- When equality should be based on identity, not attributes
- When the object has behavior that changes its state

---

## Value Object Pattern

### Definition
Value Objects are immutable objects that are defined solely by their attributes. They have no identity and are compared by their attribute values.

### Key Characteristics
- Immutable (once created, cannot be changed)
- No identity (equality is based on attributes)
- Self-validating in constructor
- Represents a descriptive aspect of the domain
- Replaces primitive obsession

### Implementation

```typescript
// Value Object with validation
record ProjectName implements ValueObject {
  private readonly value: string;

  constructor(value: string) {
    // Validation
    if (value === null || value.trim().length === 0) {
      throw new DomainError('Project name cannot be empty');
    }
    if (value.length > 100) {
      throw new DomainError('Project name cannot exceed 100 characters');
    }
    this.value = value.trim();
  }

  getValue(): string {
    return this.value;
  }

  equals(other: ValueObject): boolean {
    return other instanceof ProjectName && this.value === other.value;
  }

  // Behavior within value object
  containsKeyword(keyword: string): boolean {
    return this.value.toLowerCase().includes(keyword.toLowerCase());
  }
}

// Complex Value Object
record Money implements ValueObject {
  private readonly amount: number;
  private readonly currency: Currency;

  constructor(amount: number, currency: Currency) {
    if (amount < 0) {
      throw new DomainError('Amount cannot be negative');
    }
    this.amount = Math.round(amount * 100) / 100; // Round to 2 decimal places
    this.currency = currency;
  }

  add(other: Money): Money {
    if (!this.currency.equals(other.currency)) {
      throw new DomainError('Cannot add different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  equals(other: ValueObject): boolean {
    return other instanceof Money && 
           this.amount === other.amount &&
           this.currency.equals(other.currency);
  }
}

// Factory Method for Value Objects
class TaskId implements ValueObject {
  private readonly value: string;

  private constructor(value: string) {
    this.value = value;
  }

  // Factory method - the standard creation pattern
  static of(value: string): TaskId {
    if (!value || value.trim().length === 0) {
      throw new DomainError('Task ID cannot be empty');
    }
    return new TaskId(value);
  }

  // Alternative factory for generation
  static generate(): TaskId {
    return new TaskId(crypto.randomUUID());
  }

  equals(other: ValueObject): boolean {
    return other instanceof TaskId && this.value === other.value;
  }
}
```

### When to Use Value Objects
- When the object represents a descriptive aspect without identity
- When objects with the same attributes should be considered equal
- When the object should be immutable
- When validation belongs with the data
- When replacing primitive types with meaningful domain concepts

---

## Aggregate Root Pattern

### Definition
An Aggregate Root is a special Entity that serves as the entry point to a group of related entities and value objects (the Aggregate). The aggregate root controls access to all other objects within the aggregate and enforces consistency rules.

### Key Characteristics
- Is an Entity with additional responsibilities
- Controls consistency boundaries
- Is the only entry point to the aggregate
- Manages references to other objects in the aggregate
- Ensures invariants are maintained

### Implementation

```typescript
// Aggregate Root
class ToDoList implements Entity<ToDoListId> {
  constructor(
    private readonly id: ToDoListId,
    private name: ToDoListName,
    private projects: ProjectCollection,
    private createdAt: Date
  ) {}

  getId(): ToDoListId {
    return this.id;
  }

  // Business operations
  addProject(projectName: string): Project {
    const project = new Project(
      ProjectId.generate(),
      ProjectName.of(projectName),
      new TaskCollection(),
      this.id
    );
    this.projects.add(project);
    return project;
  }

  findProject(name: ProjectName): Project | null {
    return this.projects.findByName(name);
  }

  getAllProjects(): ReadonlyArray<Project> {
    return this.projects.getAll();
  }

  addTaskToProject(
    projectName: string, 
    taskDescription: string
  ): Task {
    const project = this.findProject(ProjectName.of(projectName));
    if (!project) {
      throw new DomainError(`Project not found: ${projectName.value}`);
    }
    return project.addTask(TaskDescription.of(taskDescription));
  }

  // Consistency enforcement
  removeProject(projectId: ProjectId): void {
    const project = this.projects.findById(projectId);
    if (!project) {
      throw new DomainError('Project not found');
    }
    // Ensure consistency: remove all tasks from project first
    project.removeAllTasks();
    this.projects.remove(projectId);
  }
}

// Collection Wrapper for Aggregate
class ProjectCollection {
  private projects: Map<string, Project> = new Map();

  add(project: Project): void {
    if (this.projects.has(project.getId().value)) {
      throw new DomainError('Project already exists');
    }
    this.projects.set(project.getId().value, project);
  }

  remove(projectId: ProjectId): void {
    this.projects.delete(projectId.value);
  }

  findById(id: ProjectId): Project | null {
    return this.projects.get(id.value) || null;
  }

  findByName(name: ProjectName): Project | null {
    for (const project of this.projects.values()) {
      if (project.getName().equals(name)) {
        return project;
      }
    }
    return null;
  }

  getAll(): ReadonlyArray<Project> {
    return Array.from(this.projects.values());
  }

  size(): number {
    return this.projects.size;
  }
}
```

### Aggregate Design Rules
1. **Reference by ID only**: External references should use identity, not direct object references
2. **Single entry point**: Only the aggregate root can be accessed from outside
3. **Consistency boundary**: All invariants are enforced within the aggregate
4. **Small aggregates**: Keep aggregates small to avoid performance issues

---

## Read-Only Interface Pattern

### Definition
Read-Only interfaces expose entities or value objects without modification capabilities, preventing unintended state changes.

### Implementation

```typescript
// Read-Only Interface
interface ReadOnlyProject {
  getId(): ProjectId;
  getName(): ProjectName;
  getTasks(): ReadonlyArray<ReadOnlyTask>;
  containsTask(taskId: TaskId): boolean;
  getTaskCount(): number;
}

// Read-Only Implementation
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

  getTaskCount(): number {
    return this.project.getTaskCount();
  }
}

// Throws on modification attempts
class ReadOnlyTaskImpl implements ReadOnlyTask {
  constructor(private task: Task) {}

  getId(): TaskId {
    return this.task.getId();
  }

  getDescription(): TaskDescription {
    return this.task.getDescription();
  }

  isDone(): boolean {
    return this.task.isDone();
  }

  // This method throws - prevents modification
  setDone(done: boolean): never {
    throw new UnsupportedOperationException('Read-only view');
  }
}
```

### Use Cases for Read-Only Views
- Returning entities from query operations
- Preventing modification in presentation layer
- Sharing entities across boundaries safely
- Implementing observer patterns

---

## Repository Pattern

### Definition
The Repository pattern abstracts data access behind a collection-like interface, hiding persistence details from the domain.

### Implementation

```typescript
// Repository Interface (Port - defined in domain)
interface ProjectRepository {
  save(project: Project): void;
  findById(id: ProjectId): Promise<Project | null>;
  findByName(name: ProjectName): Promise<Project | null>;
  delete(id: ProjectId): Promise<void>;
  exists(id: ProjectId): Promise<boolean>;
}

// In-Memory Implementation (for testing)
class InMemoryProjectRepository implements ProjectRepository {
  private projects: Map<string, Project> = new Map();

  save(project: Project): void {
    this.projects.set(project.getId().value, project);
  }

  async findById(id: ProjectId): Promise<Project | null> {
    return this.projects.get(id.value) || null;
  }

  async findByName(name: ProjectName): Promise<Project | null> {
    for (const project of this.projects.values()) {
      if (project.getName().equals(name)) {
        return project;
      }
    }
    return null;
  }

  async delete(id: ProjectId): Promise<void> {
    this.projects.delete(id.value);
  }

  async exists(id: ProjectId): Promise<boolean> {
    return this.projects.has(id.value);
  }
}

// CRUD Implementation (for production)
class CrudProjectRepository implements ProjectRepository {
  constructor(
    private entityManager: EntityManager,
    private poClass: typeof ProjectPo
  ) {}

  async save(project: Project): Promise<void> {
    const po = ProjectMapper.toPo(project);
    await this.entityManager.save(po);
  }

  async findById(id: ProjectId): Promise<Project | null> {
    const po = await this.entityManager.findOne(this.poClass, {
      where: { id: id.value }
    });
    return po ? ProjectMapper.toDomain(po) : null;
  }

  async findByName(name: ProjectName): Promise<Project | null> {
    const po = await this.entityManager.findOne(this.poClass, {
      where: { name: name.value }
    });
    return po ? ProjectMapper.toDomain(po) : null;
  }

  async delete(id: ProjectId): Promise<void> {
    await this.entityManager.delete(this.poClass, {
      where: { id: id.value }
    });
  }

  async exists(id: ProjectId): Promise<boolean> {
    const count = await this.entityManager.count(this.poClass, {
      where: { id: id.value }
    });
    return count > 0;
  }
}
```

---

## Factory Pattern

### Definition
Factories encapsulate the creation logic for complex objects, especially when creation involves validation or requires assembling multiple components.

### Implementation

```typescript
// Static Factory Method
class Task implements Entity<TaskId> {
  private constructor(
    private readonly id: TaskId,
    private description: TaskDescription,
    private isDone: boolean,
    private projectId: ProjectId,
    private createdAt: Date
  ) {}

  // Static factory method - the standard
  static create(
    description: string,
    projectId: string
  ): Task {
    return new Task(
      TaskId.generate(),
      TaskDescription.of(description),
      false,
      ProjectId.of(projectId),
      new Date()
    );
  }

  // Factory from external data
  static fromPersistence(
    id: string,
    description: string,
    isDone: boolean,
    projectId: string,
    createdAt: Date
  ): Task {
    return new Task(
      TaskId.of(id),
      TaskDescription.of(description),
      isDone,
      ProjectId.of(projectId),
      createdAt
    );
  }
}

// Builder for complex construction
class ProjectBuilder {
  private name?: ProjectName;
  private tasks: Task[] = [];
  private createdAt: Date = new Date();

  withName(name: string): this {
    this.name = ProjectName.of(name);
    return this;
  }

  withTask(task: Task): this {
    this.tasks.push(task);
    return this;
  }

  withCreatedAt(date: Date): this {
    this.createdAt = date;
    return this;
  }

  build(): Project {
    if (!this.name) {
      throw new DomainError('Project name is required');
    }
    return new Project(
      this.name,
      new TaskCollection(this.tasks),
      this.createdAt
    );
  }
}

// Usage
const project = new ProjectBuilder()
  .withName('My Project')
  .withTask(task1)
  .withTask(task2)
  .build();
```

---

## Domain Service Pattern

### Definition
Domain Services encapsulate business logic that doesn't naturally belong to a single Entity or Value Object, especially operations that involve multiple domain objects.

### Implementation

```typescript
interface ProjectTransferService {
  transferTask(
    taskId: TaskId,
    fromProjectId: ProjectId,
    toProjectId: ProjectId
  ): Promise<CqrsOutput>;
}

class ProjectTransferServiceImpl implements ProjectTransferService {
  constructor(
    private toDoListRepository: ToDoListRepository
  ) {}

  async execute(
    input: TransferTaskInput
  ): Promise<CqrsOutput> {
    // Load aggregate root
    const toDoList = await this.toDoListRepository.findById(
      ToDoListId.of(input.toDoListId)
    );

    if (!toDoList) {
      return CqrsOutput.create()
        .fail()
        .setMessage('ToDoList not found');
    }

    // Find both projects
    const fromProject = toDoList.findProject(
      ProjectName.of(input.fromProjectName)
    );
    const toProject = toDoList.findProject(
      ProjectName.of(input.toProjectName)
    );

    if (!fromProject || !toProject) {
      return CqrsOutput.create()
        .fail()
        .setMessage('Source or target project not found');
    }

    // Perform transfer
    const task = fromProject.removeTask(TaskId.of(input.taskId));
    toProject.addTask(task.getDescription());

    // Persist
    await this.toDoListRepository.save(toDoList);

    return CqrsOutput.create().succeed();
  }
}
```

---

## Anti-Patterns to Avoid

### 1. Anemic Domain Model
```typescript
// WRONG - Data without behavior
class Project {
  id: string;
  name: string;
  tasks: Task[];
}

// Behavior in separate service
class ProjectService {
  createProject(name: string): Project { ... }
  renameProject(project: Project, name: string): void { ... }
  addTask(project: Project, task: Task): void { ... }
}
```

### 2. Anemic Value Objects
```typescript
// WRONG - Value object without validation
record Email(string value) {
  // No validation - allows invalid emails
}
```

### 3. Large Aggregates
```typescript
// WRONG - Aggregate with too many objects
class EnterpriseSystem implements Entity<SystemId> {
  private departments: Department[];  // Many
  private employees: Employee[];      // Many
  private projects: Project[];        // Many
  private offices: Office[];          // Many
  // ...
}
```

### 4. Public Setters on Entities
```typescript
// WRONG - Exposing state modification
class Project {
  public id: string;
  public name: string;  // Can be modified directly
  public tasks: Task[]; // Can be mutated externally
}
```

---

## References and Further Reading

1. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
2. Vernon, Vaughn. "Implementing Domain-Driven Design." Addison-Wesley, 2013.
3. Vernon, Vaughn. "Domain-Driven Design Distilled." Addison-Wesley, 2016.
4. "Domain-Driven Design Community." https://domaindrivendesign.org/
5. "Effective Aggregate Design." Vaughn Vernon.
