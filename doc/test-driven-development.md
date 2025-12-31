# Test-Driven Development (TDD)

## Core Concept

Test-Driven Development (TDD) is a software development methodology that emphasizes writing tests before writing the implementation code. It follows a short, iterative development cycle known as the "Red-Green-Refactor" loop, where developers first write a failing test, then make the test pass with minimal code, and finally improve the code through refactoring while maintaining test coverage.

TDD was popularized by Kent Beck in his 2003 book "Test-Driven Development: By Example" and has become a fundamental practice in agile software development. The methodology promotes better design, higher code quality, and increased confidence in the codebase by ensuring that all code is covered by automated tests from the start.

The core philosophy of TDD is simple but profound: write tests first, then write just enough code to make those tests pass, and continuously improve the design through refactoring. This approach forces developers to think about the desired behavior and interface of their code before implementation, leading to more modular, flexible, and maintainable software systems.

TDD also serves as a design tool, helping developers clarify requirements and design interfaces before writing production code. By starting with a test, developers must consider how the code will be used, what inputs it will accept, and what outputs it should produce, leading to better-designed APIs and more focused, single-responsibility functions.

---

## The Red-Green-Refactor Cycle

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TEST-DRIVEN DEVELOPMENT CYCLE                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│    ┌──────────────┐         ┌──────────────┐         ┌──────────────┐  │
│    │     RED      │ ──────► │    GREEN     │ ──────► │   REFACTOR   │  │
│    │              │         │              │         │              │  │
│    │ Write failing│         │ Write minimal│         │ Improve code │  │
│    │ test first   │         │ code to pass │         │ design       │  │
│    └──────────────┘         └──────────────┘         └──────────────┘  │
│         ▲                         ▲                         ▲           │
│         │                         │                         │           │
│         │                         │                         │           │
│         └─────────────────────────┴─────────────────────────┘           │
│                           (Repeat)                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Red Phase: Write a Failing Test

The Red phase is where you write a test that describes the behavior you want to implement. This test should fail because the implementation doesn't exist yet. The test serves as a specification for the desired functionality and helps clarify what the code should do.

Key principles for the Red phase:
- Write a test that describes the behavior, not the implementation
- The test should be small and focused on a single aspect of functionality
- Run the test to confirm it fails (the test framework should show red)
- Use descriptive test names that explain what the code should do
- Start with the simplest possible test case

### Green Phase: Make the Test Pass

The Green phase involves writing the minimal amount of code necessary to make the failing test pass. The goal is not to write perfect code but to get the test green as quickly as possible. This phase validates that your test correctly identifies the desired behavior.

Key principles for the Green phase:
- Write only the code needed to pass the test
- Don't worry about optimal implementation or perfect design
- You can use shortcuts, magic values, or temporary solutions
- The focus is on correctness, not elegance
- Once the test passes, you have verified behavior

### Refactor Phase: Improve the Design

The Refactor phase is where you improve the code's structure, readability, and design while ensuring all tests continue to pass. This is your opportunity to clean up the code, remove duplication, and apply design principles without fear of breaking functionality.

Key principles for the Refactor phase:
- Improve code structure, naming, and organization
- Extract methods, remove duplication, apply design patterns
- Ensure all existing tests still pass after refactoring
- Make small, incremental changes
- Use the tests as a safety net for refactoring

---

## TDD in TypeScript with Jest

```typescript
// src/math/calculator.test.ts
import { Calculator } from './calculator';

describe('Calculator', () => {
  let calculator: Calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  describe('add', () => {
    it('should add two positive numbers correctly', () => {
      const result = calculator.add(2, 3);
      expect(result).toBe(5);
    });

    it('should handle negative numbers', () => {
      const result = calculator.add(-1, 5);
      expect(result).toBe(4);
    });

    it('should add decimal numbers correctly', () => {
      const result = calculator.add(0.1, 0.2);
      expect(result).toBeCloseTo(0.3, 10);
    });

    it('should return the first number when adding zero', () => {
      const result = calculator.add(5, 0);
      expect(result).toBe(5);
    });
  });

  describe('multiply', () => {
    it('should multiply two positive numbers correctly', () => {
      const result = calculator.multiply(3, 4);
      expect(result).toBe(12);
    });

    it('should return zero when multiplying by zero', () => {
      const result = calculator.multiply(5, 0);
      expect(result).toBe(0);
    });

    it('should handle negative number multiplication', () => {
      const result = calculator.multiply(-2, 3);
      expect(result).toBe(-6);
    });
  });

  describe('divide', () => {
    it('should divide two numbers correctly', () => {
      const result = calculator.divide(10, 2);
      expect(result).toBe(5);
    });

    it('should throw error when dividing by zero', () => {
      expect(() => calculator.divide(10, 0)).toThrow('Division by zero');
    });
  });

  describe('factorial', () => {
    it('should calculate factorial of 0 correctly', () => {
      const result = calculator.factorial(0);
      expect(result).toBe(1);
    });

    it('should calculate factorial of positive numbers', () => {
      const result = calculator.factorial(5);
      expect(result).toBe(120);
    });

    it('should throw error for negative input', () => {
      expect(() => calculator.factorial(-1)).toThrow(
        'Factorial is not defined for negative numbers'
      );
    });
  });
});

// src/math/calculator.ts
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }

  multiply(a: number, b: number): number {
    return a * b;
  }

  divide(a: number, b: number): number {
    if (b === 0) {
      throw new Error('Division by zero');
    }
    return a / b;
  }

  factorial(n: number): number {
    if (n < 0) {
      throw new Error('Factorial is not defined for negative numbers');
    }
    if (n === 0 || n === 1) {
      return 1;
    }
    return n * this.factorial(n - 1);
  }
}
```

---

## TDD with Behavior-Driven Development (Given-When-Then)

```typescript
// src/bank/account.test.ts
import { BankAccount } from './bank-account';

describe('BankAccount BDD Tests', () => {
  describe('Feature: Account Balance Management', () => {
    describe('Scenario: Depositing money into an account', () => {
      it('should increase balance after deposit', () => {
        given('an account with balance of 100', () => {
          const account = new BankAccount(100);
        });

        when('a deposit of 50 is made', (account: BankAccount) => {
          account.deposit(50);
        });

        then('the new balance should be 150', (account: BankAccount) => {
          expect(account.getBalance()).toBe(150);
        });
      });

      it('should record deposit in transaction history', () => {
        given('an account with balance of 0', () => {
          const account = new BankAccount(0);
        });

        when('a deposit of 100 is made', (account: BankAccount) => {
          account.deposit(100);
        });

        then('the transaction should be recorded', (account: BankAccount) => {
          const transactions = account.getTransactionHistory();
          expect(transactions).toContainEqual({
            type: 'deposit',
            amount: 100,
            timestamp: expect.any(Date),
          });
        });
      });
    });

    describe('Scenario: Withdrawing money from an account', () => {
      it('should decrease balance when sufficient funds exist', () => {
        given('an account with balance of 200', () => {
          const account = new BankAccount(200);
        });

        when('a withdrawal of 50 is made', (account: BankAccount) => {
          account.withdraw(50);
        });

        then('the new balance should be 150', (account: BankAccount) => {
          expect(account.getBalance()).toBe(150);
        });
      });

      it('should reject withdrawal when insufficient funds', () => {
        given('an account with balance of 50', () => {
          const account = new BankAccount(50);
        });

        when('a withdrawal of 100 is attempted', (account: BankAccount) => {
          account.withdraw(100);
        });

        then('the balance should remain unchanged', (account: BankAccount) => {
          expect(account.getBalance()).toBe(50);
        });
      });
    });
  });
});

// Helper functions for BDD-style testing
function given(description: string, setup: () => BankAccount): BankAccount {
  return setup();
}

function when(description: string, action: (account: BankAccount) => void): void {
  // This would need proper context management in practice
}

function then(
  description: string,
  assertion: (account: BankAccount) => void
): void {
  // This would need proper context management in practice
}

// src/bank/bank-account.ts
export class BankAccount {
  private balance: number;
  private transactions: Array<{
    type: 'deposit' | 'withdrawal';
    amount: number;
    timestamp: Date;
  }>;

  constructor(initialBalance: number = 0) {
    this.balance = initialBalance;
    this.transactions = [];
  }

  getBalance(): number {
    return this.balance;
  }

  deposit(amount: number): void {
    if (amount <= 0) {
      throw new Error('Deposit amount must be positive');
    }
    this.balance += amount;
    this.transactions.push({
      type: 'deposit',
      amount,
      timestamp: new Date(),
    });
  }

  withdraw(amount: number): void {
    if (amount <= 0) {
      throw new Error('Withdrawal amount must be positive');
    }
    if (amount > this.balance) {
      throw new Error('Insufficient funds');
    }
    this.balance -= amount;
    this.transactions.push({
      type: 'withdrawal',
      amount,
      timestamp: new Date(),
    });
  }

  getTransactionHistory(): Array<{
    type: 'deposit' | 'withdrawal';
    amount: number;
    timestamp: Date;
  }> {
    return [...this.transactions];
  }
}
```

---

## TDD with Mocking and Dependency Injection

```typescript
// src/notification/notification-service.test.ts
import { OrderProcessor } from './order-processor';
import { PaymentGateway } from './payment-gateway';
import { EmailService } from './email-service';
import { InventoryService } from './inventory-service';

describe('OrderProcessor', () => {
  let orderProcessor: OrderProcessor;
  let mockPaymentGateway: jest.Mocked<PaymentGateway>;
  let mockEmailService: jest.Mocked<EmailService>;
  let mockInventoryService: jest.Mocked<InventoryService>;

  beforeEach(() => {
    mockPaymentGateway = {
      processPayment: jest.fn().mockResolvedValue({ success: true, transactionId: 'TX123' }),
      refundPayment: jest.fn().mockResolvedValue({ success: true }),
    } as unknown as jest.Mocked<PaymentGateway>;

    mockEmailService = {
      sendOrderConfirmation: jest.fn().mockResolvedValue(undefined),
      sendPaymentFailedNotification: jest.fn().mockResolvedValue(undefined),
    } as unknown as jest.Mocked<EmailService>;

    mockInventoryService = {
      reserveItems: jest.fn().mockResolvedValue({ success: true }),
      releaseItems: jest.fn().mockResolvedValue({ success: true }),
    } as unknown as jest.Mocked<InventoryService>;

    orderProcessor = new OrderProcessor(
      mockPaymentGateway,
      mockEmailService,
      mockInventoryService
    );
  });

  describe('processOrder', () => {
    it('should process a valid order successfully', async () => {
      const order = {
        id: 'ORD-001',
        items: [{ productId: 'PROD-001', quantity: 2 }],
        customerEmail: 'customer@example.com',
        totalAmount: 99.99,
      };

      const result = await orderProcessor.processOrder(order);

      expect(result.success).toBe(true);
      expect(result.orderId).toBe('ORD-001');
      expect(mockPaymentGateway.processPayment).toHaveBeenCalledWith(
        99.99,
        'ORD-001'
      );
      expect(mockInventoryService.reserveItems).toHaveBeenCalledWith([
        { productId: 'PROD-001', quantity: 2 },
      ]);
      expect(mockEmailService.sendOrderConfirmation).toHaveBeenCalledWith(
        'customer@example.com',
        'ORD-001'
      );
    });

    it('should fail order when payment fails', async () => {
      mockPaymentGateway.processPayment.mockRejectedValueOnce(
        new Error('Payment declined')
      );

      const order = {
        id: 'ORD-002',
        items: [{ productId: 'PROD-001', quantity: 1 }],
        customerEmail: 'customer@example.com',
        totalAmount: 50.00,
      };

      const result = await orderProcessor.processOrder(order);

      expect(result.success).toBe(false);
      expect(result.error).toBe('Payment declined');
      expect(mockInventoryService.reserveItems).not.toHaveBeenCalled();
      expect(mockEmailService.sendPaymentFailedNotification).toHaveBeenCalledWith(
        'customer@example.com'
      );
    });

    it('should handle inventory reservation failure', async () => {
      mockInventoryService.reserveItems.mockResolvedValueOnce({
        success: false,
        error: 'Insufficient stock',
      });

      const order = {
        id: 'ORD-003',
        items: [{ productId: 'PROD-001', quantity: 100 }],
        customerEmail: 'customer@example.com',
        totalAmount: 500.00,
      };

      const result = await orderProcessor.processOrder(order);

      expect(result.success).toBe(false);
      expect(result.error).toBe('Insufficient stock');
      expect(mockPaymentGateway.processPayment).not.toHaveBeenCalled();
    });
  });

  describe('cancelOrder', () => {
    it('should refund payment and release inventory for cancelled order', async () => {
      const order = {
        id: 'ORD-004',
        items: [{ productId: 'PROD-001', quantity: 1 }],
        customerEmail: 'customer@example.com',
        totalAmount: 50.00,
      };

      const result = await orderProcessor.cancelOrder(order);

      expect(result.success).toBe(true);
      expect(mockPaymentGateway.refundPayment).toHaveBeenCalledWith('TX123');
      expect(mockInventoryService.releaseItems).toHaveBeenCalledWith([
        { productId: 'PROD-001', quantity: 1 },
      ]);
    });
  }
});

// src/notification/order-processor.ts
export interface OrderResult {
  success: boolean;
  orderId?: string;
  error?: string;
}

export class OrderProcessor {
  constructor(
    private paymentGateway: PaymentGateway,
    private emailService: EmailService,
    private inventoryService: InventoryService
  ) {}

  async processOrder(order: {
    id: string;
    items: Array<{ productId: string; quantity: number }>;
    customerEmail: string;
    totalAmount: number;
  }): Promise<OrderResult> {
    try {
      const inventoryResult = await this.inventoryService.reserveItems(order.items);
      if (!inventoryResult.success) {
        return { success: false, orderId: order.id, error: inventoryResult.error };
      }

      const paymentResult = await this.paymentGateway.processPayment(
        order.totalAmount,
        order.id
      );
      if (!paymentResult.success) {
        await this.inventoryService.releaseItems(order.items);
        await this.emailService.sendPaymentFailedNotification(
          order.customerEmail
        );
        return { success: false, orderId: order.id, error: paymentResult.error };
      }

      await this.emailService.sendOrderConfirmation(order.customerEmail, order.id);
      return { success: true, orderId: order.id };
    } catch (error) {
      return {
        success: false,
        orderId: order.id,
        error: error instanceof Error ? error.message : 'Unknown error',
      };
    }
  }

  async cancelOrder(order: {
    id: string;
    items: Array<{ productId: string; quantity: number }>;
  }): Promise<OrderResult> {
    try {
      await this.paymentGateway.refundPayment('TX123');
      await this.inventoryService.releaseItems(order.items);
      return { success: true, orderId: order.id };
    } catch (error) {
      return {
        success: false,
        orderId: order.id,
        error: error instanceof Error ? error.message : 'Unknown error',
      };
    }
  }
}

// Mock interfaces for dependencies
export interface PaymentGateway {
  processPayment(amount: number, orderId: string): Promise<{
    success: boolean;
    transactionId?: string;
    error?: string;
  }>;
  refundPayment(transactionId: string): Promise<{ success: boolean }>;
}

export interface EmailService {
  sendOrderConfirmation(email: string, orderId: string): Promise<void>;
  sendPaymentFailedNotification(email: string): Promise<void>;
}

export interface InventoryService {
  reserveItems(
    items: Array<{ productId: string; quantity: number }>
  ): Promise<{ success: boolean; error?: string }>;
  releaseItems(
    items: Array<{ productId: string; quantity: number }>
  ): Promise<{ success: boolean }>;
}
```

---

## TDD with Property-Based Testing

```typescript
// src/utils/string-utils.test.ts
import { StringUtils } from './string-utils';
import * as fc from 'fast-check';

describe('StringUtils', () => {
  let stringUtils: StringUtils;

  beforeEach(() => {
    stringUtils = new StringUtils();
  });

  describe('reverse', () => {
    it('should reverse a simple string', () => {
      expect(stringUtils.reverse('hello')).toBe('olleh');
    });

    it('should handle empty string', () => {
      expect(stringUtils.reverse('')).toBe('');
    });

    it('should handle single character', () => {
      expect(stringUtils.reverse('a')).toBe('a');
    });

    it('should preserve palindromes', () => {
      expect(stringUtils.reverse('racecar')).toBe('racecar');
    });

    it('should reverse Unicode characters correctly', () => {
      expect(stringUtils.reverse('café')).toBe('éfac');
    });

    it('should reverse any string - Property-based test', () => {
      fc.assert(
        fc.property(fc.string(), (input) => {
          const reversed = stringUtils.reverse(input);
          const doubleReversed = stringUtils.reverse(reversed);
          expect(doubleReversed).toBe(input);
        })
      );
    });
  });

  describe('isPalindrome', () => {
    it('should return true for palindrome strings', () => {
      expect(stringUtils.isPalindrome('racecar')).toBe(true);
      expect(stringUtils.isPalindrome('level')).toBe(true);
      expect(stringUtils.isPalindrome('')).toBe(true);
    });

    it('should return false for non-palindrome strings', () => {
      expect(stringUtils.isPalindrome('hello')).toBe(false);
      expect(stringUtils.isPalindrome('world')).toBe(false);
    });

    it('should handle case-insensitive palindromes', () => {
      expect(stringUtils.isPalindrome('RaceCar')).toBe(true);
    });

    it('should correctly identify palindromes - Property-based test', () => {
      fc.assert(
        fc.property(fc.string(), (input) => {
          const isPal = stringUtils.isPalindrome(input);
          if (isPal) {
            const reversed = stringUtils.reverse(input);
            expect(reversed.toLowerCase()).toBe(input.toLowerCase());
          }
        })
      );
    });
  });

  describe('truncate', () => {
    it('should not truncate short strings', () => {
      expect(stringUtils.truncate('Hi', 10)).toBe('Hi');
    });

    it('should truncate long strings with ellipsis', () => {
      expect(stringUtils.truncate('Hello World', 8)).toBe('Hello...');
    });

    it('should handle exact length boundary', () => {
      expect(stringUtils.truncate('Hello', 5)).toBe('Hello');
    });

    it('should handle maxLength less than ellipsis length', () => {
      expect(stringUtils.truncate('Hello', 2)).toBe('He');
    });

    it('should respect maxLength constraint - Property-based test', () => {
      fc.assert(
        fc.property(
          fc.string({ minLength: 0, maxLength: 100 }),
          fc.integer({ min: 0, max: 50 }),
          (input, maxLength) => {
            const result = stringUtils.truncate(input, maxLength);
            expect(result.length).toBeLessThanOrEqual(maxLength);
            if (input.length > maxLength) {
              expect(result.endsWith('...')).toBe(true);
            } else {
              expect(result).toBe(input);
            }
          }
        )
      );
    });
  });

  describe('wordCount', () => {
    it('should count words in a sentence', () => {
      expect(stringUtils.wordCount('Hello world')).toBe(2);
    });

    it('should handle multiple spaces', () => {
      expect(stringUtils.wordCount('Hello   world')).toBe(2);
    });

    it('should handle empty string', () => {
      expect(stringUtils.wordCount('')).toBe(0);
    });

    it('should handle leading and trailing whitespace', () => {
      expect(stringUtils.wordCount('  Hello world  ')).toBe(2);
    });
  }
});

// src/utils/string-utils.ts
export class StringUtils {
  reverse(input: string): string {
    return input.split('').reverse().join('');
  }

  isPalindrome(input: string): boolean {
    const cleaned = input.toLowerCase().replace(/[^a-z0-9]/g, '');
    return cleaned === this.reverse(cleaned);
  }

  truncate(input: string, maxLength: number): string {
    if (input.length <= maxLength) {
      return input;
    }
    if (maxLength <= 3) {
      return input.slice(0, maxLength);
    }
    return input.slice(0, maxLength - 3) + '...';
  }

  wordCount(input: string): number {
    if (!input.trim()) {
      return 0;
    }
    return input.trim().split(/\s+/).length;
  }
}
```

---

## TDD with React Components

```typescript
// src/components/counter.test.tsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import { Counter } from './counter';

describe('Counter Component', () => {
  it('should render with initial value of 0', () => {
    render(<Counter />);
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });

  it('should increment counter when increment button is clicked', () => {
    render(<Counter />);
    const incrementButton = screen.getByRole('button', { name: /increment/i });
    fireEvent.click(incrementButton);
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });

  it('should decrement counter when decrement button is clicked', () => {
    render(<Counter />);
    const decrementButton = screen.getByRole('button', { name: /decrement/i });
    fireEvent.click(decrementButton);
    expect(screen.getByText('Count: -1')).toBeInTheDocument();
  });

  it('should reset counter when reset button is clicked', () => {
    render(<Counter />);
    const incrementButton = screen.getByRole('button', { name: /increment/i });
    fireEvent.click(incrementButton);
    fireEvent.click(incrementButton);
    expect(screen.getByText('Count: 2')).toBeInTheDocument();

    const resetButton = screen.getByRole('button', { name: /reset/i });
    fireEvent.click(resetButton);
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });

  it('should not go below minimum value', () => {
    render(<Counter initialValue={0} minValue={0} />);
    const decrementButton = screen.getByRole('button', { name: /decrement/i });
    fireEvent.click(decrementButton);
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });

  it('should accept custom initial value', () => {
    render(<Counter initialValue={10} />);
    expect(screen.getByText('Count: 10')).toBeInTheDocument();
  });
});

// src/components/counter.tsx
import React, { useState } from 'react';

interface CounterProps {
  initialValue?: number;
  minValue?: number;
  maxValue?: number;
}

export const Counter: React.FC<CounterProps> = ({
  initialValue = 0,
  minValue = Number.MIN_SAFE_INTEGER,
  maxValue = Number.MAX_SAFE_INTEGER,
}) => {
  const [count, setCount] = useState(initialValue);

  const increment = () => {
    if (count < maxValue) {
      setCount(count + 1);
    }
  };

  const decrement = () => {
    if (count > minValue) {
      setCount(count - 1);
    }
  };

  const reset = () => {
    setCount(initialValue);
  };

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};
```

---

## TDD Best Practices

### 1. Follow the First Rule of TDD
The first rule of TDD is: "You must write a failing test before you write any production code." This ensures that every line of code is covered by a test and that you are solving the right problem.

### 2. Keep Tests Fast
Fast tests enable rapid feedback and frequent refactoring. Aim for tests that run in milliseconds. Avoid slow operations like database calls, network requests, or file I/O in unit tests by using mocks and stubs.

### 3. Write Isolated Tests
Each test should be independent and not rely on the state from other tests. Use setup and teardown methods to ensure a clean state before each test runs.

### 4. Use Descriptive Test Names
Test names should clearly describe what behavior is being tested. Use the pattern: "should [expected behavior] when [condition]". This makes tests act as documentation for your code.

### 5. Test Behavior, Not Implementation
Focus on testing the public interface and expected behavior rather than internal implementation details. This allows you to refactor the implementation without breaking tests.

### 6. Follow the Triangulation Rule
When you're unsure about the implementation, write multiple tests with different inputs to triangulate the correct solution. This helps you understand the requirements better.

### 7. Apply the "F.I.R.S.T." Principles
- **Fast**: Tests should be fast to enable quick feedback.
- **Independent**: Tests should not depend on each other.
- **Repeatable**: Tests should produce the same result every time.
- **Self-Validating**: Tests should have a clear pass/fail result.
- **Timely**: Tests should be written before the production code.

### 8. Use the "One Assertion per Test" Guideline
While not a strict rule, having one assertion per test makes it easier to understand what failed and why. However, multiple assertions that test the same behavior can be acceptable.

### 9. Maintain Test Code Quality
Tests are code too. Apply the same coding standards, refactoring practices, and design principles to test code as you do to production code.

### 10. Don't Skip Refactoring
The refactoring phase is essential for maintaining code quality. Don't skip it even when under time pressure. The safety net of tests makes refactoring safe and valuable.

---

## TDD Anti-Patterns

### 1. Testing Implementation Details
Testing private methods, internal state, or implementation-specific details makes tests brittle. When you refactor the implementation, tests break even though the behavior hasn't changed.

### 2. Large Test Setup
Tests with extensive setup code are hard to understand and maintain. Extract common setup into helper methods or use test data builders.

### 3. Brittle Tests
Tests that break with every small change indicate they are testing the wrong thing. Focus on behavior rather than implementation.

### 4. Ignoring Test Failures
Never ignore a failing test. It could indicate a regression or that the test is no longer valid. Either fix the code or update the test.

### 5. Testing Third-Party Code
Don't write tests for third-party libraries or frameworks. Trust that they have their own tests. Only test your integration with them.

### 6. Over-Mocked Tests
Too many mocks can make tests useless. Focus on mocking boundaries (external services, databases) but test the interaction between your objects directly.

### 7. Slow Tests
Tests that take too long discourage frequent execution. Keep unit tests fast by avoiding real I/O, databases, or networks.

### 8. Inconsistent Naming
Poorly named tests make it hard to understand what behavior is being tested. Use descriptive names that explain the expected behavior.

### 9. Duplicate Test Code
Repeated setup or assertions across tests indicate an opportunity for extraction. Use helper methods, fixtures, or parameterized tests.

### 10. Testing Trivial Code
Don't waste time testing trivial getters/setters or one-line functions that can't possibly fail. Focus on complex business logic.

---

## TDD in AI-Assisted Development

When working with AI assistants for TDD, follow these practices:

### 1. Provide Clear Requirements First
Before asking the AI to generate code, provide clear specifications of the expected behavior. The AI can then write tests that match these requirements.

### 2. Use the AI to Generate Test Cases
Ask the AI to help identify edge cases and generate comprehensive test scenarios that you might not have considered.

### 3. Leverage AI for Refactoring
AI assistants can help identify refactoring opportunities and ensure that refactored code maintains the same behavior and passes all tests.

### 4. Combine with BDD
Use AI to help translate natural language requirements into Gherkin scenarios or Given-When-Then test structures.

### 5. AI as a TDD Partner
Treat the AI as a TDD partner: describe what you want to test, have it write the test, then implement the code together.

### 6. Validate AI-Generated Tests
Always review and understand AI-generated tests. They should clearly express the intended behavior and not contain logical errors.

### 7. Use AI for Property-Based Testing
AI can help identify properties and invariants that should hold across all inputs, making property-based testing more effective.

---

## Advanced TDD Patterns

### 1. Transformation Priority Premise
Kent Beck's Transformation Priority Premise suggests a order of transformations to apply when moving from a failing test to a passing test:

```
( () -> () )                           // Empty test
↓
( {assertion} -> {return constant} )  // Return a constant
↓
{return constant} -> {return input}   // Return the input
↓
{return input} -> {return input + 1}  // Apply a simple transformation
...
```

### 2. London School vs. Classic TDD
- **London School (Mockist)**: Focus on isolating units with heavy mocking, testing interactions between objects.
- ** Classic TDD (Detroit School)**: Focus on testing observable behavior, using mocks sparingly.

### 3. Outside-In TDD (London School)
Start from the outside (user interface or API) and work inward, using "triangulation" to define the inner components.

### 4. Isolation Paths
Use different testing strategies for different types of code:
- **Pure functions**: Easy to test, no mocking needed
- **Stateful objects**: Test state transitions
- **Collaborative objects**: Test interactions with mocks
- **Side effects**: Test through contracts and boundaries

### 5. The London School Approach
The London School emphasizes:
- Testing interactions between objects with mocks
- Following a strict outside-in development
- Defining collaboration before implementation
- Using mock objects to drive design

### 6. The Classic Approach
The Classic approach emphasizes:
- Testing observable state changes
- Using test doubles sparingly
- Growing the design incrementally
- Trusting in-state verification

---

## Testing Pyramid and TDD

```
                    ┌─────────────┐
                   /   Manual     \
                  /    Testing     \
                 └─────────────────┘
                /    Integration     \
               /       Tests          \
              └───────────────────────┐
             /                         \
            /      End-to-End Tests     \
           /       (Few - Slow)          \
          └─────────────────────────────┐
         /                               \
        /      Integration Tests          \
       /       (Some - Medium)             \
      └─────────────────────────────────┐
     /                                   \
    /       Unit Tests (Many - Fast)      \
   /       (Foundation of TDD)             \
  └───────────────────────────────────────┘
```

---

## References and Further Reading

1. Beck, Kent. "Test-Driven Development: By Example." Addison-Wesley, 2003.
2. Martin, Robert C. "The Three Laws of TDD." Clean Coders Blog, 2009.
3. Freeman, Steve, and Nat Pryce. "Growing Object-Oriented Software, Guided by Tests." Addison-Wesley, 2009.
4. Beck, Kent. "Transformation Priority Premise." TDD Blog, 2011.
5. Meszaros, Gerard. "xUnit Test Patterns: Refactoring Test Code." Addison-Wesley, 2007.
6. "The Three Styles of Test-Driven Development." Google Testing Blog, 2020.
