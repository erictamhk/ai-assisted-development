# AI Agent Development Guidelines

## Purpose

This document provides instructions for AI agents to strictly follow all patterns, conventions, principles, and rules defined in **FOLDER_STRUCTURE.md** and **CONSTITUTION.md** when working on this project.

## Mandatory Pre-Work

### 1. Read Both Documents Before Coding
Before writing ANY code, the AI agent MUST:
- Read **FOLDER_STRUCTURE.md** completely
- Read **CONSTITUTION.md** completely
- Understand the Hexagonal Architecture pattern
- Identify all applicable design patterns for the task
- Reference specific rules when making implementation decisions

### 2. Verify Understanding
The AI agent must:
- Identify which architectural layer changes are needed
- Confirm which patterns apply to the task
- List all conventions that must be followed
- Acknowledge the dependency rules and constraints

## Architecture Adherence Rules

### Hexagonal Architecture - MANDATORY

#### Dependency Rule - STRICT
- **NEVER** violate inward dependency direction
- Adapters depend on Ports (interfaces), NOT implementations
- Use cases depend on Ports, NOT concrete adapters
- Domain layer has NO dependencies on outer layers
- Check every import statement against this rule

#### Layer Boundaries - STRICT
- **NEVER** skip layers (e.g., Controller directly calling Repository)
- **ALWAYS** communicate through interfaces (Ports)
- **NEVER** create dependencies between adapters
- **NEVER** use concrete classes from other layers

### Code Placement - MANDATORY (Vertical Slice Structure)

```
For Existing Feature (e.g., project, task, todolist):
NEW ENTITY → feature-name/domain/
NEW VALUE OBJECT → feature-name/domain/
NEW USE CASE INTERFACE → feature-name/usecase/port/in/
NEW USE CASE SERVICE → feature-name/usecase/service/
NEW CONTROLLER → feature-name/adapter/in/controller/console/ or web/
NEW PRESENTER INTERFACE → feature-name/usecase/port/out/
NEW PRESENTER IMPLEMENTATION → feature-name/adapter/out/presenter/
NEW REPOSITORY INTERFACE → feature-name/adapter/out/repository/
NEW DTO → feature-name/usecase/port/
NEW PO → feature-name/usecase/port/
NEW MAPPER → feature-name/usecase/port/

For New Feature:
Create new feature folder with complete structure:
src/main/java/.../tasks/feature-name/
├── domain/           # Entities and value objects
├── usecase/
│   ├── port/in/     # Input ports (use case interfaces)
│   ├── service/     # Use case implementations
│   └── port/       # DTOs, POs, Mappers
└── adapter/
    ├── in/          # Controllers (inbound adapters)
    └── out/        # Presenters, repositories (outbound adapters)

For Cross-Cutting Infrastructure:
NEW REPOSITORY IMPLEMENTATION → shared/adapter/out/repository/
NEW CONFIGURATION CLASS → shared/io/springboot/config/
NEW FRAMEWORK SETUP → shared/io/springboot/ or shared/io/standard/
```

## Pattern Implementation Checklist

### When Creating a New Feature

#### Step 1: Domain Layer (if new entities needed)
- [ ] Entity has identity field (implements Entity<Id>)
- [ ] Value Object is immutable (use `record`)
- [ ] Value Object has static factory `of()` method
- [ ] Validation in value object constructor
- [ ] Domain logic encapsulated in entities
- [ ] No framework dependencies in domain layer

#### Step 2: Input Port (Use Case Interface)
- [ ] Extends `Command<INPUT, OUTPUT>` for write operations
- [ ] Extends `Query<INPUT, OUTPUT>` for read operations
- [ ] Located in `usecase/port/in/`
- [ ] Interface name ends with `UseCase`
- [ ] Verb + noun naming
- [ ] Single responsibility

#### Step 3: Input Class
- [ ] Located in same package as interface
- [ ] Ends with `Input`
- [ ] Implements `Input` interface
- [ ] Public fields only
- [ ] No behavior, just data

#### Step 4: Output Class (if custom output needed)
- [ ] Located in same package as interface
- [ ] Ends with `Output`
- [ ] Extends `CqrsOutput<OUTPUT>` or `Output`
- [ ] Public fields for data
- [ ] No behavior, just data

#### Step 5: Use Case Service
- [ ] Located in `usecase/service/`
- [ ] Implements corresponding UseCase interface
- [ ] Class name ends with `Service`
- [ ] Constructor injection of dependencies
- [ ] Implements `execute()` method
- [ ] Returns `CqrsOutput` or custom Output
- [ ] Validates inputs before business logic
- [ ] Uses repository for persistence
- [ ] Handles errors and converts to output

#### Step 6: DTO (if data transfer needed)
- [ ] Located in `usecase/port/`
- [ ] Ends with `Dto`
- [ ] Public fields only
- [ ] No getters/setters
- [ ] No behavior

#### Step 7: PO (if persistence needed)
- [ ] Located in `usecase/port/out/`
- [ ] Ends with `Po`
- [ ] JPA annotations (`@Entity`, `@Table`, `@Id`, `@Column`)
- [ ] Default constructor
- [ ] Getters and setters
- [ ] No business logic

#### Step 8: Mapper (if conversion needed)
- [ ] Located in `usecase/port/`
- [ ] Ends with `Mapper`
- [ ] Static methods only
- [ ] Implements `toDto()`, `toDomain()`, `toPo()`
- [ ] Supports single and collection: `toDto(List<T>)`

#### Step 9: Output Port (Presenter Interface)
- [ ] Located in `usecase/port/out/`
- [ ] Ends with `Presenter`
- [ ] Single method: `present(Dto data)`
- [ ] No implementation details

#### Step 10: Presenter Implementation
- [ ] Located in `adapter/out/presenter/`
- [ ] Ends with `XxxConsolePresenter` or `XxxWebPresenter`
- [ ] Implements Presenter interface
- [ ] Constructor injection of output medium (PrintWriter, etc.)
- [ ] Formats and outputs data
- [ ] No business logic

#### Step 11: Controller
- [ ] Located in `adapter/in/controller/console/` or `web/`
- [ ] Ends with `XxxConsoleController` or `XxxWebController`
- [ ] Implements `ConsoleController` or appropriate interface
- [ ] Constructor injection of use cases
- [ ] Implements `execute()` method
- [ ] Creates input objects from request
- [ ] Calls use case
- [ ] Returns response object
- [ ] No business logic

#### Step 12: Dependency Configuration
- [ ] Add bean method in appropriate config class
- [ ] `RepositoryInjection` for repositories
- [ ] `UseCaseInjection` for use cases
- [ ] Constructor injection pattern
- [ ] Inject interfaces, not implementations

#### Step 13: Registration (if console command)
- [ ] Register controller in `ConsoleControllerExecutor`
- [ ] Map command string to controller
- [ ] Test command execution

## Naming Conventions - STRICT ENFORCEMENT

### Type Naming
```
Entity:              Project, Task, ToDoList
Value Object:         ProjectName, TaskId, ToDoListId (use record)
Read-Only:            ReadOnlyProject, ReadOnlyTask
Use Case Interface:   AddTaskUseCase, ShowUseCase, SetDoneUseCase
Use Case Service:     AddTaskService, ShowService, SetDoneService
Controller:           AddConsoleController, ShowConsoleController
Presenter Interface:  ShowPresenter, HelpPresenter
Presenter Impl:       ShowConsolePresenter, HelpWebPresenter
DTO:                  TaskDto, ProjectDto, ToDoListDto
PO:                   TaskPo, ProjectPo, ToDoListPo
Mapper:               TaskMapper, ProjectMapper, ToDoListMapper
Input:                AddTaskInput, ShowInput, SetDoneInput
Output:               ShowOutput, TodayOutput, HelpOutput
```

### Method Naming
```
Getters:              getName(), getId(), getDescription()
Boolean Getters:       isDone(), isActive()
Setters:              setName(), setDone() (entities only)
Actions:              addProject(), setDone(), deleteTask()
Queries:              getProjects(), containTask(), findXxx()
Factory:              of()
Mappers:              toDto(), toDomain(), toPo()
```

### Field Naming
```
Private:              private String name;
Final:                private final TaskId id;
Descriptive:          private List<Project> projects;
```

## Code Style Rules - MANDATORY

### Immutability
- Value objects MUST use `record` keyword
- Value objects MUST be immutable
- Entity identity fields MUST be final
- Return defensive copies or unmodifiable collections

### Visibility
- Fields: private by default
- Methods: public for API, private for internal
- Classes: public for API, package-private for internal

### Constructor Injection
```java
private final Dependency dependency;

public Service(Dependency dependency) {
    this.dependency = dependency;
}
```

### Factory Methods
```java
public static ProjectName of(String name) {
    return new ProjectName(name);
}
```

### Stream API Usage
```java
// Preferred
projects.stream()
    .filter(p -> p.getName().equals(name))
    .map(ProjectMapper::toDto)
    .toList();

// Avoid old-style loops
```

### Optional Usage
```java
public Optional<Task> getTask(TaskId id) {
    return tasks.stream()
        .filter(t -> t.getId().equals(id))
        .findFirst();
}

// Use map/filter instead of get()
```

### Error Handling
```java
// Use CqrsOutput for use cases
return CqrsOutput.create()
    .fail()
    .setMessage(format("Could not find project '%s'", name));

// Domain throws exceptions
if (invalid) throw new RuntimeException("Invalid state");
```

## Validation Rules - MANDATORY

### Value Objects
- Validate in constructor/compact constructor
- Throw exceptions for invalid values
- Example: `TaskId` validates no spaces, no special chars

### Use Cases
- Validate inputs before business logic
- Check for nulls
- Use contract libraries: `require()`, `reject()`
- Return appropriate error messages

### Domain Entities
- Enforce invariants in methods
- Validate state changes
- Throw exceptions for invalid state

## Testing Requirements - MANDATORY

### Test Location
```
src/test/java/.../tasks/feature-name/domain/XxxTest.java
src/test/java/.../tasks/feature-name/usecase/XxxUseCaseTest.java
```

### Test Structure
```java
@Test
public void descriptive_method_name_with_underscores() {
    // Arrange
    var repository = new InMemoryRepository(...);

    // Act
    var result = useCase.execute(input);

    // Assert
    assertEquals(ExitCode.SUCCESS, result.getExitCode());
}
```

### Test Naming
- Describe behavior, not implementation
- Use underscores for readability
- One test per scenario
- Example: `add_task_with_duplicated_id_returns_error`

## Common Patterns - REFERENCE

### Entity Pattern
```java
public class Project implements Entity<ProjectName> {
    private final ProjectName name;
    private final List<Task> tasks;

    public Project(ProjectName name) {
        this.name = name;
        this.tasks = new ArrayList<>();
    }

    @Override
    public ProjectName getId() {
        return name;
    }
}
```

### Value Object Pattern
```java
public record ProjectName(String value) implements ValueObject {
    public ProjectName {
        // Validation
        if (value == null || value.isEmpty())
            throw new RuntimeException("Name cannot be empty");
    }

    public static ProjectName of(String name) {
        return new ProjectName(name);
    }
}
```

### Read-Only Pattern
```java
public class ReadOnlyProject extends Project {
    public ReadOnlyProject(Project real) {
        super(real.getName(), real.getTasks());
    }

    @Override
    public void setTaskDone(TaskId id, boolean done) {
        throw new UnsupportedOperationException("Read only");
    }
}
```

### Use Case Pattern
```java
public interface AddTaskUseCase extends Command<AddTaskInput, CqrsOutput> {
}

public class AddTaskService implements AddTaskUseCase {
    private final ToDoListRepository repository;

    public AddTaskService(ToDoListRepository repository) {
        this.repository = repository;
    }

    @Override
    public CqrsOutput execute(AddTaskInput input) {
        var toDoList = repository.findById(ToDoListId.of(input.toDoListId)).get();
        toDoList.addTask(...);
        repository.save(toDoList);
        return CqrsOutput.create().succeed();
    }
}
```

### Controller Pattern
```java
public class AddConsoleController implements ConsoleController {
    private final AddTaskUseCase useCase;

    public AddConsoleController(AddTaskUseCase useCase) {
        this.useCase = useCase;
    }

    @Override
    public Response<CqrsOutput> execute(Request request) {
        AddTaskInput input = new AddTaskInput();
        input.projectName = request.getArg(0);
        input.description = request.getArg(1);
        return Response.of(useCase.execute(input));
    }
}
```

### Mapper Pattern
```java
public class TaskMapper {
    public static TaskDto toDto(Task task) {
        TaskDto dto = new TaskDto();
        dto.id = task.getId().value();
        dto.description = task.getDescription();
        return dto;
    }

    public static Task toDomain(TaskPo po) {
        return new Task(
            TaskId.of(po.getId()),
            po.getDescription(),
            po.getDone()
        );
    }

    public static TaskPo toPo(Task task) {
        return new TaskPo(
            task.getId().value(),
            task.getDescription(),
            task.isDone()
        );
    }
}
```

### Presenter Pattern
```java
public interface ShowPresenter {
    void present(ToDoListDto dto);
}

public class ShowConsolePresenter implements ShowPresenter {
    private final PrintWriter out;

    public ShowConsolePresenter(PrintWriter out) {
        this.out = out;
    }

    @Override
    public void present(ToDoListDto dto) {
        for (ProjectDto project : dto.projectDtos) {
            out.println(project.name);
        }
    }
}
```

## Decision Tree for AI Agents

### Question 0: Which feature does this belong to?
- **Project Management** → `project/` feature folder
- **Task Management** → `task/` feature folder
- **ToDoList Operations** → `todolist/` feature folder (aggregate root)
- **New Business Capability** → Create new feature folder
- **Infrastructure/Configuration** → `shared/` folder

### Question 1: What type of code am I creating?
- **New Entity/Value Object** → Feature domain/ → Follow Entity/VO Pattern
- **New Business Operation** → Feature usecase/ → Follow Use Case Pattern
- **New User Interface** → Feature adapter/ → Follow Controller Pattern
- **New Data Access** → Feature adapter/ or shared/ → Follow Repository Pattern
- **New Output Format** → Feature adapter/ → Follow Presenter Pattern

### Question 2: Does this modify state?
- **Yes** → Extend `Command<INPUT, OUTPUT>`
- **No** → Extend `Query<INPUT, OUTPUT>`

### Question 3: Which layer depends on which?
- **Domain Layer** → No dependencies
- **Use Case Layer** → Depends on Domain Layer + Ports
- **Adapter Layer** → Depends on Ports (interfaces only)
- **NEVER** → Adapter depends on another adapter
- **NEVER** → Domain depends on anything
- **NEVER** → Direct feature-to-feature dependencies (use aggregate root)

### Question 4: What naming convention applies?
- **Entity** → Singular noun
- **Value Object** → Identity name + `record`
- **Use Case** → Verb + noun + `UseCase`
- **Service** → Noun + `Service`
- **Controller** → Verb + `ConsoleController`/`WebController`
- **DTO** → Noun + `Dto`
- **PO** → Noun + `Po`

### Question 5: How to handle validation?
- **Value Objects** → Validate in constructor
- **Use Cases** → Validate inputs, return error output
- **Domain Entities** → Validate state, throw exceptions
- **Controllers** → Validate request structure

### Question 6: When to create a new feature?
Create new feature folder when:
- New business domain/capability unrelated to existing features
- Has its own entities, value objects, and business rules
- Needs independent lifecycle and deployment
- Multiple use cases that form a cohesive business concept
- Example: Adding "User", "Team", or "Workflow" capabilities

Use existing feature when:
- Extending functionality of current entities
- Adding new use cases for existing domain
- Modifying existing behavior within feature boundaries
- Example: Adding "archive project" goes in `project/` feature

## Vertical Slice Dependency Rules

### Feature Isolation - MANDATORY
- **NEVER** create direct dependencies between features (e.g., task/ → project/)
- **ALWAYS** communicate through aggregate root (todolist/) if needed
- **NEVER** import domain classes from another feature
- **NEVER** share use case implementations between features

### Feature Communication Rules
```
✓ CORRECT: task/ uses todolist/ repository interface
✓ CORRECT: todolist/ manages task and project entities
✗ WRONG: task/ directly calls project/ use case
✗ WRONG: task/ imports Project entity from project/ feature
✗ WRONG: Use case in task/ depends on ProjectRepository from project/
```

### Shared Infrastructure
- Repository implementations go in `shared/adapter/out/repository/`
- Configuration classes go in `shared/io/springboot/config/`
- Common interfaces (ConsoleController, Request, Response) in `todolist/adapter/`
- Framework setup in `shared/io/springboot/` or `shared/io/standard/`

### Cross-Feature Collaboration
- Through aggregate root interfaces only
- Via shared repository interfaces in `shared/`
- Domain events (if implemented) for loose coupling
- NEVER direct feature-to-feature method calls

## Prohibited Actions - NEVER DO THESE

1. **NEVER** skip layers (Controller → Repository directly)
2. **NEVER** depend on concrete classes outside your layer
3. **NEVER** use mutable value objects
4. **NEVER** expose entity internals via getters
5. **NEVER** put business logic in controllers
6. **NEVER** put business logic in presenters
7. **NEVER** use `null` returns (use Optional)
8. **NEVER** use getters on entities to modify state externally
9. **NEVER** create circular dependencies
10. **NEVER** add framework dependencies to domain layer
11. **NEVER** use `public` fields on entities (use private with methods)
12. **NEVER** skip validation in value objects
13. **NEVER** return mutable collections from domain objects
14. **NEVER** use hardcoded values (use configuration)
15. **NEVER** create generic exceptions (use specific types)
16. **NEVER** create direct dependencies between features (e.g., task/ → project/)
17. **NEVER** import domain classes from another feature
18. **NEVER** share use case implementations between features
19. **NEVER** place feature-specific code in shared/ folder
20. **NEVER** skip creating feature folders for new business capabilities

## Verification Checklist

Before submitting code, verify:

### Architecture
- [ ] Dependencies point inward
- [ ] No layer violations
- [ ] All communication through interfaces
- [ ] No circular dependencies

### Patterns
- [ ] Correct pattern applied for use case
- [ ] Proper interface segregation
- [ ] Dependency inversion followed
- [ ] Single responsibility principle applied

### Naming
- [ ] All names follow conventions
- [ ] Descriptive and clear
- [ ] No abbreviations (except common ones)
- [ ] Consistent with existing code

### Code Quality
- [ ] No code duplication
- [ ] Short, focused methods
- [ ] Clear, readable code
- [ ] Proper encapsulation

### Testing
- [ ] Tests written for new code
- [ ] Test names describe behavior
- [ ] Tests isolated and independent
- [ ] High coverage achieved

### Documentation
- [ ] Self-documenting code
- [ ] Clear intent
- [ ] No unnecessary comments
- [ ] Consistent formatting

## Error Recovery

If an AI agent makes a mistake:

1. **Identify the violation** - Which rule/pattern was broken?
2. **Understand the fix** - How should it be implemented correctly?
3. **Reference the documents** - Which section in CONSTITUTION.md applies?
4. **Implement the correction** - Apply the fix following all rules
5. **Verify the change** - Run through the checklist again

## Examples of Correct vs Incorrect

### Correct
```java
// Value object with validation
public record TaskId(String value) implements ValueObject {
    public TaskId {
        if (value == null || value.isEmpty())
            throw new RuntimeException("Task ID cannot be empty");
    }
}
```

### Incorrect
```java
// Mutable value object (VIOLATION)
public class TaskId {
    public String value;  // Public field, not immutable
}
```

### Correct
```java
// Use case following pattern
public interface AddTaskUseCase extends Command<AddTaskInput, CqrsOutput> {
}

public class AddTaskService implements AddTaskUseCase {
    private final ToDoListRepository repository;

    public AddTaskService(ToDoListRepository repository) {
        this.repository = repository;
    }
}
```

### Incorrect
```java
// Direct controller to repository (VIOLATION)
public class AddConsoleController {
    public Response execute(Request request) {
        repository.save(...);  // Skips use case layer
    }
}
```

### Correct
```java
// Read-only interface
public class ReadOnlyProject extends Project {
    @Override
    public void setTaskDone(TaskId id, boolean done) {
        throw new UnsupportedOperationException("Read only");
    }
}
```

### Incorrect
```java
// Exposing mutable collection (VIOLATION)
public class Project {
    public List<Task> tasks;  // Public, mutable
}
```

## Summary

The AI agent MUST:

1. **Read** both FOLDER_STRUCTURE.md and CONSTITUTION.md before coding
2. **Follow** Hexagonal Architecture dependency rules strictly
3. **Organize** code using vertical slice (feature-based) structure
4. **Apply** correct patterns for each use case
5. **Use** proper naming conventions for all types
6. **Implement** validation at appropriate layers
7. **Write** tests following conventions
8. **Verify** code against the checklist before submission
9. **Never** violate any prohibited action

**Vertical Slice Structure Compliance:**
- All feature code co-located in feature-specific folders
- No direct dependencies between features
- Use aggregate root (todolist/) for cross-feature communication
- Shared infrastructure in `shared/` folder only
- Each feature is self-contained with domain, use cases, and adapters

**Non-compliance with these guidelines will result in code that does not follow the project's architecture and must be corrected.**
