# Testing Conventions

Comprehensive testing conventions to ensure code quality, maintainability, and reliability. Tests are written following Test-Driven Development (TDD) principles and serve as executable specifications.

## Table of Contents

1. [Test Organization](#test-organization)
2. [Test Structure](#test-structure)
3. [Test Naming](#test-naming)
4. [Test Isolation](#test-isolation)
5. [Test Doubles](#test-doubles)
6. [Testing Strategies](#testing-strategies)
7. [Test Coverage Targets](#test-coverage-targets)
8. [Best Practices](#best-practices)
9. [Test-Driven Development](#test-driven-development)
10. [Behavior-Driven Development](#behavior-driven-development)

---

## Test Organization

### Separate Test Package Structure

Test structure mirrors main source structure for easy navigation:

```
src/
├── features/
│   ├── task/
│   │   ├── domain/
│   │   │   ├── Task.ts
│   │   │   └── TaskId.ts
│   │   ├── usecases/
│   │   │   ├── AddTaskService.ts
│   │   │   └── ShowService.ts
│   │   └── adapters/
│   │       └── controllers/
│   │           └── AddConsoleController.ts
│   └── tests/              # All tests co-located with feature
│       ├── unit/
│       │   ├── domain/
│       │   │   ├── Task.test.ts
│       │   │   └── TaskId.test.ts
│       │   ├── usecases/
│       │   │   ├── AddTaskService.test.ts
│       │   │   └── ShowService.test.ts
│       │   └── adapters/
│       │       └── AddConsoleController.test.ts
│       ├── integration/
│       │   └── usecases/
│       │       └── TaskUseCases.test.ts
│       └── e2e/
│           └── TaskFeature.test.ts
```

### Test Types

#### Unit Tests

- **Location**: `src/features/xxx/tests/unit/`
- **Purpose**: Test individual components in isolation
- **Scope**: Single class or function
- **Dependencies**: Mocked or stubbed

**Characteristics**:

- Fast execution (milliseconds)
- No external dependencies (database, network, filesystem)
- Highly focused on single unit of behavior
- Used for domain entities, value objects, utility functions

```typescript
describe('Task Entity', () => {
  @Test
  public create_task_with_valid_data_succeeds(): void {
    // Arrange
    const description = 'Test task description';

    // Act
    const task = Task.create(description);

    // Assert
    assertNotNull(task);
    assertEquals(description, task.getDescription());
    assertFalse(task.isDone());
  }
});
```

#### Integration Tests

- **Location**: `src/features/xxx/tests/integration/`
- **Purpose**: Test business logic flows
- **Scope**: Use cases and their interactions
- **Dependencies**: Real repositories (in-memory or test database)

**Characteristics**:

- Moderate execution time
- Test interactions between multiple components
- Use real implementations of repositories
- Verify data flows and business rules

```typescript
describe('AddTaskUseCase Integration', () => {
  let useCase: AddTaskService;
  let repository: InMemoryTaskRepository;

  beforeEach(() => {
    repository = new InMemoryTaskRepository();
    useCase = new AddTaskService(repository);
  });

  @Test
  public add_task_saves_to_repository(): async void {
    // Arrange
    const input: AddTaskInput = {
      description: 'Test task',
      projectId: null
    };

    // Act
    const output = await useCase.execute(input);

    // Assert
    assertTrue(output.success);
    assertNotNull(output.taskId);

    const saved = await repository.findById(output.taskId);
    assertNotNull(saved);
    assertEquals('Test task', saved.getDescription());
  }
});
```

#### E2E Tests

- **Location**: `src/features/xxx/tests/e2e/`
- **Purpose**: Test complete feature workflows
- **Scope**: Multiple use cases working together
- **Dependencies**: Real infrastructure (test database, etc.)

**Characteristics**:

- Slower execution
- Test complete user workflows
- Use production-like infrastructure
- Verify end-to-end functionality

```typescript
describe('Task Feature E2E', () => {
  @Test
  public complete_task_workflow(): async void {
    // Create task
    const createResult = await taskApi.createTask({ description: 'E2E Test' });
    assertTrue(createResult.success);

    // List tasks
    const listResult = await taskApi.listTasks();
    assertTrue(listResult.tasks.some(t => t.id === createResult.taskId));

    // Complete task
    const completeResult = await taskApi.completeTask(createResult.taskId);
    assertTrue(completeResult.success);

    // Verify completion
    const taskResult = await taskApi.getTask(createResult.taskId);
    assertTrue(taskResult.task.isCompleted);
  }
});
```

---

## Test Structure

### Arrange-Act-Assert Pattern

All tests follow the AAA pattern for clarity:

```typescript
@Test
public descriptive_test_name_with_underscores(): void {
  // Arrange - Set up test data and dependencies
  const task = Task.create('Test task');
  const repository = new InMemoryTaskRepository();
  repository.save(task);

  // Act - Execute the code being tested
  const found = repository.findById(task.id);

  // Assert - Verify the expected outcome
  assertNotNull(found);
  assertEquals(task.id, found.id);
}
```

### Each Section Explained

#### Arrange

- **Setup**: Create test data and dependencies
- **Mocks**: Configure mock behavior
- **Preconditions**: Establish initial state

```typescript
// Arrange
const user = User.create('test@example.com', 'password123');
const repository = new InMemoryUserRepository();
repository.save(user);
const authService = new AuthenticationService(repository);
```

#### Act

- **Execute**: Call the method being tested
- **Single Action**: Test one behavior at a time
- **Capture Result**: Store return value if applicable

```typescript
// Act
const result = await authService.authenticate('test@example.com', 'password123');
```

#### Assert

- **Verify**: Check expected outcomes
- **Single Focus**: Each assertion tests one thing
- **Clear Messages**: Use descriptive assertions

```typescript
// Assert
assertTrue(result.success);
assertEquals('test@example.com', result.user.email);
assertNotNull(result.token);
```

---

## Test Naming

### Descriptive Test Method Names

- **Describe behavior**: What is being tested, not how
- **Use underscores**: Separate words for readability
- **Test one thing**: Single responsibility
- **Include edge cases**: Clearly describe what's being tested

#### Examples

```typescript
// ✅ GOOD - Descriptive test names
@Test
public add_task_to_project_increases_task_count(): void {
  // ...
}

@Test
public add_task_with_empty_description_throws_exception(): void {
  // ...
}

@Test
public add_task_to_nonexistent_project_returns_not_found_error(): void {
  // ...
}

@Test
public set_task_status_to_done_updates_completion_date(): void {
  // ...
}

// ❌ BAD - Vague test names
@Test
public test_add_task(): void {
  // What does it test?
}

@Test
public task_test(): void {
  // What is being tested?
}

@Test
public test1(): void {
  // What is this test for?
}
```

### Behavior-Driven Naming

Tests describe behavior, not implementation:

```typescript
// ✅ GOOD - Behavior-driven
@Test
public when_user_logs_in_with_valid_credentials_then_token_is_returned(): void {
  // ...
}

@Test
public when_user_registers_with_duplicate_email_then_error_is_returned(): void {
  // ...
}

@Test
public when_product_quantity_is_negative_then_exception_is_thrown(): void {
  // ...
}

// ❌ BAD - Implementation-focused
@Test
public test_login_method(): void {
  // ...
}

@Test
public register_duplicate_email(): void {
  // ...
}

@Test
public check_quantity(): void {
  // ...
}
```

### Given-When-Then Format

Use Given-When-Then for complex scenarios:

```typescript
describe('Shopping Cart', () => {
  @Test
  public given_empty_cart_when_item_added_then_cart_contains_one_item(): void {
    // Given
    const cart = new ShoppingCart();

    // When
    cart.addItem(new CartItem('product-1', 'Product 1', 2, 10.00));

    // Then
    assertEquals(1, cart.getItemCount());
  }

  @Test
  public given_cart_with_items_when_cleared_then_cart_is_empty(): void {
    // Given
    const cart = new ShoppingCart();
    cart.addItem(new CartItem('product-1', 'Product 1', 2, 10.00));
    cart.addItem(new CartItem('product-2', 'Product 2', 1, 5.00));

    // When
    cart.clear();

    // Then
    assertEquals(0, cart.getItemCount());
  }
});
```

---

## Test Isolation

### Independent Tests

Each test should be independent of others:

```typescript
describe('Task Repository', () => {
  let repository: InMemoryTaskRepository;

  beforeEach(() => {
    // Create fresh repository for each test
    repository = new InMemoryTaskRepository();
  });

  @Test
  public first_test_creates_task(): void {
    const task = Task.create('Task 1');
    repository.save(task);
    // ...
  }

  @Test
  public second_test_creates_task(): void {
    // This test should not be affected by first_test
    const task = Task.create('Task 2');
    repository.save(task);
    assertEquals(1, repository.findAll().length);
  }
});
```

### Cleanup

Clean up after tests:

```typescript
describe('File Storage', () => {
  let storage: FileStorage;

  beforeEach(() => {
    storage = new FileStorage('/tmp/test-storage');
  });

  afterEach(async () => {
    // Clean up test files
    await storage.deleteAll();
  });

  @Test
  public save_and_retrieve_file(): void {
    // ...
  }
});
```

### Shared State Management

Avoid shared state between tests:

```typescript
// ❌ BAD - Shared state
let repository: InMemoryTaskRepository;

beforeAll(() => {
  repository = new InMemoryTaskRepository();
});

@Test
public test1(): void {
  repository.save(Task.create('Task 1'));
  // Test assumes empty repository
}

@Test
public test2(): void {
  repository.save(Task.create('Task 2'));
  // Test assumes empty repository - FAILS
}

// ✅ GOOD - Fresh state per test
describe('Task Repository', () => {
  let repository: InMemoryTaskRepository;

  beforeEach(() => {
    repository = new InMemoryTaskRepository();
  });

  @Test
  public test1(): void {
    repository.save(Task.create('Task 1'));
    // Fresh repository
  }

  @Test
  public test2(): void {
    repository.save(Task.create('Task 2'));
    // Fresh repository
  }
});
```

---

## Test Doubles

### Mocks

Simulate dependencies with predefined behavior:

```typescript
describe('TaskService', () => {
  let service: TaskService;
  let repository: jest.Mocked<ITaskRepository>;

  beforeEach(() => {
    repository = {
      findById: jest.fn(),
      save: jest.fn(),
      findAll: jest.fn()
    } as jest.Mocked<ITaskRepository>;

    service = new TaskService(repository);
  });

  @Test
  public find_existing_task_returns_task(): void {
    // Arrange
    const task = Task.create('Test task');
    repository.findById.mockResolvedValue(task);

    // Act
    const result = await service.findById('task-123');

    // Assert
    assertEquals(task, result);
    expect(repository.findById).toHaveBeenCalledWith('task-123');
  }
});
```

### Stubs

Provide fixed responses for specific calls:

```typescript
describe('Email Service', () => {
  let service: UserService;
  let emailService: IEmailService;

  beforeEach(() => {
    emailService = {
      sendEmail: jest.fn().mockResolvedValue(undefined)
    } as jest.Mocked<IEmailService>;

    service = new UserService(/* ... */, emailService);
  });

  @Test
  public register_user_sends_welcome_email(): async void {
    // Arrange
    emailService.sendEmail.mockResolvedValue(undefined);

    // Act
    await service.registerUser('test@example.com', 'password');

    // Assert
    expect(emailService.sendEmail).toHaveBeenCalledWith(
      'test@example.com',
      'Welcome',
      expect.any(String)
    );
  }
});
```

### Fakes

Simple implementations for testing:

```typescript
class InMemoryTaskRepository implements ITaskRepository {
  private tasks: Map<string, Task> = new Map();

  async findById(id: string): Promise<Task | null> {
    return this.tasks.get(id) || null;
  }

  async save(task: Task): Promise<void> {
    this.tasks.set(task.getId().value, task);
  }

  async findAll(): Promise<Task[]> {
    return Array.from(this.tasks.values());
  }
}

describe('Task Service', () => {
  @Test
  public create_and_retrieve_task(): void {
    // Arrange
    const repository = new InMemoryTaskRepository();
    const service = new TaskService(repository);

    // Act
    const task = await service.createTask('Test task');

    // Assert
    const found = await repository.findById(task.getId().value);
    assertNotNull(found);
  }
});
```

### Spies

Wrap existing objects to track method calls:

```typescript
describe('User Service', () => {
  @Test
  public delete_user_logs_action(): void {
    // Arrange
    const logger = {
      log: jest.fn()
    };
    const service = new UserServiceWithLogger(userRepository, logger);

    // Act
    await service.deleteUser('user-123');

    // Assert
    expect(logger.log).toHaveBeenCalledWith(
      'User deleted',
      { userId: 'user-123' }
    );
  }
});
```

---

## Testing Strategies

### Boundary Testing

Test at boundaries:

```typescript
describe('Password Validator', () => {
  @Test
  public password_with_7_characters_is_too_short(): void {
    assertFalse(PasswordValidator.isValid('Abc123!'));
  }

  @Test
  public password_with_8_characters_is_valid(): void {
    assertTrue(PasswordValidator.isValid('Abc12345!'));
  }

  @Test
  public password_with_127_characters_is_valid(): void {
    const password = 'A' + 'a'.repeat(125) + '1!';
    assertTrue(PasswordValidator.isValid(password));
  }

  @Test
  public password_with_128_characters_is_too_long(): void {
    const password = 'A' + 'a'.repeat(126) + '1!';
    assertFalse(PasswordValidator.isValid(password));
  }
});
```

### Edge Case Testing

Test edge cases and error conditions:

```typescript
describe('Calculator', () => {
  @Test
  public divide_by_zero_throws_exception(): void {
    expect(() => Calculator.divide(10, 0))
      .toThrow('Cannot divide by zero');
  }

  @Test
  public divide_negative_by_positive_returns_negative(): void {
    assertEquals(-2, Calculator.divide(10, -5));
  }

  @Test
  public divide_zero_by_any_number_returns_zero(): void {
    assertEquals(0, Calculator.divide(0, 10));
  }

  @Test
  public divide_positive_by_positive_returns_positive(): void {
    assertEquals(2, Calculator.divide(10, 5));
  }
});
```

### Happy Path vs Sad Path

Test both success and failure cases:

```typescript
describe('User Registration', () => {
  @Test
  public valid_registration_succeeds(): void {
    // Happy path
    const result = registrationService.register(
      'test@example.com',
      'password123'
    );
    assertTrue(result.success);
    assertNotNull(result.user);
  }

  @Test
  public duplicate_email_fails(): void {
    // Sad path - duplicate email
    repository.save(User.create('test@example.com', 'password'));
    
    const result = registrationService.register(
      'test@example.com',
      'password456'
    );
    assertFalse(result.success);
    assertEquals('Email already exists', result.error);
  }

  @Test
  public weak_password_fails(): void {
    // Sad path - weak password
    const result = registrationService.register(
      'test@example.com',
      '123'
    );
    assertFalse(result.success);
    assertEquals('Password too weak', result.error);
  }
});
```

### Integration Testing

Test component interactions:

```typescript
describe('Task Management Integration', () => {
  let useCase: AddTaskService;
  let repository: InMemoryTaskRepository;

  beforeEach(() => {
    repository = new InMemoryTaskRepository();
    useCase = new AddTaskService(repository);
  });

  @Test
  public add_and_retrieve_task_integration(): void {
    // Act - Add task
    const output1 = await useCase.execute({
      description: 'Test task',
      projectId: null
    });

    // Assert - Task was added
    assertTrue(output1.success);
    const task = await repository.findById(output1.taskId);
    assertNotNull(task);

    // Act - Retrieve task
    const output2 = await useCase.execute({
      taskId: output1.taskId
    });

    // Assert - Task retrieved correctly
    assertEquals('Test task', task.getDescription());
  }
});
```

---

## Test Coverage Targets

### Coverage Requirements

| Component Type    | Coverage Target | Notes                                   |
| ----------------- | --------------- | --------------------------------------- |
| Domain logic      | 100%            | All business rules must be tested       |
| Use cases         | 95%+            | Application logic thoroughly tested     |
| Adapters          | 80%+            | Controller and gateway logic tested     |
| Overall           | 85%+            | Minimum acceptable coverage             |

### Coverage Reporting

Run coverage reports:

```bash
# Run tests with coverage
npm run test:coverage

# Generate HTML report
npm run test:coverage:html

# Check coverage thresholds
npm run test:coverage:check
```

### Coverage Metrics

- **Line Coverage**: Percentage of lines executed
- **Branch Coverage**: Percentage of conditional branches tested
- **Function Coverage**: Percentage of functions called
- **Path Coverage**: Percentage of execution paths tested

### Coverage Best Practices

- **Aim for meaningful coverage**: 100% doesn't mean bug-free
- **Focus on behavior**: Test what matters, not just lines
- **Use coverage as guide**: Identify untested areas
- **Don't sacrifice quality**: Tests should be valuable, not just coverage fodder

---

## Best Practices

### AAA Pattern

Always use Arrange-Act-Assert:

```typescript
@Test
public user_login_test(): void {
  // Arrange
  const user = new User('test@example.com', 'password');
  const service = new AuthService(repository);

  // Act
  const result = service.authenticate(user);

  // Assert
  assertTrue(result.success);
}
```

### One Assertion Per Test

Keep tests focused:

```typescript
// ✅ GOOD - Single assertion
@Test
public task_id_is_unique(): void {
  const task1 = Task.create('Task 1');
  const task2 = Task.create('Task 2');
  assertNotEquals(task1.getId(), task2.getId());
}

// ❌ BAD - Multiple assertions
@Test
public task_properties(): void {
  const task = Task.create('Task 1');
  assertNotNull(task.getId()); // First assertion
  assertEquals('Task 1', task.getDescription()); // Second assertion
  assertFalse(task.isDone()); // Third assertion
}

// Exception: Related assertions for single behavior
@Test
public task_created_with_correct_properties(): void {
  const task = Task.create('Task 1');
  assertNotNull(task.getId());
  assertEquals('Task 1', task.getDescription());
  assertFalse(task.isDone());
  assertNull(task.getCompletionDate());
}
```

### Test Behavior, Not Implementation

Test what happens, not how:

```typescript
// ✅ GOOD - Tests behavior
@Test
public add_task_increases_task_count(): void {
  const cart = new ShoppingCart();
  cart.addItem(new CartItem('1', 'Product', 1, 10.00));
  assertEquals(1, cart.getItemCount());
}

// ❌ BAD - Tests implementation
@Test
public add_task_pushes_to_items_array(): void {
  const cart = new ShoppingCart();
  cart.addItem(new CartItem('1', 'Product', 1, 10.00));
  assertEquals(1, cart.getItems().length); // Exposes internal structure
}
```

### Fast Tests

Keep tests fast to encourage running them:

```typescript
// ❌ BAD - Slow test
@Test
public test_with_network_call(): void {
  const result = await httpClient.get('https://slow-api.com/data');
  assertEquals('data', result);
}

// ✅ GOOD - Fast test
@Test
public test_with_mocked_network(): void {
  httpClient.get = jest.fn().mockResolvedValue('data');
  const result = await myFunction();
  assertEquals('data', result);
}
```

### Deterministic Tests

Tests should produce the same results:

```typescript
// ❌ BAD - Non-deterministic
@Test
public test_with_current_time(): void {
  const now = new Date();
  const item = new Item(createdAt: now);
  assertEquals(now, item.createdAt); // Time changes between calls
}

// ✅ GOOD - Deterministic
@Test
public test_with_fixed_time(): void {
  const fixedTime = new Date('2024-01-01T00:00:00Z');
  const item = new Item(createdAt: fixedTime);
  assertEquals(fixedTime, item.createdAt);
}
```

---

## Test-Driven Development

### TDD Cycle

```
┌─────────────────────────────────────────────────┐
│ 1. RED - Write a failing test                   │
│    - Test describes desired behavior            │
│    - Test fails because code doesn't exist      │
│                                                 │
│ 2. GREEN - Write minimal code to pass test      │
│    - Implement just enough to make test pass    │
│    - Don't worry about perfection               │
│                                                 │
│ 3. REFACTOR - Improve code while tests pass     │
│    - Clean up implementation                    │
│    - Apply design principles                    │
│    - Ensure tests still pass                    │
└─────────────────────────────────────────────────┘
```

### Example: TDD for a User Creator

#### Step 1: RED - Write failing test

```typescript
describe('UserCreator', () => {
  let creator: UserCreator;
  let mockRepository: jest.Mocked<IUserRepository>;
  let mockEmailService: jest.Mocked<IEmailService>;

  beforeEach(() => {
    mockRepository = {
      save: jest.fn(),
      findByEmail: jest.fn()
    };
    mockEmailService = {
      sendEmail: jest.fn()
    };
    creator = new UserCreator(mockRepository, mockEmailService);
  });

  @Test
  public creating_user_saves_to_repository(): void {
    // Arrange
    const input: CreateUserInput = {
      email: 'test@example.com',
      password: 'password123'
    };
    mockRepository.findByEmail.mockResolvedValue(null);

    // Act
    const result = creator.create(input);

    // Assert
    assertTrue(result.success);
    expect(mockRepository.save).toHaveBeenCalled();
  }
});
```

#### Step 2: GREEN - Minimal implementation

```typescript
class UserCreator {
  constructor(
    private repository: IUserRepository,
    private emailService: IEmailService
  ) {}

  create(input: CreateUserInput): CreateUserResult {
    const user = new User(input.email, input.password);
    this.repository.save(user);
    return CreateUserResult.success(user);
  }
}
```

#### Step 3: REFACTOR - Improve code

```typescript
class UserCreator {
  constructor(
    private repository: IUserRepository,
    private emailService: IEmailService,
    private passwordHasher: IPasswordHasher,
    private validator: IUserValidator
  ) {}

  create(input: CreateUserInput): CreateUserResult {
    // Validate input
    if (!this.validator.isValid(input)) {
      return CreateUserResult.failure('Invalid input');
    }

    // Check for duplicate
    const existing = await this.repository.findByEmail(input.email);
    if (existing) {
      return CreateUserResult.failure('Email already exists');
    }

    // Create user
    const passwordHash = await this.passwordHasher.hash(input.password);
    const user = new User(input.email, passwordHash);
    
    // Persist and notify
    await this.repository.save(user);
    await this.emailService.sendWelcome(input.email);

    return CreateUserResult.success(user);
  }
}
```

---

## Behavior-Driven Development

### Gherkin Format

Use Given-When-Then for BDD scenarios:

```gherkin
Feature: User Registration

  Scenario: Successful registration
    Given the user is on the registration page
    When the user submits valid registration details
    Then the user should receive a confirmation email
    And the user account should be created

  Scenario: Duplicate email registration
    Given a user with email "test@example.com" already exists
    When the user submits registration with email "test@example.com"
    Then an error message "Email already exists" should be shown
    And no new account should be created
```

### BDD Test Implementation

```typescript
describe('User Registration Feature', () => {
  describe('Scenario: Successful registration', () => {
    @Test
    public user_receives_confirmation_email(): void {
      // Given
      const emailService = createMockEmailService();
      const userRepository = createMockUserRepository();
      const userCreator = new UserCreator(userRepository, emailService);

      // When
      const result = userCreator.create({
        email: 'new@example.com',
        password: 'password123'
      });

      // Then
      assertTrue(result.success);
      expect(emailService.sendWelcomeEmail).toHaveBeenCalledWith('new@example.com');
    }

    @Test
    public user_account_is_created(): void {
      // Given
      const userRepository = createMockUserRepository();
      const userCreator = new UserCreator(userRepository, createMockEmailService());

      // When
      const result = userCreator.create({
        email: 'new@example.com',
        password: 'password123'
      });

      // Then
      expect(userRepository.save).toHaveBeenCalled();
      assertNotNull(result.user);
    }
  });

  describe('Scenario: Duplicate email registration', () => {
    @Test
    public error_message_is_shown(): void {
      // Given
      const userRepository = createMockUserRepository();
      userRepository.findByEmail.mockResolvedValue(
        User.create('existing@example.com', 'hash')
      );
      const userCreator = new UserCreator(userRepository, createMockEmailService());

      // When
      const result = userCreator.create({
        email: 'existing@example.com',
        password: 'password123'
      });

      // Then
      assertFalse(result.success);
      assertEquals('Email already exists', result.error);
    }

    @Test
    public no_new_account_is_created(): void {
      // Given
      const userRepository = createMockUserRepository();
      userRepository.findByEmail.mockResolvedValue(
        User.create('existing@example.com', 'hash')
      );
      const userCreator = new UserCreator(userRepository, createMockEmailService());

      // When
      const result = userCreator.create({
        email: 'existing@example.com',
        password: 'password123'
      });

      // Then
      expect(userRepository.save).not.toHaveBeenCalled();
    }
  });
});
```

---

## References

- [TDD Workflow](tdd-workflow.md) - Test-Driven Development process
- [BDD with Gherkin](bdd-gherkin.md) - Behavior-Driven Development with Gherkin
- [Test Pyramids](test-pyramids.md) - Testing strategy and balance
- [Clean Code](clean-code.md) - Writing maintainable tests
- [AI Code Review](../07-review-checklists/ai-code-review.md) - Test review checklist
