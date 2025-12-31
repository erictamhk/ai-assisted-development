# Merging domain-driven-design.md and ddd-patterns.md into domain-driven-design-megred.md

**Session ID:** ses_48cb6817cffeH87WoKtmRlp7ag
**Created:** 31/12/2025, 15:22:35
**Updated:** 31/12/2025, 15:23:53

---

## User

compare @doc\domain-driven-design.md  and @doc\ddd-patterns.md and merge to one file domain-driven-design-megred.md

---

## Assistant

_Thinking:_

The user wants me to compare two files and merge them into one file. Let me first read both files to understand their content, then create a merged version.

The files are:
1. @doc\domain-driven-design.md
2. @doc\ddd-patterns.md

And the output should be: @doc\domain-driven-design-megred.md

Let me read both files first.

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\domain-driven-design.md"
}
```
**Output:**
```
<file>
00001| # Domain-Driven Design
00002| 
00003| ## Overview
00004| 
00005| Domain-Driven Design (DDD) is an approach to software development that emphasizes collaboration between domain experts and developers to create a shared model of the business domain. First articulated by Eric Evans in his 2003 book "Domain-Driven Design: Tackling Complexity in the Heart of Software," DDD provides a framework for tackling complex domain problems through strategic and tactical design patterns.
00006| 
00007| ## Core Concepts
00008| 
00009| ### Strategic Design Patterns
00010| 
00011| Strategic design addresses the "big picture" of the domain architecture:
00012| 
00013| ```
00014| ┌─────────────────────────────────────────────────────────────────┐
00015| │                    DOMAIN LANDSCAPE                             │
00016| ├─────────────────────────────────────────────────────────────────┤
00017| │                                                                 │
00018| │   ┌─────────────┐         ┌─────────────┐         ┌──────────┐ │
00019| │   │   CORE      │         │   CORE      │         │ SUPPORTED│ │
00020| │   │   DOMAIN    │◄────────┤   DOMAIN    │────────►│  DOMAIN  │ │
00021| │   │             │  Shared │             │ Conformist│          │ │
00022| │   │  Complex &  │  Kernel │ Complex &  │         │ Simple & │ │
00023| │   │  Valuable   │         │  Valuable  │         │ Commodity│ │
00024| │   └─────────────┘         └─────────────┘         └──────────┘ │
00025| │                                                                 │
00026| │   ┌───────────────────────────────────────────────────────┐   │
00027| │   │                    BOUNDED CONTEXTS                   │   │
00028| │   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │   │
00029| │   │  │OrderCtx │  │InvCtx   │  │PayCtx   │  │UserCtx  │  │   │
00030| │   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │   │
00031| │   └───────────────────────────────────────────────────────┘   │
00032| │                                                                 │
00033| └─────────────────────────────────────────────────────────────────┘
00034| ```
00035| 
00036| ### The Ubiquitous Language
00037| 
00038| The Ubiquitous Language is the shared vocabulary used by all team members:
00039| 
00040| ```typescript
00041| // Anti-pattern: Mixing technical and domain language
00042| class OrderService {
00043|   async processOrd() {
00044|     const ord = await this.repo.findById(id);
00045|     if (ord.status !== 'PENDING') throw new Error('Invalid');
00046|     // ... technical jargon confuses domain experts
00047|   }
00048| }
00049| 
00050| // DDD Pattern: Ubiquitous Language
00051| class OrderApplicationService {
00052|   constructor(
00053|     private readonly orderRepository: OrderRepository,
00054|     private readonly paymentGateway: PaymentGateway,
00055|     private readonly inventoryService: InventoryService
00056|   ) {}
00057| 
00058|   async submitOrder(orderId: OrderId): Promise<SubmitOrderResult> {
00059|     const order = await this.orderRepository.findById(orderId);
00060|     
00061|     // Domain experts understand this:
00062|     if (!order.canBeSubmitted()) {
00063|       throw new OrderCannotBeSubmittedError(order.status);
00064|     }
00065|     
00066|     order.submit();
00067|     
00068|     await this.paymentGateway.charge(order.paymentMethod, order.total);
00069|     await this.inventoryService.reserveItems(order.lineItems);
00070|     
00071|     order.confirm();
00072|     await this.orderRepository.save(order);
00073|     
00074|     return SubmitOrderResult.success(order.confirmationNumber);
00075|   }
00076| }
00077| ```
00078| 
00079| ## Bounded Contexts
00080| 
00081| A Bounded Context is a boundary within which a particular domain model applies:
00082| 
00083| ```typescript
00084| // Order Bounded Context
00085| namespace OrderContext {
00086|   export interface Order {
00087|     id: OrderId;
00088|     status: OrderStatus;
00089|     lineItems: OrderLineItem[];
00090|     customer: Customer;
00091|     shippingAddress: Address;
00092|     paymentMethod: PaymentMethod;
00093|     total: Money;
00094|     confirmationNumber: ConfirmationNumber;
00095|   }
00096| 
00097|   export enum OrderStatus {
00098|     DRAFT = 'DRAFT',
00099|     PENDING = 'PENDING',
00100|     SUBMITTED = 'SUBMITTED',
00101|     CONFIRMED = 'CONFIRMED',
00102|     SHIPPED = 'SHIPPED',
00103|     DELIVERED = 'DELIVERED',
00104|     CANCELLED = 'CANCELLED'
00105|   }
00106| 
00107|   export class Order {
00108|     submit(): void {
00109|       if (this.status !== OrderStatus.DRAFT) {
00110|         throw new InvalidOrderStateTransitionError();
00111|       }
00112|       this.status = OrderStatus.SUBMITTED;
00113|       this.addDomainEvent(new OrderSubmittedEvent(this.id));
00114|     }
00115|   }
00116| }
00117| 
00118| // Customer Bounded Context - Different model for same concept
00119| namespace CustomerContext {
00120|   export interface CustomerProfile {
00121|     id: CustomerId;
00122|     contactInfo: ContactInformation;
00123|     preferences: CustomerPreferences;
00124|     loyaltyTier: LoyaltyTier;
00125|     accountBalance: Money;
00126|   }
00127| }
00128| ```
00129| 
00130| ## Building Blocks
00131| 
00132| ### Entities
00133| 
00134| ```typescript
00135| class Order implements Entity<OrderId> {
00136|   private readonly id: OrderId;
00137|   private status: OrderStatus;
00138|   private lineItems: OrderLineItem[];
00139|   private readonly createdAt: DateTime;
00140|   private version: number;
00141| 
00142|   constructor(id: OrderId, lineItems: OrderLineItem[]) {
00143|     this.id = id;
00144|     this.lineItems = lineItems;
00145|     this.status = OrderStatus.DRAFT;
00146|     this.createdAt = DateTime.now();
00147|     this.version = 1;
00148|   }
00149| 
00150|   get id(): OrderId { return this.id; }
00151|   get total(): Money { return this.lineItems.reduce((sum, item) => sum.add(item.subtotal), Money.zero()); }
00152|   
00153|   isSameIdentity(other: Order): boolean {
00154|     return this.id.equals(other.id);
00155|   }
00156| }
00157| ```
00158| 
00159| ### Value Objects
00160| 
00161| ```typescript
00162| class Money implements ValueObject {
00163|   private readonly amount: number;
00164|   private readonly currency: Currency;
00165| 
00166|   constructor(amount: number, currency: Currency) {
00167|     if (amount < 0) throw new MoneyCannotBeNegativeError();
00168|     this.amount = Math.round(amount * 100) / 100; // Precision
00169|     this.currency = currency;
00170|   }
00171| 
00172|   static zero(currency: Currency): Money {
00173|     return new Money(0, currency);
00174|   }
00175| 
00176|   add(other: Money): Money {
00177|     this.ensureSameCurrency(other);
00178|     return new Money(this.amount + other.amount, this.currency);
00179|   }
00180| 
00181|   multiply(factor: number): Money {
00182|     return new Money(this.amount * factor, this.currency);
00183|   }
00184| 
00185|   private ensureSameCurrency(other: Money): void {
00186|     if (!this.currency.equals(other.currency)) {
00187|       throw new CurrencyMismatchError();
00188|     }
00189|   }
00190| }
00191| ```
00192| 
00193| ### Aggregates
00194| 
00195| ```typescript
00196| // Order Aggregate Root
00197| class Order implements AggregateRoot<OrderId> {
00198|   private _id: OrderId;
00199|   private _status: OrderStatus;
00200|   private _lineItems: OrderLineItem[];
00201|   private _customer: CustomerReference;
00202|   private _shippingAddress: Address;
00203|   private _paymentInfo: PaymentInfo;
00204|   private _domainEvents: DomainEvent[];
00205| 
00206|   get id(): OrderId { return this._id; }
00207|   get lineItems(): OrderLineItem[] { return [...this._lineItems]; }
00208|   get status(): OrderStatus { return this._status; }
00209| 
00210|   // Public interface for modifying aggregate
00211|   addLineItem(product: Product, quantity: Quantity): void {
00212|     if (this._status !== OrderStatus.DRAFT) {
00213|       throw new CannotModifyOrderError();
00214|     }
00215|     
00216|     const existing = this._lineItems.find(li => li.productId.equals(product.id));
00217|     if (existing) {
00218|       existing.increaseQuantity(quantity);
00219|     } else {
00220|       this._lineItems.push(OrderLineItem.create(product, quantity));
00221|     }
00222|   }
00223| 
00224|   submitForFulfillment(): void {
00225|     this.validateBusinessRules();
00226|     this._status = OrderStatus.SUBMITTED;
00227|     this._domainEvents.push(new OrderSubmittedEvent(this._id));
00228|   }
00229| 
00230|   private validateBusinessRules(): void {
00231|     if (this._lineItems.length === 0) {
00232|       throw new OrderMustHaveAtLeastOneLineItemError();
00233|     }
00234|     if (!this._shippingAddress.isValid()) {
00235|       throw new InvalidShippingAddressError();
00236|     }
00237|   }
00238| }
00239| ```
00240| 
00241| ### Domain Events
00242| 
00243| ```typescript
00244| // Domain Event
00245| class OrderSubmitted implements DomainEvent {
00246|   readonly orderId: OrderId;
00247|   readonly customerId: CustomerId;
00248|   readonly totalAmount: Money;
00249|   readonly submittedAt: DateTime;
00250|   readonly eventId: EventId;
00251|   readonly occurredAt: DateTime;
00252| 
00253|   constructor(
00254|     orderId: OrderId,
00255|     customerId: CustomerId,
00256|     totalAmount: Money
00257|   ) {
00258|     this.orderId = orderId;
00259|     this.customerId = customerId;
00260|     this.totalAmount = totalAmount;
00261|     this.submittedAt = DateTime.now();
00262|     this.eventId = EventId.generate();
00263|     this.occurredAt = DateTime.now();
00264|   }
00265| }
00266| 
00267| // Publishing and handling
00268| class Order extends AggregateRoot<OrderId> {
00269|   submit(): void {
00270|     // Business logic
00271|     this._status = OrderStatus.SUBMITTED;
00272|     
00273|     // Raise domain event
00274|     this.recordEvent(new OrderSubmitted(
00275|       this._id,
00276|       this._customerId,
00277|       this.calculateTotal()
00278|     ));
00279|   }
00280| }
00281| ```
00282| 
00283| ### Repositories
00284| 
00285| ```typescript
00286| interface OrderRepository {
00287|   findById(id: OrderId): Promise<Order | null>;
00288|   findByCustomerId(customerId: CustomerId): Promise<Order[]>;
00289|   findByStatus(status: OrderStatus): Promise<Order[]>;
00290|   save(order: Order): Promise<void>;
00291|   delete(order: Order): Promise<void>;
00292| }
00293| 
00294| // Implementation
00295| class PostgresOrderRepository implements OrderRepository {
00296|   async findById(id: OrderId): Promise<Order | null> {
00297|     const result = await this.db.query(
00298|       'SELECT * FROM orders WHERE id = $1',
00299|       [id.value]
00300|     );
00301|     
00302|     if (result.rows.length === 0) return null;
00303|     
00304|     return OrderMapper.toDomain(result.rows[0]);
00305|   }
00306| 
00307|   async save(order: Order): Promise<void> {
00308|     await this.db.transaction(async (tx) => {
00309|       await tx.query(
00310|         `INSERT INTO orders (id, status, created_at, version, data)
00311|          VALUES ($1, $2, $3, $4, $5)
00312|          ON CONFLICT (id) DO UPDATE SET status = $2, version = $4, data = $5`,
00313|         [order.id.value, order.status, order.createdAt, order.version + 1, OrderMapper.toPersistence(order)]
00314|       );
00315|       
00316|       // Publish domain events
00317|       for (const event of order.getUncommittedEvents()) {
00318|         await this.eventStore.publish(event);
00319|       }
00320|     });
00321|   }
00322| }
00323| ```
00324| 
00325| ### Domain Services
00326| 
00327| ```typescript
00328| class OrderFactory implements DomainService {
00329|   constructor(
00330|     private readonly productRepository: ProductRepository,
00331|     private readonly pricingService: PricingService
00332|   ) {}
00333| 
00334|   createOrder(customerId: CustomerId, items: OrderItemInput[]): Order {
00335|     const lineItems = items.map(input => {
00336|       const product = this.productRepository.findById(input.productId)
00337|         .orElseThrow(() => new ProductNotFoundError(input.productId));
00338|       
00339|       const price = this.pricingService.calculatePrice(
00340|         product.id,
00341|         input.quantity,
00342|         customerId
00343|       );
00344|       
00345|       return OrderLineItem.create(product, input.quantity, price);
00346|     });
00347|     
00348|     const orderId = OrderId.generate();
00349|     return new Order(orderId, customerId, lineItems);
00350|   }
00351| }
00352| ```
00353| 
00354| ## Context Mapping
00355| 
00356| ```typescript
00357| // Context Map showing relationships between bounded contexts
00358| const contextMap = {
00359|   // Upstream-Downstream relationship
00360|   upstream: {
00361|     name: 'OrderManagement',
00362|     responsibilities: [
00363|       'Order creation',
00364|       'Order lifecycle management',
00365|       'Order history'
00366|     ]
00367|   },
00368|   
00369|   downstream: {
00370|     name: 'Inventory',
00371|     relationship: 'Customer-Supplier',
00372|     protocol: 'Domain Events'
00373|   },
00374|   
00375|   // Anticorruption Layer
00376|   inventoryACL: {
00377|     name: 'InventoryACL',
00378|     role: 'Translate between contexts',
00379|     implementations: [
00380|       'Event consumer from Inventory context',
00381|       'Adapter for inventory data format',
00382|       'Translator for Inventory-specific concepts'
00383|     ]
00384|   },
00385|   
00386|   // Shared Kernel
00387|   shared: {
00388|     name: 'CoreTypes',
00389|     sharedBy: ['OrderContext', 'CustomerContext', 'InventoryContext'],
00390|     artifacts: [
00391|       'Money (value object)',
00392|       'Address (value object)',
00393|       'Event types (domain events)'
00394|     ]
00395|   }
00396| };
00397| ```
00398| 
00399| ## DDD in AI-Assisted Development
00400| 
00401| ```typescript
00402| // AI Prompts for DDD
00403| const dddPrompt = `Implement the following domain model following DDD principles:
00404| 
00405| Domain: E-commerce Order Processing
00406| 
00407| Bounded Context: Order Management
00408| 
00409| Entities:
00410| - Order (aggregate root)
00411| - OrderLineItem
00412| - Customer
00413| 
00414| Value Objects:
00415| - Money (with currency)
00416| - Address
00417| - OrderId
00418| - Quantity
00419| 
00420| Aggregate Rules:
00421| 1. Order is the only entry point for modifications
00422| 2. Order status transitions: DRAFT → SUBMITTED → CONFIRMED → SHIPPED → DELIVERED
00423| 3. Total is calculated from line items
00424| 4. Inventory must be checked before submission
00425| 
00426| Domain Events:
00427| - OrderCreated
00428| - OrderSubmitted
00429| - OrderConfirmed
00430| - OrderShipped
00431| - OrderCancelled
00432| 
00433| Please generate:
00434| 1. TypeScript interfaces for entities and value objects
00435| 2. Aggregate root with business logic
00436| 3. Domain events
00437| 4. Repository interface
00438| 5. Unit tests for aggregate invariants
00439| `;
00440| ```
00441| 
00442| ## Best Practices
00443| 
00444| 1. **Start with Strategic Design**: Map bounded contexts before modeling entities
00445| 
00446| 2. **Use the Ubiquitous Language**: Ensure all code reflects domain terminology
00447| 
00448| 3. **Protect Aggregate Invariants**: Enforce business rules within aggregates
00449| 
00450| 4. **Design Around Behavior**: Focus on what the domain does, not just data structures
00451| 
00452| 5. **Separate Contexts**: Don't let bounded contexts leak into each other
00453| 
00454| 6. **Leverage Domain Events**: Decouple contexts through event-driven communication
00455| 
00456| 7. **Iterate with Domain Experts**: Continue refining the model through collaboration
00457| 
00458| ## Anti-Patterns to Avoid
00459| 
00460| | Anti-Pattern | Description | Solution |
00461| |--------------|-------------|----------|
00462| | Anemic Domain Model | Entities without behavior | Move logic from services to domain objects |
00463| | God Objects | One entity doing too much | Split into multiple aggregates |
00464| | Anemic Aggregates | Violating aggregate invariants | Enforce rules in aggregate root |
00465| | Context Bleed | Leaking concepts between contexts | Define clear boundaries |
00466| | Anemic Services | Services with no domain logic | Move behavior to domain |
00467| 
00468| ## References
00469| 
00470| - "Domain-Driven Design" by Eric Evans
00471| - "Implementing Domain-Driven Design" by Vaughn Vernon
00472| - "Domain-Driven Design Reference" by Eric Evans
00473| - "Patterns, Principles, and Practices of Domain-Driven Design" by Scott Millett
00474| 

(End of file - total 474 lines)
</file>
```
```

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\ddd-patterns.md"
}
```
**Output:**
```
<file>
00001| # Domain-Driven Design Patterns
00002| 
00003| ## Core Concept
00004| 
00005| Domain-Driven Design (DDD) is an approach to software development that focuses on modeling the business domain accurately and completely. The key insight is that complex business domains should be reflected in the code structure, making the software a faithful representation of the business concepts it serves.
00006| 
00007| DDD provides a set of patterns for modeling domain logic, organizing code around business concepts, and managing the complexity of large systems. These patterns help bridge the gap between domain experts' mental models and the actual software implementation.
00008| 
00009| The patterns in this document have been extracted from real-world refactoring experience and represent universal, timeless principles that apply across different projects and technology stacks.
00010| 
00011| ---
00012| 
00013| ## Entity Pattern
00014| 
00015| ### Definition
00016| Entities are domain objects that have a distinct identity that persists through time and state changes. Unlike value objects, entities are compared by identity, not by their attributes.
00017| 
00018| ### Key Characteristics
00019| - Has a unique identity (ID) that distinguishes it from other entities
00020| - Equality is based on identity, not attributes
00021| - Has a lifecycle (creation, modification, deletion)
00022| - Encapsulates business behavior and rules
00023| - Can change state over time
00024| 
00025| ### Implementation
00026| 
00027| ```typescript
00028| // Entity Interface
00029| interface Entity<TId> {
00030|   getId(): TId;
00031| }
00032| 
00033| // Concrete Entity
00034| class Project implements Entity<ProjectId> {
00035|   constructor(
00036|     private readonly id: ProjectId,
00037|     private name: ProjectName,
00038|     private tasks: TaskCollection,
00039|     private createdAt: Date
00040|   ) {}
00041| 
00042|   getId(): ProjectId {
00043|     return this.id;
00044|   }
00045| 
00046|   getName(): ProjectName {
00047|     return this.name;
00048|   }
00049| 
00050|   getTasks(): ReadonlyArray<Task> {
00051|     return this.tasks.getAll();
00052|   }
00053| 
00054|   getCreatedAt(): Date {
00055|     return this.createdAt;
00056|   }
00057| 
00058|   // Business behavior
00059|   addTask(description: TaskDescription): Task {
00060|     const task = new Task(
00061|       TaskId.generate(),
00062|       description,
00063|       false,
00064|       this.id
00065|     );
00066|     this.tasks.add(task);
00067|     return task;
00068|   }
00069| 
00070|   removeTask(taskId: TaskId): void {
00071|     this.tasks.remove(taskId);
00072|   }
00073| 
00074|   rename(newName: ProjectName): void {
00075|     this.name = newName;
00076|   }
00077| 
00078|   // Equality by identity
00079|   equals(other: Entity<ProjectId>): boolean {
00080|     if (other === null || other === undefined) return false;
00081|     return this.id.equals(other.getId());
00082|   }
00083| }
00084| ```
00085| 
00086| ### When to Use Entities
00087| - When the object has a lifecycle (can be created, modified, deleted)
00088| - When the object needs to be tracked and referenced
00089| - When equality should be based on identity, not attributes
00090| - When the object has behavior that changes its state
00091| 
00092| ---
00093| 
00094| ## Value Object Pattern
00095| 
00096| ### Definition
00097| Value Objects are immutable objects that are defined solely by their attributes. They have no identity and are compared by their attribute values.
00098| 
00099| ### Key Characteristics
00100| - Immutable (once created, cannot be changed)
00101| - No identity (equality is based on attributes)
00102| - Self-validating in constructor
00103| - Represents a descriptive aspect of the domain
00104| - Replaces primitive obsession
00105| 
00106| ### Implementation
00107| 
00108| ```typescript
00109| // Value Object with validation
00110| record ProjectName implements ValueObject {
00111|   private readonly value: string;
00112| 
00113|   constructor(value: string) {
00114|     // Validation
00115|     if (value === null || value.trim().length === 0) {
00116|       throw new DomainError('Project name cannot be empty');
00117|     }
00118|     if (value.length > 100) {
00119|       throw new DomainError('Project name cannot exceed 100 characters');
00120|     }
00121|     this.value = value.trim();
00122|   }
00123| 
00124|   getValue(): string {
00125|     return this.value;
00126|   }
00127| 
00128|   equals(other: ValueObject): boolean {
00129|     return other instanceof ProjectName && this.value === other.value;
00130|   }
00131| 
00132|   // Behavior within value object
00133|   containsKeyword(keyword: string): boolean {
00134|     return this.value.toLowerCase().includes(keyword.toLowerCase());
00135|   }
00136| }
00137| 
00138| // Complex Value Object
00139| record Money implements ValueObject {
00140|   private readonly amount: number;
00141|   private readonly currency: Currency;
00142| 
00143|   constructor(amount: number, currency: Currency) {
00144|     if (amount < 0) {
00145|       throw new DomainError('Amount cannot be negative');
00146|     }
00147|     this.amount = Math.round(amount * 100) / 100; // Round to 2 decimal places
00148|     this.currency = currency;
00149|   }
00150| 
00151|   add(other: Money): Money {
00152|     if (!this.currency.equals(other.currency)) {
00153|       throw new DomainError('Cannot add different currencies');
00154|     }
00155|     return new Money(this.amount + other.amount, this.currency);
00156|   }
00157| 
00158|   multiply(factor: number): Money {
00159|     return new Money(this.amount * factor, this.currency);
00160|   }
00161| 
00162|   equals(other: ValueObject): boolean {
00163|     return other instanceof Money && 
00164|            this.amount === other.amount &&
00165|            this.currency.equals(other.currency);
00166|   }
00167| }
00168| 
00169| // Factory Method for Value Objects
00170| class TaskId implements ValueObject {
00171|   private readonly value: string;
00172| 
00173|   private constructor(value: string) {
00174|     this.value = value;
00175|   }
00176| 
00177|   // Factory method - the standard creation pattern
00178|   static of(value: string): TaskId {
00179|     if (!value || value.trim().length === 0) {
00180|       throw new DomainError('Task ID cannot be empty');
00181|     }
00182|     return new TaskId(value);
00183|   }
00184| 
00185|   // Alternative factory for generation
00186|   static generate(): TaskId {
00187|     return new TaskId(crypto.randomUUID());
00188|   }
00189| 
00190|   equals(other: ValueObject): boolean {
00191|     return other instanceof TaskId && this.value === other.value;
00192|   }
00193| }
00194| ```
00195| 
00196| ### When to Use Value Objects
00197| - When the object represents a descriptive aspect without identity
00198| - When objects with the same attributes should be considered equal
00199| - When the object should be immutable
00200| - When validation belongs with the data
00201| - When replacing primitive types with meaningful domain concepts
00202| 
00203| ---
00204| 
00205| ## Aggregate Root Pattern
00206| 
00207| ### Definition
00208| An Aggregate Root is a special Entity that serves as the entry point to a group of related entities and value objects (the Aggregate). The aggregate root controls access to all other objects within the aggregate and enforces consistency rules.
00209| 
00210| ### Key Characteristics
00211| - Is an Entity with additional responsibilities
00212| - Controls consistency boundaries
00213| - Is the only entry point to the aggregate
00214| - Manages references to other objects in the aggregate
00215| - Ensures invariants are maintained
00216| 
00217| ### Implementation
00218| 
00219| ```typescript
00220| // Aggregate Root
00221| class ToDoList implements Entity<ToDoListId> {
00222|   constructor(
00223|     private readonly id: ToDoListId,
00224|     private name: ToDoListName,
00225|     private projects: ProjectCollection,
00226|     private createdAt: Date
00227|   ) {}
00228| 
00229|   getId(): ToDoListId {
00230|     return this.id;
00231|   }
00232| 
00233|   // Business operations
00234|   addProject(projectName: string): Project {
00235|     const project = new Project(
00236|       ProjectId.generate(),
00237|       ProjectName.of(projectName),
00238|       new TaskCollection(),
00239|       this.id
00240|     );
00241|     this.projects.add(project);
00242|     return project;
00243|   }
00244| 
00245|   findProject(name: ProjectName): Project | null {
00246|     return this.projects.findByName(name);
00247|   }
00248| 
00249|   getAllProjects(): ReadonlyArray<Project> {
00250|     return this.projects.getAll();
00251|   }
00252| 
00253|   addTaskToProject(
00254|     projectName: string, 
00255|     taskDescription: string
00256|   ): Task {
00257|     const project = this.findProject(ProjectName.of(projectName));
00258|     if (!project) {
00259|       throw new DomainError(`Project not found: ${projectName.value}`);
00260|     }
00261|     return project.addTask(TaskDescription.of(taskDescription));
00262|   }
00263| 
00264|   // Consistency enforcement
00265|   removeProject(projectId: ProjectId): void {
00266|     const project = this.projects.findById(projectId);
00267|     if (!project) {
00268|       throw new DomainError('Project not found');
00269|     }
00270|     // Ensure consistency: remove all tasks from project first
00271|     project.removeAllTasks();
00272|     this.projects.remove(projectId);
00273|   }
00274| }
00275| 
00276| // Collection Wrapper for Aggregate
00277| class ProjectCollection {
00278|   private projects: Map<string, Project> = new Map();
00279| 
00280|   add(project: Project): void {
00281|     if (this.projects.has(project.getId().value)) {
00282|       throw new DomainError('Project already exists');
00283|     }
00284|     this.projects.set(project.getId().value, project);
00285|   }
00286| 
00287|   remove(projectId: ProjectId): void {
00288|     this.projects.delete(projectId.value);
00289|   }
00290| 
00291|   findById(id: ProjectId): Project | null {
00292|     return this.projects.get(id.value) || null;
00293|   }
00294| 
00295|   findByName(name: ProjectName): Project | null {
00296|     for (const project of this.projects.values()) {
00297|       if (project.getName().equals(name)) {
00298|         return project;
00299|       }
00300|     }
00301|     return null;
00302|   }
00303| 
00304|   getAll(): ReadonlyArray<Project> {
00305|     return Array.from(this.projects.values());
00306|   }
00307| 
00308|   size(): number {
00309|     return this.projects.size;
00310|   }
00311| }
00312| ```
00313| 
00314| ### Aggregate Design Rules
00315| 1. **Reference by ID only**: External references should use identity, not direct object references
00316| 2. **Single entry point**: Only the aggregate root can be accessed from outside
00317| 3. **Consistency boundary**: All invariants are enforced within the aggregate
00318| 4. **Small aggregates**: Keep aggregates small to avoid performance issues
00319| 
00320| ---
00321| 
00322| ## Read-Only Interface Pattern
00323| 
00324| ### Definition
00325| Read-Only interfaces expose entities or value objects without modification capabilities, preventing unintended state changes.
00326| 
00327| ### Implementation
00328| 
00329| ```typescript
00330| // Read-Only Interface
00331| interface ReadOnlyProject {
00332|   getId(): ProjectId;
00333|   getName(): ProjectName;
00334|   getTasks(): ReadonlyArray<ReadOnlyTask>;
00335|   containsTask(taskId: TaskId): boolean;
00336|   getTaskCount(): number;
00337| }
00338| 
00339| // Read-Only Implementation
00340| class ReadOnlyProjectImpl implements ReadOnlyProject {
00341|   constructor(private project: Project) {}
00342| 
00343|   getId(): ProjectId {
00344|     return this.project.getId();
00345|   }
00346| 
00347|   getName(): ProjectName {
00348|     return this.project.getName();
00349|   }
00350| 
00351|   getTasks(): ReadonlyArray<ReadOnlyTask> {
00352|     return this.project.getTasks()
00353|       .map(task => new ReadOnlyTaskImpl(task));
00354|   }
00355| 
00356|   containsTask(taskId: TaskId): boolean {
00357|     return this.project.containsTask(taskId);
00358|   }
00359| 
00360|   getTaskCount(): number {
00361|     return this.project.getTaskCount();
00362|   }
00363| }
00364| 
00365| // Throws on modification attempts
00366| class ReadOnlyTaskImpl implements ReadOnlyTask {
00367|   constructor(private task: Task) {}
00368| 
00369|   getId(): TaskId {
00370|     return this.task.getId();
00371|   }
00372| 
00373|   getDescription(): TaskDescription {
00374|     return this.task.getDescription();
00375|   }
00376| 
00377|   isDone(): boolean {
00378|     return this.task.isDone();
00379|   }
00380| 
00381|   // This method throws - prevents modification
00382|   setDone(done: boolean): never {
00383|     throw new UnsupportedOperationException('Read-only view');
00384|   }
00385| }
00386| ```
00387| 
00388| ### Use Cases for Read-Only Views
00389| - Returning entities from query operations
00390| - Preventing modification in presentation layer
00391| - Sharing entities across boundaries safely
00392| - Implementing observer patterns
00393| 
00394| ---
00395| 
00396| ## Repository Pattern
00397| 
00398| ### Definition
00399| The Repository pattern abstracts data access behind a collection-like interface, hiding persistence details from the domain.
00400| 
00401| ### Implementation
00402| 
00403| ```typescript
00404| // Repository Interface (Port - defined in domain)
00405| interface ProjectRepository {
00406|   save(project: Project): void;
00407|   findById(id: ProjectId): Promise<Project | null>;
00408|   findByName(name: ProjectName): Promise<Project | null>;
00409|   delete(id: ProjectId): Promise<void>;
00410|   exists(id: ProjectId): Promise<boolean>;
00411| }
00412| 
00413| // In-Memory Implementation (for testing)
00414| class InMemoryProjectRepository implements ProjectRepository {
00415|   private projects: Map<string, Project> = new Map();
00416| 
00417|   save(project: Project): void {
00418|     this.projects.set(project.getId().value, project);
00419|   }
00420| 
00421|   async findById(id: ProjectId): Promise<Project | null> {
00422|     return this.projects.get(id.value) || null;
00423|   }
00424| 
00425|   async findByName(name: ProjectName): Promise<Project | null> {
00426|     for (const project of this.projects.values()) {
00427|       if (project.getName().equals(name)) {
00428|         return project;
00429|       }
00430|     }
00431|     return null;
00432|   }
00433| 
00434|   async delete(id: ProjectId): Promise<void> {
00435|     this.projects.delete(id.value);
00436|   }
00437| 
00438|   async exists(id: ProjectId): Promise<boolean> {
00439|     return this.projects.has(id.value);
00440|   }
00441| }
00442| 
00443| // CRUD Implementation (for production)
00444| class CrudProjectRepository implements ProjectRepository {
00445|   constructor(
00446|     private entityManager: EntityManager,
00447|     private poClass: typeof ProjectPo
00448|   ) {}
00449| 
00450|   async save(project: Project): Promise<void> {
00451|     const po = ProjectMapper.toPo(project);
00452|     await this.entityManager.save(po);
00453|   }
00454| 
00455|   async findById(id: ProjectId): Promise<Project | null> {
00456|     const po = await this.entityManager.findOne(this.poClass, {
00457|       where: { id: id.value }
00458|     });
00459|     return po ? ProjectMapper.toDomain(po) : null;
00460|   }
00461| 
00462|   async findByName(name: ProjectName): Promise<Project | null> {
00463|     const po = await this.entityManager.findOne(this.poClass, {
00464|       where: { name: name.value }
00465|     });
00466|     return po ? ProjectMapper.toDomain(po) : null;
00467|   }
00468| 
00469|   async delete(id: ProjectId): Promise<void> {
00470|     await this.entityManager.delete(this.poClass, {
00471|       where: { id: id.value }
00472|     });
00473|   }
00474| 
00475|   async exists(id: ProjectId): Promise<boolean> {
00476|     const count = await this.entityManager.count(this.poClass, {
00477|       where: { id: id.value }
00478|     });
00479|     return count > 0;
00480|   }
00481| }
00482| ```
00483| 
00484| ---
00485| 
00486| ## Factory Pattern
00487| 
00488| ### Definition
00489| Factories encapsulate the creation logic for complex objects, especially when creation involves validation or requires assembling multiple components.
00490| 
00491| ### Implementation
00492| 
00493| ```typescript
00494| // Static Factory Method
00495| class Task implements Entity<TaskId> {
00496|   private constructor(
00497|     private readonly id: TaskId,
00498|     private description: TaskDescription,
00499|     private isDone: boolean,
00500|     private projectId: ProjectId,
00501|     private createdAt: Date
00502|   ) {}
00503| 
00504|   // Static factory method - the standard
00505|   static create(
00506|     description: string,
00507|     projectId: string
00508|   ): Task {
00509|     return new Task(
00510|       TaskId.generate(),
00511|       TaskDescription.of(description),
00512|       false,
00513|       ProjectId.of(projectId),
00514|       new Date()
00515|     );
00516|   }
00517| 
00518|   // Factory from external data
00519|   static fromPersistence(
00520|     id: string,
00521|     description: string,
00522|     isDone: boolean,
00523|     projectId: string,
00524|     createdAt: Date
00525|   ): Task {
00526|     return new Task(
00527|       TaskId.of(id),
00528|       TaskDescription.of(description),
00529|       isDone,
00530|       ProjectId.of(projectId),
00531|       createdAt
00532|     );
00533|   }
00534| }
00535| 
00536| // Builder for complex construction
00537| class ProjectBuilder {
00538|   private name?: ProjectName;
00539|   private tasks: Task[] = [];
00540|   private createdAt: Date = new Date();
00541| 
00542|   withName(name: string): this {
00543|     this.name = ProjectName.of(name);
00544|     return this;
00545|   }
00546| 
00547|   withTask(task: Task): this {
00548|     this.tasks.push(task);
00549|     return this;
00550|   }
00551| 
00552|   withCreatedAt(date: Date): this {
00553|     this.createdAt = date;
00554|     return this;
00555|   }
00556| 
00557|   build(): Project {
00558|     if (!this.name) {
00559|       throw new DomainError('Project name is required');
00560|     }
00561|     return new Project(
00562|       this.name,
00563|       new TaskCollection(this.tasks),
00564|       this.createdAt
00565|     );
00566|   }
00567| }
00568| 
00569| // Usage
00570| const project = new ProjectBuilder()
00571|   .withName('My Project')
00572|   .withTask(task1)
00573|   .withTask(task2)
00574|   .build();
00575| ```
00576| 
00577| ---
00578| 
00579| ## Domain Service Pattern
00580| 
00581| ### Definition
00582| Domain Services encapsulate business logic that doesn't naturally belong to a single Entity or Value Object, especially operations that involve multiple domain objects.
00583| 
00584| ### Implementation
00585| 
00586| ```typescript
00587| interface ProjectTransferService {
00588|   transferTask(
00589|     taskId: TaskId,
00590|     fromProjectId: ProjectId,
00591|     toProjectId: ProjectId
00592|   ): Promise<CqrsOutput>;
00593| }
00594| 
00595| class ProjectTransferServiceImpl implements ProjectTransferService {
00596|   constructor(
00597|     private toDoListRepository: ToDoListRepository
00598|   ) {}
00599| 
00600|   async execute(
00601|     input: TransferTaskInput
00602|   ): Promise<CqrsOutput> {
00603|     // Load aggregate root
00604|     const toDoList = await this.toDoListRepository.findById(
00605|       ToDoListId.of(input.toDoListId)
00606|     );
00607| 
00608|     if (!toDoList) {
00609|       return CqrsOutput.create()
00610|         .fail()
00611|         .setMessage('ToDoList not found');
00612|     }
00613| 
00614|     // Find both projects
00615|     const fromProject = toDoList.findProject(
00616|       ProjectName.of(input.fromProjectName)
00617|     );
00618|     const toProject = toDoList.findProject(
00619|       ProjectName.of(input.toProjectName)
00620|     );
00621| 
00622|     if (!fromProject || !toProject) {
00623|       return CqrsOutput.create()
00624|         .fail()
00625|         .setMessage('Source or target project not found');
00626|     }
00627| 
00628|     // Perform transfer
00629|     const task = fromProject.removeTask(TaskId.of(input.taskId));
00630|     toProject.addTask(task.getDescription());
00631| 
00632|     // Persist
00633|     await this.toDoListRepository.save(toDoList);
00634| 
00635|     return CqrsOutput.create().succeed();
00636|   }
00637| }
00638| ```
00639| 
00640| ---
00641| 
00642| ## Anti-Patterns to Avoid
00643| 
00644| ### 1. Anemic Domain Model
00645| ```typescript
00646| // WRONG - Data without behavior
00647| class Project {
00648|   id: string;
00649|   name: string;
00650|   tasks: Task[];
00651| }
00652| 
00653| // Behavior in separate service
00654| class ProjectService {
00655|   createProject(name: string): Project { ... }
00656|   renameProject(project: Project, name: string): void { ... }
00657|   addTask(project: Project, task: Task): void { ... }
00658| }
00659| ```
00660| 
00661| ### 2. Anemic Value Objects
00662| ```typescript
00663| // WRONG - Value object without validation
00664| record Email(string value) {
00665|   // No validation - allows invalid emails
00666| }
00667| ```
00668| 
00669| ### 3. Large Aggregates
00670| ```typescript
00671| // WRONG - Aggregate with too many objects
00672| class EnterpriseSystem implements Entity<SystemId> {
00673|   private departments: Department[];  // Many
00674|   private employees: Employee[];      // Many
00675|   private projects: Project[];        // Many
00676|   private offices: Office[];          // Many
00677|   // ...
00678| }
00679| ```
00680| 
00681| ### 4. Public Setters on Entities
00682| ```typescript
00683| // WRONG - Exposing state modification
00684| class Project {
00685|   public id: string;
00686|   public name: string;  // Can be modified directly
00687|   public tasks: Task[]; // Can be mutated externally
00688| }
00689| ```
00690| 
00691| ---
00692| 
00693| ## References and Further Reading
00694| 
00695| 1. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
00696| 2. Vernon, Vaughn. "Implementing Domain-Driven Design." Addison-Wesley, 2013.
00697| 3. Vernon, Vaughn. "Domain-Driven Design Distilled." Addison-Wesley, 2016.
00698| 4. "Domain-Driven Design Community." https://domaindrivendesign.org/
00699| 5. "Effective Aggregate Design." Vaughn Vernon.
00700| 

(End of file - total 700 lines)
</file>
```
```

---

## Assistant

_Thinking:_

Now I have read both files. Let me analyze them:

**domain-driven-design.md (474 lines)**
- Overview and Core Concepts
- Strategic Design Patterns with ASCII diagram
- Ubiquitous Language section with code examples
- Bounded Contexts section
- Building Blocks: Entities, Value Objects, Aggregates, Domain Events, Repositories, Domain Services
- Context Mapping with code examples
- DDD in AI-Assisted Development
- Best Practices
- Anti-Patterns to Avoid
- References

**ddd-patterns.md (700 lines)**
- Core Concept
- Entity Pattern (detailed implementation)
- Value Object Pattern (detailed implementation)
- Aggregate Root Pattern (detailed implementation)
- Read-Only Interface Pattern
- Repository Pattern
- Factory Pattern
- Domain Service Pattern
- Anti-Patterns to Avoid
- References

I need to merge these into a single comprehensive file. I'll create an organized structure that combines the best of both documents:

1. Overview/Core Concepts
2. Strategic Design (from domain-driven-design.md)
3. Ubiquitous Language
4. Bounded Contexts
5. Tactical Design Patterns (building blocks):
   - Entities
   - Value Objects
   - Aggregates
   - Domain Events
   - Repositories
   - Factories
   - Domain Services
   - Read-Only Interfaces
6. Context Mapping
7. DDD in AI-Assisted Development
8. Best Practices
9. Anti-Patterns to Avoid
10. References

I'll write a comprehensive merged file.

```
Tool: write

**Input:**
```json
{
  "content": "# Domain-Driven Design: Patterns and Principles\n\n## Overview\n\nDomain-Driven Design (DDD) is an approach to software development that emphasizes collaboration between domain experts and developers to create a shared model of the business domain. First articulated by Eric Evans in his 2003 book \"Domain-Driven Design: Tackling Complexity in the Heart of Software,\" DDD provides a framework for tackling complex domain problems through strategic and tactical design patterns.\n\nDDD is an approach to software development that focuses on modeling the business domain accurately and completely. The key insight is that complex business domains should be reflected in the code structure, making the software a faithful representation of the business concepts it serves.\n\nDDD provides a set of patterns for modeling domain logic, organizing code around business concepts, and managing the complexity of large systems. These patterns help bridge the gap between domain experts' mental models and the actual software implementation.\n\nThe patterns in this document have been extracted from real-world refactoring experience and represent universal, timeless principles that apply across different projects and technology stacks.\n\n---\n\n## Strategic Design\n\nStrategic design addresses the \"big picture\" of the domain architecture:\n\n```\n┌─────────────────────────────────────────────────────────────────┐\n│                    DOMAIN LANDSCAPE                             │\n├─────────────────────────────────────────────────────────────────┤\n│                                                                 │\n│   ┌─────────────┐         ┌─────────────┐         ┌──────────┐ │\n│   │   CORE      │         │   CORE      │         │ SUPPORTED│ │\n│   │   DOMAIN    │◄────────┤   DOMAIN    │────────►│  DOMAIN  │ │\n│   │             │  Shared │             │ Conformist│          │ │\n│   │  Complex &  │  Kernel │ Complex &  │         │ Simple & │ │\n│   │  Valuable   │         │  Valuable  │         │ Commodity│ │\n│   └─────────────┘         └─────────────┘         └──────────┘ │\n│                                                                 │\n│   ┌───────────────────────────────────────────────────────┐   │\n│   │                    BOUNDED CONTEXTS                   │   │\n│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │   │\n│   │  │OrderCtx │  │InvCtx   │  │PayCtx   │  │UserCtx  │  │   │\n│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │   │\n│   └───────────────────────────────────────────────────────┘   │\n│                                                                 │\n└─────────────────────────────────────────────────────────────────┘\n```\n\n---\n\n## The Ubiquitous Language\n\nThe Ubiquitous Language is the shared vocabulary used by all team members:\n\n```typescript\n// Anti-pattern: Mixing technical and domain language\nclass OrderService {\n  async processOrd() {\n    const ord = await this.repo.findById(id);\n    if (ord.status !== 'PENDING') throw new Error('Invalid');\n    // ... technical jargon confuses domain experts\n  }\n}\n\n// DDD Pattern: Ubiquitous Language\nclass OrderApplicationService {\n  constructor(\n    private readonly orderRepository: OrderRepository,\n    private readonly paymentGateway: PaymentGateway,\n    private readonly inventoryService: InventoryService\n  ) {}\n\n  async submitOrder(orderId: OrderId): Promise<SubmitOrderResult> {\n    const order = await this.orderRepository.findById(orderId);\n    \n    // Domain experts understand this:\n    if (!order.canBeSubmitted()) {\n      throw new OrderCannotBeSubmittedError(order.status);\n    }\n    \n    order.submit();\n    \n    await this.paymentGateway.charge(order.paymentMethod, order.total);\n    await this.inventoryService.reserveItems(order.lineItems);\n    \n    order.confirm();\n    await this.orderRepository.save(order);\n    \n    return SubmitOrderResult.success(order.confirmationNumber);\n  }\n}\n```\n\n---\n\n## Bounded Contexts\n\nA Bounded Context is a boundary within which a particular domain model applies:\n\n```typescript\n// Order Bounded Context\nnamespace OrderContext {\n  export interface Order {\n    id: OrderId;\n    status: OrderStatus;\n    lineItems: OrderLineItem[];\n    customer: Customer;\n    shippingAddress: Address;\n    paymentMethod: PaymentMethod;\n    total: Money;\n    confirmationNumber: ConfirmationNumber;\n  }\n\n  export enum OrderStatus {\n    DRAFT = 'DRAFT',\n    PENDING = 'PENDING',\n    SUBMITTED = 'SUBMITTED',\n    CONFIRMED = 'CONFIRMED',\n    SHIPPED = 'SHIPPED',\n    DELIVERED = 'DELIVERED',\n    CANCELLED = 'CANCELLED'\n  }\n\n  export class Order {\n    submit(): void {\n      if (this.status !== OrderStatus.DRAFT) {\n        throw new InvalidOrderStateTransitionError();\n      }\n      this.status = OrderStatus.SUBMITTED;\n      this.addDomainEvent(new OrderSubmittedEvent(this.id));\n    }\n  }\n}\n\n// Customer Bounded Context - Different model for same concept\nnamespace CustomerContext {\n  export interface CustomerProfile {\n    id: CustomerId;\n    contactInfo: ContactInformation;\n    preferences: CustomerPreferences;\n    loyaltyTier: LoyaltyTier;\n    accountBalance: Money;\n  }\n}\n```\n\n---\n\n## Tactical Design Patterns\n\n### Entity Pattern\n\nEntities are domain objects that have a distinct identity that persists through time and state changes. Unlike value objects, entities are compared by identity, not by their attributes.\n\n**Key Characteristics:**\n- Has a unique identity (ID) that distinguishes it from other entities\n- Equality is based on identity, not attributes\n- Has a lifecycle (creation, modification, deletion)\n- Encapsulates business behavior and rules\n- Can change state over time\n\n**Implementation:**\n\n```typescript\ninterface Entity<TId> {\n  getId(): TId;\n}\n\nclass Order implements Entity<OrderId> {\n  private readonly id: OrderId;\n  private status: OrderStatus;\n  private lineItems: OrderLineItem[];\n  private readonly createdAt: DateTime;\n  private version: number;\n\n  constructor(id: OrderId, lineItems: OrderLineItem[]) {\n    this.id = id;\n    this.lineItems = lineItems;\n    this.status = OrderStatus.DRAFT;\n    this.createdAt = DateTime.now();\n    this.version = 1;\n  }\n\n  get id(): OrderId { return this.id; }\n  get total(): Money { return this.lineItems.reduce((sum, item) => sum.add(item.subtotal), Money.zero()); }\n  \n  isSameIdentity(other: Order): boolean {\n    return this.id.equals(other.id);\n  }\n}\n```\n\n**When to Use Entities:**\n- When the object has a lifecycle (can be created, modified, deleted)\n- When the object needs to be tracked and referenced\n- When equality should be based on identity, not attributes\n- When the object has behavior that changes its state\n\n---\n\n### Value Object Pattern\n\nValue Objects are immutable objects that are defined solely by their attributes. They have no identity and are compared by their attribute values.\n\n**Key Characteristics:**\n- Immutable (once created, cannot be changed)\n- No identity (equality is based on attributes)\n- Self-validating in constructor\n- Represents a descriptive aspect of the domain\n- Replaces primitive obsession\n\n**Implementation:**\n\n```typescript\nclass Money implements ValueObject {\n  private readonly amount: number;\n  private readonly currency: Currency;\n\n  constructor(amount: number, currency: Currency) {\n    if (amount < 0) throw new MoneyCannotBeNegativeError();\n    this.amount = Math.round(amount * 100) / 100; // Precision\n    this.currency = currency;\n  }\n\n  static zero(currency: Currency): Money {\n    return new Money(0, currency);\n  }\n\n  add(other: Money): Money {\n    this.ensureSameCurrency(other);\n    return new Money(this.amount + other.amount, this.currency);\n  }\n\n  multiply(factor: number): Money {\n    return new Money(this.amount * factor, this.currency);\n  }\n\n  private ensureSameCurrency(other: Money): void {\n    if (!this.currency.equals(other.currency)) {\n      throw new CurrencyMismatchError();\n    }\n  }\n}\n\nrecord ProjectName implements ValueObject {\n  private readonly value: string;\n\n  constructor(value: string) {\n    if (value === null || value.trim().length === 0) {\n      throw new DomainError('Project name cannot be empty');\n    }\n    if (value.length > 100) {\n      throw new DomainError('Project name cannot exceed 100 characters');\n    }\n    this.value = value.trim();\n  }\n\n  getValue(): string {\n    return this.value;\n  }\n\n  equals(other: ValueObject): boolean {\n    return other instanceof ProjectName && this.value === other.value;\n  }\n\n  containsKeyword(keyword: string): boolean {\n    return this.value.toLowerCase().includes(keyword.toLowerCase());\n  }\n}\n```\n\n**When to Use Value Objects:**\n- When the object represents a descriptive aspect without identity\n- When objects with the same attributes should be considered equal\n- When the object should be immutable\n- When validation belongs with the data\n- When replacing primitive types with meaningful domain concepts\n\n---\n\n### Aggregate Root Pattern\n\nAn Aggregate Root is a special Entity that serves as the entry point to a group of related entities and value objects (the Aggregate). The aggregate root controls access to all other objects within the aggregate and enforces consistency rules.\n\n**Key Characteristics:**\n- Is an Entity with additional responsibilities\n- Controls consistency boundaries\n- Is the only entry point to the aggregate\n- Manages references to other objects in the aggregate\n- Ensures invariants are maintained\n\n**Implementation:**\n\n```typescript\nclass Order implements AggregateRoot<OrderId> {\n  private _id: OrderId;\n  private _status: OrderStatus;\n  private _lineItems: OrderLineItem[];\n  private _customer: CustomerReference;\n  private _shippingAddress: Address;\n  private _paymentInfo: PaymentInfo;\n  private _domainEvents: DomainEvent[];\n\n  get id(): OrderId { return this._id; }\n  get lineItems(): OrderLineItem[] { return [...this._lineItems]; }\n  get status(): OrderStatus { return this._status; }\n\n  // Public interface for modifying aggregate\n  addLineItem(product: Product, quantity: Quantity): void {\n    if (this._status !== OrderStatus.DRAFT) {\n      throw new CannotModifyOrderError();\n    }\n    \n    const existing = this._lineItems.find(li => li.productId.equals(product.id));\n    if (existing) {\n      existing.increaseQuantity(quantity);\n    } else {\n      this._lineItems.push(OrderLineItem.create(product, quantity));\n    }\n  }\n\n  submitForFulfillment(): void {\n    this.validateBusinessRules();\n    this._status = OrderStatus.SUBMITTED;\n    this._domainEvents.push(new OrderSubmittedEvent(this._id));\n  }\n\n  private validateBusinessRules(): void {\n    if (this._lineItems.length === 0) {\n      throw new OrderMustHaveAtLeastOneLineItemError();\n    }\n    if (!this._shippingAddress.isValid()) {\n      throw new InvalidShippingAddressError();\n    }\n  }\n}\n```\n\n**Aggregate Design Rules:**\n1. **Reference by ID only**: External references should use identity, not direct object references\n2. **Single entry point**: Only the aggregate root can be accessed from outside\n3. **Consistency boundary**: All invariants are enforced within the aggregate\n4. **Small aggregates**: Keep aggregates small to avoid performance issues\n\n---\n\n### Domain Events Pattern\n\nDomain events capture something significant that happened in the domain, enabling loose coupling between bounded contexts.\n\n**Implementation:**\n\n```typescript\nclass OrderSubmitted implements DomainEvent {\n  readonly orderId: OrderId;\n  readonly customerId: CustomerId;\n  readonly totalAmount: Money;\n  readonly submittedAt: DateTime;\n  readonly eventId: EventId;\n  readonly occurredAt: DateTime;\n\n  constructor(\n    orderId: OrderId,\n    customerId: CustomerId,\n    totalAmount: Money\n  ) {\n    this.orderId = orderId;\n    this.customerId = customerId;\n    this.totalAmount = totalAmount;\n    this.submittedAt = DateTime.now();\n    this.eventId = EventId.generate();\n    this.occurredAt = DateTime.now();\n  }\n}\n\nclass Order extends AggregateRoot<OrderId> {\n  submit(): void {\n    this._status = OrderStatus.SUBMITTED;\n    this.recordEvent(new OrderSubmitted(\n      this._id,\n      this._customerId,\n      this.calculateTotal()\n    ));\n  }\n}\n```\n\n---\n\n### Repository Pattern\n\nThe Repository pattern abstracts data access behind a collection-like interface, hiding persistence details from the domain.\n\n**Implementation:**\n\n```typescript\ninterface OrderRepository {\n  findById(id: OrderId): Promise<Order | null>;\n  findByCustomerId(customerId: CustomerId): Promise<Order[]>;\n  findByStatus(status: OrderStatus): Promise<Order[]>;\n  save(order: Order): Promise<void>;\n  delete(order: Order): Promise<void>;\n}\n\nclass PostgresOrderRepository implements OrderRepository {\n  async findById(id: OrderId): Promise<Order | null> {\n    const result = await this.db.query(\n      'SELECT * FROM orders WHERE id = $1',\n      [id.value]\n    );\n    \n    if (result.rows.length === 0) return null;\n    \n    return OrderMapper.toDomain(result.rows[0]);\n  }\n\n  async save(order: Order): Promise<void> {\n    await this.db.transaction(async (tx) => {\n      await tx.query(\n        `INSERT INTO orders (id, status, created_at, version, data)\n         VALUES ($1, $2, $3, $4, $5)\n         ON CONFLICT (id) DO UPDATE SET status = $2, version = $4, data = $5`,\n        [order.id.value, order.status, order.createdAt, order.version + 1, OrderMapper.toPersistence(order)]\n      );\n      \n      for (const event of order.getUncommittedEvents()) {\n        await this.eventStore.publish(event);\n      }\n    });\n  }\n}\n```\n\n---\n\n### Factory Pattern\n\nFactories encapsulate the creation logic for complex objects, especially when creation involves validation or requires assembling multiple components.\n\n**Implementation:**\n\n```typescript\nclass OrderFactory implements DomainService {\n  constructor(\n    private readonly productRepository: ProductRepository,\n    private readonly pricingService: PricingService\n  ) {}\n\n  createOrder(customerId: CustomerId, items: OrderItemInput[]): Order {\n    const lineItems = items.map(input => {\n      const product = this.productRepository.findById(input.productId)\n        .orElseThrow(() => new ProductNotFoundError(input.productId));\n      \n      const price = this.pricingService.calculatePrice(\n        product.id,\n        input.quantity,\n        customerId\n      );\n      \n      return OrderLineItem.create(product, input.quantity, price);\n    });\n    \n    const orderId = OrderId.generate();\n    return new Order(orderId, customerId, lineItems);\n  }\n}\n\nclass ProjectBuilder {\n  private name?: ProjectName;\n  private tasks: Task[] = [];\n  private createdAt: Date = new Date();\n\n  withName(name: string): this {\n    this.name = ProjectName.of(name);\n    return this;\n  }\n\n  withTask(task: Task): this {\n    this.tasks.push(task);\n    return this;\n  }\n\n  build(): Project {\n    if (!this.name) {\n      throw new DomainError('Project name is required');\n    }\n    return new Project(\n      this.name,\n      new TaskCollection(this.tasks),\n      this.createdAt\n    );\n  }\n}\n```\n\n---\n\n### Domain Service Pattern\n\nDomain Services encapsulate business logic that doesn't naturally belong to a single Entity or Value Object, especially operations that involve multiple domain objects.\n\n**Implementation:**\n\n```typescript\ninterface ProjectTransferService {\n  transferTask(\n    taskId: TaskId,\n    fromProjectId: ProjectId,\n    toProjectId: ProjectId\n  ): Promise<CqrsOutput>;\n}\n\nclass ProjectTransferServiceImpl implements ProjectTransferService {\n  constructor(\n    private toDoListRepository: ToDoListRepository\n  ) {}\n\n  async execute(input: TransferTaskInput): Promise<CqrsOutput> {\n    const toDoList = await this.toDoListRepository.findById(\n      ToDoListId.of(input.toDoListId)\n    );\n\n    if (!toDoList) {\n      return CqrsOutput.create().fail().setMessage('ToDoList not found');\n    }\n\n    const fromProject = toDoList.findProject(ProjectName.of(input.fromProjectName));\n    const toProject = toDoList.findProject(ProjectName.of(input.toProjectName));\n\n    if (!fromProject || !toProject) {\n      return CqrsOutput.create().fail().setMessage('Source or target project not found');\n    }\n\n    const task = fromProject.removeTask(TaskId.of(input.taskId));\n    toProject.addTask(task.getDescription());\n\n    await this.toDoListRepository.save(toDoList);\n\n    return CqrsOutput.create().succeed();\n  }\n}\n```\n\n---\n\n### Read-Only Interface Pattern\n\nRead-Only interfaces expose entities or value objects without modification capabilities, preventing unintended state changes.\n\n**Implementation:**\n\n```typescript\ninterface ReadOnlyProject {\n  getId(): ProjectId;\n  getName(): ProjectName;\n  getTasks(): ReadonlyArray<ReadOnlyTask>;\n  containsTask(taskId: TaskId): boolean;\n  getTaskCount(): number;\n}\n\nclass ReadOnlyProjectImpl implements ReadOnlyProject {\n  constructor(private project: Project) {}\n\n  getId(): ProjectId {\n    return this.project.getId();\n  }\n\n  getTasks(): ReadonlyArray<ReadOnlyTask> {\n    return this.project.getTasks()\n      .map(task => new ReadOnlyTaskImpl(task));\n  }\n\n  containsTask(taskId: TaskId): boolean {\n    return this.project.containsTask(taskId);\n  }\n\n  getTaskCount(): number {\n    return this.project.getTaskCount();\n  }\n}\n\nclass ReadOnlyTaskImpl implements ReadOnlyTask {\n  constructor(private task: Task) {}\n\n  getId(): TaskId {\n    return this.task.getId();\n  }\n\n  getDescription(): TaskDescription {\n    return this.task.getDescription();\n  }\n\n  isDone(): boolean {\n    return this.task.isDone();\n  }\n\n  setDone(done: boolean): never {\n    throw new UnsupportedOperationException('Read-only view');\n  }\n}\n```\n\n**Use Cases for Read-Only Views:**\n- Returning entities from query operations\n- Preventing modification in presentation layer\n- Sharing entities across boundaries safely\n- Implementing observer patterns\n\n---\n\n## Context Mapping\n\nContext Map showing relationships between bounded contexts:\n\n```typescript\nconst contextMap = {\n  upstream: {\n    name: 'OrderManagement',\n    responsibilities: [\n      'Order creation',\n      'Order lifecycle management',\n      'Order history'\n    ]\n  },\n  \n  downstream: {\n    name: 'Inventory',\n    relationship: 'Customer-Supplier',\n    protocol: 'Domain Events'\n  },\n  \n  inventoryACL: {\n    name: 'InventoryACL',\n    role: 'Translate between contexts',\n    implementations: [\n      'Event consumer from Inventory context',\n      'Adapter for inventory data format',\n      'Translator for Inventory-specific concepts'\n    ]\n  },\n  \n  shared: {\n    name: 'CoreTypes',\n    sharedBy: ['OrderContext', 'CustomerContext', 'InventoryContext'],\n    artifacts: [\n      'Money (value object)',\n      'Address (value object)',\n      'Event types (domain events)'\n    ]\n  }\n};\n```\n\n---\n\n## DDD in AI-Assisted Development\n\n```typescript\nconst dddPrompt = `Implement the following domain model following DDD principles:\n\nDomain: E-commerce Order Processing\n\nBounded Context: Order Management\n\nEntities:\n- Order (aggregate root)\n- OrderLineItem\n- Customer\n\nValue Objects:\n- Money (with currency)\n- Address\n- OrderId\n- Quantity\n\nAggregate Rules:\n1. Order is the only entry point for modifications\n2. Order status transitions: DRAFT → SUBMITTED → CONFIRMED → SHIPPED → DELIVERED\n3. Total is calculated from line items\n4. Inventory must be checked before submission\n\nDomain Events:\n- OrderCreated\n- OrderSubmitted\n- OrderConfirmed\n- OrderShipped\n- OrderCancelled\n\nPlease generate:\n1. TypeScript interfaces for entities and value objects\n2. Aggregate root with business logic\n3. Domain events\n4. Repository interface\n5. Unit tests for aggregate invariants\n`;\n```\n\n---\n\n## Best Practices\n\n1. **Start with Strategic Design**: Map bounded contexts before modeling entities\n\n2. **Use the Ubiquitous Language**: Ensure all code reflects domain terminology\n\n3. **Protect Aggregate Invariants**: Enforce business rules within aggregates\n\n4. **Design Around Behavior**: Focus on what the domain does, not just data structures\n\n5. **Separate Contexts**: Don't let bounded contexts leak into each other\n\n6. **Leverage Domain Events**: Decouple contexts through event-driven communication\n\n7. **Iterate with Domain Experts**: Continue refining the model through collaboration\n\n8. **Keep Aggregates Small**: Design focused aggregates for better performance and consistency\n\n9. **Make Value Objects Immutable**: Prevent side effects by making value objects immutable\n\n10. **Use Factories for Complex Creation**: Encapsulate creation logic where it involves validation or assembly\n\n---\n\n## Anti-Patterns to Avoid\n\n| Anti-Pattern | Description | Solution |\n|--------------|-------------|----------|\n| Anemic Domain Model | Entities without behavior | Move logic from services to domain objects |\n| God Objects | One entity doing too much | Split into multiple aggregates |\n| Anemic Aggregates | Violating aggregate invariants | Enforce rules in aggregate root |\n| Context Bleed | Leaking concepts between contexts | Define clear boundaries |\n| Anemic Services | Services with no domain logic | Move behavior to domain |\n| Large Aggregates | Aggregate with too many objects | Split into smaller aggregates |\n| Public Setters on Entities | Exposing state modification | Use private fields with controlled methods |\n| Value Object Without Validation | Allowing invalid values | Validate in constructor |\n\n```typescript\n// WRONG - Data without behavior (Anemic Domain Model)\nclass Project {\n  id: string;\n  name: string;\n  tasks: Task[];\n}\n\n// Behavior in separate service\nclass ProjectService {\n  createProject(name: string): Project { ... }\n  renameProject(project: Project, name: string): void { ... }\n  addTask(project: Project, task: Task): void { ... }\n}\n\n// WRONG - Value object without validation\nrecord Email(string value) {\n  // No validation - allows invalid emails\n}\n\n// WRONG - Public setters on entities\nclass Project {\n  public id: string;\n  public name: string;  // Can be modified directly\n  public tasks: Task[]; // Can be mutated externally\n}\n```\n\n---\n\n## References\n\n1. Evans, Eric. \"Domain-Driven Design: Tackling Complexity in the Heart of Software.\" Addison-Wesley, 2004.\n\n2. Vernon, Vaughn. \"Implementing Domain-Driven Design.\" Addison-Wesley, 2013.\n\n3. Vernon, Vaughn. \"Domain-Driven Design Distilled.\" Addison-Wesley, 2016.\n\n4. \"Domain-Driven Design Reference\" by Eric Evans\n\n5. \"Patterns, Principles, and Practices of Domain-Driven Design\" by Scott Millett\n\n6. \"Effective Aggregate Design.\" Vaughn Vernon.\n\n7. \"Domain-Driven Design Community.\" https://domaindrivendesign.org/\n",
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\domain-driven-design-megred.md"
}
```
```

---

## Assistant

_Thinking:_

The file has been created successfully. The merged file combines the content from both source files, organizing them into a comprehensive document that covers:

1. Overview (combined from both files)
2. Strategic Design (from domain-driven-design.md)
3. The Ubiquitous Language (from domain-driven-design.md)
4. Bounded Contexts (from domain-driven-design.md)
5. Tactical Design Patterns:
   - Entity Pattern (detailed from ddd-patterns.md)
   - Value Object Pattern (detailed from ddd-patterns.md)
   - Aggregate Root Pattern (detailed from ddd-patterns.md)
   - Domain Events Pattern (from domain-driven-design.md)
   - Repository Pattern (combined)
   - Factory Pattern (from ddd-patterns.md)
   - Domain Service Pattern (combined)
   - Read-Only Interface Pattern (from ddd-patterns.md)
6. Context Mapping (from domain-driven-design.md)
7. DDD in AI-Assisted Development (from domain-driven-design.md)
8. Best Practices (combined)
9. Anti-Patterns to Avoid (combined)
10. References (combined)

The task is complete.

Merged both DDD documents into `doc\domain-driven-design-megred.md`. The combined file includes:

- Overview and strategic design concepts
- Ubiquitous Language and Bounded Contexts
- All tactical patterns (Entities, Value Objects, Aggregates, Domain Events, Repositories, Factories, Domain Services, Read-Only Interfaces)
- Context Mapping
- AI-Assisted Development section
- Best practices and anti-patterns
- Comprehensive references

---

