# Event Sourcing

## Overview

Event Sourcing is an architectural pattern where changes to the application state are stored as a sequence of events rather than maintaining just the current state. This approach provides a complete audit trail, enables temporal queries, and supports sophisticated business workflows.

## Core Concept

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    EVENT SOURCING CONCEPT                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TRADITIONAL APPROACH              EVENT SOURCING                         │
│  ┌───────────────────┐           ┌───────────────────┐                    │
│  │                   │           │                   │                    │
│  │  Account: $1000  │           │  [Event Log]       │                    │
│  │                   │           │                   │                    │
│  │  UPDATE balance  │           │  AccountOpened     │                    │
│  │  TO $800         │           │  → Initial: $1000  │                    │
│  │                   │           │                   │                    │
│  │                   │           │  Deposit           │                    │
│  │  State is overwritten│         │  → +$500 = $1500   │                    │
│  │                   │           │                   │                    │
│  │                   │           │  Withdraw          │                    │
│  │                   │           │  → -$500 = $1000   │                    │
│  │                   │           │                   │                    │
│  │                   │           │  Withdraw          │                    │
│  │                   │           │  → -$200 = $800    │                    │
│  │                   │           │                   │                    │
│  │  Current: $800    │           │  Current: $800      │                    │
│  │  History: LOST     │           │  History: COMPLETE  │                    │
│  │                   │           │                   │                    │
│  └───────────────────┘           └───────────────────┘                    │
│                                                                      │
│  KEY INSIGHT: Events are immutable facts; state is derived from events   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Event Sourcing Structure

```typescript
// Domain events are the source of truth
interface DomainEvent {
  readonly eventId: string;
  readonly aggregateId: string;
  readonly aggregateType: string;
  readonly eventType: string;
  readonly payload: Record<string, unknown>;
  readonly occurredAt: DateTime;
  readonly version: number;
}

// Example events
namespace AccountEvents {
  export class AccountOpened implements DomainEvent {
    readonly eventType = 'AccountOpened';
    
    constructor(
      readonly aggregateId: string,
      readonly payload: {
        ownerId: string;
        initialBalance: number;
        currency: string;
      },
      readonly occurredAt = DateTime.now()
    ) {}
  }

  export class Deposited implements DomainEvent {
    readonly eventType = 'Deposited';
    
    constructor(
      readonly aggregateId: string,
      readonly payload: {
        amount: number;
        depositId: string;
        method: string;
      },
      readonly occurredAt = DateTime.now()
    ) {}
  }

  export class Withdrawn implements DomainEvent {
    readonly eventType = 'Withdrawn';
    
    constructor(
      readonly aggregateId: string,
      readonly payload: {
        amount: number;
        withdrawalId: string;
        method: string;
      },
      readonly occurredAt = DateTime.now()
    ) {}
  }

  export class AccountClosed implements DomainEvent {
    readonly eventType = 'AccountClosed';
    
    constructor(
      readonly aggregateId: string,
      readonly payload: {
        reason: string;
        finalBalance: number;
      },
      readonly occurredAt = DateTime.now()
    ) {}
  }
}
```

## Aggregate with Event Sourcing

```typescript
// Aggregate that derives state from events
class BankAccount implements AggregateRoot<AccountId> {
  private _id: AccountId;
  private _ownerId: string;
  private _balance: number;
  private _currency: string;
  private _status: AccountStatus;
  private _version: number;
  private _uncommittedEvents: DomainEvent[];

  // Private constructor - use factory methods
  private constructor() {
    this._uncommittedEvents = [];
  }

  // Apply event to state
  private apply(event: DomainEvent): void {
    switch (event.eventType) {
      case 'AccountOpened':
        const openEvent = event as AccountEvents.AccountOpened;
        this._id = AccountId.fromString(event.aggregateId);
        this._ownerId = openEvent.payload.ownerId;
        this._balance = openEvent.payload.initialBalance;
        this._currency = openEvent.payload.currency;
        this._status = AccountStatus.ACTIVE;
        this._version = 1;
        break;

      case 'Deposited':
        const depositEvent = event as AccountEvents.Deposited;
        this._balance += depositEvent.payload.amount;
        this._version++;
        break;

      case 'Withdrawn':
        const withdrawEvent = event as AccountEvents.Withdrawn;
        this._balance -= withdrawEvent.payload.amount;
        this._version++;
        break;

      case 'AccountClosed':
        this._status = AccountStatus.CLOSED;
        this._version++;
        break;
    }
  }

  // Replay events to build current state
  static fromEvents(id: AccountId, events: DomainEvent[]): BankAccount {
    const account = new BankAccount();
    events.forEach(event => account.apply(event));
    return account;
  }

  // Command: Deposit money
  deposit(amount: number, method: string): void {
    if (amount <= 0) {
      throw new InvalidOperationError('Deposit amount must be positive');
    }

    const event = new AccountEvents.Deposited(
      this._id.value,
      {
        amount,
        depositId: DepositId.generate().value,
        method
      }
    );

    this.apply(event);
    this._uncommittedEvents.push(event);
  }

  // Command: Withdraw money
  withdraw(amount: number, method: string): void {
    if (amount <= 0) {
      throw new InvalidOperationError('Withdrawal amount must be positive');
    }

    if (this._balance < amount) {
      throw new InsufficientFundsError('Insufficient balance');
    }

    const event = new AccountEvents.Withdrawn(
      this._id.value,
      {
        amount,
        withdrawalId: WithdrawalId.generate().value,
        method
      }
    );

    this.apply(event);
    this._uncommittedEvents.push(event);
  }

  // Get uncommitted events and clear
  getUncommittedEvents(): DomainEvent[] {
    return [...this._uncommittedEvents];
  }

  clearUncommittedEvents(): void {
    this._uncommittedEvents = [];
  }
}
```

## Event Store

```typescript
// Event Store interface
interface EventStore {
  append(events: DomainEvent[]): Promise<void>;
  getEvents(aggregateId: string, fromVersion?: number): Promise<DomainEvent[]>;
  getAllEvents(since?: DateTime): Promise<DomainEvent[]>;
  subscribe(eventType: string, handler: EventHandler): Promise<void>;
}

// Implementation using a database
class PostgresEventStore implements EventStore {
  constructor(
    private readonly pool: Pool,
    private readonly eventPublisher: EventPublisher
  ) {}

  async append(events: DomainEvent[]): Promise<void> {
    const client = await this.pool.connect();
    
    try {
      await client.query('BEGIN');

      for (const event of events) {
        // Check for concurrency
        const currentVersion = await this.getCurrentVersion(
          client,
          event.aggregateId
        );

        if (currentVersion !== event.version - 1) {
          throw new ConcurrencyError(
            `Expected version ${event.version - 1}, ` +
            `but found ${currentVersion}`
          );
        }

        // Store event
        await client.query(
          `INSERT INTO event_store 
           (aggregate_id, aggregate_type, event_type, payload, occurred_at, version)
           VALUES ($1, $2, $3, $4, $5, $6)`,
          [
            event.aggregateId,
            event.aggregateType,
            event.eventType,
            JSON.stringify(event.payload),
            event.occurredAt,
            event.version
          ]
        );
      }

      await client.query('COMMIT');

      // Publish events for other consumers
      for (const event of events) {
        await this.eventPublisher.publish(event);
      }
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }

  private async getCurrentVersion(client: PoolClient, aggregateId: string): Promise<number> {
    const result = await client.query(
      'SELECT COALESCE(MAX(version), 0) FROM event_store WHERE aggregate_id = $1',
      [aggregateId]
    );
    return parseInt(result.rows[0].coalesce);
  }

  async getEvents(aggregateId: string, fromVersion = 0): Promise<DomainEvent[]> {
    const result = await this.pool.query(
      `SELECT * FROM event_store 
       WHERE aggregate_id = $1 AND version > $2
       ORDER BY version ASC`,
      [aggregateId, fromVersion]
    );

    return result.rows.map(row => this.deserializeEvent(row));
  }

  private deserializeEvent(row: any): DomainEvent {
    const payload = JSON.parse(row.payload);
    
    switch (row.event_type) {
      case 'AccountOpened':
        return new AccountEvents.AccountOpened(
          row.aggregate_id,
          payload
        );
      case 'Deposited':
        return new AccountEvents.Deposited(
          row.aggregate_id,
          payload
        );
      case 'Withdrawn':
        return new AccountEvents.Withdrawn(
          row.aggregate_id,
          payload
        );
      default:
        throw new Error(`Unknown event type: ${row.event_type}`);
    }
  }
}
```

## Projections (Read Models)

```typescript
// Project events to read models
interface Projection {
  handles: string[];
  apply(event: DomainEvent): Promise<void>;
}

class AccountBalanceProjection implements Projection {
  handles = ['AccountOpened', 'Deposited', 'Withdrawn', 'AccountClosed'];

  constructor(private readonly readRepository: AccountBalanceRepository) {}

  async apply(event: DomainEvent): Promise<void> {
    switch (event.eventType) {
      case 'AccountOpened':
        await this.readRepository.create({
          accountId: event.aggregateId,
          balance: event.payload.initialBalance,
          lastUpdated: DateTime.now()
        });
        break;

      case 'Deposited':
        await this.readRepository.updateBalance(
          event.aggregateId,
          event.payload.amount,
          'ADD'
        );
        break;

      case 'Withdrawn':
        await this.readRepository.updateBalance(
          event.aggregateId,
          event.payload.amount,
          'SUBTRACT'
        );
        break;

      case 'AccountClosed':
        await this.readRepository.markClosed(event.aggregateId);
        break;
    }
  }
}

// Projection runner
class ProjectionRunner {
  private projections: Projection[];

  constructor(
    private readonly eventStore: EventStore,
    private readonly checkpointStore: CheckpointStore
  ) {
    this.projections = [
      new AccountBalanceProjection(...),
      new TransactionHistoryProjection(...),
      new AccountActivityFeedProjection(...)
    ];
  }

  async run(): Promise<void> {
    let checkpoint = await this.checkpointStore.get();
    
    while (true) {
      const events = await this.eventStore.getAllEvents(checkpoint);
      
      if (events.length === 0) {
        break;
      }

      for (const event of events) {
        for (const projection of this.projections) {
          if (projection.handles.includes(event.eventType)) {
            await projection.apply(event);
          }
        }
      }

      checkpoint = events[events.length - 1].version;
      await this.checkpointStore.save(checkpoint);
    }
  }
}
```

## Temporal Queries

```typescript
// Query state at a point in time
class TemporalAccountService {
  constructor(private readonly eventStore: EventStore) {}

  async getBalanceAt(accountId: string, at: DateTime): Promise<number> {
    const events = await this.eventStore.getEvents(accountId);
    
    let balance = 0;
    
    for (const event of events) {
      if (event.occurredAt > at) {
        break;
      }
      
      switch (event.eventType) {
        case 'AccountOpened':
          balance = event.payload.initialBalance;
          break;
        case 'Deposited':
          balance += event.payload.amount;
          break;
        case 'Withdrawn':
          balance -= event.payload.amount;
          break;
      }
    }
    
    return balance;
  }

  async getTransactionHistory(
    accountId: string,
    from: DateTime,
    to: DateTime
  ): Promise<Transaction[]> {
    const events = await this.eventStore.getEvents(accountId);
    
    return events
      .filter(e => 
        e.occurredAt >= from && 
        e.occurredAt <= to &&
        (e.eventType === 'Deposited' || e.eventType === 'Withdrawn')
      )
      .map(e => ({
        type: e.eventType,
        amount: e.eventType === 'Deposited' 
          ? e.payload.amount 
          : -e.payload.amount,
        date: e.occurredAt
      }));
  }
}
```

## Benefits

| Aspect | Benefit |
|---------|----------|
| **Audit Trail** | Complete history of all changes |
| **Temporal Queries** | Query state at any point in time |
| **Debugging** | Replay events to understand issues |
| **Business Insights** | Analyze event patterns |
| **Reliability** | Events are immutable facts |

## Challenges

| Challenge | Solution |
|-----------|----------|
| Event Versioning | Upcasters, schema evolution |
| Performance | Snapshots, projections |
| Data Size | Archival, compression |
| Complexity | CQRS to separate read/write |

## Event Sourcing with CQRS

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    EVENT SOURCING + CQRS                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   WRITE SIDE                    │    READ SIDE                       │
│   ┌───────────────────┐        │    ┌───────────────────┐           │
│   │                   │        │    │                   │           │
│   │  Commands ───►   │ Events │    │  Projections ◄─── │Events     │
│   │  Aggregate        │───────►│    │  Read Models      │───────►   │
│   │                   │        │    │                   │           │
│   │  Event Store     │        │    │  Query APIs       │           │
│   │  (Source of      │        │    │  (Optimized for  │           │
│   │   Truth)         │        │    │   reads)          │           │
│   │                   │        │    │                   │           │
│   └───────────────────┘        │    └───────────────────┘           │
│                                                                      │
│   Benefits:                        Benefits:                           │
│   - Complete audit trail            - Fast reads                      │
│   - Temporal queries               - Denormalized views              │
│   - Event replay                  - Flexible data models           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## References

- "Event Sourcing" by Martin Fowler
- "Practical Event Sourcing" by Greg Young
- EventStoreDB
- "Versioning in an Event Sourced System" by Greg Young
