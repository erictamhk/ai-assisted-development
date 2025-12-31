# File Organization

This document defines file and directory organization conventions for AI-assisted development, focusing on feature-based structure, DDD patterns, and clean architecture.

---

## Core Principle: Feature-Based Organization

Organize code by feature (vertical slice), not by type. Each feature should be self-contained with all related code grouped together.

```
✅ GOOD - Feature-based
src/
├── features/
│   ├── auth/
│   │   ├── domain/
│   │   ├── usecase/
│   │   ├── adapter/
│   │   └── auth.module.ts
│   └── orders/
│       ├── domain/
│       ├── usecase/
│       ├── adapter/
│       └── orders.module.ts

❌ BAD - Type-based
src/
├── controllers/
│   ├── auth-controller.ts
│   └── orders-controller.ts
├── services/
│   ├── auth-service.ts
│   └── orders-service.ts
├── models/
│   ├── user.model.ts
│   └── order.model.ts
```

---

## Standard Directory Structure

### Feature-Based Structure

```
src/
├── features/
│   └── {feature-name}/
│       ├── domain/
│       │   ├── entities/
│       │   │   └── {feature}-entity.ts
│       │   ├── value-objects/
│       │   │   └── {feature}-value-object.ts
│       │   ├── aggregates/
│       │   │   └── {feature}-aggregate.ts
│       │   ├── events/
│       │   │   └── {feature}-event.ts
│       │   └── domain-services/
│       │       └── {feature}-service.ts
│       ├── usecase/
│       │   ├── port/
│       │   │   ├── in/
│       │   │   │   ├── {verb}-{feature}.input.ts
│       │   │   │   └── {verb}-{feature}.usecase.ts
│       │   │   └── out/
│       │   │       ├── {feature}.dto.ts
│       │   │       └── {feature}-mapper.ts
│       │   └── service/
│       │       └── {verb}-{feature}.service.ts
│       ├── adapter/
│       │   ├── in/
│       │   │   ├── controller/
│       │   │   │   ├── {feature}-web.controller.ts
│       │   │   │   └── {feature}-console.controller.ts
│       │   │   └── presenter/
│       │   │       ├── {feature}-presenter.ts
│       │   │       └── {feature}-view-model.ts
│       │   └── out/
│       │       ├── repository/
│       │       │   └── {feature}-repository.ts
│       │       └── external/
│       │           └── {feature}-external-client.ts
│       ├── {feature}.module.ts
│       └── index.ts
├── shared/
│   ├── domain/
│   │   ├── entity.ts
│   │   ├── value-object.ts
│   │   └── aggregate-root.ts
│   ├── usecase/
│   │   └── port/
│   │       └── repository.interface.ts
│   └── utils/
│       └── {utility}.ts
├── config/
│   ├── app.config.ts
│   └── database.config.ts
├── infrastructure/
│   ├── database/
│   │   ├── connection.ts
│   │   └── {feature}-repository.impl.ts
│   └── messaging/
│       └── event-bus.ts
└── main.ts
```

---

## DDD Layer Organization

### Domain Layer

```
feature/domain/
├── entities/
│   └── {aggregate-name}.ts          # Main aggregate entity
├── value-objects/
│   ├── {name}.ts                    # Value objects for the feature
│   └── index.ts
├── aggregates/
│   └── {aggregate-name}.ts          # Aggregate root
├── events/
│   ├── {feature}-event.ts           # Domain events
│   └── {feature}-event-handler.ts
├── repository-interface/
│   └── {feature}-repository.interface.ts
└── domain-services/
    └── {feature}-domain-service.ts
```

### Use Case Layer

```
feature/usecase/
├── port/
│   ├── in/
│   │   ├── {verb}-{feature}.input.ts       # Input DTO
│   │   └── {verb}-{feature}.usecase.ts     # Use case interface
│   └── out/
│       ├── {feature}.dto.ts                # Output DTO
│       ├── {feature}.mapper.ts             # Mapper
│       └── {feature}-repository.interface.ts
└── service/
    └── {verb}-{feature}.service.ts         # Use case implementation
```

### Adapter Layer

```
feature/adapter/
├── in/
│   ├── controller/
│   │   ├── {feature}-web.controller.ts     # HTTP/REST
│   │   ├── {feature}-graphql.resolver.ts   # GraphQL
│   │   └── {feature}-console.controller.ts # CLI
│   ├── presenter/
│   │   ├── {feature}-presenter.ts          # Presenter interface
│   │   └── {feature}-console.presenter.ts  # Console implementation
│   └── dto/
│       ├── {feature}-request.dto.ts
│       └── {feature}-response.dto.ts
└── out/
    ├── repository/
    │   └── {feature}-repository.ts          # Repository implementation
    ├── persistence/
    │   └── {feature}-entity.mapper.ts
    └── external/
        └── {feature}-external-client.ts
```

---

## Shared Layer

```
shared/
├── domain/
│   ├── entity.ts                    # Base Entity class
│   ├── value-object.ts              # Base ValueObject class
│   ├── aggregate-root.ts            # Base AggregateRoot class
│   ├── domain-event.ts              # Base DomainEvent class
│   └── result.ts                    # Result type
├── usecase/
│   ├── input.ts                     # Base Input class
│   └── output.ts                    # Base Output class
├── utils/
│   ├── validation.ts                # Validation helpers
│   ├── date.ts                      # Date utilities
│   └── string.ts                    # String utilities
├── errors/
│   ├── domain-error.ts              # Base DomainError
│   └── validation-error.ts          # Validation errors
└── types/
    └── common.ts                    # Shared types
```

---

## File Naming Conventions

### General Rules

| Pattern | Example | Use For |
|---------|---------|---------|
| `{feature}.ts` | `user.ts` | Main feature exports |
| `{feature}.module.ts` | `order.module.ts` | Feature module/entry point |
| `{verb}-{feature}.ts` | `create-user.ts` | Single-action files |
| `{feature}.interface.ts` | `user-repository.interface.ts` | Interface files |
| `{feature}.type.ts` | `user.type.ts` | Type definitions |
| `{feature}.constant.ts` | `user-status.constant.ts` | Constants |
| `{feature}.util.ts` | `date.util.ts` | Utility functions |

### DDD-Specific Naming

| Pattern | Example | Layer |
|---------|---------|-------|
| `{feature}-entity.ts` | `order-entity.ts` | domain/entities |
| `{feature}-vo.ts` | `money-vo.ts` | domain/value-objects |
| `{feature}-aggregate.ts` | `order-aggregate.ts` | domain/aggregates |
| `{feature}-event.ts` | `order-event.ts` | domain/events |
| `{verb}-{feature}.usecase.ts` | `create-order.usecase.ts` | usecase/port/in |
| `{verb}-{feature}.service.ts` | `create-order.service.ts` | usecase/service |
| `{feature}-repository.ts` | `order-repository.ts` | adapter/out/repository |
| `{feature}-controller.ts` | `order-controller.ts` | adapter/in/controller |

---

## Barrel Exports (index.ts)

Use barrel exports for clean imports:

```typescript
// features/orders/domain/index.ts
export * from './entities/order';
export * from './value-objects/money';
export * from './value-objects/order-id';
export * from './aggregates/order';
export * from './events/order-event';

// features/orders/usecase/port/in/create-order/index.ts
export * from './create-order.input';
export * from './create-order.usecase';
```

---

## File Size Guidelines

| File Type | Max Lines | Guideline |
|-----------|-----------|-----------|
| Entity/Value Object | 150 | Single responsibility |
| Service/Use Case | 200 | One use case per file |
| Controller | 100 | Delegate to services |
| Mapper | 100 | One per aggregate |
| Utility | 200 | Group related functions |

**AI Guidelines:**
- If file exceeds limits, refactor into smaller files
- Extract helper methods to separate files if reused
- Consider splitting large aggregates into multiple files

---

## Test Organization

```
feature/
├── domain/
│   └── __tests__/
│       ├── order.aggregate.test.ts
│       └── money.vo.test.ts
├── usecase/
│   └── __tests__/
│       ├── create-order.service.test.ts
│       └── order-mapper.test.ts
└── adapter/
    └── __tests__/
        ├── order.controller.test.ts
        └── order.repository.test.ts
```

---

## Import Order

```typescript
// 1. Node.js built-ins
import { readFile } from 'fs';
import { join } from 'path';

// 2. External packages
import { Injectable } from '@nestjs/common';
import { Result } from '@types/result';

// 3. Shared/domain modules
import { Entity } from '../../../../shared/domain/entity';
import { ValueObject } from '../../../../shared/domain/value-object';

// 4. Feature internal imports (relative)
import { Order } from '../domain/entities/order';
import { CreateOrderInput } from './port/in/create-order/input';
import { OrderRepository } from './port/out/order-repository';
```

---

## AI-Specific Guidelines

When organizing files:

1. **Feature first** - Always group by feature, not type
2. **DDD structure** - Follow domain/usecase/adapter pattern
3. **Barrel exports** - Use index.ts for clean imports
4. **File naming** - Use consistent naming patterns
5. **Size limits** - Keep files focused and small
6. **Single responsibility** - One file per concern
7. **Test co-location** - Place tests next to implementation
8. **Shared extraction** - Extract common patterns to shared/

---

## Example: Complete Feature Structure

```
src/features/orders/
├── domain/
│   ├── entities/
│   │   ├── order.ts
│   │   ├── order-line-item.ts
│   │   └── index.ts
│   ├── value-objects/
│   │   ├── order-id.ts
│   │   ├── order-status.ts
│   │   ├── money.ts
│   │   └── index.ts
│   ├── aggregates/
│   │   ├── order-aggregate.ts
│   │   └── index.ts
│   ├── events/
│   │   ├── order-created.event.ts
│   │   └── index.ts
│   └── index.ts
├── usecase/
│   ├── port/
│   │   ├── in/
│   │   │   ├── create-order/
│   │   │   │   ├── create-order.input.ts
│   │   │   │   ├── create-order.usecase.ts
│   │   │   │   └── index.ts
│   │   │   ├── update-order/
│   │   │   │   ├── update-order.input.ts
│   │   │   │   ├── update-order.usecase.ts
│   │   │   │   └── index.ts
│   │   │   └── index.ts
│   │   └── out/
│   │       ├── order.dto.ts
│   │       ├── order.mapper.ts
│   │       ├── order.repository.interface.ts
│   │       └── index.ts
│   ├── service/
│   │   ├── create-order.service.ts
│   │   ├── update-order.service.ts
│   │   └── index.ts
│   └── index.ts
├── adapter/
│   ├── in/
│   │   ├── controller/
│   │   │   ├── order-web.controller.ts
│   │   │   └── index.ts
│   │   ├── presenter/
│   │   │   ├── order-presenter.ts
│   │   │   └── index.ts
│   │   └── dto/
│   │       ├── create-order-request.dto.ts
│   │       ├── create-order-response.dto.ts
│   │       └── index.ts
│   └── out/
│       ├── repository/
│       │   ├── order-repository.ts
│       │   └── index.ts
│       └── persistence/
│           ├── order.entity.ts
│           └── order.mapper.ts
├── orders.module.ts
└── index.ts
```

---

## References

1. Clean Architecture by Robert C. Martin
2. Domain-Driven Design by Eric Evans
3. Vertical Slice Architecture patterns
4. Feature-Sliced Design (FSD) methodology
