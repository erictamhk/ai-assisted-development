# Design by Contract

## Overview

Design by Contract (DbC) is a software design methodology where software components specify preconditions, postconditions, and invariants that define their contract. Formalized by Bertrand Meyer and implemented in Eiffel, this approach ensures that components guarantee their behavior through explicit assertions.

## Core Concepts

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DESIGN BY CONTRACT CONCEPT                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    PRECONDITIONS         INVARIANTS         POSTCONDITIONS               │
│    ┌─────────┐         ┌─────────┐        ┌─────────┐               │
│    │  Must   │  before │  Always │   after │   Must   │               │
│    │  Be     │────────►│  Must   │────────►│   Ensure │               │
│    │  True   │  call   │  Be     │   call  │   True   │               │
│    └─────────┘         └─────────┘        └─────────┘               │
│                                                                      │
│    Caller's         Contract                 Implementer's               │
│    Responsibility   ENFORCED                 Guarantee                 │
│                                                                      │
│    If violated:     The contract            If violated:               │
│    - Caller error   is broken              - Implementation error    │
│    - Call rejected                                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Contract Elements

### Preconditions

```typescript
// Precondition: What must be true before calling a function
class BankAccount {
  private balance: number;
  private readonly minimumBalance: number = 0;

  withdraw(amount: number): void {
    // Precondition: Amount must be positive
    if (amount <= 0) {
      throw new PreconditionViolationError(
        'Withdrawal amount must be positive'
      );
    }

    // Precondition: Sufficient funds
    if (this.balance - amount < this.minimumBalance) {
      throw new PreconditionViolationError(
        'Insufficient funds for withdrawal'
      );
    }

    this.balance -= amount;
  }
}

// Using a contract library
import { contract } from 'contracts.ts';

class BankAccount {
  private balance: number = 0;
  private readonly minimumBalance: number = 0;

  @requires('amount > 0', 'Withdrawal amount must be positive')
  @requires(
    'balance - amount >= minimumBalance',
    'Cannot overdraft account'
  )
  withdraw(amount: number): void {
    this.balance -= amount;
  }
}
```

### Postconditions

```typescript
// Postcondition: What must be true after a function completes
class BankAccount {
  private balance: number;

  deposit(amount: number): number {
    // Precondition
    if (amount <= 0) {
      throw new Error('Deposit amount must be positive');
    }

    const previousBalance = this.balance;
    this.balance += amount;

    // Postcondition: Balance increased by exact amount
    if (this.balance !== previousBalance + amount) {
      throw new PostconditionViolationError(
        'Balance should increase by exact deposit amount'
      );
    }

    // Postcondition: Return new balance
    return this.balance;
  }
}

// Using assertion-based postconditions
class BankAccount {
  private transactions: Transaction[] = [];

  @ensures('result === balance', 'Return current balance')
  getBalance(): number {
    return this.balance;
  }

  @ensures(
    'old(transactions).length + 1 === transactions.length',
    'Transaction should be recorded'
  )
  deposit(amount: number): void {
    this.balance += amount;
    this.transactions.push(new Transaction('deposit', amount));
  }

  @ensures(
    'old(balance) - amount === balance',
    'Balance should decrease by exact amount'
  )
  @ensures(
    'old(transactions).length + 1 === transactions.length',
    'Transaction should be recorded'
  )
  withdraw(amount: number): void {
    this.balance -= amount;
    this.transactions.push(new Transaction('withdrawal', amount));
  }
}
```

### Invariants

```typescript
// Invariant: What must always be true
class BankAccount {
  private _balance: number;
  private readonly _accountNumber: string;
  private readonly _minimumBalance: number = 0;
  private _isClosed: boolean = false;

  // Invariant: Balance cannot go below minimum
  private get balance(): number {
    return this._balance;
  }

  private set balance(value: number) {
    if (value < this._minimumBalance) {
      throw new InvariantViolationError(
        `Balance cannot be below ${this._minimumBalance}`
      );
    }
    this._balance = value;
  }

  // Invariant: Balance cannot be NaN
  constructor(accountNumber: string, initialBalance: number) {
    if (!Number.isFinite(initialBalance)) {
      throw new InvariantViolationError('Balance must be a valid number');
    }
    this._accountNumber = accountNumber;
    this.balance = initialBalance;
  }

  // Invariant: Account number never changes
  get accountNumber(): string {
    return this._accountNumber;
  }

  // Invariant: Closed account cannot be modified
  private set closed(value: boolean) {
    if (this._isClosed && value === false) {
      throw new InvariantViolationError(
        'Cannot reopen a closed account'
      );
    }
    this._isClosed = value;
  }

  close(): void {
    if (this.balance !== 0) {
      throw new InvariantViolationError(
        'Cannot close account with non-zero balance'
      );
    }
    this._isClosed = true;
  }
}
```

## Implementation Approaches

### Runtime Assertion Checking

```typescript
// contracts.ts inspired implementation
type Contract = {
  requires: (condition: string, message: string) => MethodDecorator;
  ensures: (condition: string, message: string) => MethodDecorator;
  invariant: (condition: string, message: string) => ClassDecorator;
};

const contract: Contract = {
  requires(condition, message) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
      const original = descriptor.value;
      descriptor.value = function (...args: any[]) {
        if (!eval(condition)) {
          throw new PreconditionViolationError(message);
        }
        return original.apply(this, args);
      };
    };
  },

  ensures(condition, message) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
      const original = descriptor.value;
      descriptor.value = function (...args: any[]) {
        const result = original.apply(this, args);
        const oldValues = {};
        
        // Capture old values
        Object.keys(eval('result') || {}).forEach(key => {
          (oldValues as any)[`old(${key})`] = (this as any)[key];
        });

        if (!eval(condition)) {
          throw new PostconditionViolationError(message);
        }
        return result;
      };
    };
  },

  invariant(condition, message) {
    return function (constructor: any) {
      const original = constructor;
      return class extends original {
        constructor(...args: any[]) {
          super(...args);
          if (!eval(condition)) {
            throw new InvariantViolationError(message);
          }
        }
      };
    };
  }
};

@contract.invariant('balance >= 0', 'Balance cannot be negative')
class Account {
  @contract.requires('amount > 0', 'Amount must be positive')
  @contract.ensures('old(balance) + amount === balance', 'Balance increased correctly')
  deposit(amount: number): void {
    this.balance += amount;
  }
}
```

### TypeScript with Runtime Validation

```typescript
// Runtime validation with io-ts or zod
import { z } from 'zod';

const DepositSchema = z.object({
  accountId: z.string().uuid(),
  amount: z.number().positive(),
  currency: z.string().length(3),
  description: z.string().max(200).optional()
});

type DepositInput = z.infer<typeof DepositSchema>;

class AccountService {
  async deposit(input: DepositInput): Promise<Account> {
    // Runtime validation of preconditions
    const validated = DepositSchema.parse(input);
    
    const account = await this.repository.findById(validated.accountId);
    
    // Business logic preconditions
    if (account.isClosed) {
      throw new PreconditionViolationError('Cannot deposit to closed account');
    }
    
    // Perform deposit
    account.deposit(validated.amount);
    
    // Postcondition checks
    const newBalance = await account.getBalance();
    if (newBalance !== account.expectedBalance) {
      throw new PostconditionViolationError('Balance mismatch');
    }
    
    return account;
  }
}
```

### Design by Contract in Testing

```typescript
// Contracts as test specifications
import { describe, it, expect } from 'vitest';

describe('BankAccount - Contract Tests', () => {
  describe('Preconditions', () => {
    it('should reject negative withdrawal amounts', () => {
      const account = new BankAccount(100);
      
      expect(() => account.withdraw(-50))
        .toThrow(PreconditionViolationError);
    });

    it('should reject withdrawals exceeding balance', () => {
      const account = new BankAccount(100);
      
      expect(() => account.withdraw(150))
        .toThrow(PreconditionViolationError);
    });
  });

  describe('Postconditions', () => {
    it('should return correct new balance', () => {
      const account = new BankAccount(100);
      
      const newBalance = account.withdraw(30);
      
      expect(newBalance).toBe(70);
    });
  });

  describe('Invariants', () => {
    it('should always maintain non-negative balance', () => {
      const account = new BankAccount(100);
      
      account.withdraw(50);
      account.deposit(20);
      account.withdraw(70);
      
      expect(account.balance).toBeGreaterThanOrEqual(0);
    });

    it('should never allow balance below minimum', () => {
      const account = new BankAccount(100, 50); // minimum 50
      
      expect(() => account.withdraw(60))
        .toThrow(InvariantViolationError);
    });
  });
});
```

## Benefits

| Aspect | Benefit |
|---------|---------|
| **Correctness** | Explicit behavior guarantees |
| **Documentation** | Self-documenting contracts |
| **Debugging** | Clear failure messages |
| **Design** | Forces thinking about edge cases |
| **Testing** | Contracts guide test cases |

## DbC with AI

```markdown
## AI Prompt for Design by Contract

Generate a class with full contracts for the following:

```
Feature: Order Processing System

Requirements:
1. Orders have unique IDs
2. Orders must have at least one item
3. Order status transitions: DRAFT → SUBMITTED → PAID → SHIPPED → DELIVERED
4. Orders can only be submitted if valid
5. Payment must match order total
6. Shipping only after payment
7. Orders cannot be modified after submission
```

Output:
1. TypeScript class with preconditions, postconditions, invariants
2. Contract violations with meaningful error messages
3. Unit tests for contract enforcement
```

## Best Practices

1. **Start with Preconditions**: What must be true before calling?

2. **Define Postconditions**: What will be true after completion?

3. **Identify Invariants**: What must always hold true?

4. **Use Meaningful Messages**: Error messages should explain the contract

5. **Balance Rigor**: Don't over-specify trivial conditions

6. **Combine with Testing**: Contracts complement, not replace, tests

## Anti-Patterns

| Anti-Pattern | Description | Solution |
|--------------|-------------|----------|
| **Over-Contracting** | Contracts on trivial methods | Focus on complex behavior |
| **Weak Contracts** | Vague or useless conditions | Make contracts meaningful |
| **Missing Invariants** | Forgetting class constraints | Identify invariants early |
| **No Enforcement** | Writing contracts but not checking | Use assertion libraries |

## References

- "Object-Oriented Software Construction" by Bertrand Meyer
- "Design by Contract" by Bertrand Meyer
- Contracts.ts library
- Eiffel Programming Language
