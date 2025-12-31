# Refactor-to-Clean Architecture Analysis

A comprehensive analysis of a real-world 14-step refactoring journey from a monolithic application to Clean Architecture with Domain-Driven Design.

## Overview

This repository (`refactor-to-ca`) demonstrates a methodical, incremental approach to refactoring legacy code into Clean Architecture. Each step builds upon the previous, maintaining working code throughout the transformation.

**Project**: Task List Application (ToDo List)
**Language**: Java 21
**Build Tool**: Maven
**Testing**: JUnit 5
**Framework**: Spring Boot 2.7.18
**Branches**: main, step1-step14

## Architecture Evolution Summary

### Final Architecture (Step 14)

```
tw.teddysoft.tasks/
├── entity/                          # Domain Layer
│   ├── ToDoList.java               # Aggregate Root
│   ├── Project.java                # Entity
│   ├── Task.java                   # Entity
│   ├── ProjectName.java            # Value Object
│   ├── TaskId.java                 # Value Object
│   ├── ToDoListId.java             # Value Object
│   ├── ReadOnlyProject.java        # Interface
│   └── ReadOnlyTask.java           # Interface
│
├── usecase/                        # Application Layer
│   ├── AddProjectUseCase/          # Command
│   ├── AddTaskUseCase/             # Command
│   ├── SetDoneUseCase/             # Command
│   ├── DeleteTaskUseCase/          # Command
│   ├── ShowUseCase/                # Query
│   ├── HelpUseCase/                # Query
│   ├── ErrorUseCase/               # Query
│   ├── DeadlineUseCase/            # Query
│   ├── TodayUseCase/               # Query
│   ├── ViewTaskUseCase/            # Query
│   ├── port/                       # Output Port (Mappers)
│   └── TestUtil.java
│
├── adapter/
│   ├── in/
│   │   ├── controller/
│   │   │   ├── console/           # Console Controllers
│   │   │   │   ├── ConsoleControllerExecutor.java
│   │   │   │   ├── ToDoListCommands.java
│   │   │   │   ├── Request.java
│   │   │   │   ├── Response.java
│   │   │   │   ├── AddProjectConsoleController.java
│   │   │   │   ├── AddTaskConsoleController.java
│   │   │   │   ├── ...
│   │   │   └── web/               # Web Controllers (REST)
│   │   │       ├── AddProjectController.java
│   │   │       ├── AddTaskController.java
│   │   │       ├── SetDoneController.java
│   │   │       ├── ShowController.java
│   │   │       └── HelpController.java
│   └── out/
│       ├── presenter/              # Presenters
│       │   ├── ShowConsolePresenter.java
│       │   ├── HelpConsolePresenter.java
│       │   ├── ViewTaskConsolePresenter.java
│       │   └── HelpWebPresenter.java
│       └── repository/             # Repositories
│           ├── ToDoListCrudRepository.java
│           ├── ToDoListInMemoryRepository.java
│           ├── ToDoListCrudRepositoryPeer.java
│           └── ToDoListInMemoryRepositoryPeer.java
│
└── io/
    ├── standard/
    │   └── ToDoListApp.java        # Console Entry Point
    └── springboot/
        ├── ToDoListSpringBootApp.java
        └── config/
            ├── UseCaseInjection.java
            ├── RepositoryInjection.java
            └── ToDoListDataSourceConfiguration.java
```

## Step-by-Step Analysis

### Step 1: Initial Commit

**Focus**: Foundation setup

**Changes**:
- JDK 21, JUnit 5 configuration
- Test structure for help and error commands
- Error handling messages for add project, add task, check commands
- Dependencies: ezddd, ezcqrs, uContract, ezspec

**Pattern**: Greenfield test setup

---

### Step 2: Establish Clean Architecture Layers

**Focus**: Layer creation and DDD foundation

**Changes**:
1. Create four-layer package structure:
   - `entity/` - Domain model
   - `usecase/` - Business logic
   - `adapter/` - Controllers, Presenters, Repositories
   - `io/` - Entry points

2. Move Task.java to entity package

3. DDD Tactical Design:
   - ToDoList → Aggregate Root
   - Project, Task → Entities
   - ProjectName, TaskId, ToDoListId → Value Objects

4. Remove Primitive Obsession:
   - Replace `Map<String, List<Task>>` with `Tasks` class
   - Replace `Map<ProjectName, List<Task>>` with `Project` class
   - Create `ProjectName` record

5. Establish Ubiquitous Language:
   - Rename `Tasks` to `ToDoList`
   - Rename methods to meaningful domain terms

**Patterns Applied**:
- Layered Architecture
- Value Object
- Entity
- Aggregate Root

**DDD Concepts Implemented**:
- Aggregate boundaries
- Invariant enforcement
- Read-only interfaces (ReadOnlyTask, ReadOnlyProject)

---

### Step 3: Find More Classes in Entity Package

**Focus**: Extract command/query logic from monolithic code

**Changes** (all using "Extract Class from Private Method" pattern):
1. Extract TaskList::show → Show class
2. Extract TaskList::add → Add class
3. Extract TaskList::setDone → SetDone class
4. Extract TaskList::help → Help class
5. Extract TaskList::error → Error class
6. Extract TaskList::execute → Execute class

7. Clean entity package:
   - Move classes with external/IO references to usecase package

**Patterns Applied**:
- Extract Class
- Single Responsibility Principle
- Method Object

**Key Insight**: Each extracted class represents a potential Use Case

---

### Step 4: Form Commands and Queries in Use Case Package

**Focus**: Implement Use Case pattern with Input/Output

**Changes**:

**Command Use Cases**:
1. AddProjectUseCase:
   - +AddProjectInput
   - +AddProjectService
   - ToDoListRepository
   - ToDoListInMemoryRepository

2. AddTaskUseCase:
   - AddTaskInput
   - AddTaskService
   - Inline Add to Execute

3. SetDoneUseCase:
   - SetDoneInput
   - SetDoneService

**Query Use Cases**:
1. ShowUseCase:
   - +ShowInput, +ShowOutput
   - +ShowService, +ShowPresenter
   - +ShowConsolePresenter
   - +ToDoListDot, +ProjectDot, +TaskDto
   - +ToDoListMapper, +ProjectMapper, +TaskMapper

2. HelpUseCase:
   - +HelpInput, +HelpOutput
   - +HelpService, +HelpDot
   - +HelpPresenter, +HelpConsolePresenter

3. ErrorUseCase:
   - +ErrorInput, +ErrorService

**Patterns Applied**:
- Command Query Separation
- Input/Output Port Pattern
- Service Layer
- Data Mapper

---

### Step 5: Form Controllers

**Focus**: Extract and inject use case dependencies

**Changes**:
1. Move Execute to adapter.controller package, rename to ToDoListConsoleController
2. Inject ShowUseCase and ShowPresenter
3. Inject AddProjectUseCase
4. Inject AddTaskUseCase
5. Inject SetDoneUseCase
6. Inject HelpUseCase
7. Inject ErrorUseCase
8. Remove unused ToDoList from controller

**Patterns Applied**:
- Dependency Injection (Constructor Injection)
- Controller Pattern
- Single Responsibility

**Key Insight**: Controllers become thin, delegating all business logic to use cases

---

### Step 6: Form Main Component

**Focus**: Identify entry point and apply DI framework

**Changes**:
1. Move TaskList to io package, rename to ToDoListApp (Main Component)
2. Use ToDoListApp::main and ApplicationTest as DI frameworks manually
3. Add SpringBoot dependencies to pom.xml
4. Create ToDoListSpringBootApp
5. Create SpringBootApplicationTest

**Patterns Applied**:
- Application Entry Point
- Dependency Injection Container
- Main Component Pattern

**Key Insight**: Main component orchestrates dependency wiring

---

### Step 7: Create Web Controllers

**Focus**: Support REST API alongside console

**Changes**:
1. Create AddProjectController in adapter.controller.web
2. Move ToDoListController to adapter.controller.console
3. Create AddTaskController
4. Create SetDoneController
5. Create HelpController
6. Create ShowController

**Patterns Applied**:
- Adapter Pattern
- Strategy Pattern (multiple controller implementations)
- Interface Segregation

**Key Insight**: Same use cases can be accessed via different adapters (console/web)

---

### Step 8: Add Persistent Objects

**Focus**: Introduce persistence layer

**Changes**:
1. Define ToDoListPoRepository interface
2. Implement ToDoListPo, ProjectPo, TaskPo
3. Write Mappers (ToDoListMapperTest, ProjectMapperTest, TaskMapperTest)
4. Apply Bridge Pattern:
   - Move ToDoListPoRepository to adapter.repository
   - Rename to ToDoListRepositoryPeer
   - Implement ToDoListInMemoryRepositoryPeer
5. Revise DI to inject repository peer

**Patterns Applied**:
- Bridge Pattern
- Data Mapper Pattern
- Repository Pattern
- Strategy Pattern (in-memory vs persistence)

**Key Insight**: Bridge pattern decouples abstraction (repository) from implementation (peer)

---

### Step 9: Use Relational Database via JPA

**Focus**: Switch from in-memory to persistent storage

**Changes**:
1. Add JPA annotations to ToDoListPo, ProjectPo, TaskPo
2. Create ToDoListCrudRepositoryPeer (Spring Data JPA)
3. Create ToDoListCrudRepository
4. Create ToDoListDataSourceConfiguration
5. Revise RepositoryInjection for SpringBoot

**Patterns Applied**:
- Spring Data JPA
- Repository Pattern
- Dependency Injection

**Key Insight**: Same repository interface, different implementations (in-memory/JPA)

---

### Step 10-11: Feature Expansion

**Focus**: Add new features using established patterns

**Step 10 - Add New Feature Deadlines**:
1. TDD DeadlineUseCase with DeadlineUseCaseTest
2. Implement DeadlineUseCase, DeadlineInput, DeadlineService
3. Revise Task to support deadline
4. Create TaskPo deadline field
5. Revise console controller for deadline command

**Step 11 - Add New Feature Customisable IDs**:
1. TDD AddTaskUseCase with custom ID support
2. Revise AddTaskInput, AddTaskService
3. Implement TaskId value object
4. Revise console controller for task2 command

**Patterns Applied**:
- Test-Driven Development
- Value Object
- Open/Closed Principle (extend without modifying existing code)

**Key Insight**: New features integrate seamlessly with existing architecture

---

### Step 12-13: Console Controller Refinement

**Focus**: Improve console interface

**Step 12 - Add New Feature Deletion**:
1. TDD DeleteTaskUseCase
2. Implement DeleteTaskUseCase, DeleteTaskInput, DeleteTaskService
3. Revise console controller for delete command
4. Refactor adapter package: add in/out packages
5. Move controller to in, presenter and repository to out

**Step 13 - Add New Feature View by Deadline**:
1. TDD ViewTaskUseCase
2. Implement ViewTaskUseCase, ViewTaskInput, ViewTaskOutput
3. Create ViewTaskDto, ViewTaskService
4. Revise console controller and tests

**Patterns Applied**:
- Input/Output Port Naming Convention (in/out)
- RESTful resource naming
- Use Case reusability

---

### Step 14: Extract Console Controllers

**Focus**: Decouple console controllers using executor pattern

**Changes**:
1. Define Request, Response, ConsoleController interfaces
2. Create ToDoListCommands enum (commands mapping)
3. Implement ConsoleControllerExecutor
4. Extract individual controllers:
   - AddProjectConsoleController
   - AddTaskConsoleController
   - CheckConsoleController
   - SetDoneConsoleController
   - DeleteConsoleController
   - ShowConsoleController
   - HelpConsoleController
   - DeadlineConsoleController
   - TodayConsoleController
   - ErrorConsoleController

**Patterns Applied**:
- Command Pattern
- Strategy Pattern
- Extensible Enum
- Template Method

**Key Insight**: Each command becomes a standalone controller, executor dispatches based on command type

---

## Design Pattern Usage Summary

| Pattern | Steps Applied | Purpose |
|---------|---------------|---------|
| Layered Architecture | 2 | Clear separation of concerns |
| Aggregate Root | 2 | Domain model encapsulation |
| Value Object | 2 | Type safety, domain modeling |
| Entity | 2 | Domain identity |
| Extract Class | 3 | Single responsibility |
| Command/Query Separation | 4 | Clear use case definition |
| Input/Output Port | 4 | Use case interface |
| Service Layer | 4 | Business logic orchestration |
| Data Mapper | 4, 8 | Object-relational mapping |
| Dependency Injection | 5, 6 | Loose coupling |
| Adapter Pattern | 7 | Interface adaptation |
| Bridge Pattern | 8 | Decouple abstraction from implementation |
| Repository Pattern | 8, 9 | Data access abstraction |
| Strategy Pattern | 8, 14 | Interchangeable algorithms |
| Command Pattern | 14 | Encapsulate request as object |
| Template Method | 7, 14 | Algorithm skeleton |

## Dependency Evolution

### Step 1-4
```
ToDoList (monolith) → direct dependencies
```

### Step 5-6
```
ToDoListConsoleController → UseCase → Service → Repository → ToDoList (Aggregate)
```

### Step 7-9
```
Controller (in) → UseCase (usecase) → Repository (out) → PO (out) → DB
Presenter (out)
```

### Step 14
```
ConsoleControllerExecutor → Request/Response → ConsoleController (in) → UseCase
                                                                  ↓
                                                      Presenter/Repository (out)
```

## Testing Strategy Evolution

### Step 1
- Basic unit tests for commands
- Error message validation

### Step 4
- Use Case Tests (TDD approach)
- Input/Output boundary testing
- Service layer testing

### Step 8
- Mapper Tests (domain ↔ PO)
- Repository Tests (with TestContainers or in-memory)

### Step 12-13
- Feature tests following TDD
- Integration tests for console/web

## Key Architectural Principles Demonstrated

1. **Dependency Inversion**: High-level modules (use cases) don't depend on low-level modules (frameworks)

2. **Single Responsibility**: Each class has one reason to change

3. **Open/Closed**: Open for extension, closed for modification

4. **Interface Segregation**: Small, focused interfaces

5. **Aggregate Boundary**: Aggregate root controls access to entities

6. **Ubiquitous Language**: Code expresses domain concepts clearly

7. **Testability**: Every layer can be tested in isolation

## Evaluation

| Criterion | Score | Notes |
|-----------|-------|-------|
| Architecture Clarity | 10/10 | Clear 4-layer structure |
| DDD Implementation | 9/10 | Full tactical patterns |
| Incremental Refactoring | 10/14 steps | Maintains working code |
| Test Coverage | 9/10 | TDD at each feature |
| Design Patterns | 10/10 | Appropriate pattern selection |
| Framework Independence | 8/10 | Some Spring coupling in io layer |
| Documentation | 8/10 | Commit messages explain rationale |

**Overall Score: 95/100**

## Lessons Learned

1. **Start with tests**: Each refactoring step is verified by existing tests

2. **Extract by responsibility**: When code does multiple things, extract until single responsibility

3. **Bridge for abstractions**: Use bridge pattern when you need multiple implementations

4. **Controllers are adapters**: They're entry points, not business logic

5. **Use cases are stable**: Once use cases are defined, they're rarely changed

6. **Value objects add safety**: Replace primitives with domain-specific types

7. **Inject dependencies**: Makes testing and future changes easier

## Comparison with ai-coding-exercise

| Aspect | ai-coding-exercise | refactor-to-ca |
|--------|-------------------|----------------|
| Approach | Template/Scaffold | Refactoring Journey |
| Framework | ezddd/ezcqrs | Plain DDD + Spring |
| Steps | Sub-agent workflows | 14 git steps |
| Focus | AI-assisted development | Manual refactoring |
| Feature Set | Event sourcing, CQRS | CRUD + queries |
| Complexity | Higher | Lower |
| Learning Value | Pattern templates | Refactoring process |

## Conclusion

This repository serves as an excellent reference for:
1. Understanding Clean Architecture evolution
2. Learning incremental refactoring techniques
3. Applying DDD tactical patterns
4. Implementing test-driven feature development
5. Managing dependencies across layers

The 14-step progression demonstrates that refactoring to Clean Architecture doesn't require a "big bang" rewrite—small, focused changes maintain working code while improving structure.
