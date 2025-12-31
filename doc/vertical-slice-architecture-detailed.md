# Vertical Slice Architecture

## Core Concept

Vertical Slice Architecture is an organizational pattern that structures code by business features (vertical slices) rather than by technical layers (horizontal layers). Each vertical slice contains all the code needed to implement a specific feature, from the user interface down to the database, while maintaining architectural boundaries within the slice.

The key insight is that features are the primary unit of change in software systems. When requirements change, they typically change within a single feature, not across all entities or all controllers. By organizing code around features, each slice becomes a self-contained unit that can be developed, tested, and deployed independently.

This approach combines the benefits of modular architecture with the simplicity of feature-focused development. Each slice maintains clean separation from others, but within a slice, all related code is co-located for easy navigation and understanding.

Vertical Slice Architecture is particularly valuable in large, complex applications where purely layered architectures can lead to code that is difficult to navigate and modify. By keeping feature-related code together, developers can work more efficiently and with better context.

---

## Vertical Slice vs Layer-Based Structure

```
TRADITIONAL LAYER-BASED STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

src/
├── domain/
│   ├── Project.java
│   ├── Task.java
│   └── ToDoList.java
├── usecase/
│   ├── AddProjectUseCase.java
│   ├── AddTaskUseCase.java
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
├── project/              # Feature: Project management
│   ├── domain/           # Domain objects for this feature
│   ├── usecase/          # Use cases for this feature
│   └── adapter/          # Adapters for this feature
│
├── task/                 # Feature: Task management
│   ├── domain/
│   ├── usecase/
│   └── adapter/
│
├── todolist/             # Feature: ToDoList aggregate root
│   ├── domain/
│   ├── usecase/
│   └── adapter/
│
└── shared/               # Cross-cutting concerns
    ├── adapter/
    └── io/
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Benefit: All code for a feature is in one place
         Clear feature boundaries
         Easy to navigate and modify
```

---

## Vertical Slice Structure

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
   todolist/ → domain + usecase + adapter

✓ CORRECT: Cross-feature communication through aggregate root
   task/ → todolist/ (aggregate root) → project/
   (task communicates with project through todolist)

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

The aggregate root feature (todolist/) serves as the entry point for managing other features:

```
AGGREGATE ROOT COMMUNICATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

todolist/ (Aggregate Root)
    │
    ├── Manages project/ entities
    │   └── Controls consistency boundaries
    │
    ├── Manages task/ entities
    │   └── Ensures business invariants
    │
    └── Provides repository interfaces
        → used by all features

task/ Feature
    ├── Depends on todolist/ repository interface
    ├── NO direct dependency on project/
    └── Imports NO domain classes from project/

project/ Feature
    ├── Depends on todolist/ repository interface
    ├── NO direct dependency on task/
    └── Imports NO domain classes from task/

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

todolist/ (aggregate root)
    ↓ manages
project/ and task/
    → communicate through todolist/ ONLY
    → NO direct imports between features

task/ and project/
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
# In todolist/domain/ToDoList.java:
public NewFeature getNewFeature() {
    // If new feature is managed by ToDoList
}
```

---

## Cross-Cutting Infrastructure

Repository implementations and configuration classes go in `shared/`:

```bash
shared/
├── adapter/
│   └── out/
│       └── repository/
│           ├── ProjectCrudRepository.java
│           ├── TaskCrudRepository.java
│           ├── ToDoListCrudRepository.java
│           ├── ProjectInMemoryRepository.java
│           ├── TaskInMemoryRepository.java
│           └── ToDoListInMemoryRepository.java
│
└── io/
    ├── springboot/
    │   ├── Application.java
    │   └── config/
    │       ├── RepositoryInjection.java
    │       ├── UseCaseInjection.java
    │       ├── DataSourceConfig.java
    │       └── SecurityConfig.java
    │
    └── standard/
        └── Main.java
```

---

## Benefits of Vertical Slice Architecture

1. **Feature Independence**: Each feature can be developed, tested, and deployed independently
2. **Easy Navigation**: All code for a feature is in one directory
3. **Clear Boundaries**: Feature boundaries are explicit and enforced
4. **Better Scalability**: Adding new features doesn't clutter existing code
5. **Team Collaboration**: Different teams can work on different features with minimal conflicts
6. **Faster Development**: Context switching between related code is minimized
7. **Easier Refactoring**: Impact of changes is contained within feature boundaries
8. **Simpler Onboarding**: New developers can understand one feature at a time

---

## When to Use Vertical Slice

### Choose Vertical Slice When:
- Building medium to large applications
- Multiple teams working on different features
- Features have distinct business logic
- Clear domain boundaries exist
- Independent deployment is desired

### Consider Layer-Based When:
- Building small, simple applications
- Single developer or small team
- All features are tightly coupled
- Simpler navigation is preferred
- Rapid prototyping

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

---

## References and Further Reading

1. "Vertical Slice Architecture." Jimmy Bogard.
2. "Feature Folders." Rachel Appel.
3. "Modular monoliths." Simon Brown.
4. "Domain-Driven Design." Eric Evans. Addison-Wesley, 2004.
5. "Implementing Domain-Driven Design." Vaughn Vernon. Addison-Wesley, 2013.
