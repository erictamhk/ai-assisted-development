# Clean Architecture Refactoring Journey

## Overview

This document captures a proven 14-step refactoring journey from a monolithic application to Clean Architecture with Domain-Driven Design. Each step builds upon the previous, maintaining working code throughout the transformation.

**Project**: E-Commerce Order Management System  
**Language**: TypeScript  
**Framework**: Node.js with Express  
**Testing**: Jest

---

## Architecture Evolution Summary

### Final Architecture Structure

```
com.example.orders/
├── entity/                          # Domain Layer
│   ├── Order.ts                     # Aggregate Root
│   ├── Customer.ts                  # Entity
│   ├── Item.ts                      # Entity
│   ├── CustomerName.ts              # Value Object
│   ├── ItemId.ts                    # Value Object
│   ├── OrderId.ts                   # Value Object
│   ├── ReadOnlyCustomer.ts          # Interface
│   └── ReadOnlyItem.ts              # Interface
│
├── usecase/                        # Application Layer
│   ├── AddCustomerUseCase/          # Command
│   ├── AddItemUseCase/              # Command
│   ├── RemoveItemUseCase/           # Command
│   ├── SubmitOrderUseCase/          # Command
│   ├── ShowOrderUseCase/            # Query
│   ├── OrderHistoryUseCase/         # Query
│   ├── ErrorUseCase/                # Query
│   ├── port/                        # Output Port (Mappers)
│   └── TestUtil.ts
│
├── adapter/
│   ├── in/
│   │   ├── controller/
│   │   │   ├── console/           # Console Controllers
│   │   │   │   ├── ConsoleControllerExecutor.ts
│   │   │   │   ├── OrderCommands.ts
│   │   │   │   ├── Request.ts
│   │   │   │   ├── Response.ts
│   │   │   │   ├── AddCustomerConsoleController.ts
│   │   │   │   ├── AddItemConsoleController.ts
│   │   │   │   └── ...
│   │   │   └── web/               # Web Controllers (REST)
│   │   │       ├── AddCustomerController.ts
│   │   │       ├── AddItemController.ts
│   │   │       ├── RemoveItemController.ts
│   │   │       ├── SubmitOrderController.ts
│   │   │       └── ShowOrderController.ts
│   └── out/
│       ├── presenter/              # Presenters
│       │   ├── ShowConsolePresenter.ts
│       │   ├── OrderHistoryConsolePresenter.ts
│       │   └── ShowWebPresenter.ts
│       └── repository/             # Repositories
│           ├── OrderCrudRepository.ts
│           ├── OrderInMemoryRepository.ts
│           ├── OrderCrudRepositoryPeer.ts
│           └── OrderInMemoryRepositoryPeer.ts
│
└── io/
    ├── standard/
    │   └── OrderApp.ts             # Console Entry Point
    └── express/
        ├── OrderExpressApp.ts
        └── config/
            ├── UseCaseInjection.ts
            ├── RepositoryInjection.ts
            └── OrderDataSourceConfiguration.ts
```

---

## Step-by-Step Refactoring Guide

### Step 1: Initial Foundation Setup

**Focus**: Test structure and error handling setup

**Changes**:
- Configure JDK 21, JUnit 5
- Establish test structure for help and error commands
- Define error handling messages for add project, add task, check commands
- Set up dependencies (ezddd, ezcqrs, uContract, ezspec)

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

3. Apply DDD Tactical Design:
    - Order → Aggregate Root
    - Customer, Item → Entities
    - CustomerName, ItemId, OrderId → Value Objects

4. Remove Primitive Obsession:
    - Replace `Map<String, List<Item>>` with `Items` class
    - Replace `Map<CustomerName, List<Item>>` with `Customer` class
    - Create `CustomerName` record

5. Establish Ubiquitous Language:
    - Rename `ItemCollection` to `Order`
    - Rename methods to meaningful domain terms

**Patterns Applied**:
- Layered Architecture
- Value Object
- Entity
- Aggregate Root

**Key Insight**: Aggregate boundaries and invariant enforcement are established early

---

### Step 3: Extract Classes from Monolith

**Focus**: Find more classes in entity package by extracting command/query logic

**Changes** (using "Extract Class from Private Method" pattern):
1. Extract Order::show → Show class
2. Extract Order::add → Add class
3. Extract Order::remove → Remove class
4. Extract Order::submit → Submit class
5. Extract Order::error → Error class
6. Extract Order::execute → Execute class

7. Clean entity package:
   - Move classes with external/IO references to usecase package

**Patterns Applied**:
- Extract Class
- Single Responsibility Principle
- Method Object

**Key Insight**: Each extracted class represents a potential Use Case

---

### Step 4: Form Commands and Queries

**Focus**: Implement Use Case pattern with Input/Output

**Command Use Cases**:
1. AddCustomerUseCase:
    - AddCustomerInput
    - AddCustomerService
    - OrderRepository
    - OrderInMemoryRepository

2. AddItemUseCase:
    - AddItemInput
    - AddItemService
    - Inline Add to Execute

3. SubmitOrderUseCase:
    - SubmitOrderInput
    - SubmitOrderService

**Query Use Cases**:
1. ShowOrderUseCase:
    - ShowInput, ShowOutput
    - ShowService, ShowPresenter
    - ShowConsolePresenter
    - OrderDto, CustomerDto, ItemDto
    - OrderMapper, CustomerMapper, ItemMapper

2. OrderHistoryUseCase:
    - OrderHistoryInput, OrderHistoryOutput
    - OrderHistoryService, OrderDto
    - OrderHistoryPresenter, OrderHistoryConsolePresenter

3. ErrorUseCase:
    - ErrorInput, ErrorService

**Patterns Applied**:
- Command Query Separation
- Input/Output Port Pattern
- Service Layer
- Data Mapper

---

### Step 5: Form Controllers

**Focus**: Extract and inject use case dependencies

**Changes**:
1. Move Execute to adapter.controller package, rename to OrderConsoleController
2. Inject ShowOrderUseCase and ShowPresenter
3. Inject AddCustomerUseCase
4. Inject AddItemUseCase
5. Inject SubmitOrderUseCase
6. Inject OrderHistoryUseCase
7. Inject ErrorUseCase
8. Remove unused Order from controller

**Patterns Applied**:
- Dependency Injection (Constructor Injection)
- Controller Pattern
- Single Responsibility

**Key Insight**: Controllers become thin, delegating all business logic to use cases

---

### Step 6: Form Main Component

**Focus**: Identify entry point and apply DI framework

**Changes**:
1. Move OrderCollection to io package, rename to OrderApp (Main Component)
2. Use OrderApp::main and ApplicationTest as DI frameworks manually
3. Add Express dependencies to package.json
4. Create OrderExpressApp
5. Create ExpressApplicationTest

**Patterns Applied**:
- Application Entry Point
- Dependency Injection Container
- Main Component Pattern

**Key Insight**: Main component orchestrates dependency wiring

---

### Step 7: Create Web Controllers

**Focus**: Support REST API alongside console

**Changes**:
1. Create AddCustomerController in adapter.controller.web
2. Move OrderController to adapter.controller.console
3. Create AddItemController
4. Create RemoveItemController
5. Create OrderHistoryController
6. Create ShowOrderController

**Patterns Applied**:
- Adapter Pattern
- Strategy Pattern (multiple controller implementations)
- Interface Segregation

**Key Insight**: Same use cases can be accessed via different adapters (console/web)

---

### Step 8: Add Persistent Objects

**Focus**: Introduce persistence layer

**Changes**:
1. Define OrderPoRepository interface
2. Implement OrderPo, CustomerPo, ItemPo
3. Write Mappers (OrderMapperTest, CustomerMapperTest, ItemMapperTest)
4. Apply Bridge Pattern:
   - Move OrderPoRepository to adapter.repository
   - Rename to OrderRepositoryPeer
   - Implement OrderInMemoryRepositoryPeer
5. Revise DI to inject repository peer

**Patterns Applied**:
- Bridge Pattern
- Data Mapper Pattern
- Repository Pattern
- Strategy Pattern (in-memory vs persistence)

**Key Insight**: Bridge pattern decouples abstraction (repository) from implementation (peer)

---

### Step 9: Use Relational Database via ORM

**Focus**: Switch from in-memory to persistent storage

**Changes**:
1. Add ORM annotations to OrderPo, CustomerPo, ItemPo
2. Create OrderCrudRepositoryPeer (TypeORM/Prisma)
3. Create OrderCrudRepository
4. Create OrderDataSourceConfiguration
5. Revise RepositoryInjection for Express

**Patterns Applied**:
- TypeORM/Prisma
- Repository Pattern
- Dependency Injection

**Key Insight**: Same repository interface, different implementations (in-memory/ORM)

---

### Step 10-11: Feature Expansion

**Focus**: Add new features using established patterns

**Step 10 - Add New Feature Shipment**:
1. TDD ShipOrderUseCase with ShipOrderUseCaseTest
2. Implement ShipOrderUseCase, ShipOrderInput, ShipOrderService
3. Revise Order to support shipment
4. Create OrderPo shipment field
5. Revise REST controller for shipment endpoint

**Step 11 - Add New Feature Customizable IDs**:
1. TDD AddItemUseCase with custom ID support
2. Revise AddItemInput, AddItemService
3. Implement ItemId value object
4. Revise REST controller for item2 endpoint

**Patterns Applied**:
- Test-Driven Development
- Value Object
- Open/Closed Principle (extend without modifying existing code)

**Key Insight**: New features integrate seamlessly with existing architecture

---

### Step 12-13: Controller Refinement

**Focus**: Improve API interface

**Step 12 - Add New Feature Cancellation**:
1. TDD CancelOrderUseCase
2. Implement CancelOrderUseCase, CancelOrderInput, CancelOrderService
3. Revise REST controller for cancel endpoint
4. Refactor adapter package: add in/out packages
5. Move controller to in, presenter and repository to out

**Step 13 - Add New Feature Order History**:
1. TDD OrderHistoryUseCase
2. Implement OrderHistoryUseCase, OrderHistoryInput, OrderHistoryOutput
3. Create OrderHistoryDto, OrderHistoryService
4. Revise REST controller and tests

**Patterns Applied**:
- Input/Output Port Naming Convention (in/out)
- RESTful resource naming
- Use Case reusability

---

### Step 14: Extract API Handlers

**Focus**: Decouple API handlers using executor pattern

**Changes**:
1. Define Request, Response, ApiController interfaces
2. Create OrderCommands enum (commands mapping)
3. Implement ApiControllerExecutor
4. Extract individual controllers:
   - AddCustomerApiController
   - AddItemApiController
   - RemoveItemApiController
   - SubmitOrderApiController
   - ShowOrderApiController
   - OrderHistoryApiController
   - ErrorApiController

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

---

## Dependency Evolution

### Step 1-4
```
Order (monolith) → direct dependencies
```

### Step 5-6
```
OrderConsoleController → UseCase → Service → Repository → Order (Aggregate)
```

### Step 7-9
```
Controller (in) → UseCase (usecase) → Repository (out) → PO (out) → DB
Presenter (out)
```

### Step 14
```
ApiControllerExecutor → Request/Response → ApiController (in) → UseCase
                                                              ↓
                                                  Presenter/Repository (out)
```

---

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

---

## Key Architectural Principles Demonstrated

1. **Dependency Inversion**: High-level modules (use cases) don't depend on low-level modules (frameworks)

2. **Single Responsibility**: Each class has one reason to change

3. **Open/Closed**: Open for extension, closed for modification

4. **Interface Segregation**: Small, focused interfaces

5. **Aggregate Boundary**: Aggregate root controls access to entities

6. **Ubiquitous Language**: Code expresses domain concepts clearly

7. **Testability**: Every layer can be tested in isolation

---

## Lessons Learned

1. **Start with tests**: Each refactoring step is verified by existing tests

2. **Extract by responsibility**: When code does multiple things, extract until single responsibility

3. **Bridge for abstractions**: Use bridge pattern when you need multiple implementations

4. **Controllers are adapters**: They're entry points, not business logic

5. **Use cases are stable**: Once use cases are defined, they're rarely changed

6. **Value objects add safety**: Replace primitives with domain-specific types

7. **Inject dependencies**: Makes testing and future changes easier

---

## References

- Original repository: `refactor-to-ca` (14 git branches: main, step1-step14)
- Related documentation: Clean Architecture, Domain-Driven Design, Test-Driven Development

---

## Architecture Patterns Quick Reference

| Pattern | When to Use | Key Benefit |
|---------|-------------|-------------|
| **Layered Architecture** | Small-medium applications | Clear separation by technical concern |
| **Ports & Adapters** | Framework-independent design | Technology independence |
| **Clean Architecture** | Enterprise applications | Business logic isolation |
| **Vertical Slice** | Large, feature-rich apps | Feature isolation and team scaling |
| **Aggregate Root** | Domain-driven systems | Consistency boundary enforcement |
| **CQRS** | Read/Write separation | Scalability and flexibility |
| **Repository Pattern** | Data access abstraction | Persistence technology independence |
| **Dependency Injection** | All architectures | Loose coupling and testability |

---

## Architecture Decision Checklist

### Layer Compliance
- [ ] Domain layer has no dependencies on outer layers
- [ ] Application layer depends only on domain and ports
- [ ] Adapter layer implements ports defined in inner layers
- [ ] No framework dependencies in domain layer

### Dependency Direction
- [ ] All source code dependencies point inward
- [ ] Ports defined in inner layers, implemented in outer
- [ ] No circular dependencies between layers
- [ ] No adapter-to-adapter direct dependencies

### Code Organization
- [ ] Related code co-located (feature slice or layer)
- [ ] Clear boundaries between features/layers
- [ ] Cross-cutting concerns in shared/ folder
- [ ] No feature-specific code in shared/

### Design Patterns
- [ ] Entities contain behavior (not anemic)
- [ ] Value objects are immutable and self-validating
- [ ] Use cases follow Command/Query separation
- [ ] Repositories are abstractions, not implementations

### Testing
- [ ] Domain logic tested without external dependencies
- [ ] Use cases tested with mock ports
- [ ] Adapters tested with real external systems
- [ ] Integration tests verify layer interactions
