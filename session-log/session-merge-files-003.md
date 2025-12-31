# Comparing clean-architecture.md hexagonal-architecture.md merging into layered-architecture.md

**Session ID:** ses_48cb364cfffefKI5sa5yDwcfge
**Created:** 31/12/2025, 15:25:58
**Updated:** 31/12/2025, 15:45:33

---

## User

compare @doc\clean-architecture.md  and @doc\hexagonal-architecture.md and merge to one file layered-architecture.md

---

## Assistant

_Thinking:_

I need to compare two files: doc/clean-architecture.md and doc/hexagonal-architecture.md, then merge them into layered-architecture.md.

Let me start by reading both files to understand their content.

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\clean-architecture.md"
}
```
**Output:**
```
<file>
00001| # Clean Architecture
00002| 
00003| ## Overview
00004| 
00005| Clean Architecture is a software design philosophy introduced by Robert C. Martin (Uncle Bob) that emphasizes the separation of concerns through architectural layers. The core principle is that business rules and application logic should not depend on external frameworks, databases, or delivery mechanisms.
00006| 
00007| ## Core Principles
00008| 
00009| ### The Dependency Rule
00010| 
00011| ```
00012|                     ┌───────────────────────────────────────┐
00013|                     │          Frameworks & Tools          │  ← External
00014|                     │  (Web, DB, UI, External Services)   │
00015|                     └───────────────────┬───────────────────┘
00016|                                         │
00017|                     ┌───────────────────▼───────────────────┐
00018|                     │         Interface Adapters          │  ← Entry/Exit
00019|                     │  (Controllers, Gateways, Presenters)│
00020|                     └───────────────────┬───────────────────┘
00021|                                         │
00022|                     ┌───────────────────▼───────────────────┐
00023|                     │          Application Services        │  ← Use Cases
00024|                     │  (Interactors, Use Case Coordinators)│
00025|                     └───────────────────┬───────────────────┘
00026|                                         │
00027|                     ┌───────────────────▼───────────────────┐
00028|                     │           Domain Layer              │  ← Business Rules
00029|                     │  (Entities, Domain Services, Value Objects)│
00030|                     └───────────────────────────────────────┘
00031| 
00032|      Source code dependencies point INWARD toward the domain.
00033|      Inner layers do NOT know about outer layers.
00034| ```
00035| 
00036| ### Key Principles
00037| 
00038| 1. **Independence of Frameworks**: Business logic independent of frameworks
00039| 2. **Testability**: Business rules testable without UI, DB, or external systems
00040| 3. **Independence of UI**: UI can change without changing business logic
00041| 4. **Independence of Database**: Business logic independent of database
00042| 5. **Independence of External Agencies**: Business rules don't know about outside world
00043| 
00044| ## Layer Structure
00045| 
00046| ```typescript
00047| src/
00048| ├── enterprise/
00049| │   └── business-rules/        # Domain Layer ( innermost)
00050| │       ├── entities/
00051| │       ├── value-objects/
00052| │       ├── domain-services/
00053| │       └── events/
00054| │
00055| ├── application/
00056| │   ├── use-cases/            # Application Services
00057| │   │   ├── create-order/
00058| │   │   ├── process-payment/
00059| │   │   └── ship-order/
00060| │   ├── ports/
00061| │   │   ├── inbound/          # Input ports (use case interfaces)
00062| │   │   └── outbound/         # Output ports (repository interfaces)
00063| │   └── coordinators/
00064| │
00065| ├── adapters/
00066| │   ├── inbound/              # Interface Adapters
00067| │   │   ├── http/             # HTTP controllers
00068| │   │   ├── grpc/
00069| │   │   └── messaging/
00070| │   ├── outbound/
00071| │   │   ├── persistence/      # Database adapters
00072| │   │   ├── external-api/     # External service clients
00073| │   │   └── events/           # Event publishers
00074| │   └── presenters/
00075| │
00076| └── frameworks/
00077|     ├── web/                  # Frameworks & Drivers
00078|     │   ├── express/
00079|     │   ├── fastify/
00080|     │   └── swagger/
00081|     ├── database/
00082|     │   ├── typeorm/
00083|     │   ├── prisma/
00084|     │   └── mongoose/
00085|     └── external/
00086|         ├── stripe/
00087|         ├── sendgrid/
00088|         └── aws/
00089| ```
00090| 
00091| ## Implementation Examples
00092| 
00093| ### Domain Layer (Enterprise Business Rules)
00094| 
00095| ```typescript
00096| // entities/order.ts
00097| import { Entity } from '../../shared/entity';
00098| import { Money } from '../../value-objects/money';
00099| import { OrderId } from '../../value-objects/order-id';
00100| import { OrderStatus } from './order-status';
00101| import { OrderLineItem } from './order-line-item';
00102| 
00103| export interface OrderProps {
00104|   id: OrderId;
00105|   customerId: string;
00106|   lineItems: OrderLineItem[];
00107|   status: OrderStatus;
00108|   createdAt: Date;
00109| }
00110| 
00111| export class Order extends Entity<OrderId, OrderProps> {
00112|   private constructor(props: OrderProps) {
00113|     super(props);
00114|   }
00115| 
00116|   static create(props: OrderProps): Order {
00117|     const order = new Order(props);
00118|     order.validate();
00119|     return order;
00120|   }
00121| 
00122|   private validate(): void {
00123|     if (this._props.lineItems.length === 0) {
00124|       throw new Error('Order must have at least one line item');
00125|     }
00126|     if (!this._props.status) {
00127|       throw new Error('Order must have a status');
00128|     }
00129|   }
00130| 
00131|   get total(): Money {
00132|     return this._props.lineItems.reduce(
00133|       (sum, item) => sum.add(item.subtotal),
00134|       Money.zero()
00135|     );
00136|   }
00137| 
00138|   get canBeSubmitted(): boolean {
00139|     return this._props.status === OrderStatus.DRAFT;
00140|   }
00141| 
00142|   submit(): void {
00143|     if (!this.canBeSubmitted) {
00144|       throw new Error('Order cannot be submitted');
00145|     }
00146|     this._props.status = OrderStatus.SUBMITTED;
00147|     this.recordDomainEvent(new OrderSubmitted(this._props.id));
00148|   }
00149| }
00150| 
00151| // value-objects/money.ts
00152| export class Money {
00153|   private constructor(
00154|     private readonly amount: number,
00155|     private readonly currency: string
00156|   ) {}
00157| 
00158|   static zero(currency: string = 'USD'): Money {
00159|     return new Money(0, currency);
00160|   }
00161| 
00162|   static fromAmount(amount: number, currency: string = 'USD'): Money {
00163|     return new Money(Math.round(amount * 100) / 100, currency);
00164|   }
00165| 
00166|   add(other: Money): Money {
00167|     this.ensureSameCurrency(other);
00168|     return new Money(this.amount + other.amount, this.currency);
00169|   }
00170| 
00171|   multiply(factor: number): Money {
00172|     return new Money(this.amount * factor, this.currency);
00173|   }
00174| 
00175|   private ensureSameCurrency(other: Money): void {
00176|     if (this.currency !== other.currency) {
00177|       throw new Error('Cannot operate on different currencies');
00178|     }
00179|   }
00180| }
00181| ```
00182| 
00183| ### Application Layer (Use Cases)
00184| 
00185| ```typescript
00186| // use-cases/create-order/create-order.handler.ts
00187| import { OrderRepository } from '../../ports/outbound/order.repository';
00188| import { InventoryService } from '../../ports/outbound/inventory.service';
00189| import { OrderFactory } from '../../domain/factories/order.factory';
00190| 
00191| export interface CreateOrderInput {
00192|   customerId: string;
00193|   items: Array<{
00194|     productId: string;
00195|     quantity: number;
00196|   }>;
00197| }
00198| 
00199| export interface CreateOrderResult {
00200|   orderId: string;
00201|   total: number;
00202|   status: string;
00203| }
00204| 
00205| export class CreateOrderHandler {
00206|   constructor(
00207|     private readonly orderRepository: OrderRepository,
00208|     private readonly inventoryService: InventoryService,
00209|     private readonly orderFactory: OrderFactory
00210|   ) {}
00211| 
00212|   async execute(input: CreateOrderInput): Promise<CreateOrderResult> {
00213|     // Validate availability
00214|     await this.inventoryService.checkAvailability(input.items);
00215|     
00216|     // Create order using factory
00217|     const order = this.orderFactory.createFromItems(
00218|       input.customerId,
00219|       input.items
00220|     );
00221|     
00222|     // Persist
00223|     await this.orderRepository.save(order);
00224|     
00225|     // Return result
00226|     return {
00227|       orderId: order.id.value,
00228|       total: order.total.amount,
00229|       status: order.status.value
00230|     };
00231|   }
00232| }
00233| 
00234| // use-cases/create-order/create-order.controller.ts
00235| import { Request, Response } from 'express';
00236| import { CreateOrderHandler } from './create-order.handler';
00237| import { CreateOrderInput } from './create-order.handler';
00238| 
00239| export class CreateOrderController {
00240|   constructor(private readonly handler: CreateOrderHandler) {}
00241| 
00242|   async handle(req: Request, res: Response): Promise<void> {
00243|     try {
00244|       const input: CreateOrderInput = {
00245|         customerId: req.body.customerId,
00246|         items: req.body.items.map((item: any) => ({
00247|           productId: item.productId,
00248|           quantity: item.quantity
00249|         }))
00250|       };
00251| 
00252|       const result = await this.handler.execute(input);
00253| 
00254|       res.status(201).json({
00255|         success: true,
00256|         data: result
00257|       });
00258|     } catch (error) {
00259|       res.status(400).json({
00260|         success: false,
00261|         error: error instanceof Error ? error.message : 'Unknown error'
00262|       });
00263|     }
00264|   }
00265| }
00266| ```
00267| 
00268| ### Ports (Interfaces)
00269| 
00270| ```typescript
00271| // ports/inbound/order-input.port.ts
00272| export interface CreateOrderInputPort {
00273|   createOrder(input: CreateOrderInput): Promise<CreateOrderOutput>;
00274| }
00275| 
00276| // ports/outbound/order.repository.ts
00277| export interface OrderRepository {
00278|   findById(id: string): Promise<Order | null>;
00279|   findByCustomerId(customerId: string): Promise<Order[]>;
00280|   save(order: Order): Promise<void>;
00281|   delete(id: string): Promise<void>;
00282| }
00283| 
00284| // ports/outbound/inventory.service.ts
00285| export interface InventoryService {
00286|   checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00287|   reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00288|   releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00289| }
00290| ```
00291| 
00292| ### Adapters (Interface Adapters)
00293| 
00294| ```typescript
00295| // adapters/outbound/typeorm-order.repository.ts
00296| import { OrderRepository } from '../../application/ports/outbound/order.repository';
00297| import { Order } from '../../domain/entities/order';
00298| import { OrderMapper } from './order.mapper';
00299| 
00300| export class TypeORMOrderRepository implements OrderRepository {
00301|   constructor(
00302|     private readonly orderRepository: any,
00303|     private readonly eventPublisher: EventPublisher
00304|   ) {}
00305| 
00306|   async findById(id: string): Promise<Order | null> {
00307|     const entity = await this.orderRepository.findOne({ where: { id } });
00308|     if (!entity) return null;
00309|     return OrderMapper.toDomain(entity);
00310|   }
00311| 
00312|   async save(order: Order): Promise<void> {
00313|     const entity = OrderMapper.toPersistence(order);
00314|     await this.orderRepository.save(entity);
00315|     
00316|     // Publish domain events
00317|     for (const event of order.getUncommittedEvents()) {
00318|       await this.eventPublisher.publish(event);
00319|     }
00320|   }
00321| }
00322| 
00323| // adapters/inbound/http-order.controller.ts
00324| import { Request, Response } from 'express';
00325| import { CreateOrderHandler } from '../../application/use-cases/create-order/create-order.handler';
00326| 
00327| export class HTTPOrderController {
00328|   constructor(private readonly handler: CreateOrderHandler) {}
00329| 
00330|   async createOrder(req: Request, res: Response): Promise<void> {
00331|     const result = await this.handler.execute(req.body);
00332|     res.status(201).json(result);
00333|   }
00334| 
00335|   async getOrder(req: Request, res: Response): Promise<void> {
00336|     const order = await this.handler.getOrder(req.params.id);
00337|     res.status(200).json(order);
00338|   }
00339| }
00340| ```
00341| 
00342| ## Dependency Injection Structure
00343| 
00344| ```typescript
00345| // container.ts - Composition Root
00346| class Container {
00347|   private readonly services = new Map<string, any>();
00348| 
00349|   register<T>(token: string, factory: (container: Container) => T): void {
00350|     this.services.set(token, factory(this));
00351|   }
00352| 
00353|   resolve<T>(token: string): T {
00354|     const factory = this.services.get(token);
00355|     if (!factory) {
00356|       throw new Error(`Service ${token} not registered`);
00357|     }
00358|     return factory(this);
00359|   }
00360| }
00361| 
00362| // Setup
00363| const container = new Container();
00364| 
00365| // Ports
00366| container.register('OrderRepository', () => new TypeORMOrderRepository(...));
00367| container.register('InventoryService', () => new StripeInventoryService(...));
00368| container.register('EventPublisher', () => new KafkaEventPublisher(...));
00369| 
00370| // Use Cases
00371| container.register('CreateOrderHandler', (c) => 
00372|   new CreateOrderHandler(
00373|     c.resolve('OrderRepository'),
00374|     c.resolve('InventoryService'),
00375|     c.resolve('OrderFactory')
00376|   )
00377| );
00378| 
00379| // Controllers
00380| container.register('CreateOrderController', (c) =>
00381|   new HTTPOrderController(c.resolve('CreateOrderHandler'))
00382| );
00383| ```
00384| 
00385| ## Clean Architecture in AI-Assisted Development
00386| 
00387| ```markdown
00388| ## AI Prompt for Clean Architecture
00389| 
00390| Generate a feature following Clean Architecture:
00391| 
00392| Context: E-commerce Payment Processing
00393| 
00394| Requirements:
00395| 1. Process payment for an order
00396| 2. Handle payment failures with retries
00397| 3. Publish events on success/failure
00398| 
00399| Generate:
00400| 1. Domain entities (Payment, PaymentResult)
00401| 2. Value objects (PaymentAmount, PaymentMethod)
00402| 3. Domain events (PaymentProcessed, PaymentFailed)
00403| 4. Use case handler (ProcessPaymentHandler)
00404| 5. Input/output ports (interfaces)
00405| 6. Repository interface for payments
00406| 7. HTTP controller adapter
00407| 
00408| Constraints:
00409| - All dependencies point inward to domain
00410| - Domain has no imports from outer layers
00411| - Use TypeScript
00412| - Include JSDoc documentation
00413| - Add unit tests for domain rules
00414| ```
00415| 
00416| ## Benefits
00417| 
00418| | Benefit | Description |
00419| |---------|-------------|
00420| | Testability | Test business rules without external dependencies |
00421| | Maintainability | Changes to outer layers don't affect inner layers |
00422| | Flexibility | Swap implementations without code changes |
00423| | Scalability | Clear separation enables team scaling |
00424| | Framework Independence | Business logic survives framework changes |
00425| 
00426| ## Anti-Patterns to Avoid
00427| 
00428| | Anti-Pattern | Description | Solution |
00429| |--------------|-------------|----------|
00430| | Anemic Domain | Entities without behavior | Move logic from services to domain |
00431| | Dependency Inversion Violation | Domain depending on frameworks | Use dependency injection |
00432| | Feature Scattered Across Layers | Related code in multiple layers | Organize by feature |
00433| | God Interface | One port for everything | Separate inbound/outbound ports |
00434| 
00435| ## References
00436| 
00437| - "Clean Architecture" by Robert C. Martin
00438| - "Architecture: The Hard Parts" by Neal Ford
00439| - "Hands-On Domain-Driven Design with .NET Core" by Alexey Zimarev
00440| 

(End of file - total 440 lines)
</file>
```
```

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\hexagonal-architecture.md"
}
```
**Output:**
```
<file>
00001| # Hexagonal Architecture (Ports and Adapters)
00002| 
00003| ## Core Concept
00004| 
00005| Hexagonal Architecture, also known as Ports and Adapters, is an architectural pattern that structures applications around the business logic while keeping it independent of external concerns. The architecture creates a hexagonal boundary where the internal application (the domain) communicates with external systems (adapters) through well-defined interfaces (ports).
00006| 
00007| The fundamental insight is that business logic should not depend on databases, web frameworks, or any external system. Instead, external systems depend on abstractions defined by the business logic. This inversion of dependencies ensures that the core application can be tested independently of its delivery mechanisms and that external systems can be replaced without changing business code.
00008| 
00009| The pattern was formalized by Alistair Cockburn as a response to the limitations of layered architectures where business logic often became coupled to infrastructure concerns. By explicitly defining ports (interfaces) and placing adapters at the boundaries, the architecture creates clear separation between what the application does (core) and how it integrates with the outside world (adapters).
00010| 
00011| ---
00012| 
00013| ## Architecture Overview
00014| 
00015| ```
00016| ┌─────────────────────────────────────────────────────────────────────────┐
00017| │                    HEXAGONAL ARCHITECTURE                                │
00018| ├─────────────────────────────────────────────────────────────────────────┤
00019| │                                                                         │
00020| │                         ┌─────────────────────┐                         │
00021| │                         │                     │                         │
00022| │                         │     APPLICATION     │                         │
00023| │                         │        CORE         │                         │
00024| │                         │                     │                         │
00025| │                         │  ┌───────────────┐  │                         │
00026| │                         │  │   DOMAIN      │  │                         │
00027| │                         │  │   LAYER       │  │                         │
00028| │                         │  │               │  │                         │
00029| │                         │  │  • Entities   │  │                         │
00030| │                         │  │  • Value Obs  │  │                         │
00031| │                         │  │  • Aggregates │  │                         │
00032| │                         │  └───────────────┘  │                         │
00033| │                         │           │          │                         │
00034| │                         │           ▼          │                         │
00035| │                         │  ┌───────────────┐  │                         │
00036| │                         │  │   USE CASE    │  │                         │
00037| │                         │  │   LAYER       │  │                         │
00038| │                         │  │               │  │                         │
00039| │                         │  │  • Services   │  │                         │
00040| │                         │  │  • Use Cases  │  │                         │
00041| │                         │  │  • Ports      │  │                         │
00042| │                         │  └───────────────┘  │                         │
00043| │                         │                     │                         │
00044| │                         └──────────┬──────────┘                         │
00045| │                                    │                                    │
00046| │              ┌─────────────────────┼─────────────────────┐              │
00047| │              │                     │                     │              │
00048| │              ▼                     ▼                     ▼              │
00049| │    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │
00050| │    │     PORTS       │  │                 │  │     PORTS       │       │
00051| │    │   (Interfaces)  │  │                 │  │   (Interfaces)  │       │
00052| │    │                 │  │                 │  │                 │       │
00053| │    │  • Input Ports  │  │                 │  │  • Output Ports │       │
00054| │    │    (Use Cases)  │  │                 │  │  • Repositories │       │
00055| │    │                 │  │                 │  │  • Presenters   │       │
00056| │    └────────┬────────┘  │                 │  └────────┬────────┘       │
00057| │             │           │                 │           │                │
00058| │             │           │                 │           │                │
00059| │             ▼           │                 │           ▼                │
00060| │    ┌─────────────────┐  │                 │  ┌─────────────────┐       │
00061| │    │   ADAPTERS      │  │                 │  │   ADAPTERS      │       │
00062| │    │   (Inbound)     │  │                 │  │   (Outbound)    │       │
00063| │    │                 │  │                 │  │                 │       │
00064| │    │  • Web          │  │                 │  │  • Database     │       │
00065| │    │  • Console      │  │                 │  │  • File System  │       │
00066| │    │  • API          │  │                 │  │  • External APIs│       │
00067| │    │  • CLI          │  │                 │  │  • Messaging    │       │
00068| │    └─────────────────┘  │                 │  └─────────────────┘       │
00069| │                         │                 │                            │
00070| └─────────────────────────┼─────────────────┼────────────────────────────┘
00071|                           │                 │
00072|                           ▼                 ▼
00073|               ┌───────────────────┐ ┌───────────────────┐
00074|               │    EXTERNAL       │ │    EXTERNAL       │
00075|               │    SYSTEMS        │ │    SYSTEMS        │
00076|               │                   │ │                   │
00077|               │  • Web Browsers   │ │  • Databases      │
00078|               │  • Mobile Apps    │ │  • File Systems   │
00079|               │  • CLI Users      │ │  • Web Services   │
00080|               └───────────────────┘ └───────────────────┘
00081| ```
00082| 
00083| ### The Three Core Layers
00084| 
00085| **Domain Layer (Core)**
00086| The domain layer contains the business logic of the application. It consists of entities, value objects, aggregates, and domain services. This layer has NO dependencies on any other layer. It defines interfaces (ports) that describe what it needs from the outside world.
00087| 
00088| **Use Case Layer (Application)**
00089| The use case layer contains application-specific business logic. It orchestrates operations by using domain objects and calling through ports to interact with external systems. This layer depends only on domain abstractions.
00090| 
00091| **Adapter Layer (External Integration)**
00092| The adapter layer contains implementations of ports that integrate with external systems. Inbound adapters handle user input, while outbound adapters handle external system integration. Adapters depend on ports, never the reverse.
00093| 
00094| ---
00095| 
00096| ## Ports and Adapters Explained
00097| 
00098| ### Ports (Interfaces)
00099| 
00100| Ports define the boundary between the application core and external systems. They are abstract contracts that specify what the application expects from the outside world.
00101| 
00102| **Input Ports (Use Case Interfaces)**
00103| Input ports define operations that external systems can perform on the application. They represent the application's API for incoming operations.
00104| 
00105| ```typescript
00106| // Input Port - What the outside world can DO
00107| interface AddTaskUseCase {
00108|   execute(input: AddTaskInput): Promise<CqrsOutput>;
00109| }
00110| 
00111| interface GetTaskUseCase {
00112|   execute(input: GetTaskInput): Promise<GetTaskOutput>;
00113| }
00114| 
00115| interface DeleteTaskUseCase {
00116|   execute(input: DeleteTaskInput): Promise<CqrsOutput>;
00117| }
00118| ```
00119| 
00120| **Output Ports (Repository and Presenter Interfaces)**
00121| Output ports define operations that the application needs to perform on external systems. These are implemented by outbound adapters.
00122| 
00123| ```typescript
00124| // Output Port - What the application NEEDS
00125| interface TaskRepository {
00126|   save(task: Task): void;
00127|   findById(id: TaskId): Promise<Task | null>;
00128|   delete(id: TaskId): void;
00129| }
00130| 
00131| interface TaskPresenter {
00132|   present(dto: TaskDto): void;
00133| }
00134| 
00135| interface NotificationService {
00136|   sendNotification(message: string): Promise<void>;
00137| }
00138| ```
00139| 
00140| ### Adapters (Implementations)
00141| 
00142| Adapters implement ports and handle the specifics of interacting with external systems.
00143| 
00144| **Inbound Adapters (Delivery Mechanisms)**
00145| Inbound adapters translate external input into calls to input ports.
00146| 
00147| ```typescript
00148| // Web Controller (Inbound Adapter)
00149| class AddTaskWebController {
00150|   constructor(private addTaskUseCase: AddTaskUseCase) {}
00151| 
00152|   async handleRequest(request: HttpRequest): Promise<HttpResponse> {
00153|     const input = new AddTaskInput();
00154|     input.description = request.body.description;
00155|     input.projectId = request.params.projectId;
00156|     
00157|     const result = await this.addTaskUseCase.execute(input);
00158|     
00159|     return new HttpResponse(
00160|       result.isSuccess ? 200 : 400,
00161|       result.toDto()
00162|     );
00163|   }
00164| }
00165| 
00166| // Console Controller (Inbound Adapter)
00167| class AddTaskConsoleController {
00168|   constructor(private addTaskUseCase: AddTaskUseCase) {}
00169| 
00170|   handleRequest(request: ConsoleRequest): Response {
00171|     const input = new AddTaskInput();
00172|     input.description = request.getArg('description');
00173|     input.projectId = request.getArg('projectId');
00174|     
00175|     const result = this.addTaskUseCase.execute(input);
00176|     return Response.of(result);
00177|   }
00178| }
00179| ```
00180| 
00181| **Outbound Adapters (External System Integrations)**
00182| Outbound adapters implement output ports and handle communication with external systems.
00183| 
00184| ```typescript
00185| // Repository Implementation (Outbound Adapter)
00186| class TaskCrudRepository implements TaskRepository {
00187|   constructor(
00188|     private entityManager: EntityManager,
00189|     private taskPoClass: typeof TaskPo
00190|   ) {}
00191| 
00192|   async save(task: Task): Promise<void> {
00193|     const po = TaskMapper.toPo(task);
00194|     await this.entityManager.save(po);
00195|   }
00196| 
00197|   async findById(id: TaskId): Promise<Task | null> {
00198|     const po = await this.entityManager.findOne(this.taskPoClass, {
00199|       where: { id: id.value }
00200|     });
00201|     return po ? TaskMapper.toDomain(po) : null;
00202|   }
00203| 
00204|   async delete(id: TaskId): Promise<void> {
00205|     await this.entityManager.delete(this.taskPoClass, {
00206|       where: { id: id.value }
00207|     });
00208|   }
00209| }
00210| 
00211| // In-Memory Repository (Outbound Adapter for Testing)
00212| class InMemoryTaskRepository implements TaskRepository {
00213|   private tasks: Map<string, Task> = new Map();
00214| 
00215|   save(task: Task): void {
00216|     this.tasks.set(task.getId().value, task);
00217|   }
00218| 
00219|   findById(id: TaskId): Task | null {
00220|     return this.tasks.get(id.value) || null;
00221|   }
00222| 
00223|   delete(id: TaskId): void {
00224|     this.tasks.delete(id.value);
00225|   }
00226| }
00227| 
00228| // Presenter Implementation (Outbound Adapter)
00229| class TaskConsolePresenter implements TaskPresenter {
00230|   constructor(private output: PrintWriter) {}
00231| 
00232|   present(dto: TaskDto): void {
00233|     this.output.println(`Task: ${dto.description}`);
00234|     this.output.println(`Status: ${dto.isDone ? 'Done' : 'Pending'}`);
00235|     this.output.println(`ID: ${dto.id}`);
00236|   }
00237| }
00238| ```
00239| 
00240| ---
00241| 
00242| ## Dependency Direction
00243| 
00244| The most critical rule in Hexagonal Architecture is that **dependencies always point inward** toward the domain. This is the Dependency Inversion Principle applied at the architectural level.
00245| 
00246| ```
00247| Dependency Flow (CORRECT)
00248| ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
00249| 
00250| External Input (Web/Console)
00251|         │
00252|         ▼
00253| Inbound Adapter (Controller)
00254|         │
00255|         │ depends on
00256|         ▼
00257| Input Port (UseCase Interface)
00258|         │
00259|         │ depends on
00260|         ▼
00261| Use Case Service
00262|         │
00263|         │ depends on
00264|         ▼
00265| Domain Entities / Output Ports
00266|         │
00267|         │ ←─── implement
00268|         ▼
00269| Outbound Adapter (Repository/Presenter Implementation)
00270|         │
00271|         │ implements
00272|         ▼
00273| External Systems (DB, APIs, etc.)
00274| ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
00275| 
00276| Dependency Flow (INCORRECT - Violations)
00277| ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
00278| 
00279| ✗ Domain depends on Infrastructure
00280| ✗ Controller directly calls Repository
00281| ✗ Use Case imports Framework classes
00282| ✗ Entity has JPA annotations
00283| ✗ Service uses HttpServletRequest
00284| ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
00285| ```
00286| 
00287| ---
00288| 
00289| ## The Core Without Dependencies
00290| 
00291| The domain layer should have zero dependencies on any external library or framework. This makes it:
00292| 
00293| 1. **Testable**: Core business logic can be tested without mocking infrastructure
00294| 2. **Maintainable**: Changes to external systems don't affect business logic
00295| 3. **Portable**: Core can be reused in different contexts
00296| 
00297| ```typescript
00298| // Domain Layer - Pure business logic, no dependencies
00299| interface Entity<TId> {
00300|   getId(): TId;
00301| }
00302| 
00303| interface ValueObject {
00304|   equals(other: ValueObject): boolean;
00305| }
00306| 
00307| // Example: Entity with Value Object ID
00308| class Project implements Entity<ProjectId> {
00309|   constructor(
00310|     private readonly id: ProjectId,
00311|     private readonly name: ProjectName,
00312|     private readonly tasks: TaskCollection
00313|   ) {}
00314| 
00315|   getId(): ProjectId {
00316|     return this.id;
00317|   }
00318| 
00319|   addTask(task: Task): void {
00320|     this.tasks.add(task);
00321|   }
00322| 
00323|   removeTask(taskId: TaskId): void {
00324|     this.tasks.remove(taskId);
00325|   }
00326| }
00327| 
00328| // Example: Value Object with validation
00329| record ProjectName implements ValueObject {
00330|   private readonly value: string;
00331| 
00332|   constructor(value: string) {
00333|     if (value === null || value.trim().length === 0) {
00334|       throw new DomainError('Project name cannot be empty');
00335|     }
00336|     if (value.length > 100) {
00337|       throw new DomainError('Project name cannot exceed 100 characters');
00338|     }
00339|     this.value = value;
00340|   }
00341| 
00342|   equals(other: ValueObject): boolean {
00343|     return other instanceof ProjectName && this.value === other.value;
00344|   }
00345| }
00346| ```
00347| 
00348| ---
00349| 
00350| ## Use Case Implementation Pattern
00351| 
00352| Use cases encapsulate a single business operation. Each use case follows a consistent pattern:
00353| 
00354| ```typescript
00355| // Step 1: Input Port (Interface)
00356| interface AddTaskUseCase extends Command<AddTaskInput, CqrsOutput> {
00357| }
00358| 
00359| // Step 2: Input Object
00360| class AddTaskInput implements Input {
00361|   toDoListId: string;
00362|   projectName: string;
00363|   description: string;
00364| }
00365| 
00366| // Step 3: Output Object (if custom needed)
00367| class AddTaskOutput implements Output {
00368|   taskId: string;
00369|   createdAt: Date;
00370| }
00371| 
00372| // Step 4: Use Case Service (Implementation)
00373| class AddTaskService implements AddTaskUseCase {
00374|   constructor(
00375|     private toDoListRepository: ToDoListRepository,
00376|     private taskPresenter: TaskPresenter
00377|   ) {}
00378| 
00379|   async execute(input: AddTaskInput): Promise<CqrsOutput> {
00380|     // 1. Validate input
00381|     if (!input.description) {
00382|       return CqrsOutput.create()
00383|         .fail()
00384|         .setMessage('Task description is required');
00385|     }
00386| 
00387|     // 2. Load aggregate root
00388|     const toDoList = await this.toDoListRepository.findById(
00389|       ToDoListId.of(input.toDoListId)
00390|     );
00391| 
00392|     if (!toDoList) {
00393|       return CqrsOutput.create()
00394|         .fail()
00395|         .setMessage('ToDoList not found');
00396|     }
00397| 
00398|     // 3. Perform business operation
00399|     const project = toDoList.findProject(ProjectName.of(input.projectName));
00400|     if (!project) {
00401|       return CqrsOutput.create()
00402|         .fail()
00403|         .setMessage('Project not found');
00404|     }
00405| 
00406|     const task = project.addTask(TaskDescription.of(input.description));
00407| 
00408|     // 4. Persist changes
00409|     await this.toDoListRepository.save(toDoList);
00410| 
00411|     // 5. Return result
00412|     return CqrsOutput.create()
00413|       .succeed()
00414|       .setData(TaskMapper.toDto(task));
00415|   }
00416| }
00417| ```
00418| 
00419| ---
00420| 
00421| ## CQRS Pattern in Hexagonal Architecture
00422| 
00423| Command Query Responsibility Segregation (CQRS) separates operations that change state (Commands) from operations that read data (Queries).
00424| 
00425| ```typescript
00426| // Command - Modifies state
00427| interface Command<Input, Output> {
00428|   execute(input: Input): Promise<Output>;
00429| }
00430| 
00431| // Query - Reads state
00432| interface Query<Input, Output> {
00433|   execute(input: Input): Promise<Output>;
00434| }
00435| 
00436| // Write Operation (Command)
00437| interface CreateProjectCommand extends Command<CreateProjectInput, CqrsOutput> {
00438| }
00439| 
00440| // Read Operation (Query)
00441| interface GetProjectQuery extends Query<GetProjectInput, GetProjectOutput> {
00442| }
00443| 
00444| // Different use cases for different operations
00445| class CreateProjectService implements CreateProjectCommand {
00446|   // Modifies state - returns success/failure
00447| }
00448| 
00449| class GetProjectService implements GetProjectQuery {
00450|   // Only reads - returns data
00451| }
00452| ```
00453| 
00454| ---
00455| 
00456| ## Data Transfer Objects (DTOs)
00457| 
00458| DTOs transfer data between layers without behavior.
00459| 
00460| ```typescript
00461| // DTO - Simple data container
00462| class TaskDto {
00463|   id: string;
00464|   description: string;
00465|   isDone: boolean;
00466|   projectName: string;
00467|   createdAt: Date;
00468| }
00469| 
00470| // PO - Database representation (in adapter layer)
00471| class TaskPo {
00472|   id: string;
00473|   description: string;
00474|   is_done: boolean;
00475|   project_id: string;
00476|   created_at: Date;
00477| }
00478| 
00479| // Mapper - Transforms between layers
00480| class TaskMapper {
00481|   // Domain → DTO
00482|   static toDto(task: Task): TaskDto {
00483|     const dto = new TaskDto();
00484|     dto.id = task.getId().value;
00485|     dto.description = task.getDescription().value;
00486|     dto.isDone = task.isDone();
00487|     dto.projectName = task.getProjectName().value;
00488|     dto.createdAt = task.getCreatedAt();
00489|     return dto;
00490|   }
00491| 
00492|   // PO → Domain
00493|   static toDomain(po: TaskPo): Task {
00494|     return new Task(
00495|       TaskId.of(po.id),
00496|       TaskDescription.of(po.description),
00497|       po.is_done,
00498|       ProjectName.of(po.project_name),
00499|       po.created_at
00500|     );
00501|   }
00502| 
00503|   // Domain → PO
00504|   static toPo(task: Task): TaskPo {
00505|     const po = new TaskPo();
00506|     po.id = task.getId().value;
00507|     po.description = task.getDescription().value;
00508|     po.is_done = task.isDone();
00509|     po.project_id = task.getProjectName().value;
00510|     po.created_at = task.getCreatedAt();
00511|     return po;
00512|   }
00513| 
00514|   // Support collections
00515|   static toDtoList(tasks: Task[]): TaskDto[] {
00516|     return tasks.map(TaskMapper.toDto);
00517|   }
00518| }
00519| ```
00520| 
00521| ---
00522| 
00523| ## Read-Only Interface Pattern
00524| 
00525| Prevent unintended modifications by exposing read-only views.
00526| 
00527| ```typescript
00528| // Read-Only Interface
00529| interface ReadOnlyProject {
00530|   getId(): ProjectId;
00531|   getName(): ProjectName;
00532|   getTasks(): ReadonlyArray<ReadOnlyTask>;
00533|   containsTask(taskId: TaskId): boolean;
00534| }
00535| 
00536| // Implementation
00537| class ReadOnlyProjectImpl implements ReadOnlyProject {
00538|   constructor(private project: Project) {}
00539| 
00540|   getId(): ProjectId {
00541|     return this.project.getId();
00542|   }
00543| 
00544|   getName(): ProjectName {
00545|     return this.project.getName();
00546|   }
00547| 
00548|   getTasks(): ReadonlyArray<ReadOnlyTask> {
00549|     return this.project.getTasks()
00550|       .map(task => new ReadOnlyTaskImpl(task));
00551|   }
00552| 
00553|   containsTask(taskId: TaskId): boolean {
00554|     return this.project.containsTask(taskId);
00555|   }
00556| }
00557| 
00558| // Throws on modification attempts
00559| class ReadOnlyTaskImpl implements ReadOnlyTask {
00560|   constructor(private task: Task) {}
00561| 
00562|   getId(): TaskId {
00563|     return this.task.getId();
00564|   }
00565| 
00566|   setDone(done: boolean): void {
00567|     throw new UnsupportedOperationException('Read-only view');
00568|   }
00569| }
00570| ```
00571| 
00572| ---
00573| 
00574| ## Testing Strategy
00575| 
00576| Because of the clear separation, testing can be done at multiple levels:
00577| 
00578| ```typescript
00579| // 1. Domain Layer - Pure unit tests, no dependencies
00580| describe('Project', () => {
00581|   it('should create project with valid name', () => {
00582|     const name = ProjectName.of('Test Project');
00583|     const project = new Project(name);
00584|     expect(project.getName()).toEqual(name);
00585|   });
00586| 
00587|   it('should reject empty project name', () => {
00588|     expect(() => new ProjectName.of(''))
00589|       .toThrow(DomainError);
00590|   });
00591| });
00592| 
00593| // 2. Use Case Layer - Tests with in-memory repositories
00594| describe('AddTaskService', () => {
00595|   it('should add task to project', async () => {
00596|     const repository = new InMemoryToDoListRepository();
00597|     const service = new AddTaskService(repository);
00598|     
00599|     const input = new AddTaskInput();
00600|     input.toDoListId = 'list-1';
00601|     input.projectName = 'Project A';
00602|     input.description = 'New Task';
00603|     
00604|     const result = await service.execute(input);
00605|     
00606|     expect(result.isSuccess).toBe(true);
00607|   });
00608| });
00609| 
00610| // 3. Integration Tests - Full stack with test adapters
00611| describe('AddTaskFlow', () => {
00612|   it('should handle end-to-end flow', async () => {
00613|     const controller = new AddTaskConsoleController(
00614|       new AddTaskService(TestRepository)
00615|     );
00616|     
00617|     const request = new ConsoleRequest(['add', 'Project A', 'Task 1']);
00618|     const response = controller.execute(request);
00619|     
00620|     expect(response.exitCode).toBe(ExitCode.SUCCESS);
00621|   });
00622| });
00623| ```
00624| 
00625| ---
00626| 
00627| ## Benefits of Hexagonal Architecture
00628| 
00629| 1. **Testability**: Core business logic can be tested without external dependencies
00630| 2. **Flexibility**: External systems can be swapped without changing business code
00631| 3. **Maintainability**: Clear boundaries make code easier to understand and modify
00632| 4. **Adaptability**: Application can be delivered through multiple channels
00633| 5. **Parallel Development**: Teams can work on different parts independently
00634| 6. **Longevity**: Core business logic outlives specific technology choices
00635| 
00636| ---
00637| 
00638| ## Anti-Patterns to Avoid
00639| 
00640| ### 1. Leaking Infrastructure into Domain
00641| ```typescript
00642| // WRONG - Domain depends on JPA
00643| @Entity
00644| class Project {
00645|   @Id
00646|   private id: string;
00647|   
00648|   @Column
00649|   private name: string;
00650| }
00651| ```
00652| 
00653| ### 2. Skipping Layers
00654| ```typescript
00655| // WRONG - Controller directly calls repository
00656| class Controller {
00657|   async handle() {
00658|     // Bypasses use case layer
00659|     await this.entityManager.save(entity);
00660|   }
00661| }
00662| ```
00663| 
00664| ### 3. Concrete Dependencies in Use Cases
00665| ```typescript
00666| // WRONG - Use case depends on concrete implementation
00667| class Service {
00668|   constructor(
00669|     private repository: CrudRepository // Should be TaskRepository interface
00670|   ) {}
00671| }
00672| ```
00673| 
00674| ---
00675| 
00676| ## References and Further Reading
00677| 
00678| 1. Cockburn, Alistair. "Hexagonal Architecture." 2005.
00679| 2. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
00680| 3. Martin, Robert C. "The Dependency Inversion Principle." 1996.
00681| 4. "Implementing Domain-Driven Design." Vaughn Vernon. Addison-Wesley, 2013.
00682| 5. "Architecture Patterns with Python." Harry Percival and Bob Gregory. O'Reilly, 2021.
00683| 

(End of file - total 683 lines)
</file>
```
```

---

## Assistant

_Thinking:_

The user wants me to compare the two architecture documents (clean-architecture.md and hexagonal-architecture.md) and merge them into a single file called layered-architecture.md.

Looking at both files:

1. **Clean Architecture** (440 lines):
   - Introduced by Robert C. Martin (Uncle Bob)
   - Core principle: business rules independent of frameworks, databases, delivery mechanisms
   - Four layers: Frameworks & Tools, Interface Adapters, Application Services, Domain Layer
   - Dependency rule: dependencies point inward
   - Key principles: independence of frameworks, testability, UI independence, database independence, external agency independence
   - Includes directory structure, implementation examples, dependency injection, benefits, anti-patterns

2. **Hexagonal Architecture** (683 lines):
   - Also known as Ports and Adapters
   - Formalized by Alistair Cockburn
   - Core concept: hexagonal boundary with domain at center, ports as interfaces, adapters as implementations
   - Three layers: Domain, Use Case (Application), Adapter
   - Ports: Input ports (use case interfaces), Output ports (repository/presenter interfaces)
   - Adapters: Inbound (delivery mechanisms like Web, Console, API), Outbound (external system integrations like DB, APIs)
   - More detailed on CQRS pattern, DTOs, mappers, read-only interfaces, testing strategy
   - Benefits and anti-patterns

Both architectures share many similarities:
- Both emphasize separation of concerns
- Both have the dependency rule (dependencies point inward)
- Both separate business logic from external concerns
- Both use ports/interfaces to define boundaries
- Both enable testability and flexibility

Key differences:
- Clean Architecture has 4 layers, Hexagonal has 3 main layers (though both conceptually similar)
- Clean Architecture uses "Frameworks & Tools" as outermost layer, Hexagonal uses "External Systems"


- Hexagonal provides more detailed patterns and patterns (CQRS, DTOs, mappers, read-only interfaces)

I'll develop a comprehensive document that integrates the strengths of both architectural approaches. The layered-architecture.md will synthesize their core principles, creating a robust framework that combines Clean Architecture's structured layering with Hexagonal Architecture's detailed interface and adapter strategies. This approach will provide a clear, flexible blueprint for software design that maximizes modularity and separation of concerns.

```
Tool: write

**Input:**
```json
{
  "content": "# Layered Architecture (Ports and Adapters / Clean Architecture)\n\n## Overview\n\nLayered Architecture, also known as **Hexagonal Architecture** (Ports and Adapters) or **Clean Architecture**, is a software design philosophy that emphasizes the separation of concerns through architectural layers. The core principle is that business rules and application logic should remain independent of external frameworks, databases, delivery mechanisms, and user interfaces.\n\nThis architectural pattern was formalized by Alistair Cockburn as Hexagonal Architecture and further developed by Robert C. Martin (Uncle Bob) as Clean Architecture. Both approaches share the same fundamental insight: business logic should not depend on databases, web frameworks, or any external system. Instead, external systems depend on abstractions defined by the business logic. This inversion of dependencies ensures that the core application can be tested independently of its delivery mechanisms and that external systems can be replaced without changing business code.\n\nThe architecture creates clear boundaries between what the application does (core/domain) and how it integrates with the outside world (adapters). This separation enables teams to work on different parts independently, allows technology choices to be deferred, and ensures that core business logic outlives specific technology choices.\n\n---\n\n## Core Principles\n\n### The Dependency Rule\n\nThe most critical rule in layered architectures is that **dependencies always point inward** toward the domain. This is the Dependency Inversion Principle applied at the architectural level. Inner layers do not know about outer layers, and all source code dependencies point toward the more abstract inner layers.\n\n```\n                    ┌───────────────────────────────────────┐\n                    │          External Systems             │  ← Outermost Layer\n                    │  (Web, DB, UI, External Services)    │\n                    └───────────────────┬───────────────────┘\n                                        │\n                    ┌───────────────────▼───────────────────┐\n                    │            Adapters Layer             │  ← Interface Adapters\n                    │  (Controllers, Gateways, Presenters,  │\n                    │   Database Adapters, External Clients)│\n                    └───────────────────┬───────────────────┘\n                                        │\n                    ┌───────────────────▼───────────────────┐\n                    │         Application Layer             │  ← Use Cases / Services\n                    │  (Interactors, Use Case Coordinators, │\n                    │   Application Services)               │\n                    └───────────────────┬───────────────────┘\n                                        │\n                    ┌───────────────────▼───────────────────┐\n                    │            Domain Layer               │  ← Innermost Layer\n                    │  (Entities, Value Objects, Aggregates,│\n                    │   Domain Services, Domain Events)     │\n                    └───────────────────────────────────────┘\n\n     Source code dependencies point INWARD toward the domain.\n     Inner layers do NOT know about outer layers.\n```\n\n### Key Principles\n\n1. **Independence of Frameworks**: Business logic remains independent of frameworks and can be tested without them\n2. **Testability**: Business rules are testable without UI, database, or external systems\n3. **Independence of UI**: User interfaces can change without affecting business logic\n4. **Independence of Database**: Business logic is decoupled from database implementation details\n5. **Independence of External Agencies**: Business rules have no knowledge of the outside world\n6. **Framework Independence**: Business logic survives framework changes and upgrades\n\n---\n\n## Architecture Overview\n\n```\n┌─────────────────────────────────────────────────────────────────────────┐\n│                    LAYERED ARCHITECTURE                                 │\n├─────────────────────────────────────────────────────────────────────────┤\n│                                                                         │\n│                         ┌─────────────────────┐                         │\n│                         │                     │                         │\n│                         │        DOMAIN       │                         │\n│                         │        LAYER        │                         │\n│                         │                     │                         │\n│                         │  ┌───────────────┐  │                         │\n│                         │  │   Entities    │  │                         │\n│                         │  │  Value Objects│  │                         │\n│                         │  │  Aggregates   │  │                         │\n│                         │  │  Domain Events│  │                         │\n│                         │  └───────────────┘  │                         │\n│                         │                     │                         │\n│                         └──────────┬──────────┘                         │\n│                                    │                                    │\n│                                    ▼                                    │\n│                         ┌─────────────────────┐                         │\n│                         │     APPLICATION     │                         │\n│                         │        LAYER        │                         │\n│                         │                     │                         │\n│                         │  ┌───────────────┐  │                         │\n│                         │  │   Use Cases   │  │                         │\n│                         │  │   Services    │  │                         │\n│                         │  │   Ports       │  │                         │\n│                         │  └───────────────┘  │                         │\n│                         │                     │                         │\n│                         └──────────┬──────────┘                         │\n│                                    │                                    │\n│              ┌─────────────────────┼─────────────────────┐              │\n│              │                     │                     │              │\n│              ▼                     ▼                     ▼              │\n│    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │\n│    │     PORTS       │  │                 │  │     PORTS       │       │\n│    │   (Interfaces)  │  │                 │  │   (Interfaces)  │       │\n│    │                 │  │                 │  │                 │       │\n│    │  • Input Ports  │  │                 │  │  • Output Ports │       │\n│    │    (Use Cases)  │  │                 │  │  • Repositories │       │\n│    │                 │  │                 │  │  • Presenters   │       │\n│    └────────┬────────┘  │                 │  └────────┬────────┘       │\n│             │           │                 │           │                │\n│             │           │                 │           │                │\n│             ▼           │                 │           ▼                │\n│    ┌─────────────────┐  │                 │  ┌─────────────────┐       │\n│    │   ADAPTERS      │  │                 │  │   ADAPTERS      │       │\n│    │   (Inbound)     │  │                 │  │   (Outbound)    │       │\n│    │                 │  │                 │  │                 │       │\n│    │  • Web/HTTP     │  │                 │  │  • Database     │       │\n│    │  • Console/CLI  │  │                 │  │  • File System  │       │\n│    │  • API/Gateway  │  │                 │  │  • External APIs│       │\n│    │  • Messaging    │  │                 │  │  • Messaging    │       │\n│    │  • Mobile       │  │                 │  │  • Event Bus    │       │\n│    └─────────────────┘  │                 │  └─────────────────┘       │\n│                         │                 │                            │\n└─────────────────────────┼─────────────────┼────────────────────────────┘\n                          │                 │\n                          ▼                 ▼\n              ┌───────────────────┐ ┌───────────────────┐\n              │    EXTERNAL       │ │    EXTERNAL       │\n              │    SYSTEMS        │ │    SYSTEMS        │\n              │                   │ │                   │\n              │  • Web Browsers   │ │  • Databases      │\n              │  • Mobile Apps    │ │  • File Systems   │\n              │  • CLI Users      │ │  • Web Services   │\n              │  • API Clients    │ │  • Message Queues │\n              └───────────────────┘ └───────────────────┘\n```\n\n---\n\n## The Three Core Layers\n\n### Domain Layer (Core)\n\nThe domain layer contains the business logic of the application at its purest form. It consists of entities, value objects, aggregates, domain services, and domain events. This layer has **zero dependencies** on any other layer. It defines interfaces (ports) that describe what it needs from the outside world. The domain layer represents the heart of the application and should be completely isolated from infrastructure concerns.\n\nThe domain layer is where the most stable code resides because it contains the core business rules that rarely change, regardless of how the application is delivered or what technologies are used. This layer can be thoroughly tested without any external dependencies, making it the most reliable and valuable part of the codebase.\n\n### Application Layer (Use Cases)\n\nThe application layer contains application-specific business logic that orchestrates operations. It coordinates between the domain layer and external systems by using domain objects and calling through ports to interact with external concerns. This layer depends only on domain abstractions and contains use cases, application services, and input/output port definitions.\n\nThe application layer defines what the system does without caring about how it communicates with the outside world. Each use case represents a single business operation and follows a consistent pattern of validating input, performing business operations, persisting changes, and returning results.\n\n### Adapter Layer (External Integration)\n\nThe adapter layer contains implementations of ports that integrate with external systems. Adapters are divided into two categories: inbound adapters handle user input and external requests, while outbound adapters handle communication with external systems. Adapters depend on ports (interfaces), never the reverse.\n\nThis layer translates between external protocols and internal abstractions, allowing the core application to remain unchanged regardless of how it is accessed or what external systems it integrates with.\n\n---\n\n## Ports and Adapters Explained\n\n### Ports (Interfaces)\n\nPorts define the boundary between the application core and external systems. They are abstract contracts that specify what the application expects from and provides to the outside world. Ports create a stable interface that shields the core from changes in external systems.\n\n**Input Ports (Use Case Interfaces)**\n\nInput ports define operations that external systems can perform on the application. They represent the application's API for incoming operations. These ports are implemented by application services and called by inbound adapters.\n\n```typescript\n// Input Port - What the outside world can DO\ninterface CreateOrderUseCase {\n  execute(input: CreateOrderInput): Promise<CreateOrderOutput>;\n}\n\ninterface GetOrderUseCase {\n  execute(input: GetOrderInput): Promise<GetOrderOutput>;\n}\n\ninterface ProcessPaymentUseCase {\n  execute(input: ProcessPaymentInput): Promise<ProcessPaymentOutput>;\n}\n\ninterface ShipOrderUseCase {\n  execute(input: ShipOrderInput): Promise<ShipOrderOutput>;\n}\n```\n\n**Output Ports (Repository and Service Interfaces)**\n\nOutput ports define operations that the application needs to perform on external systems. These ports are implemented by outbound adapters and called by application services and domain logic. Output ports decouple the core from specific implementations of persistence, external services, and other infrastructure concerns.\n\n```typescript\n// Output Port - What the application NEEDS from external systems\ninterface OrderRepository {\n  findById(id: OrderId): Promise<Order | null>;\n  findByCustomerId(customerId: string): Promise<Order[]>;\n  save(order: Order): Promise<void>;\n  delete(id: OrderId): Promise<void>;\n}\n\ninterface InventoryService {\n  checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;\n  reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;\n  releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;\n}\n\ninterface PaymentGateway {\n  processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;\n  refundPayment(transactionId: string): Promise<RefundResult>;\n}\n\ninterface EventPublisher {\n  publish(event: DomainEvent): Promise<void>;\n  publishAll(events: DomainEvent[]): Promise<void>;\n}\n\ninterface NotificationService {\n  sendNotification(customerId: string, message: string): Promise<void>;\n}\n```\n\n### Adapters (Implementations)\n\nAdapters implement ports and handle the specifics of interacting with external systems. They translate between external protocols and internal abstractions, keeping the core completely unaware of how external communication happens.\n\n**Inbound Adapters (Delivery Mechanisms)**\n\nInbound adapters translate external input into calls to input ports. They handle the specifics of receiving requests from various delivery mechanisms and translating them into a format the application can understand.\n\n```typescript\n// HTTP Controller (Inbound Adapter)\nclass CreateOrderController {\n  constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}\n\n  async handleRequest(request: HttpRequest): Promise<HttpResponse> {\n    try {\n      const input: CreateOrderInput = {\n        customerId: request.body.customerId,\n        items: request.body.items.map((item: any) => ({\n          productId: item.productId,\n          quantity: item.quantity\n        })),\n        shippingAddress: request.body.shippingAddress,\n        paymentMethod: request.body.paymentMethod\n      };\n\n      const result = await this.createOrderUseCase.execute(input);\n\n      return new HttpResponse(\n        result.isSuccess ? 201 : 400,\n        {\n          success: result.isSuccess,\n          data: result.isSuccess ? result.orderId : null,\n          error: result.isSuccess ? null : result.error\n        }\n      );\n    } catch (error) {\n      return new HttpResponse(\n        500,\n        {\n          success: false,\n          error: error instanceof Error ? error.message : 'Unknown error'\n        }\n      );\n    }\n  }\n}\n\n// gRPC Controller (Inbound Adapter)\nclass OrderGrpcController {\n  constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}\n\n  async createOrder(call: ServerUnaryCall<CreateOrderRequest>): Promise<CreateOrderResponse> {\n    const input: CreateOrderInput = {\n      customerId: call.request.customerId,\n      items: call.request.items.map(item => ({\n        productId: item.productId,\n        quantity: item.quantity\n      }))\n    };\n\n    const result = await this.createOrderUseCase.execute(input);\n\n    return {\n      success: result.isSuccess,\n      orderId: result.isSuccess ? result.orderId : '',\n      error: result.error || ''\n    };\n  }\n}\n\n// Message Queue Consumer (Inbound Adapter)\nclass OrderMessageHandler {\n  constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}\n\n  async handleMessage(message: OrderMessage): Promise<void> {\n    const input: CreateOrderInput = {\n      customerId: message.customerId,\n      items: message.items\n    };\n\n    const result = await this.createOrderUseCase.execute(input);\n\n    if (!result.isSuccess) {\n      throw new Error(`Failed to process order: ${result.error}`);\n    }\n  }\n}\n\n// Console Controller (Inbound Adapter)\nclass OrderConsoleController {\n  constructor(\n    private readonly createOrderUseCase: CreateOrderUseCase,\n    private readonly getOrderUseCase: GetOrderUseCase\n  ) {}\n\n  handleCommand(args: string[]): Response {\n    const command = args[0];\n\n    switch (command) {\n      case 'create':\n        return this.handleCreateOrder(args);\n      case 'get':\n        return this.handleGetOrder(args);\n      default:\n        return Response.error('Unknown command');\n    }\n  }\n\n  private handleCreateOrder(args: string[]): Response {\n    const input: CreateOrderInput = {\n      customerId: args[1],\n      items: JSON.parse(args[2])\n    };\n\n    const result = this.createOrderUseCase.execute(input);\n    return Response.of(result);\n  }\n\n  private handleGetOrder(args: string[]): Response {\n    const input: GetOrderInput = { orderId: args[1] };\n    const result = this.getOrderUseCase.execute(input);\n    return Response.of(result);\n  }\n}\n```\n\n**Outbound Adapters (External System Integrations)**\n\nOutbound adapters implement output ports and handle communication with external systems. They translate between internal abstractions and external system specifics.\n\n```typescript\n// TypeOutbound Adapter)\nclass TypeORMOrderORM Repository Implementation (Repository implements OrderRepository {\n  constructor(\n    private readonly entityManager: EntityManager,\n    private readonly eventPublisher: EventPublisher\n  ) {}\n\n  async findById(id: OrderId): Promise<Order | null> {\n    const entity = await this.entityManager.findOne(OrderEntity, {\n      where: { id: id.value }\n    });\n\n    if (!entity) return null;\n\n    return OrderMapper.toDomain(entity);\n  }\n\n  async findByCustomerId(customerId: string): Promise<Order[]> {\n    const entities = await this.entityManager.find(OrderEntity, {\n      where: { customerId }\n    });\n\n    return entities.map(OrderMapper.toDomain);\n  }\n\n  async save(order: Order): Promise<void> {\n    const entity = OrderMapper.toPersistence(order);\n\n    if (order.isNew) {\n      await this.entityManager.save(entity);\n    } else {\n      await this.entityManager.update(OrderEntity, entity.id, entity);\n    }\n\n    for (const event of order.getUncommittedEvents()) {\n      await this.eventPublisher.publish(event);\n    }\n  }\n\n  async delete(id: OrderId): Promise<void> {\n    await this.entityManager.delete(OrderEntity, { id: id.value });\n  }\n}\n\n// In-Memory Repository (Outbound Adapter for Testing)\nclass InMemoryOrderRepository implements OrderRepository {\n  private orders: Map<string, Order> = new Map();\n\n  async findById(id: OrderId): Promise<Order | null> {\n    return this.orders.get(id.value) || null;\n  }\n\n  async findByCustomerId(customerId: string): Promise<Order[]> {\n    return Array.from(this.orders.values()).filter(\n      order => order.customerId === customerId\n    );\n  }\n\n  async save(order: Order): Promise<void> {\n    this.orders.set(order.id.value, order);\n  }\n\n  async delete(id: OrderId): Promise<void> {\n    this.orders.delete(id.value);\n  }\n}\n\n// Stripe Payment Gateway (Outbound Adapter)\nclass StripePaymentGateway implements PaymentGateway {\n  constructor(\n    private readonly stripeClient: StripeClient,\n    private readonly webhookSecret: string\n  ) {}\n\n  async processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult> {\n    try {\n      const paymentIntent = await this.stripeClient.paymentIntents.create({\n        amount: amount.inCents,\n        currency: amount.currency.toLowerCase(),\n        payment_method: paymentMethod.stripeToken,\n        confirm: true,\n        automatic_payment_methods: {\n          enabled: true,\n          allow_redirects: 'never'\n        }\n      });\n\n      return {\n        success: paymentIntent.status === 'succeeded',\n        transactionId: paymentIntent.id,\n        error: paymentIntent.last_payment_error?.message\n      };\n    } catch (error) {\n      return {\n        success: false,\n        transactionId: '',\n        error: error instanceof Error ? error.message : 'Payment failed'\n      };\n    }\n  }\n\n  async refundPayment(transactionId: string): Promise<RefundResult> {\n    try {\n      const refund = await this.stripeClient.refunds.create({\n        payment_intent: transactionId\n      });\n\n      return {\n        success: refund.status === 'succeeded',\n        refundId: refund.id\n      };\n    } catch (error) {\n      return {\n        success: false,\n        refundId: '',\n        error: error instanceof Error ? error.message : 'Refund failed'\n      };\n    }\n  }\n}\n\n// Kafka Event Publisher (Outbound Adapter)\nclass KafkaEventPublisher implements EventPublisher {\n  constructor(\n    private readonly kafkaProducer: KafkaProducer,\n    private readonly topicPrefix: string\n  ) {}\n\n  async publish(event: DomainEvent): Promise<void> {\n    const topic = `${this.topicPrefix}.${event.constructor.name}`;\n    const message = {\n      key: event.aggregateId,\n      value: JSON.stringify(event),\n      timestamp: Date.now()\n    };\n\n    await this.kafkaProducer.send({ topic, messages: [message] });\n  }\n\n  async publishAll(events: DomainEvent[]): Promise<void> {\n    const messages = events.map(event => ({\n      topic: `${this.topicPrefix}.${event.constructor.name}`,\n      key: event.aggregateId,\n      value: JSON.stringify(event),\n      timestamp: Date.now()\n    }));\n\n    await this.kafkaProducer.send({ messages });\n  }\n}\n```\n\n---\n\n## Layer Structure\n\n```\nsrc/\n├── domain/                           # Domain Layer (innermost)\n│   ├── entities/                     # Domain entities with behavior\n│   │   ├── order.ts\n│   │   ├── customer.ts\n│   │   └── product.ts\n│   ├── value-objects/                # Value objects with validation\n│   │   ├── money.ts\n│   │   ├── address.ts\n│   │   └── email.ts\n│   ├── aggregates/                   # Aggregate roots\n│   │   ├── order-aggregate.ts\n│   │   └── customer-aggregate.ts\n│   ├── domain-services/              # Domain services for complex logic\n│   │   └── pricing-service.ts\n│   ├── events/                       # Domain events\n│   │   ├── order-created.event.ts\n│   │   └── payment-completed.event.ts\n│   ├── repositories/                 # Repository interfaces (ports)\n│   │   ├── order.repository.interface.ts\n│   │   └── customer.repository.interface.ts\n│   ├── services/                     # Service interfaces (ports)\n│   │   ├── inventory.service.interface.ts\n│   │   └── payment.service.interface.ts\n│   └── factories/                    # Domain factories\n│       └── order.factory.ts\n│\n├── application/                      # Application Layer\n│   ├── use-cases/                    # Use case implementations\n│   │   ├── create-order/\n│   │   │   ├── create-order.handler.ts\n│   │   │   ├── create-order.input.ts\n│   │   │   └── create-order.output.ts\n│   │   ├── process-payment/\n│   │   │   ├── process-payment.handler.ts\n│   │   │   ├── process-payment.input.ts\n│   │   │   └── process-payment.output.ts\n│   │   └── ship-order/\n│   │       ├── ship-order.handler.ts\n│   │       ├── ship-order.input.ts\n│   │       └── ship-order.output.ts\n│   ├── ports/\n│   │   ├── inbound/                  # Input port interfaces\n│   │   │   ├── create-order.port.ts\n│   │   │   └── process-payment.port.ts\n│   │   └── outbound/                 # Output port interfaces\n│   │       ├── order.repository.port.ts\n│   │       └── payment-gateway.port.ts\n│   └── coordinators/                 # Application coordinators\n│       └── order-coordinator.ts\n│\n├── adapters/                         # Adapter Layer\n│   ├── inbound/                      # Inbound adapters (controllers)\n│   │   ├── http/\n│   │   │   ├── controllers/\n│   │   │   │   ├── order.controller.ts\n│   │   │   │   └── payment.controller.ts\n│   │   │   ├── middlewares/\n│   │   │   │   ├── auth.middleware.ts\n│   │   │   │   └── validation.middleware.ts\n│   │   │   └── routes/\n│   │   │       ├── order.routes.ts\n│   │   │       └── payment.routes.ts\n│   │   ├── grpc/\n│   │   │   └── order.grpc.controller.ts\n│   │   ├── messaging/\n│   │   │   ├── order.message-handler.ts\n│   │   │   └── payment.message-handler.ts\n│   │   └── cli/\n│   │       └── order.cli.controller.ts\n│   ├── outbound/                     # Outbound adapters (implementations)\n│   │   ├── persistence/\n│   │   │   ├── typeorm/\n│   │   │   │   ├── order.repository.ts\n│   │   │   │   ├── customer.repository.ts\n│   │   │   │   └── entities/\n│   │   │   │       ├── order.entity.ts\n│   │   │   │       └── customer.entity.ts\n│   │   │   ├── prisma/\n│   │   │   │   ├── order.repository.ts\n│   │   │   │   └── customer.repository.ts\n│   │   │   └── mongodb/\n│   │   │       ├── order.repository.ts\n│   │   │       └── customer.repository.ts\n│   │   ├── external-api/\n│   │   │   ├── stripe/\n│   │   │   │   ├── payment-gateway.adapter.ts\n│   │   │   │   └── stripe-client.ts\n│   │   │   ├── sendgrid/\n│   │   │   │   └── notification.adapter.ts\n│   │   │   └── aws/\n│   │   │       └── email.service.ts\n│   │   ├── events/\n│   │   │   ├── kafka/\n│   │   │   │   └── event-publisher.ts\n│   │   │   └── rabbitmq/\n│   │   │       └── event-publisher.ts\n│   │   └── cache/\n│   │       └── redis/\n│   │           └── cache.adapter.ts\n│   └── presenters/                   # Response formatters\n│       ├── json/\n│       │   └── order.presenter.ts\n│       └── grpc/\n│           └── order.presenter.ts\n│\n├── frameworks/                       # Frameworks and Drivers\n│   ├── web/\n│   │   ├── express/\n│   │   │   ├── app.ts\n│   │   │   └── server.ts\n│   │   ├── fastify/\n│   │   │   ├── app.ts\n│   │   │   └── server.ts\n│   │   └── nestjs/                   # NestJS modules\n│   │       ├── order.module.ts\n│   │       └── payment.module.ts\n│   ├── database/\n│   │   ├── typeorm/\n│   │   │   ├── data-source.ts\n│   │   │   └── migrations/\n│   │   ├── prisma/\n│   │   │   ├── schema.prisma\n│   │   │   └── client.ts\n│   │   └── mongoose/\n│   │       ├── connection.ts\n│   │       └── schemas/\n│   ├── messaging/\n│   │   ├── kafka/\n│   │   │   ├── producer.ts\n│   │   │   └── consumer.ts\n│   │   └── rabbitmq/\n│   │       ├── connection.ts\n│   │       └── publisher.ts\n│   └── external/\n│       ├── stripe/\n│       │   └── client.ts\n│       ├── sendgrid/\n│       │   └── client.ts\n│       └── aws/\n│           ├── config.ts\n│           └── client.ts\n│\n└── shared/\n    ├── utils/                        # Shared utilities\n    ├── types/                        # Shared TypeScript types\n    ├── errors/                       # Custom error classes\n    └── mappers/                      # Cross-layer mappers\n```\n\n---\n\n## Implementation Examples\n\n### Domain Layer (Enterprise Business Rules)\n\nThe domain layer contains pure business logic with no dependencies on external frameworks or libraries. This makes it completely testable and portable.\n\n```typescript\n// domain/entities/order.ts\nimport { Entity } from '../../shared/entity';\nimport { Money } from '../../value-objects/money';\nimport { OrderId } from '../../value-objects/order-id';\nimport { OrderStatus } from './order-status';\nimport { OrderLineItem } from './order-line-item';\n\nexport interface OrderProps {\n  id: OrderId;\n  customerId: string;\n  lineItems: OrderLineItem[];\n  status: OrderStatus;\n  shippingAddress: Address;\n  paymentMethod: PaymentMethod;\n  createdAt: Date;\n}\n\nexport class Order extends Entity<OrderId, OrderProps> {\n  private constructor(props: OrderProps) {\n    super(props);\n  }\n\n  static create(props: OrderProps): Order {\n    const order = new Order(props);\n    order.validate();\n    return order;\n  }\n\n  private validate(): void {\n    if (this._props.lineItems.length === 0) {\n      throw new DomainError('Order must have at least one line item');\n    }\n    if (!this._props.status) {\n      throw new DomainError('Order must have a status');\n    }\n    if (!this._props.shippingAddress) {\n      throw new DomainError('Order must have a shipping address');\n    }\n  }\n\n  get total(): Money {\n    return this._props.lineItems.reduce(\n      (sum, item) => sum.add(item.subtotal),\n      Money.zero(this._props.lineItems[0]?.productCurrency ?? 'USD')\n    );\n  }\n\n  get canBeSubmitted(): boolean {\n    return this._props.status === OrderStatus.DRAFT;\n  }\n\n  get canBeCancelled(): boolean {\n    return this._props.status !== OrderStatus.SHIPPED &&\n           this._props.status !== OrderStatus.DELIVERED;\n  }\n\n  submit(): void {\n    if (!this.canBeSubmitted) {\n      throw new DomainError('Order cannot be submitted');\n    }\n    this._props.status = OrderStatus.SUBMITTED;\n    this.recordDomainEvent(new OrderSubmitted(this._props.id));\n  }\n\n  cancel(): void {\n    if (!this.canBeCancelled) {\n      throw new DomainError('Order cannot be cancelled');\n    }\n    this._props.status = OrderStatus.CANCELLED;\n    this.recordDomainEvent(new OrderCancelled(this._props.id));\n  }\n\n  ship(): void {\n    if (this._props.status !== OrderStatus.PAID) {\n      throw new DomainError('Order must be paid before shipping');\n    }\n    this._props.status = OrderStatus.SHIPPED;\n    this.recordDomainEvent(new OrderShipped(this._props.id));\n  }\n}\n\n// domain/value-objects/money.ts\nexport class Money {\n  private constructor(\n    private readonly amount: number,\n    private readonly currency: string\n  ) {}\n\n  static zero(currency: string = 'USD'): Money {\n    return new Money(0, currency);\n  }\n\n  static fromAmount(amount: number, currency: string = 'USD'): Money {\n    if (amount < 0) {\n      throw new DomainError('Amount cannot be negative');\n    }\n    return new Money(Math.round(amount * 100) / 100, currency);\n  }\n\n  add(other: Money): Money {\n    this.ensureSameCurrency(other);\n    return new Money(this.amount + other.amount, this.currency);\n  }\n\n  subtract(other: Money): Money {\n    this.ensureSameCurrency(other);\n    const result = this.amount - other.amount;\n    if (result < 0) {\n      throw new DomainError('Result cannot be negative');\n    }\n    return new Money(result, this.currency);\n  }\n\n  multiply(factor: number): Money {\n    if (factor < 0) {\n      throw new DomainError('Factor cannot be negative');\n    }\n    return new Money(this.amount * factor, this.currency);\n  }\n\n  isGreaterThan(other: Money): boolean {\n    this.ensureSameCurrency(other);\n    return this.amount > other.amount;\n  }\n\n  isLessThan(other: Money): boolean {\n    this.ensureSameCurrency(other);\n    return this.amount < other.amount;\n  }\n\n  get amountValue(): number {\n    return this.amount;\n  }\n\n  get currencyCode(): string {\n    return this.currency;\n  }\n\n  private ensureSameCurrency(other: Money): void {\n    if (this.currency !== other.currency) {\n      throw new DomainError('Cannot operate on different currencies');\n    }\n  }\n}\n\n// domain/aggregates/order-aggregate.ts\nexport class OrderAggregate {\n  private constructor(\n    public readonly order: Order,\n    private readonly lineItems: OrderLineItem[],\n    private readonly payments: Payment[]\n  ) {}\n\n  static createFromOrder(order: Order): OrderAggregate {\n    return new OrderAggregate(order, [], []);\n  }\n\n  get totalPaid(): Money {\n    return this.payments\n      .filter(p => p.status === PaymentStatus.COMPLETED)\n      .reduce((sum, payment) => sum.add(payment.amount), Money.zero());\n  }\n\n  isFullyPaid(): boolean {\n    return this.totalPaid.isGreaterThanOrEqual(this.order.total);\n  }\n\n  addPayment(payment: Payment): void {\n    if (this.isFullyPaid()) {\n      throw new DomainError('Order is already fully paid');\n    }\n    this.payments.push(payment);\n  }\n}\n\n// domain/events/order-submitted.event.ts\nexport class OrderSubmitted extends DomainEvent {\n  constructor(\n    public readonly orderId: OrderId,\n    public readonly customerId: string,\n    public readonly total: Money\n  ) {\n    super('OrderSubmitted', orderId.value);\n  }\n}\n```\n\n### Application Layer (Use Cases)\n\nApplication layer contains use case implementations that orchestrate domain objects and external adapters through ports.\n\n```typescript\n// application/use-cases/create-order/create-order.handler.ts\nimport { OrderRepository } from '../../ports/outbound/order.repository';\nimport { InventoryService } from '../../ports/outbound/inventory.service';\nimport { PaymentGateway } from '../../ports/outbound/payment-gateway';\nimport { EventPublisher } from '../../ports/outbound/event-publisher';\nimport { OrderFactory } from '../../domain/factories/order.factory';\n\nexport interface CreateOrderInput {\n  customerId: string;\n  items: Array<{\n    productId: string;\n    quantity: number;\n  }>;\n  shippingAddress: Address;\n  paymentMethod: PaymentMethod;\n}\n\nexport interface CreateOrderOutput {\n  isSuccess: boolean;\n  orderId?: string;\n  total?: number;\n  error?: string;\n}\n\nexport class CreateOrderHandler {\n  constructor(\n    private readonly orderRepository: OrderRepository,\n    private readonly inventoryService: InventoryService,\n    private readonly paymentGateway: PaymentGateway,\n    private readonly eventPublisher: EventPublisher,\n    private readonly orderFactory: OrderFactory\n  ) {}\n\n  async execute(input: CreateOrderInput): Promise<CreateOrderOutput> {\n    try {\n      // Step 1: Validate input\n      this.validateInput(input);\n\n      // Step 2: Check inventory availability\n      await this.inventoryService.checkAvailability(input.items);\n\n      // Step 3: Create order using factory\n      const order = this.orderFactory.createFromItems(\n        input.customerId,\n        input.items,\n        input.shippingAddress,\n        input.paymentMethod\n      );\n\n      // Step 4: Persist order\n      await this.orderRepository.save(order);\n\n      // Step 5: Publish domain events\n      await this.eventPublisher.publishAll(order.getUncommittedEvents());\n\n      // Step 6: Return success result\n      return {\n        isSuccess: true,\n        orderId: order.id.value,\n        total: order.total.amountValue\n      };\n    } catch (error) {\n      return {\n        isSuccess: false,\n        error: error instanceof Error ? error.message : 'Unknown error'\n      };\n    }\n  }\n\n  private validateInput(input: CreateOrderInput): void {\n    if (!input.customerId) {\n      throw new DomainError('Customer ID is required');\n    }\n    if (!input.items || input.items.length === 0) {\n      throw new DomainError('Order must have at least one item');\n    }\n    if (!input.shippingAddress) {\n      throw new DomainError('Shipping address is required');\n    }\n    if (!input.paymentMethod) {\n      throw new DomainError('Payment method is required');\n    }\n  }\n}\n\n// application/use-cases/process-payment/process-payment.handler.ts\nexport interface ProcessPaymentInput {\n  orderId: string;\n  paymentToken: string;\n  amount: number;\n  currency: string;\n}\n\nexport interface ProcessPaymentOutput {\n  isSuccess: boolean;\n  transactionId?: string;\n  error?: string;\n}\n\nexport class ProcessPaymentHandler {\n  constructor(\n    private readonly orderRepository: OrderRepository,\n    private readonly paymentGateway: PaymentGateway\n  ) {}\n\n  async execute(input: ProcessPaymentInput): Promise<ProcessPaymentOutput> {\n    try {\n      // Step 1: Load order\n      const order = await this.orderRepository.findById(\n        OrderId.of(input.orderId)\n      );\n\n      if (!order) {\n        return {\n          isSuccess: false,\n          error: 'Order not found'\n        };\n      }\n\n      // Step 2: Create payment\n      const amount = Money.fromAmount(input.amount, input.currency);\n      const paymentMethod = PaymentMethod.fromToken(input.paymentToken);\n\n      // Step 3: Process payment through gateway\n      const result = await this.paymentGateway.processPayment(\n        amount,\n        paymentMethod\n      );\n\n      if (!result.success) {\n        order.recordDomainEvent(new PaymentFailed(order.id, result.error!));\n        await this.orderRepository.save(order);\n\n        return {\n          isSuccess: false,\n          error: result.error\n        };\n      }\n\n      // Step 4: Update order status\n      order.markAsPaid(result.transactionId!);\n      await this.orderRepository.save(order);\n\n      // Step 5: Return success\n      return {\n        isSuccess: true,\n        transactionId: result.transactionId\n      };\n    } catch (error) {\n      return {\n        isSuccess: false,\n        error: error instanceof Error ? error.message : 'Payment processing failed'\n      };\n    }\n  }\n}\n```\n\n### Ports (Interfaces)\n\nPorts define the contracts between layers, enabling dependency inversion.\n\n```typescript\n// application/ports/inbound/create-order.port.ts\nexport interface CreateOrderInputPort {\n  createOrder(input: CreateOrderInput): Promise<CreateOrderOutput>;\n}\n\n// application/ports/outbound/order.repository.ts\nexport interface OrderRepository {\n  findById(id: OrderId): Promise<Order | null>;\n  findByCustomerId(customerId: string): Promise<Order[]>;\n  findByStatus(status: OrderStatus): Promise<Order[]>;\n  save(order: Order): Promise<void>;\n  delete(id: OrderId): Promise<void>;\n}\n\n// application/ports/outbound/inventory.service.ts\nexport interface InventoryService {\n  checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;\n  reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;\n  releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;\n  getProduct(productId: string): Promise<Product | null>;\n}\n\n// application/ports/outbound/payment-gateway.ts\nexport interface PaymentGateway {\n  processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;\n  refundPayment(transactionId: string, amount?: Money): Promise<RefundResult>;\n  getPaymentStatus(transactionId: string): Promise<PaymentStatus>;\n}\n\n// application/ports/outbound/event-publisher.ts\nexport interface EventPublisher {\n  publish(event: DomainEvent): Promise<void>;\n  publishAll(events: DomainEvent[]): Promise<void>;\n}\n\n// application/ports/outbound/notification.service.ts\nexport interface NotificationService {\n  sendEmail(to: string, subject: string, body: string): Promise<void>;\n  sendSms(to: string, message: string): Promise<void>;\n  sendPushNotification(userId: string, notification: PushNotification): Promise<void>;\n}\n```\n\n---\n\n## Dependency Injection Structure\n\nThe composition root wires all dependencies together, following the dependency inversion principle.\n\n```typescript\n// infrastructure/container.ts - Composition Root\nclass Container {\n  private readonly services = new Map<string, any>();\n  private readonly factories = new Map<string, (container: Container) => any>();\n\n  register<T>(token: string, factory: (container: Container) => T): void {\n    this.factories.set(token, factory);\n  }\n\n  resolve<T>(token: string): T {\n    if (this.services.has(token)) {\n      return this.services.get(token);\n    }\n\n    const factory = this.factories.get(token);\n    if (!factory) {\n      throw new Error(`Service ${token} not registered`);\n    }\n\n    const instance = factory(this);\n    this.services.set(token, instance);\n    return instance;\n  }\n\n  registerSingleton<T>(token: string, instance: T): void {\n    this.services.set(token, instance);\n  }\n}\n\n// Setup dependency container\nconst container = new Container();\n\n// Register ports (abstractions)\ncontainer.register('OrderRepository', () => new TypeORMOrderRepository(\n  container.resolve('EntityManager'),\n  container.resolve('EventPublisher')\n));\n\ncontainer.register('InventoryService', () => new InventoryServiceImpl(\n  container.resolve('InventoryRepository')\n));\n\ncontainer.register('PaymentGateway', () => new StripePaymentGateway(\n  container.resolve('StripeClient')\n));\n\ncontainer.register('EventPublisher', () => new KafkaEventPublisher(\n  container.resolve('KafkaProducer'),\n  container.resolve('TopicPrefix')\n));\n\ncontainer.register('NotificationService', () => new SendgridNotificationService(\n  container.resolve('SendgridClient')\n));\n\n// Register use case handlers\ncontainer.register('CreateOrderHandler', (c) => new CreateOrderHandler(\n  c.resolve('OrderRepository'),\n  c.resolve('InventoryService'),\n  c.resolve('PaymentGateway'),\n  c.resolve('EventPublisher'),\n  c.resolve('OrderFactory')\n));\n\ncontainer.register('ProcessPaymentHandler', (c) => new ProcessPaymentHandler(\n  c.resolve('OrderRepository'),\n  c.resolve('PaymentGateway')\n));\n\ncontainer.register('ShipOrderHandler', (c) => new ShipOrderHandler(\n  c.resolve('OrderRepository'),\n  c.resolve('InventoryService'),\n  c.resolve('NotificationService')\n));\n\n// Register inbound adapters (controllers)\ncontainer.register('CreateOrderController', (c) =>\n  new HTTPOrderController(c.resolve('CreateOrderHandler'))\n);\n\ncontainer.register('ProcessPaymentController', (c) =>\n  new HTTPPaymentController(c.resolve('ProcessPaymentHandler'))\n);\n\n// Register domain factories\ncontainer.register('OrderFactory', () => new OrderFactory(\n  container.resolve('ProductRepository')\n));\n\nexport { container };\n```\n\n---\n\n## CQRS Pattern in Layered Architecture\n\nCommand Query Responsibility Segregation (CQRS) separates operations that change state (Commands) from operations that read data (Queries). This pattern fits naturally with layered architecture.\n\n```typescript\n// application/ports/inbound/command.port.ts\nexport interface Command<Input, Output> {\n  execute(input: Input): Promise<Output>;\n}\n\n// application/ports/inbound/query.port.ts\nexport interface Query<Input, Output> {\n  execute(input: Input): Promise<Output>;\n}\n\n// application/use-cases/commands/create-order.command.ts\nexport interface CreateOrderCommandInput {\n  customerId: string;\n  items: Array<{ productId: string; quantity: number }>;\n  shippingAddress: Address;\n}\n\nexport interface CreateOrderCommandOutput {\n  orderId: string;\n  total: number;\n}\n\nexport interface CreateOrderCommand extends Command<CreateOrderCommandInput, CreateOrderCommandOutput> {\n}\n\n// application/use-cases/queries/get-order.query.ts\nexport interface GetOrderQueryInput {\n  orderId: string;\n  includeItems?: boolean;\n}\n\nexport interface GetOrderQueryOutput {\n  order: OrderDto | null;\n  error?: string;\n}\n\nexport interface GetOrderQuery extends Query<GetOrderQueryInput, GetOrderQueryOutput> {\n}\n\n// Different handlers for different operations\nexport class CreateOrderCommandHandler implements CreateOrderCommand {\n  // Modifies state - returns new order ID and total\n}\n\nexport class GetOrderQueryHandler implements GetOrderQuery {\n  // Only reads - returns order data, never modifies state\n}\n```\n\n---\n\n## Data Transfer Objects (DTOs) and Mappers\n\nDTOs transfer data between layers without behavior. Mappers translate between different representations in different layers.\n\n```typescript\n// application/dtos/order.dto.ts\nexport class OrderDto {\n  id: string;\n  customerId: string;\n  status: string;\n  total: number;\n  currency: string;\n  items: OrderLineItemDto[];\n  shippingAddress: AddressDto;\n  createdAt: Date;\n  updatedAt: Date;\n}\n\nexport class OrderLineItemDto {\n  productId: string;\n  productName: string;\n  quantity: number;\n  unitPrice: number;\n  subtotal: number;\n}\n\nexport class AddressDto {\n  street: string;\n  city: string;\n  state: string;\n  postalCode: string;\n  country: string;\n}\n\n// infrastructure/mappers/order.mapper.ts\nexport class OrderMapper {\n  // Domain → DTO\n  static toDto(order: Order): OrderDto {\n    return {\n      id: order.id.value,\n      customerId: order.customerId,\n      status: order.status.value,\n      total: order.total.amountValue,\n      currency: order.total.currencyCode,\n      items: order.lineItems.map(item => ({\n        productId: item.productId,\n        productName: item.productName,\n        quantity: item.quantity,\n        unitPrice: item.unitPrice.amountValue,\n        subtotal: item.subtotal.amountValue\n      })),\n      shippingAddress: {\n        street: order.shippingAddress.street,\n        city: order.shippingAddress.city,\n        state: order.shippingAddress.state,\n        postalCode: order.shippingAddress.postalCode,\n        country: order.shippingAddress.country\n      },\n      createdAt: order.createdAt,\n      updatedAt: order.updatedAt\n    };\n  }\n\n  // Domain → Persistence Object\n  static toPersistence(order: Order): OrderPo {\n    return {\n      id: order.id.value,\n      customer_id: order.customerId,\n      status: order.status.value,\n      total_amount: order.total.amountValue,\n      currency: order.total.currencyCode,\n      shipping_address: JSON.stringify(order.shippingAddress),\n      created_at: order.createdAt,\n      updated_at: new Date()\n    };\n  }\n\n  // Persistence Object → Domain\n  static toDomain(po: OrderPo): Order {\n    return Order.reconstitute({\n      id: OrderId.of(po.id),\n      customerId: po.customer_id,\n      lineItems: [], // Would need to load from separate table\n      status: OrderStatus.of(po.status),\n      shippingAddress: JSON.parse(po.shipping_address),\n      createdAt: po.created_at\n    });\n  }\n\n  // DTO → Input (for use cases)\n  static toCreateOrderInput(dto: OrderDto): CreateOrderInput {\n    return {\n      customerId: dto.customerId,\n      items: dto.items.map(item => ({\n        productId: item.productId,\n        quantity: item.quantity\n      })),\n      shippingAddress: Address.of(dto.shippingAddress)\n    };\n  }\n}\n```\n\n---\n\n## Read-Only Interface Pattern\n\nPrevent unintended modifications by exposing read-only views of domain objects.\n\n```typescript\n// domain/interfaces/read-only-order.interface.ts\nexport interface ReadOnlyOrder {\n  getId(): OrderId;\n  getCustomerId(): string;\n  getStatus(): OrderStatus;\n  getTotal(): Money;\n  getLineItems(): ReadonlyArray<ReadOnlyOrderLineItem>;\n  getCreatedAt(): Date;\n}\n\n// infrastructure/adapters/presenters/read-only-order.adapter.ts\nexport class ReadOnlyOrderAdapter implements ReadOnlyOrder {\n  constructor(private order: Order) {}\n\n  getId(): OrderId {\n    return this.order.id;\n  }\n\n  getCustomerId(): string {\n    return this.order.customerId;\n  }\n\n  getStatus(): OrderStatus {\n    return this.order.status;\n  }\n\n  getTotal(): Money {\n    return this.order.total;\n  }\n\n  getLineItems(): ReadonlyArray<ReadOnlyOrderLineItem> {\n    return this.order.lineItems.map(item => new ReadOnlyOrderLineItemAdapter(item));\n  }\n\n  getCreatedAt(): Date {\n    return this.order.createdAt;\n  }\n}\n\nexport class ReadOnlyOrderLineItemAdapter implements ReadOnlyOrderLineItem {\n  constructor(private item: OrderLineItem) {}\n\n  getProductId(): string {\n    return this.item.productId;\n  }\n\n  getQuantity(): number {\n    return this.item.quantity;\n  }\n\n  getUnitPrice(): Money {\n    return this.item.unitPrice;\n  }\n\n  getSubtotal(): Money {\n    return this.item.subtotal;\n  }\n}\n```\n\n---\n\n## Testing Strategy\n\nBecause of the clear separation, testing can be done at multiple levels, each with different characteristics and purposes.\n\n```typescript\n// 1. Domain Layer - Pure unit tests, no dependencies\ndescribe('Order Entity', () => {\n  it('should create order with valid data', () => {\n    const orderId = OrderId.of('order-123');\n    const customerId = 'customer-456';\n    const lineItems = [\n      OrderLineItem.create({\n        productId: 'product-1',\n        quantity: 2,\n        unitPrice: Money.fromAmount(25.00)\n      })\n    ];\n    const status = OrderStatus.DRAFT;\n\n    const order = Order.create({\n      id: orderId,\n      customerId,\n      lineItems,\n      status,\n      shippingAddress: TestAddresses.VALID_ADDRESS,\n      paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD\n    });\n\n    expect(order.id).toEqual(orderId);\n    expect(order.customerId).toBe(customerId);\n    expect(order.total.amountValue).toBe(50.00);\n  });\n\n  it('should reject empty line items', () => {\n    expect(() => {\n      Order.create({\n        id: OrderId.of('order-123'),\n        customerId: 'customer-456',\n        lineItems: [],\n        status: OrderStatus.DRAFT,\n        shippingAddress: TestAddresses.VALID_ADDRESS,\n        paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD\n      });\n    }).toThrow(DomainError);\n  });\n\n  it('should calculate total correctly', () => {\n    const order = Order.create({\n      id: OrderId.of('order-123'),\n      customerId: 'customer-456',\n      lineItems: [\n        OrderLineItem.create({\n          productId: 'product-1',\n          quantity: 2,\n          unitPrice: Money.fromAmount(25.00)\n        }),\n        OrderLineItem.create({\n          productId: 'product-2',\n          quantity: 1,\n          unitPrice: Money.fromAmount(50.00)\n        })\n      ],\n      status: OrderStatus.DRAFT,\n      shippingAddress: TestAddresses.VALID_ADDRESS,\n      paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD\n    });\n\n    expect(order.total.amountValue).toBe(100.00);\n  });\n\n  it('should transition to SUBMITTED when submitted', () => {\n    const order = Order.create(TestOrderData.VALID_DRAFT_ORDER);\n\n    order.submit();\n\n    expect(order.status).toBe(OrderStatus.SUBMITTED);\n  });\n});\n\n// 2. Use Case Layer - Tests with in-memory repositories\ndescribe('CreateOrderHandler', () => {\n  let handler: CreateOrderHandler;\n  let orderRepository: InMemoryOrderRepository;\n  let inventoryService: MockInventoryService;\n  let paymentGateway: MockPaymentGateway;\n  let eventPublisher: MockEventPublisher;\n  let orderFactory: OrderFactory;\n\n  beforeEach(() => {\n    orderRepository = new InMemoryOrderRepository();\n    inventoryService = new MockInventoryService();\n    paymentGateway = new MockPaymentGateway();\n    eventPublisher = new MockEventPublisher();\n    orderFactory = new OrderFactory();\n\n    handler = new CreateOrderHandler(\n      orderRepository,\n      inventoryService,\n      paymentGateway,\n      eventPublisher,\n      orderFactory\n    );\n  });\n\n  it('should create order successfully', async () => {\n    const input: CreateOrderInput = {\n      customerId: 'customer-123',\n      items: [\n        { productId: 'product-1', quantity: 2 }\n      ],\n      shippingAddress: TestAddresses.VALID_ADDRESS,\n      paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD\n    };\n\n    const result = await handler.execute(input);\n\n    expect(result.isSuccess).toBe(true);\n    expect(result.orderId).toBeDefined();\n    expect(result.total).toBe(50.00);\n    expect(orderRepository.orders.size).toBe(1);\n  });\n\n  it('should fail when inventory is unavailable', async () => {\n    inventoryService.setAvailabilityCheckError(\n      new DomainError('Product is out of stock')\n    );\n\n    const input: CreateOrderInput = {\n      customerId: 'customer-123',\n      items: [\n        { productId: 'out-of-stock-product', quantity: 1 }\n      ],\n      shippingAddress: TestAddresses.VALID_ADDRESS,\n      paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD\n    };\n\n    const result = await handler.execute(input);\n\n    expect(result.isSuccess).toBe(false);\n    expect(result.error).toBe('Product is out of stock');\n    expect(orderRepository.orders.size).toBe(0);\n  });\n});\n\n// 3. Integration Tests - Full stack with test adapters\ndescribe('Order API Integration', () => {\n  it('should create order via HTTP API', async () => {\n    const app = createTestApp();\n    const response = await request(app)\n      .post('/api/orders')\n      .send({\n        customerId: 'customer-123',\n        items: [{ productId: 'product-1', quantity: 2 }],\n        shippingAddress: TestAddresses.VALID_ADDRESS,\n        paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD\n      })\n      .expect(201);\n\n    expect(response.body.success).toBe(true);\n    expect(response.body.data.orderId).toBeDefined();\n  });\n});\n\n// 4. Contract Tests - Adapter contract verification\ndescribe('OrderRepository Contract', () => {\n  let repository: OrderRepository;\n\n  beforeEach(() => {\n    repository = new InMemoryOrderRepository();\n  });\n\n  it('should save and find order by id', async () => {\n    const order = Order.create(TestOrderData.VALID_DRAFT_ORDER);\n\n    await repository.save(order);\n    const found = await repository.findById(order.id);\n\n    expect(found).toEqual(order);\n  });\n\n  it('should return null for non-existent order', async () => {\n    const found = await repository.findById(OrderId.of('non-existent'));\n\n    expect(found).toBeNull();\n  });\n});\n```\n\n---\n\n## Benefits of Layered Architecture\n\n| Benefit | Description |\n|---------|-------------|\n| **Testability** | Business rules can be tested without UI, database, or external systems using pure unit tests |\n| **Maintainability** | Changes to outer layers (frameworks, databases) do not affect inner layers (domain, use cases) |\n| **Flexibility** | External systems and delivery mechanisms can be swapped without changing business code |\n| **Scalability** | Clear separation enables parallel development by multiple teams |\n| **Framework Independence** | Business logic survives framework changes and technology migrations |\n| **Adaptability** | Application can be delivered through multiple channels (HTTP, gRPC, CLI, messaging) |\n| **Longevity** | Core business logic outlives specific technology choices |\n| **Test doubles** | Easy to create mock implementations of ports for testing |\n| **Technology Deferral** | Technology choices can be deferred until the outer layers |\n| **Domain Focus** | Business logic remains pure and focused on domain rules |\n\n---\n\n## Anti-Patterns to Avoid\n\n| Anti-Pattern | Description | Solution |\n|--------------|-------------|----------|\n| **Anemic Domain** | Entities with no behavior, logic in services | Move business logic from services to domain entities |\n| **Dependency Inversion Violation** | Domain depending on frameworks or infrastructure | Use dependency injection, define ports in domain/application |\n| **Feature Scattered Across Layers** | Related code in multiple layers | Organize by feature module, not by layer |\n| **God Interface** | One port/interface for everything | Separate inbound/outbound ports, single responsibility |\n| **Skipping Layers** | Controllers directly calling repositories | Always go through use cases for business operations |\n| **Leaking Infrastructure** | Domain entities with framework annotations | Keep domain pure, use adapters for persistence mapping |\n| **Concrete Dependencies** | Use cases depending on concrete implementations | Depend on abstractions (interfaces), inject implementations |\n| **Missing Ports** | Direct imports of infrastructure in core | Define ports, implement in adapters |\n\n### Common Violations and Fixes\n\n```typescript\n// WRONG - Domain depends on JPA/ORM annotations\n@Entity\nclass Order {\n  @Id\n  private id: string;\n  \n  @Column\n  private status: string;\n}\n\n// RIGHT - Pure domain entity\nexport class Order {\n  private constructor(\n    private readonly id: OrderId,\n    private readonly status: OrderStatus\n  ) {}\n}\n\n// WRONG - Controller directly calls repository\nclass OrderController {\n  async handle() {\n    // Bypasses use case layer\n    await this.entityManager.save(order);\n  }\n}\n\n// RIGHT - Controller calls use case\nclass OrderController {\n  async handle(request: Request): Promise<Response> {\n    const result = await this.createOrderHandler.execute(request.body);\n    return Response.of(result);\n  }\n}\n\n// WRONG - Use case depends on concrete implementation\nclass CreateOrderHandler {\n  constructor(\n    private repository: TypeORMOrderRepository // Should be interface\n  ) {}\n}\n\n// RIGHT - Use case depends on abstraction\nclass CreateOrderHandler {\n  constructor(\n    private repository: OrderRepository // Interface\n  ) {}\n}\n```\n\n---\n\n## Layered Architecture in AI-Assisted Development\n\n```markdown\n## AI Prompt for Layered Architecture Implementation\n\nGenerate a feature following Layered Architecture (Ports and Adapters):\n\nContext: E-commerce Order Management System\n\nRequirements:\n1. Create order from shopping cart items\n2. Validate inventory availability\n3. Process payment\n4. Send order confirmation notification\n5. Publish order created event\n\nGenerate:\n1. Domain entities (Order, OrderLineItem)\n2. Value objects (Money, Address, OrderId)\n3. Domain events (OrderCreated, PaymentProcessed)\n4. Use case handler (CreateOrderHandler)\n5. Input/output ports (interfaces)\n6. Repository interface for orders\n7. HTTP controller adapter\n8. TypeORM repository adapter\n9. Unit tests for domain rules\n10. Integration tests for use cases\n\nConstraints:\n- All dependencies point inward to domain\n- Domain has no imports from outer layers\n- Use TypeScript\n- Include JSDoc documentation\n- Follow dependency inversion principle\n- Support multiple payment gateways (strategy pattern)\n```\n\n---\n\n## Comparison with Related Patterns\n\n### Layered Architecture vs. Clean Architecture\n\nClean Architecture, introduced by Robert C. Martin, is essentially the same concept as Hexagonal Architecture with a slightly different layer naming convention. Clean Architecture typically includes four layers: Frameworks & Drivers, Interface Adapters, Application Business Rules, and Enterprise Business Rules. The core principles and dependency rules are identical.\n\n### Layered Architecture vs. Onion Architecture\n\nOnion Architecture, proposed by Jeffrey Palermo, uses similar concentric layer concepts but emphasizes the \"onion\" metaphor with dependencies pointing toward the center. The layers are often named: Application Core (inner), Domain Services, Application Services (outer), and Infrastructure. The fundamental principles are the same as Layered Architecture.\n\n### Layered Architecture vs. Microservices\n\nWhile both promote separation of concerns, Layered Architecture is an application-level pattern for organizing code within a single service, whereas Microservices is an architectural style for decomposing an entire system into independently deployable services. Layered Architecture principles can be applied within each microservice.\n\n---\n\n## References and Further Reading\n\n1. Cockburn, Alistair. \"Hexagonal Architecture.\" 2005.\n2. Martin, Robert C. \"Clean Architecture: A Craftsman's Guide to Software Structure and Design.\" Prentice Hall, 2017.\n3. Evans, Eric. \"Domain-Driven Design: Tackling Complexity in the Heart of Software.\" Addison-Wesley, 2004.\n4. Vernon, Vaughn. \"Implementing Domain-Driven Design.\" Addison-Wesley, 2013.\n5. Martin, Robert C. \"The Dependency Inversion Principle.\" 1996.\n6. Percival, Harry, and Bob Gregory. \"Architecture Patterns with Python.\" O'Reilly, 2021.\n7. Ford, Neal, et al. \"Architecture: The Hard Parts.\" O'Reilly, 2021.\n8. Zimarev, Alexey. \"Hands-On Domain-Driven Design with .NET Core.\" Packt, 2019.\n",
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md"
}
```
```

---

## Assistant

_Thinking:_

The user asked me to compare the two architecture documents and merge them into layered-architecture.md. I've now created a comprehensive merged document that combines both Clean Architecture and Hexagonal Architecture concepts.

Let me verify the file was created successfully.

```
Tool: bash

**Input:**
```json
{
  "command": "ls -la E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "description": "Verify merged file was created"
}
```
**Output:**
```
ls: cannot access 'E:AI-Learnai-coding-promptsdoclayered-architecture.md': No such file or directory

```
```

---

## Assistant

_Thinking:_

The ls command failed, likely because of the Windows path. Let me try with a different approach to verify the file exists.

```
Tool: glob

**Input:**
```json
{
  "path": "E:\\AI-Learn\\ai-coding-prompts",
  "pattern": "**/layered-architecture.md"
}
```
**Output:**
```
E:\AI-Learn\ai-coding-prompts\doc\layered-architecture.md
```
```

---

## Assistant

_Thinking:_

The file was created successfully at E:\AI-Learn\ai-coding-prompts\doc\layered-architecture.md. The task is complete.

Created `doc/layered-architecture.md` by merging Clean Architecture and Hexagonal Architecture documentation. The merged file includes:

- **Unified overview** combining both patterns under "Layered Architecture"
- **Core principles** with dependency rule diagram showing inward-pointing dependencies
- **Three-layer structure** (Domain, Application, Adapter) with ports (input/output) and adapters (inbound/outbound)
- **Comprehensive directory structure** for a TypeScript/Node.js project
- **Implementation examples** for all layers with practical code samples
- **Dependency injection** patterns with composition root
- **CQRS pattern** integration
- **DTOs and mappers** for data transformation
- **Read-only interface pattern** for encapsulation
- **Testing strategy** with multiple test levels
- **Benefits table** and **anti-patterns to avoid** with code examples
- **AI prompt template** for generating layered architecture code

The document preserves the strengths of both source documents while eliminating redundancy and creating a cohesive reference.

---

## User

read this webpage https://teddy-chen-tw.blogspot.com/2020/08/clean-architecture.html and add this 3 principle in english into layered-architecture.md

---

## Assistant

_Thinking:_

The user wants me to:
1. Fetch the webpage from the given URL
2. Extract the 3 principles mentioned there (in Chinese based on the blog URL)
3. Translate those principles to English
4. Add them to the layered-architecture.md file

Let me fetch the webpage first.

```
Tool: webfetch

**Input:**
```json
{
  "format": "markdown",
  "url": "https://teddy-chen-tw.blogspot.com/2020/08/clean-architecture.html"
}
```
**Output:**
```
                  搞笑談軟工: 再談Clean Architecture三原則   

 

 

l

# [搞笑談軟工](https://teddy-chen-tw.blogspot.com/)

敏捷開發，設計模式，精實開發，Scrum，軟體設計，軟體架構

## 網頁

-   [首頁](https://teddy-chen-tw.blogspot.com/)
-   [泰迪軟體最新課表](http://teddysoft.tw/courses/)
-   [泰迪軟體Facebook](https://www.facebook.com/teddysoft.tw)
-   [搞笑談軟工Facebook社團](https://www.facebook.com/groups/teddy.tw/)

## 2020年8月19日 星期三

  

### 再談Clean Architecture三原則

August 19 08:45~10:00；12:56~13:38

![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhXStWo78JXq5d6H5T8R_0yYk0LnYbau3T2C9GEocQgQ2Xv7bTcGQZK9ECLJ75O09DCLuUOnQiPtiwztkObR9LmSa_1rmah8FrIwLYGjNFAw918G4jtY-g4tky33o5pg1zrmi6T2wtr_Ln0/w1390-h1332-no/?authuser=0)

  

2018年Teddy寫了幾篇介紹Clean Architecture的文章如下，其中有三篇提到Clean Architecture三原則：**分層**、**相依性**、**跨層**，今天再一次一起討論這三個原則。  

-   [Clean Architecture（1）：軟體架構的定義與目的](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture1.html)
-   [Clean Architecture（2）：Port and Adapter Architecture](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html)
-   [Clean Architecture（3）：精選參考範例](https://teddy-chen-tw.blogspot.com/2018/06/clean-architecture.html)
-   [Clean Architecture（4）：架構三原則首部曲—分層原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html)
-   [Clean Architecture（5）：架構三原則二部曲—相依性原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html)
-   [Clean Architecture（6）：架構三原則三部曲—跨層原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture6.html)

\*\*\*

**分層原則**

如圖1所示，Clean Architecture有四層：Entities、Use Cases、Interface Adapters、Frameworks & Drivers。分層原則乍看之下最簡單也最沒爭議，但實際上有些物件要放在哪一層還是存在灰色地帶。

最核心的Entities存放domain model物件。如果對照領域驅動設計（Domain-Driven Design；DDD），Aggregate、Entity、Value Object、Domain Service就放在這一層。應用程式邏輯放在Use Cases這一層，也就是DDD裡面的Application Service。

先看這兩層就好，**請問DDD的Repository要放在哪裡？**大部分的人覺得要放在Use Cases這一層，但也有人主張要放在Entities。所以，雖然Clean Architecture這四層大方向來講區分得很清楚，但在實作的時候有一些「支援物件」，像是剛剛提到的Repository以及Mapper等實際上要放在哪裡，還是有一些討論的空間。

  

[![CleanArchitecture](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiofyVn5BrjrUx64_FhXuCslg8jX7HZiX5Z_MW-0QvcYw-qS_h0XUIY-iNXk2yfV1LNPnumnP22_6zwdGhOMnllqbbMwMiXJJVVtrC2f2FLirNXBppQPGySWqwuPg77xG1lNwMQ6NwyKbvF/?imgmax=800 "CleanArchitecture")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgHOugVUfASncB4KxEx7TI050IHQP4FCrNY1Ghx0bL5KS_5orpSd4ATCjoaW0016KCRhAbg4GCyN2pJdvKZ0c4GkVh5cDGazHG8VaMD3Wr6uc3Ak_OZfikBja_MjQcQNSpucYDX1RUsBd52/s1600-h/CleanArchitecture%255B4%255D)

▲圖1：《Clean Architecture》建議的四層架構，畫成同心圓（圖片來源[在此](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)）。

\*\*\*

**相依性原則**

原始碼的依賴方向必須由外往內、由低層往高層。也就是說Entities層的物件只可以參考（依賴）同一層的其他物件，不可以參考其他層的物件。Use Cases層只可以往內參考Entities層以及自己這一層的物件，不可以往外參考其他層的物件，依此類推。

但是，如果Use Cases需要將Aggregates儲存到資料庫，那一定要參考到最外層的資料庫物件啊，這怎麼辦？如圖2所示，Clean Architecture透過**依賴反轉（Dependency Inversion Principle；DIP）**確保相依性原則可以被遵守。

  

![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMgSwAr7b5jH-KtbPd14R9dclABccJomHiNQ-He8J0kFCjc5_cyE7F4CyrrER2hESgWBp6J-6P8_3bG8-6ClfdH-9BHoGnFrDK-icnHHLOnvHTA92Cfgfu1ytc-reaMoL0iQk2VWX-r0IU/w2618-h1560-no/?authuser=0)

▲圖2：透過依賴反轉做到相依性由外往內。

現在問題來了，圖2的Workflow aggregate透過AddWorkflowUseCase把Workflow儲存到MySQL資料庫，**這樣子有沒有違反相依性原則？**

雖然SqlWorkflowRepository與MySQL都**跨層**直接參考Workflow，但相依性的方向還是由外向內，因此並沒有違反相依性原則。但是，這麼做違反了Clean Architecture的第三個原則：跨層原則。

相依性原則要完全遵守也是不容易的一件事，例如Entities層的Aggregate要發Domain Event（領域事件）需要用到event bus公具程式，最簡單的方式是找人家已經寫好的工具來使用，但是這樣就違反了相依性原則。為了滿足相依性原則，就要幫著個event bus工具程式定義介面，透過依賴反轉做到相依性原則。有時候定義依賴反轉介面很簡單，但有時候卻不容易。

\*\*\*

**跨層原則**

當Entities層的物件跨越離開Use Case層往外傳遞時（傳到UI或是DB或其他外部系統），不要直接把Entities層的物件傳出Use Case層，而是要在Use Cases層定義往外傳遞的介面與資料結構。簡單說就是把Entities層的物件轉成DTO（Data Transfer Object）再往外傳遞。

如果直接把Entities層的物件往外傳，例如直接傳給UI，很有可能因為UI端的需求，導致回頭影響Entities層的物件。也就是說，presentation logic影響了domain logic。為了做到分層負責與單一責任，所以物件跨層的時候要轉一次資料型態。

\*\*\*

**潔癖**

完全做到Clean Architecture的三個原則，系統架構就會變得很乾淨，但實作上卻會很麻煩。就好像跟一個有潔癖的人住在一起，居住環境一定很乾淨，但你可能會被對方煩死。一天要擦三次地板、衣服髒了要馬上洗、裝冷飲的杯子要用杯墊，不可以讓水滴到桌上、回家要馬上消毒等等。

這幾年實作Clean Architecture的經驗，Teddy覺得只要做到以下幾點即可，其餘部分可以不用那麼「乾淨」：

-   相依性原則在大部分的情況下都要遵守，特別是Entities層與Use Case層。如果真的需要使用到一些工具軟體且又**不能**或**不想**透過依賴反轉滿足相依性原則，這樣的特例不能太多，且要自己心裡有數，承受這樣的依賴關係。
-   Interface Adapters層經常與框架緊密相關，例如Web Controller和所使用的Web Container框架緊密相關。在這種情況下，要做的完全的依賴反轉是很痛苦的一件事。所以Interface Adapters層放生，就讓它和框架緊密耦合吧。
-   把Use Cases分成Command與Query可以簡化很多Interface Adapters層的物件設計。Command是會改變系統狀態但不回傳值（或只回傳物件的id等特定資料）的操作，Query是回傳資料但不會改變系統狀態的操作。基本上Command可以共用同一個Presenter與View Model，只有Query需要客製化的Presenter和View Model。區分Command和Query之後，Interface Adapters層可以清爽許多。

\*\*\*

友藏內心獨白：沒有絕對的乾淨，無塵室也是有灰塵。

張貼者： [Teddy Chen](https://www.blogger.com/profile/02066842119056439711 "author profile") 於  [上午8:57](https://teddy-chen-tw.blogspot.com/2020/08/clean-architecture.html "permanent link")  [![](//img1.blogblog.com/img/icon18_email.gif)](https://www.blogger.com/email-post/1298974142445162186/2204387477147378159 "電子郵件發布")[![](https://resources.blogblog.com/img/icon18_edit_allbkg.gif)](https://www.blogger.com/post-edit.g?blogID=1298974142445162186&postID=2204387477147378159&from=pencil "編輯文章")

[以電子郵件傳送這篇文章](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2204387477147378159&target=email "以電子郵件傳送這篇文章")[BlogThis！](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2204387477147378159&target=blog "BlogThis！")[分享至 X](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2204387477147378159&target=twitter "分享至 X")[分享至 Facebook](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2204387477147378159&target=facebook "分享至 Facebook")[分享到 Pinterest](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2204387477147378159&target=pinterest "分享到 Pinterest")

標籤： [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B), [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88), [DDD](https://teddy-chen-tw.blogspot.com/search/label/DDD)

#### 沒有留言:

#### 張貼留言

[](https://www.blogger.com/comment/frame/1298974142445162186?po=2204387477147378159&hl=zh-TW&saa=85391&origin=https://teddy-chen-tw.blogspot.com)

[較新的文章](https://teddy-chen-tw.blogspot.com/2020/08/clean-architecture7cqrspresenterview.html "較新的文章") [較舊的文章](https://teddy-chen-tw.blogspot.com/2020/08/10repository.html "較舊的文章") [首頁](https://teddy-chen-tw.blogspot.com/)

訂閱： [張貼留言 (Atom)](https://teddy-chen-tw.blogspot.com/feeds/2204387477147378159/comments/default)

## 搜尋此網站

 

## 總網頁瀏覽量

## 關於

[![我的相片](//blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBBqKlknBrSZ3EEXiWqeZasJOxrwoUfdqqF4Es3OR0YFtwDAYNHp6bBx3PqLGVJXAJQ4INhfp0gUlRZyRoSF9vzraU_JlLOECrERAWag3VIa0JdxRhpLwyEz7w9QtX8w/s220/Teddy.jpg)](https://www.blogger.com/profile/02066842119056439711)

[Teddy Chen](https://www.blogger.com/profile/02066842119056439711)

[檢視我的完整簡介](https://www.blogger.com/profile/02066842119056439711)

## 請幫Teddy按個讚^\_^

## 標籤

-   [工商服務](https://teddy-chen-tw.blogspot.com/search/label/%E5%B7%A5%E5%95%86%E6%9C%8D%E5%8B%99) (94)
-   [生活](https://teddy-chen-tw.blogspot.com/search/label/%E7%94%9F%E6%B4%BB) (342)
-   [改行寫網路小說算了](https://teddy-chen-tw.blogspot.com/search/label/%E6%94%B9%E8%A1%8C%E5%AF%AB%E7%B6%B2%E8%B7%AF%E5%B0%8F%E8%AA%AA%E7%AE%97%E4%BA%86) (25)
-   [其他](https://teddy-chen-tw.blogspot.com/search/label/%E5%85%B6%E4%BB%96) (25)
-   [客戶教我的事](https://teddy-chen-tw.blogspot.com/search/label/%E5%AE%A2%E6%88%B6%E6%95%99%E6%88%91%E7%9A%84%E4%BA%8B) (2)
-   [持續整合](https://teddy-chen-tw.blogspot.com/search/label/%E6%8C%81%E7%BA%8C%E6%95%B4%E5%90%88) (29)
-   [哲學](https://teddy-chen-tw.blogspot.com/search/label/%E5%93%B2%E5%AD%B8) (13)
-   [旅遊](https://teddy-chen-tw.blogspot.com/search/label/%E6%97%85%E9%81%8A) (568)
-   [視力測驗](https://teddy-chen-tw.blogspot.com/search/label/%E8%A6%96%E5%8A%9B%E6%B8%AC%E9%A9%97) (3)
-   [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5) (410)
-   [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B) (110)
-   [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88) (188)
-   [最新課程](https://teddy-chen-tw.blogspot.com/search/label/%E6%9C%80%E6%96%B0%E8%AA%B2%E7%A8%8B) (81)
-   [創業](https://teddy-chen-tw.blogspot.com/search/label/%E5%89%B5%E6%A5%AD) (84)
-   [測試](https://teddy-chen-tw.blogspot.com/search/label/%E6%B8%AC%E8%A9%A6) (106)
-   [盡信書不如無書](https://teddy-chen-tw.blogspot.com/search/label/%E7%9B%A1%E4%BF%A1%E6%9B%B8%E4%B8%8D%E5%A6%82%E7%84%A1%E6%9B%B8) (2)
-   [需求](https://teddy-chen-tw.blogspot.com/search/label/%E9%9C%80%E6%B1%82) (23)
-   [課程實錄](https://teddy-chen-tw.blogspot.com/search/label/%E8%AA%B2%E7%A8%8B%E5%AF%A6%E9%8C%84) (22)
-   [學習](https://teddy-chen-tw.blogspot.com/search/label/%E5%AD%B8%E7%BF%92) (92)
-   [貓](https://teddy-chen-tw.blogspot.com/search/label/%E8%B2%93) (15)
-   [還少一本書](https://teddy-chen-tw.blogspot.com/search/label/%E9%82%84%E5%B0%91%E4%B8%80%E6%9C%AC%E6%9B%B8) (36)
-   [agile](https://teddy-chen-tw.blogspot.com/search/label/agile) (638)
-   [C. C. Agile](https://teddy-chen-tw.blogspot.com/search/label/C.%20C.%20Agile) (33)
-   [DDD](https://teddy-chen-tw.blogspot.com/search/label/DDD) (85)
-   [DevOps](https://teddy-chen-tw.blogspot.com/search/label/DevOps) (3)
-   [Eclipse](https://teddy-chen-tw.blogspot.com/search/label/Eclipse) (6)
-   [exception handling](https://teddy-chen-tw.blogspot.com/search/label/exception%20handling) (84)
-   [HCI](https://teddy-chen-tw.blogspot.com/search/label/HCI) (44)
-   [Implementation Patterns](https://teddy-chen-tw.blogspot.com/search/label/Implementation%20Patterns) (5)
-   [Kanban](https://teddy-chen-tw.blogspot.com/search/label/Kanban) (79)
-   [Mobile](https://teddy-chen-tw.blogspot.com/search/label/Mobile) (2)
-   [Patterns](https://teddy-chen-tw.blogspot.com/search/label/Patterns) (198)
-   [Programming](https://teddy-chen-tw.blogspot.com/search/label/Programming) (54)
-   [Refactoring](https://teddy-chen-tw.blogspot.com/search/label/Refactoring) (65)
-   [Scrum](https://teddy-chen-tw.blogspot.com/search/label/Scrum) (461)
-   [TDD/BDD](https://teddy-chen-tw.blogspot.com/search/label/TDD%2FBDD) (54)

## 網誌存檔

-   [►](javascript:void\(0\))  [2025](https://teddy-chen-tw.blogspot.com/2025/) (1)
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2025/06/) (1)

-   [►](javascript:void\(0\))  [2024](https://teddy-chen-tw.blogspot.com/2024/) (7)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2024/09/) (6)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2024/03/) (1)

-   [►](javascript:void\(0\))  [2023](https://teddy-chen-tw.blogspot.com/2023/) (25)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2023/09/) (5)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2023/07/) (3)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2023/06/) (5)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2023/03/) (10)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2023/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2023/01/) (1)

-   [►](javascript:void\(0\))  [2022](https://teddy-chen-tw.blogspot.com/2022/) (29)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2022/10/) (1)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2022/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2022/07/) (18)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2022/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2022/03/) (2)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2022/02/) (1)

-   [►](javascript:void\(0\))  [2021](https://teddy-chen-tw.blogspot.com/2021/) (27)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2021/12/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2021/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2021/09/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2021/08/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2021/06/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2021/04/) (1)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2021/03/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2021/01/) (8)

-   [▼](javascript:void\(0\))  [2020](https://teddy-chen-tw.blogspot.com/2020/) (32)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2020/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2020/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2020/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2020/09/) (4)
    
    -   [▼](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2020/08/) (4)
        -   [三種團隊互動模式](https://teddy-chen-tw.blogspot.com/2020/08/blog-post.html)
        -   [Clean Architecture（7）：套用CQRS簡化Presenter與View Model](https://teddy-chen-tw.blogspot.com/2020/08/clean-architecture7cqrspresenterview.html)
        -   [再談Clean Architecture三原則](https://teddy-chen-tw.blogspot.com/2020/08/clean-architecture.html)
        -   [領域驅動設計學習筆記（10）：實作Repository](https://teddy-chen-tw.blogspot.com/2020/08/10repository.html)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2020/07/) (2)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2020/06/) (2)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2020/05/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2020/04/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2020/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2020/02/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2020/01/) (1)

-   [►](javascript:void\(0\))  [2019](https://teddy-chen-tw.blogspot.com/2019/) (50)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2019/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2019/11/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2019/10/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2019/07/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2019/06/) (1)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2019/05/) (8)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2019/04/) (2)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2019/03/) (8)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2019/02/) (8)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2019/01/) (7)

-   [►](javascript:void\(0\))  [2018](https://teddy-chen-tw.blogspot.com/2018/) (28)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2018/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2018/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2018/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2018/08/) (6)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2018/07/) (6)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2018/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2018/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2018/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2018/01/) (3)

-   [►](javascript:void\(0\))  [2017](https://teddy-chen-tw.blogspot.com/2017/) (142)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2017/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2017/11/) (8)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2017/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2017/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2017/08/) (15)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2017/07/) (10)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2017/06/) (14)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2017/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2017/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2017/03/) (23)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2017/02/) (16)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2017/01/) (25)

-   [►](javascript:void\(0\))  [2016](https://teddy-chen-tw.blogspot.com/2016/) (301)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2016/12/) (14)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2016/11/) (19)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2016/10/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2016/09/) (18)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2016/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2016/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2016/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2016/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2016/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2016/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2016/02/) (29)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2016/01/) (31)

-   [►](javascript:void\(0\))  [2015](https://teddy-chen-tw.blogspot.com/2015/) (366)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2015/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2015/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2015/10/) (32)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2015/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2015/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2015/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2015/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2015/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2015/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2015/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2015/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2015/01/) (31)

-   [►](javascript:void\(0\))  [2014](https://teddy-chen-tw.blogspot.com/2014/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2014/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2014/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2014/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2014/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2014/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2014/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2014/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2014/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2014/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2014/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2014/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2014/01/) (31)

-   [►](javascript:void\(0\))  [2013](https://teddy-chen-tw.blogspot.com/2013/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2013/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2013/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2013/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2013/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2013/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2013/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2013/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2013/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2013/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2013/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2013/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2013/01/) (31)

-   [►](javascript:void\(0\))  [2012](https://teddy-chen-tw.blogspot.com/2012/) (373)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2012/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2012/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2012/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2012/09/) (31)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2012/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2012/07/) (32)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2012/06/) (31)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2012/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2012/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2012/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2012/02/) (32)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2012/01/) (32)

-   [►](javascript:void\(0\))  [2011](https://teddy-chen-tw.blogspot.com/2011/) (120)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2011/12/) (16)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2011/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2011/10/) (4)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2011/09/) (3)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2011/08/) (9)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2011/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2011/06/) (10)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2011/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2011/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2011/03/) (16)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2011/02/) (12)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2011/01/) (11)

-   [►](javascript:void\(0\))  [2010](https://teddy-chen-tw.blogspot.com/2010/) (104)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2010/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2010/11/) (6)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2010/10/) (10)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2010/09/) (5)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2010/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2010/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2010/06/) (19)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2010/05/) (12)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2010/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2010/03/) (12)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2010/02/) (2)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2010/01/) (7)

-   [►](javascript:void\(0\))  [2009](https://teddy-chen-tw.blogspot.com/2009/) (31)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2009/12/) (5)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2009/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2009/10/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2009/08/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2009/07/) (8)

-   [►](javascript:void\(0\))  [2008](https://teddy-chen-tw.blogspot.com/2008/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2008/09/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2008/03/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2008/01/) (2)

-   [►](javascript:void\(0\))  [2007](https://teddy-chen-tw.blogspot.com/2007/) (4)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2007/12/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2007/06/) (3)

## 追蹤者

## 最新回應

[載入中…](http://teddy-chen-tw.blogspot.tw/feeds/comments/default)

## 熱門文章

-   [爭什麼，摻在一起做成撒尿牛丸啊](https://teddy-chen-tw.blogspot.com/2012/03/blog-post_20.html)
    
    March 20 13:53~14:56 (影像節錄自http://www.youtube.com/watch?v=LGV5PcM9SQo) 鄉民們都看過食神這部電影吧，電影中有一個經典的橋段，就是火鷄姐及鵝頭在爭辯誰做的撒尿蝦好吃，誰做的牛丸比較優。正當雙方吵得不可...
    
-   [擁抱改變，遠離Hard Code](https://teddy-chen-tw.blogspot.com/2018/09/hard-code.html)
    
    September 19 08:37~09:12 ▲圖片來源 在此 寫死（Hard Code） 幾天前一位朋友在Facebook上發問：「如何對非資訊人員解釋寫死（hard code）？」 維基百科 對於寫死的解釋如下： 寫死 （英語： Hard Code 或 Hard Co...
    
-   [什麼是物件導向（3）：Polymorphism](https://teddy-chen-tw.blogspot.com/2012/01/3polymorphism.html)
    
    January 26 23:01~January 27 00:23 Polymorphism，中文翻譯成「多型」，說真的這個概念當年Teddy學OO的時候搞了好久才弄懂，後來看了「課本」的定義之後，覺得寫得還滿好的，請參考Object-Oriented Software En...
    
-   [幫Macbook Pro 15” Late 2016尋找外接4K 27”螢幕](https://teddy-chen-tw.blogspot.com/2017/07/macbook-pro-15-late-20164k-27.html)
    
    June 17 21:28~23:24 先打個廣告，【 敏捷開發懶人包：物件導向技能 】確定於8月19日（週六）開課。 課程設定目標希望讓鄉民們在找工作面試的時候，如果面試官問到物件導向技術方面的問題，能夠具備輕鬆回答的能力 。 \*\*\* 動心起念 去年底買了MacBook P...
    
-   [Top-down和Bottom-up設計方法](https://teddy-chen-tw.blogspot.com/2014/03/top-downbottom-up.html)
    
    Mar. 12 08:40~10:10 Top-down（由上而下）和Bottom-up（由下而上）是兩種設計與解決問題的技巧。前者對問題先有一個整體的概念，然後再逐步加上設計細節，最後讓整體的輪廓越來越清楚。後者則是先將解決問題可能所需的基本元件、方案給準備好，然後再將這...
    
-   [羅技Logitech R500 雷射簡報筆開箱](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html)
    
    June 22 12:51~14:16 需求分析 因為工作的關係Teddy經常需要使用簡報筆，雜牌的不算，光是羅技的簡報筆Teddy就買過以下四種（R500是最新上市產品，也是今天介紹的主角）  ： ▼畫面節錄自羅技官網 Teddy心中理想的簡報筆應該有以下幾個功能： 重量輕且...
    

## 分析

 

技術提供：[Blogger](https://www.blogger.com).
```
```

---

## Assistant

_Thinking:_

I found the article about Clean Architecture's three principles. Let me extract the key points from the article:

The three principles are:
1. **分層原則** (Layering Principle) - Divide the architecture into layers
2. **相依性原則** (Dependency Principle) - Source code dependencies must point inward, from outer to inner layers
3. **跨層原則** (Cross-Layer Principle) - When domain objects cross layers to the outside, convert them to DTOs instead of passing them directly

Let me now read the current layered-architecture.md file and add these three principles in English.

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md"
}
```
**Output:**
```
<file>
00001| # Layered Architecture (Ports and Adapters / Clean Architecture)
00002| 
00003| ## Overview
00004| 
00005| Layered Architecture, also known as **Hexagonal Architecture** (Ports and Adapters) or **Clean Architecture**, is a software design philosophy that emphasizes the separation of concerns through architectural layers. The core principle is that business rules and application logic should remain independent of external frameworks, databases, delivery mechanisms, and user interfaces.
00006| 
00007| This architectural pattern was formalized by Alistair Cockburn as Hexagonal Architecture and further developed by Robert C. Martin (Uncle Bob) as Clean Architecture. Both approaches share the same fundamental insight: business logic should not depend on databases, web frameworks, or any external system. Instead, external systems depend on abstractions defined by the business logic. This inversion of dependencies ensures that the core application can be tested independently of its delivery mechanisms and that external systems can be replaced without changing business code.
00008| 
00009| The architecture creates clear boundaries between what the application does (core/domain) and how it integrates with the outside world (adapters). This separation enables teams to work on different parts independently, allows technology choices to be deferred, and ensures that core business logic outlives specific technology choices.
00010| 
00011| ---
00012| 
00013| ## Core Principles
00014| 
00015| ### The Dependency Rule
00016| 
00017| The most critical rule in layered architectures is that **dependencies always point inward** toward the domain. This is the Dependency Inversion Principle applied at the architectural level. Inner layers do not know about outer layers, and all source code dependencies point toward the more abstract inner layers.
00018| 
00019| ```
00020|                     ┌───────────────────────────────────────┐
00021|                     │          External Systems             │  ← Outermost Layer
00022|                     │  (Web, DB, UI, External Services)    │
00023|                     └───────────────────┬───────────────────┘
00024|                                         │
00025|                     ┌───────────────────▼───────────────────┐
00026|                     │            Adapters Layer             │  ← Interface Adapters
00027|                     │  (Controllers, Gateways, Presenters,  │
00028|                     │   Database Adapters, External Clients)│
00029|                     └───────────────────┬───────────────────┘
00030|                                         │
00031|                     ┌───────────────────▼───────────────────┐
00032|                     │         Application Layer             │  ← Use Cases / Services
00033|                     │  (Interactors, Use Case Coordinators, │
00034|                     │   Application Services)               │
00035|                     └───────────────────┬───────────────────┘
00036|                                         │
00037|                     ┌───────────────────▼───────────────────┐
00038|                     │            Domain Layer               │  ← Innermost Layer
00039|                     │  (Entities, Value Objects, Aggregates,│
00040|                     │   Domain Services, Domain Events)     │
00041|                     └───────────────────────────────────────┘
00042| 
00043|      Source code dependencies point INWARD toward the domain.
00044|      Inner layers do NOT know about outer layers.
00045| ```
00046| 
00047| ### Key Principles
00048| 
00049| 1. **Independence of Frameworks**: Business logic remains independent of frameworks and can be tested without them
00050| 2. **Testability**: Business rules are testable without UI, database, or external systems
00051| 3. **Independence of UI**: User interfaces can change without affecting business logic
00052| 4. **Independence of Database**: Business logic is decoupled from database implementation details
00053| 5. **Independence of External Agencies**: Business rules have no knowledge of the outside world
00054| 6. **Framework Independence**: Business logic survives framework changes and upgrades
00055| 
00056| ---
00057| 
00058| ## Architecture Overview
00059| 
00060| ```
00061| ┌─────────────────────────────────────────────────────────────────────────┐
00062| │                    LAYERED ARCHITECTURE                                 │
00063| ├─────────────────────────────────────────────────────────────────────────┤
00064| │                                                                         │
00065| │                         ┌─────────────────────┐                         │
00066| │                         │                     │                         │
00067| │                         │        DOMAIN       │                         │
00068| │                         │        LAYER        │                         │
00069| │                         │                     │                         │
00070| │                         │  ┌───────────────┐  │                         │
00071| │                         │  │   Entities    │  │                         │
00072| │                         │  │  Value Objects│  │                         │
00073| │                         │  │  Aggregates   │  │                         │
00074| │                         │  │  Domain Events│  │                         │
00075| │                         │  └───────────────┘  │                         │
00076| │                         │                     │                         │
00077| │                         └──────────┬──────────┘                         │
00078| │                                    │                                    │
00079| │                                    ▼                                    │
00080| │                         ┌─────────────────────┐                         │
00081| │                         │     APPLICATION     │                         │
00082| │                         │        LAYER        │                         │
00083| │                         │                     │                         │
00084| │                         │  ┌───────────────┐  │                         │
00085| │                         │  │   Use Cases   │  │                         │
00086| │                         │  │   Services    │  │                         │
00087| │                         │  │   Ports       │  │                         │
00088| │                         │  └───────────────┘  │                         │
00089| │                         │                     │                         │
00090| │                         └──────────┬──────────┘                         │
00091| │                                    │                                    │
00092| │              ┌─────────────────────┼─────────────────────┐              │
00093| │              │                     │                     │              │
00094| │              ▼                     ▼                     ▼              │
00095| │    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │
00096| │    │     PORTS       │  │                 │  │     PORTS       │       │
00097| │    │   (Interfaces)  │  │                 │  │   (Interfaces)  │       │
00098| │    │                 │  │                 │  │                 │       │
00099| │    │  • Input Ports  │  │                 │  │  • Output Ports │       │
00100| │    │    (Use Cases)  │  │                 │  │  • Repositories │       │
00101| │    │                 │  │                 │  │  • Presenters   │       │
00102| │    └────────┬────────┘  │                 │  └────────┬────────┘       │
00103| │             │           │                 │           │                │
00104| │             │           │                 │           │                │
00105| │             ▼           │                 │           ▼                │
00106| │    ┌─────────────────┐  │                 │  ┌─────────────────┐       │
00107| │    │   ADAPTERS      │  │                 │  │   ADAPTERS      │       │
00108| │    │   (Inbound)     │  │                 │  │   (Outbound)    │       │
00109| │    │                 │  │                 │  │                 │       │
00110| │    │  • Web/HTTP     │  │                 │  │  • Database     │       │
00111| │    │  • Console/CLI  │  │                 │  │  • File System  │       │
00112| │    │  • API/Gateway  │  │                 │  │  • External APIs│       │
00113| │    │  • Messaging    │  │                 │  │  • Messaging    │       │
00114| │    │  • Mobile       │  │                 │  │  • Event Bus    │       │
00115| │    └─────────────────┘  │                 │  └─────────────────┘       │
00116| │                         │                 │                            │
00117| └─────────────────────────┼─────────────────┼────────────────────────────┘
00118|                           │                 │
00119|                           ▼                 ▼
00120|               ┌───────────────────┐ ┌───────────────────┐
00121|               │    EXTERNAL       │ │    EXTERNAL       │
00122|               │    SYSTEMS        │ │    SYSTEMS        │
00123|               │                   │ │                   │
00124|               │  • Web Browsers   │ │  • Databases      │
00125|               │  • Mobile Apps    │ │  • File Systems   │
00126|               │  • CLI Users      │ │  • Web Services   │
00127|               │  • API Clients    │ │  • Message Queues │
00128|               └───────────────────┘ └───────────────────┘
00129| ```
00130| 
00131| ---
00132| 
00133| ## The Three Core Layers
00134| 
00135| ### Domain Layer (Core)
00136| 
00137| The domain layer contains the business logic of the application at its purest form. It consists of entities, value objects, aggregates, domain services, and domain events. This layer has **zero dependencies** on any other layer. It defines interfaces (ports) that describe what it needs from the outside world. The domain layer represents the heart of the application and should be completely isolated from infrastructure concerns.
00138| 
00139| The domain layer is where the most stable code resides because it contains the core business rules that rarely change, regardless of how the application is delivered or what technologies are used. This layer can be thoroughly tested without any external dependencies, making it the most reliable and valuable part of the codebase.
00140| 
00141| ### Application Layer (Use Cases)
00142| 
00143| The application layer contains application-specific business logic that orchestrates operations. It coordinates between the domain layer and external systems by using domain objects and calling through ports to interact with external concerns. This layer depends only on domain abstractions and contains use cases, application services, and input/output port definitions.
00144| 
00145| The application layer defines what the system does without caring about how it communicates with the outside world. Each use case represents a single business operation and follows a consistent pattern of validating input, performing business operations, persisting changes, and returning results.
00146| 
00147| ### Adapter Layer (External Integration)
00148| 
00149| The adapter layer contains implementations of ports that integrate with external systems. Adapters are divided into two categories: inbound adapters handle user input and external requests, while outbound adapters handle communication with external systems. Adapters depend on ports (interfaces), never the reverse.
00150| 
00151| This layer translates between external protocols and internal abstractions, allowing the core application to remain unchanged regardless of how it is accessed or what external systems it integrates with.
00152| 
00153| ---
00154| 
00155| ## Ports and Adapters Explained
00156| 
00157| ### Ports (Interfaces)
00158| 
00159| Ports define the boundary between the application core and external systems. They are abstract contracts that specify what the application expects from and provides to the outside world. Ports create a stable interface that shields the core from changes in external systems.
00160| 
00161| **Input Ports (Use Case Interfaces)**
00162| 
00163| Input ports define operations that external systems can perform on the application. They represent the application's API for incoming operations. These ports are implemented by application services and called by inbound adapters.
00164| 
00165| ```typescript
00166| // Input Port - What the outside world can DO
00167| interface CreateOrderUseCase {
00168|   execute(input: CreateOrderInput): Promise<CreateOrderOutput>;
00169| }
00170| 
00171| interface GetOrderUseCase {
00172|   execute(input: GetOrderInput): Promise<GetOrderOutput>;
00173| }
00174| 
00175| interface ProcessPaymentUseCase {
00176|   execute(input: ProcessPaymentInput): Promise<ProcessPaymentOutput>;
00177| }
00178| 
00179| interface ShipOrderUseCase {
00180|   execute(input: ShipOrderInput): Promise<ShipOrderOutput>;
00181| }
00182| ```
00183| 
00184| **Output Ports (Repository and Service Interfaces)**
00185| 
00186| Output ports define operations that the application needs to perform on external systems. These ports are implemented by outbound adapters and called by application services and domain logic. Output ports decouple the core from specific implementations of persistence, external services, and other infrastructure concerns.
00187| 
00188| ```typescript
00189| // Output Port - What the application NEEDS from external systems
00190| interface OrderRepository {
00191|   findById(id: OrderId): Promise<Order | null>;
00192|   findByCustomerId(customerId: string): Promise<Order[]>;
00193|   save(order: Order): Promise<void>;
00194|   delete(id: OrderId): Promise<void>;
00195| }
00196| 
00197| interface InventoryService {
00198|   checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00199|   reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00200|   releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00201| }
00202| 
00203| interface PaymentGateway {
00204|   processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;
00205|   refundPayment(transactionId: string): Promise<RefundResult>;
00206| }
00207| 
00208| interface EventPublisher {
00209|   publish(event: DomainEvent): Promise<void>;
00210|   publishAll(events: DomainEvent[]): Promise<void>;
00211| }
00212| 
00213| interface NotificationService {
00214|   sendNotification(customerId: string, message: string): Promise<void>;
00215| }
00216| ```
00217| 
00218| ### Adapters (Implementations)
00219| 
00220| Adapters implement ports and handle the specifics of interacting with external systems. They translate between external protocols and internal abstractions, keeping the core completely unaware of how external communication happens.
00221| 
00222| **Inbound Adapters (Delivery Mechanisms)**
00223| 
00224| Inbound adapters translate external input into calls to input ports. They handle the specifics of receiving requests from various delivery mechanisms and translating them into a format the application can understand.
00225| 
00226| ```typescript
00227| // HTTP Controller (Inbound Adapter)
00228| class CreateOrderController {
00229|   constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}
00230| 
00231|   async handleRequest(request: HttpRequest): Promise<HttpResponse> {
00232|     try {
00233|       const input: CreateOrderInput = {
00234|         customerId: request.body.customerId,
00235|         items: request.body.items.map((item: any) => ({
00236|           productId: item.productId,
00237|           quantity: item.quantity
00238|         })),
00239|         shippingAddress: request.body.shippingAddress,
00240|         paymentMethod: request.body.paymentMethod
00241|       };
00242| 
00243|       const result = await this.createOrderUseCase.execute(input);
00244| 
00245|       return new HttpResponse(
00246|         result.isSuccess ? 201 : 400,
00247|         {
00248|           success: result.isSuccess,
00249|           data: result.isSuccess ? result.orderId : null,
00250|           error: result.isSuccess ? null : result.error
00251|         }
00252|       );
00253|     } catch (error) {
00254|       return new HttpResponse(
00255|         500,
00256|         {
00257|           success: false,
00258|           error: error instanceof Error ? error.message : 'Unknown error'
00259|         }
00260|       );
00261|     }
00262|   }
00263| }
00264| 
00265| // gRPC Controller (Inbound Adapter)
00266| class OrderGrpcController {
00267|   constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}
00268| 
00269|   async createOrder(call: ServerUnaryCall<CreateOrderRequest>): Promise<CreateOrderResponse> {
00270|     const input: CreateOrderInput = {
00271|       customerId: call.request.customerId,
00272|       items: call.request.items.map(item => ({
00273|         productId: item.productId,
00274|         quantity: item.quantity
00275|       }))
00276|     };
00277| 
00278|     const result = await this.createOrderUseCase.execute(input);
00279| 
00280|     return {
00281|       success: result.isSuccess,
00282|       orderId: result.isSuccess ? result.orderId : '',
00283|       error: result.error || ''
00284|     };
00285|   }
00286| }
00287| 
00288| // Message Queue Consumer (Inbound Adapter)
00289| class OrderMessageHandler {
00290|   constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}
00291| 
00292|   async handleMessage(message: OrderMessage): Promise<void> {
00293|     const input: CreateOrderInput = {
00294|       customerId: message.customerId,
00295|       items: message.items
00296|     };
00297| 
00298|     const result = await this.createOrderUseCase.execute(input);
00299| 
00300|     if (!result.isSuccess) {
00301|       throw new Error(`Failed to process order: ${result.error}`);
00302|     }
00303|   }
00304| }
00305| 
00306| // Console Controller (Inbound Adapter)
00307| class OrderConsoleController {
00308|   constructor(
00309|     private readonly createOrderUseCase: CreateOrderUseCase,
00310|     private readonly getOrderUseCase: GetOrderUseCase
00311|   ) {}
00312| 
00313|   handleCommand(args: string[]): Response {
00314|     const command = args[0];
00315| 
00316|     switch (command) {
00317|       case 'create':
00318|         return this.handleCreateOrder(args);
00319|       case 'get':
00320|         return this.handleGetOrder(args);
00321|       default:
00322|         return Response.error('Unknown command');
00323|     }
00324|   }
00325| 
00326|   private handleCreateOrder(args: string[]): Response {
00327|     const input: CreateOrderInput = {
00328|       customerId: args[1],
00329|       items: JSON.parse(args[2])
00330|     };
00331| 
00332|     const result = this.createOrderUseCase.execute(input);
00333|     return Response.of(result);
00334|   }
00335| 
00336|   private handleGetOrder(args: string[]): Response {
00337|     const input: GetOrderInput = { orderId: args[1] };
00338|     const result = this.getOrderUseCase.execute(input);
00339|     return Response.of(result);
00340|   }
00341| }
00342| ```
00343| 
00344| **Outbound Adapters (External System Integrations)**
00345| 
00346| Outbound adapters implement output ports and handle communication with external systems. They translate between internal abstractions and external system specifics.
00347| 
00348| ```typescript
00349| // TypeOutbound Adapter)
00350| class TypeORMOrderORM Repository Implementation (Repository implements OrderRepository {
00351|   constructor(
00352|     private readonly entityManager: EntityManager,
00353|     private readonly eventPublisher: EventPublisher
00354|   ) {}
00355| 
00356|   async findById(id: OrderId): Promise<Order | null> {
00357|     const entity = await this.entityManager.findOne(OrderEntity, {
00358|       where: { id: id.value }
00359|     });
00360| 
00361|     if (!entity) return null;
00362| 
00363|     return OrderMapper.toDomain(entity);
00364|   }
00365| 
00366|   async findByCustomerId(customerId: string): Promise<Order[]> {
00367|     const entities = await this.entityManager.find(OrderEntity, {
00368|       where: { customerId }
00369|     });
00370| 
00371|     return entities.map(OrderMapper.toDomain);
00372|   }
00373| 
00374|   async save(order: Order): Promise<void> {
00375|     const entity = OrderMapper.toPersistence(order);
00376| 
00377|     if (order.isNew) {
00378|       await this.entityManager.save(entity);
00379|     } else {
00380|       await this.entityManager.update(OrderEntity, entity.id, entity);
00381|     }
00382| 
00383|     for (const event of order.getUncommittedEvents()) {
00384|       await this.eventPublisher.publish(event);
00385|     }
00386|   }
00387| 
00388|   async delete(id: OrderId): Promise<void> {
00389|     await this.entityManager.delete(OrderEntity, { id: id.value });
00390|   }
00391| }
00392| 
00393| // In-Memory Repository (Outbound Adapter for Testing)
00394| class InMemoryOrderRepository implements OrderRepository {
00395|   private orders: Map<string, Order> = new Map();
00396| 
00397|   async findById(id: OrderId): Promise<Order | null> {
00398|     return this.orders.get(id.value) || null;
00399|   }
00400| 
00401|   async findByCustomerId(customerId: string): Promise<Order[]> {
00402|     return Array.from(this.orders.values()).filter(
00403|       order => order.customerId === customerId
00404|     );
00405|   }
00406| 
00407|   async save(order: Order): Promise<void> {
00408|     this.orders.set(order.id.value, order);
00409|   }
00410| 
00411|   async delete(id: OrderId): Promise<void> {
00412|     this.orders.delete(id.value);
00413|   }
00414| }
00415| 
00416| // Stripe Payment Gateway (Outbound Adapter)
00417| class StripePaymentGateway implements PaymentGateway {
00418|   constructor(
00419|     private readonly stripeClient: StripeClient,
00420|     private readonly webhookSecret: string
00421|   ) {}
00422| 
00423|   async processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult> {
00424|     try {
00425|       const paymentIntent = await this.stripeClient.paymentIntents.create({
00426|         amount: amount.inCents,
00427|         currency: amount.currency.toLowerCase(),
00428|         payment_method: paymentMethod.stripeToken,
00429|         confirm: true,
00430|         automatic_payment_methods: {
00431|           enabled: true,
00432|           allow_redirects: 'never'
00433|         }
00434|       });
00435| 
00436|       return {
00437|         success: paymentIntent.status === 'succeeded',
00438|         transactionId: paymentIntent.id,
00439|         error: paymentIntent.last_payment_error?.message
00440|       };
00441|     } catch (error) {
00442|       return {
00443|         success: false,
00444|         transactionId: '',
00445|         error: error instanceof Error ? error.message : 'Payment failed'
00446|       };
00447|     }
00448|   }
00449| 
00450|   async refundPayment(transactionId: string): Promise<RefundResult> {
00451|     try {
00452|       const refund = await this.stripeClient.refunds.create({
00453|         payment_intent: transactionId
00454|       });
00455| 
00456|       return {
00457|         success: refund.status === 'succeeded',
00458|         refundId: refund.id
00459|       };
00460|     } catch (error) {
00461|       return {
00462|         success: false,
00463|         refundId: '',
00464|         error: error instanceof Error ? error.message : 'Refund failed'
00465|       };
00466|     }
00467|   }
00468| }
00469| 
00470| // Kafka Event Publisher (Outbound Adapter)
00471| class KafkaEventPublisher implements EventPublisher {
00472|   constructor(
00473|     private readonly kafkaProducer: KafkaProducer,
00474|     private readonly topicPrefix: string
00475|   ) {}
00476| 
00477|   async publish(event: DomainEvent): Promise<void> {
00478|     const topic = `${this.topicPrefix}.${event.constructor.name}`;
00479|     const message = {
00480|       key: event.aggregateId,
00481|       value: JSON.stringify(event),
00482|       timestamp: Date.now()
00483|     };
00484| 
00485|     await this.kafkaProducer.send({ topic, messages: [message] });
00486|   }
00487| 
00488|   async publishAll(events: DomainEvent[]): Promise<void> {
00489|     const messages = events.map(event => ({
00490|       topic: `${this.topicPrefix}.${event.constructor.name}`,
00491|       key: event.aggregateId,
00492|       value: JSON.stringify(event),
00493|       timestamp: Date.now()
00494|     }));
00495| 
00496|     await this.kafkaProducer.send({ messages });
00497|   }
00498| }
00499| ```
00500| 
00501| ---
00502| 
00503| ## Layer Structure
00504| 
00505| ```
00506| src/
00507| ├── domain/                           # Domain Layer (innermost)
00508| │   ├── entities/                     # Domain entities with behavior
00509| │   │   ├── order.ts
00510| │   │   ├── customer.ts
00511| │   │   └── product.ts
00512| │   ├── value-objects/                # Value objects with validation
00513| │   │   ├── money.ts
00514| │   │   ├── address.ts
00515| │   │   └── email.ts
00516| │   ├── aggregates/                   # Aggregate roots
00517| │   │   ├── order-aggregate.ts
00518| │   │   └── customer-aggregate.ts
00519| │   ├── domain-services/              # Domain services for complex logic
00520| │   │   └── pricing-service.ts
00521| │   ├── events/                       # Domain events
00522| │   │   ├── order-created.event.ts
00523| │   │   └── payment-completed.event.ts
00524| │   ├── repositories/                 # Repository interfaces (ports)
00525| │   │   ├── order.repository.interface.ts
00526| │   │   └── customer.repository.interface.ts
00527| │   ├── services/                     # Service interfaces (ports)
00528| │   │   ├── inventory.service.interface.ts
00529| │   │   └── payment.service.interface.ts
00530| │   └── factories/                    # Domain factories
00531| │       └── order.factory.ts
00532| │
00533| ├── application/                      # Application Layer
00534| │   ├── use-cases/                    # Use case implementations
00535| │   │   ├── create-order/
00536| │   │   │   ├── create-order.handler.ts
00537| │   │   │   ├── create-order.input.ts
00538| │   │   │   └── create-order.output.ts
00539| │   │   ├── process-payment/
00540| │   │   │   ├── process-payment.handler.ts
00541| │   │   │   ├── process-payment.input.ts
00542| │   │   │   └── process-payment.output.ts
00543| │   │   └── ship-order/
00544| │   │       ├── ship-order.handler.ts
00545| │   │       ├── ship-order.input.ts
00546| │   │       └── ship-order.output.ts
00547| │   ├── ports/
00548| │   │   ├── inbound/                  # Input port interfaces
00549| │   │   │   ├── create-order.port.ts
00550| │   │   │   └── process-payment.port.ts
00551| │   │   └── outbound/                 # Output port interfaces
00552| │   │       ├── order.repository.port.ts
00553| │   │       └── payment-gateway.port.ts
00554| │   └── coordinators/                 # Application coordinators
00555| │       └── order-coordinator.ts
00556| │
00557| ├── adapters/                         # Adapter Layer
00558| │   ├── inbound/                      # Inbound adapters (controllers)
00559| │   │   ├── http/
00560| │   │   │   ├── controllers/
00561| │   │   │   │   ├── order.controller.ts
00562| │   │   │   │   └── payment.controller.ts
00563| │   │   │   ├── middlewares/
00564| │   │   │   │   ├── auth.middleware.ts
00565| │   │   │   │   └── validation.middleware.ts
00566| │   │   │   └── routes/
00567| │   │   │       ├── order.routes.ts
00568| │   │   │       └── payment.routes.ts
00569| │   │   ├── grpc/
00570| │   │   │   └── order.grpc.controller.ts
00571| │   │   ├── messaging/
00572| │   │   │   ├── order.message-handler.ts
00573| │   │   │   └── payment.message-handler.ts
00574| │   │   └── cli/
00575| │   │       └── order.cli.controller.ts
00576| │   ├── outbound/                     # Outbound adapters (implementations)
00577| │   │   ├── persistence/
00578| │   │   │   ├── typeorm/
00579| │   │   │   │   ├── order.repository.ts
00580| │   │   │   │   ├── customer.repository.ts
00581| │   │   │   │   └── entities/
00582| │   │   │   │       ├── order.entity.ts
00583| │   │   │   │       └── customer.entity.ts
00584| │   │   │   ├── prisma/
00585| │   │   │   │   ├── order.repository.ts
00586| │   │   │   │   └── customer.repository.ts
00587| │   │   │   └── mongodb/
00588| │   │   │       ├── order.repository.ts
00589| │   │   │       └── customer.repository.ts
00590| │   │   ├── external-api/
00591| │   │   │   ├── stripe/
00592| │   │   │   │   ├── payment-gateway.adapter.ts
00593| │   │   │   │   └── stripe-client.ts
00594| │   │   │   ├── sendgrid/
00595| │   │   │   │   └── notification.adapter.ts
00596| │   │   │   └── aws/
00597| │   │   │       └── email.service.ts
00598| │   │   ├── events/
00599| │   │   │   ├── kafka/
00600| │   │   │   │   └── event-publisher.ts
00601| │   │   │   └── rabbitmq/
00602| │   │   │       └── event-publisher.ts
00603| │   │   └── cache/
00604| │   │       └── redis/
00605| │   │           └── cache.adapter.ts
00606| │   └── presenters/                   # Response formatters
00607| │       ├── json/
00608| │       │   └── order.presenter.ts
00609| │       └── grpc/
00610| │           └── order.presenter.ts
00611| │
00612| ├── frameworks/                       # Frameworks and Drivers
00613| │   ├── web/
00614| │   │   ├── express/
00615| │   │   │   ├── app.ts
00616| │   │   │   └── server.ts
00617| │   │   ├── fastify/
00618| │   │   │   ├── app.ts
00619| │   │   │   └── server.ts
00620| │   │   └── nestjs/                   # NestJS modules
00621| │   │       ├── order.module.ts
00622| │   │       └── payment.module.ts
00623| │   ├── database/
00624| │   │   ├── typeorm/
00625| │   │   │   ├── data-source.ts
00626| │   │   │   └── migrations/
00627| │   │   ├── prisma/
00628| │   │   │   ├── schema.prisma
00629| │   │   │   └── client.ts
00630| │   │   └── mongoose/
00631| │   │       ├── connection.ts
00632| │   │       └── schemas/
00633| │   ├── messaging/
00634| │   │   ├── kafka/
00635| │   │   │   ├── producer.ts
00636| │   │   │   └── consumer.ts
00637| │   │   └── rabbitmq/
00638| │   │       ├── connection.ts
00639| │   │       └── publisher.ts
00640| │   └── external/
00641| │       ├── stripe/
00642| │       │   └── client.ts
00643| │       ├── sendgrid/
00644| │       │   └── client.ts
00645| │       └── aws/
00646| │           ├── config.ts
00647| │           └── client.ts
00648| │
00649| └── shared/
00650|     ├── utils/                        # Shared utilities
00651|     ├── types/                        # Shared TypeScript types
00652|     ├── errors/                       # Custom error classes
00653|     └── mappers/                      # Cross-layer mappers
00654| ```
00655| 
00656| ---
00657| 
00658| ## Implementation Examples
00659| 
00660| ### Domain Layer (Enterprise Business Rules)
00661| 
00662| The domain layer contains pure business logic with no dependencies on external frameworks or libraries. This makes it completely testable and portable.
00663| 
00664| ```typescript
00665| // domain/entities/order.ts
00666| import { Entity } from '../../shared/entity';
00667| import { Money } from '../../value-objects/money';
00668| import { OrderId } from '../../value-objects/order-id';
00669| import { OrderStatus } from './order-status';
00670| import { OrderLineItem } from './order-line-item';
00671| 
00672| export interface OrderProps {
00673|   id: OrderId;
00674|   customerId: string;
00675|   lineItems: OrderLineItem[];
00676|   status: OrderStatus;
00677|   shippingAddress: Address;
00678|   paymentMethod: PaymentMethod;
00679|   createdAt: Date;
00680| }
00681| 
00682| export class Order extends Entity<OrderId, OrderProps> {
00683|   private constructor(props: OrderProps) {
00684|     super(props);
00685|   }
00686| 
00687|   static create(props: OrderProps): Order {
00688|     const order = new Order(props);
00689|     order.validate();
00690|     return order;
00691|   }
00692| 
00693|   private validate(): void {
00694|     if (this._props.lineItems.length === 0) {
00695|       throw new DomainError('Order must have at least one line item');
00696|     }
00697|     if (!this._props.status) {
00698|       throw new DomainError('Order must have a status');
00699|     }
00700|     if (!this._props.shippingAddress) {
00701|       throw new DomainError('Order must have a shipping address');
00702|     }
00703|   }
00704| 
00705|   get total(): Money {
00706|     return this._props.lineItems.reduce(
00707|       (sum, item) => sum.add(item.subtotal),
00708|       Money.zero(this._props.lineItems[0]?.productCurrency ?? 'USD')
00709|     );
00710|   }
00711| 
00712|   get canBeSubmitted(): boolean {
00713|     return this._props.status === OrderStatus.DRAFT;
00714|   }
00715| 
00716|   get canBeCancelled(): boolean {
00717|     return this._props.status !== OrderStatus.SHIPPED &&
00718|            this._props.status !== OrderStatus.DELIVERED;
00719|   }
00720| 
00721|   submit(): void {
00722|     if (!this.canBeSubmitted) {
00723|       throw new DomainError('Order cannot be submitted');
00724|     }
00725|     this._props.status = OrderStatus.SUBMITTED;
00726|     this.recordDomainEvent(new OrderSubmitted(this._props.id));
00727|   }
00728| 
00729|   cancel(): void {
00730|     if (!this.canBeCancelled) {
00731|       throw new DomainError('Order cannot be cancelled');
00732|     }
00733|     this._props.status = OrderStatus.CANCELLED;
00734|     this.recordDomainEvent(new OrderCancelled(this._props.id));
00735|   }
00736| 
00737|   ship(): void {
00738|     if (this._props.status !== OrderStatus.PAID) {
00739|       throw new DomainError('Order must be paid before shipping');
00740|     }
00741|     this._props.status = OrderStatus.SHIPPED;
00742|     this.recordDomainEvent(new OrderShipped(this._props.id));
00743|   }
00744| }
00745| 
00746| // domain/value-objects/money.ts
00747| export class Money {
00748|   private constructor(
00749|     private readonly amount: number,
00750|     private readonly currency: string
00751|   ) {}
00752| 
00753|   static zero(currency: string = 'USD'): Money {
00754|     return new Money(0, currency);
00755|   }
00756| 
00757|   static fromAmount(amount: number, currency: string = 'USD'): Money {
00758|     if (amount < 0) {
00759|       throw new DomainError('Amount cannot be negative');
00760|     }
00761|     return new Money(Math.round(amount * 100) / 100, currency);
00762|   }
00763| 
00764|   add(other: Money): Money {
00765|     this.ensureSameCurrency(other);
00766|     return new Money(this.amount + other.amount, this.currency);
00767|   }
00768| 
00769|   subtract(other: Money): Money {
00770|     this.ensureSameCurrency(other);
00771|     const result = this.amount - other.amount;
00772|     if (result < 0) {
00773|       throw new DomainError('Result cannot be negative');
00774|     }
00775|     return new Money(result, this.currency);
00776|   }
00777| 
00778|   multiply(factor: number): Money {
00779|     if (factor < 0) {
00780|       throw new DomainError('Factor cannot be negative');
00781|     }
00782|     return new Money(this.amount * factor, this.currency);
00783|   }
00784| 
00785|   isGreaterThan(other: Money): boolean {
00786|     this.ensureSameCurrency(other);
00787|     return this.amount > other.amount;
00788|   }
00789| 
00790|   isLessThan(other: Money): boolean {
00791|     this.ensureSameCurrency(other);
00792|     return this.amount < other.amount;
00793|   }
00794| 
00795|   get amountValue(): number {
00796|     return this.amount;
00797|   }
00798| 
00799|   get currencyCode(): string {
00800|     return this.currency;
00801|   }
00802| 
00803|   private ensureSameCurrency(other: Money): void {
00804|     if (this.currency !== other.currency) {
00805|       throw new DomainError('Cannot operate on different currencies');
00806|     }
00807|   }
00808| }
00809| 
00810| // domain/aggregates/order-aggregate.ts
00811| export class OrderAggregate {
00812|   private constructor(
00813|     public readonly order: Order,
00814|     private readonly lineItems: OrderLineItem[],
00815|     private readonly payments: Payment[]
00816|   ) {}
00817| 
00818|   static createFromOrder(order: Order): OrderAggregate {
00819|     return new OrderAggregate(order, [], []);
00820|   }
00821| 
00822|   get totalPaid(): Money {
00823|     return this.payments
00824|       .filter(p => p.status === PaymentStatus.COMPLETED)
00825|       .reduce((sum, payment) => sum.add(payment.amount), Money.zero());
00826|   }
00827| 
00828|   isFullyPaid(): boolean {
00829|     return this.totalPaid.isGreaterThanOrEqual(this.order.total);
00830|   }
00831| 
00832|   addPayment(payment: Payment): void {
00833|     if (this.isFullyPaid()) {
00834|       throw new DomainError('Order is already fully paid');
00835|     }
00836|     this.payments.push(payment);
00837|   }
00838| }
00839| 
00840| // domain/events/order-submitted.event.ts
00841| export class OrderSubmitted extends DomainEvent {
00842|   constructor(
00843|     public readonly orderId: OrderId,
00844|     public readonly customerId: string,
00845|     public readonly total: Money
00846|   ) {
00847|     super('OrderSubmitted', orderId.value);
00848|   }
00849| }
00850| ```
00851| 
00852| ### Application Layer (Use Cases)
00853| 
00854| Application layer contains use case implementations that orchestrate domain objects and external adapters through ports.
00855| 
00856| ```typescript
00857| // application/use-cases/create-order/create-order.handler.ts
00858| import { OrderRepository } from '../../ports/outbound/order.repository';
00859| import { InventoryService } from '../../ports/outbound/inventory.service';
00860| import { PaymentGateway } from '../../ports/outbound/payment-gateway';
00861| import { EventPublisher } from '../../ports/outbound/event-publisher';
00862| import { OrderFactory } from '../../domain/factories/order.factory';
00863| 
00864| export interface CreateOrderInput {
00865|   customerId: string;
00866|   items: Array<{
00867|     productId: string;
00868|     quantity: number;
00869|   }>;
00870|   shippingAddress: Address;
00871|   paymentMethod: PaymentMethod;
00872| }
00873| 
00874| export interface CreateOrderOutput {
00875|   isSuccess: boolean;
00876|   orderId?: string;
00877|   total?: number;
00878|   error?: string;
00879| }
00880| 
00881| export class CreateOrderHandler {
00882|   constructor(
00883|     private readonly orderRepository: OrderRepository,
00884|     private readonly inventoryService: InventoryService,
00885|     private readonly paymentGateway: PaymentGateway,
00886|     private readonly eventPublisher: EventPublisher,
00887|     private readonly orderFactory: OrderFactory
00888|   ) {}
00889| 
00890|   async execute(input: CreateOrderInput): Promise<CreateOrderOutput> {
00891|     try {
00892|       // Step 1: Validate input
00893|       this.validateInput(input);
00894| 
00895|       // Step 2: Check inventory availability
00896|       await this.inventoryService.checkAvailability(input.items);
00897| 
00898|       // Step 3: Create order using factory
00899|       const order = this.orderFactory.createFromItems(
00900|         input.customerId,
00901|         input.items,
00902|         input.shippingAddress,
00903|         input.paymentMethod
00904|       );
00905| 
00906|       // Step 4: Persist order
00907|       await this.orderRepository.save(order);
00908| 
00909|       // Step 5: Publish domain events
00910|       await this.eventPublisher.publishAll(order.getUncommittedEvents());
00911| 
00912|       // Step 6: Return success result
00913|       return {
00914|         isSuccess: true,
00915|         orderId: order.id.value,
00916|         total: order.total.amountValue
00917|       };
00918|     } catch (error) {
00919|       return {
00920|         isSuccess: false,
00921|         error: error instanceof Error ? error.message : 'Unknown error'
00922|       };
00923|     }
00924|   }
00925| 
00926|   private validateInput(input: CreateOrderInput): void {
00927|     if (!input.customerId) {
00928|       throw new DomainError('Customer ID is required');
00929|     }
00930|     if (!input.items || input.items.length === 0) {
00931|       throw new DomainError('Order must have at least one item');
00932|     }
00933|     if (!input.shippingAddress) {
00934|       throw new DomainError('Shipping address is required');
00935|     }
00936|     if (!input.paymentMethod) {
00937|       throw new DomainError('Payment method is required');
00938|     }
00939|   }
00940| }
00941| 
00942| // application/use-cases/process-payment/process-payment.handler.ts
00943| export interface ProcessPaymentInput {
00944|   orderId: string;
00945|   paymentToken: string;
00946|   amount: number;
00947|   currency: string;
00948| }
00949| 
00950| export interface ProcessPaymentOutput {
00951|   isSuccess: boolean;
00952|   transactionId?: string;
00953|   error?: string;
00954| }
00955| 
00956| export class ProcessPaymentHandler {
00957|   constructor(
00958|     private readonly orderRepository: OrderRepository,
00959|     private readonly paymentGateway: PaymentGateway
00960|   ) {}
00961| 
00962|   async execute(input: ProcessPaymentInput): Promise<ProcessPaymentOutput> {
00963|     try {
00964|       // Step 1: Load order
00965|       const order = await this.orderRepository.findById(
00966|         OrderId.of(input.orderId)
00967|       );
00968| 
00969|       if (!order) {
00970|         return {
00971|           isSuccess: false,
00972|           error: 'Order not found'
00973|         };
00974|       }
00975| 
00976|       // Step 2: Create payment
00977|       const amount = Money.fromAmount(input.amount, input.currency);
00978|       const paymentMethod = PaymentMethod.fromToken(input.paymentToken);
00979| 
00980|       // Step 3: Process payment through gateway
00981|       const result = await this.paymentGateway.processPayment(
00982|         amount,
00983|         paymentMethod
00984|       );
00985| 
00986|       if (!result.success) {
00987|         order.recordDomainEvent(new PaymentFailed(order.id, result.error!));
00988|         await this.orderRepository.save(order);
00989| 
00990|         return {
00991|           isSuccess: false,
00992|           error: result.error
00993|         };
00994|       }
00995| 
00996|       // Step 4: Update order status
00997|       order.markAsPaid(result.transactionId!);
00998|       await this.orderRepository.save(order);
00999| 
01000|       // Step 5: Return success
01001|       return {
01002|         isSuccess: true,
01003|         transactionId: result.transactionId
01004|       };
01005|     } catch (error) {
01006|       return {
01007|         isSuccess: false,
01008|         error: error instanceof Error ? error.message : 'Payment processing failed'
01009|       };
01010|     }
01011|   }
01012| }
01013| ```
01014| 
01015| ### Ports (Interfaces)
01016| 
01017| Ports define the contracts between layers, enabling dependency inversion.
01018| 
01019| ```typescript
01020| // application/ports/inbound/create-order.port.ts
01021| export interface CreateOrderInputPort {
01022|   createOrder(input: CreateOrderInput): Promise<CreateOrderOutput>;
01023| }
01024| 
01025| // application/ports/outbound/order.repository.ts
01026| export interface OrderRepository {
01027|   findById(id: OrderId): Promise<Order | null>;
01028|   findByCustomerId(customerId: string): Promise<Order[]>;
01029|   findByStatus(status: OrderStatus): Promise<Order[]>;
01030|   save(order: Order): Promise<void>;
01031|   delete(id: OrderId): Promise<void>;
01032| }
01033| 
01034| // application/ports/outbound/inventory.service.ts
01035| export interface InventoryService {
01036|   checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;
01037|   reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
01038|   releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
01039|   getProduct(productId: string): Promise<Product | null>;
01040| }
01041| 
01042| // application/ports/outbound/payment-gateway.ts
01043| export interface PaymentGateway {
01044|   processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;
01045|   refundPayment(transactionId: string, amount?: Money): Promise<RefundResult>;
01046|   getPaymentStatus(transactionId: string): Promise<PaymentStatus>;
01047| }
01048| 
01049| // application/ports/outbound/event-publisher.ts
01050| export interface EventPublisher {
01051|   publish(event: DomainEvent): Promise<void>;
01052|   publishAll(events: DomainEvent[]): Promise<void>;
01053| }
01054| 
01055| // application/ports/outbound/notification.service.ts
01056| export interface NotificationService {
01057|   sendEmail(to: string, subject: string, body: string): Promise<void>;
01058|   sendSms(to: string, message: string): Promise<void>;
01059|   sendPushNotification(userId: string, notification: PushNotification): Promise<void>;
01060| }
01061| ```
01062| 
01063| ---
01064| 
01065| ## Dependency Injection Structure
01066| 
01067| The composition root wires all dependencies together, following the dependency inversion principle.
01068| 
01069| ```typescript
01070| // infrastructure/container.ts - Composition Root
01071| class Container {
01072|   private readonly services = new Map<string, any>();
01073|   private readonly factories = new Map<string, (container: Container) => any>();
01074| 
01075|   register<T>(token: string, factory: (container: Container) => T): void {
01076|     this.factories.set(token, factory);
01077|   }
01078| 
01079|   resolve<T>(token: string): T {
01080|     if (this.services.has(token)) {
01081|       return this.services.get(token);
01082|     }
01083| 
01084|     const factory = this.factories.get(token);
01085|     if (!factory) {
01086|       throw new Error(`Service ${token} not registered`);
01087|     }
01088| 
01089|     const instance = factory(this);
01090|     this.services.set(token, instance);
01091|     return instance;
01092|   }
01093| 
01094|   registerSingleton<T>(token: string, instance: T): void {
01095|     this.services.set(token, instance);
01096|   }
01097| }
01098| 
01099| // Setup dependency container
01100| const container = new Container();
01101| 
01102| // Register ports (abstractions)
01103| container.register('OrderRepository', () => new TypeORMOrderRepository(
01104|   container.resolve('EntityManager'),
01105|   container.resolve('EventPublisher')
01106| ));
01107| 
01108| container.register('InventoryService', () => new InventoryServiceImpl(
01109|   container.resolve('InventoryRepository')
01110| ));
01111| 
01112| container.register('PaymentGateway', () => new StripePaymentGateway(
01113|   container.resolve('StripeClient')
01114| ));
01115| 
01116| container.register('EventPublisher', () => new KafkaEventPublisher(
01117|   container.resolve('KafkaProducer'),
01118|   container.resolve('TopicPrefix')
01119| ));
01120| 
01121| container.register('NotificationService', () => new SendgridNotificationService(
01122|   container.resolve('SendgridClient')
01123| ));
01124| 
01125| // Register use case handlers
01126| container.register('CreateOrderHandler', (c) => new CreateOrderHandler(
01127|   c.resolve('OrderRepository'),
01128|   c.resolve('InventoryService'),
01129|   c.resolve('PaymentGateway'),
01130|   c.resolve('EventPublisher'),
01131|   c.resolve('OrderFactory')
01132| ));
01133| 
01134| container.register('ProcessPaymentHandler', (c) => new ProcessPaymentHandler(
01135|   c.resolve('OrderRepository'),
01136|   c.resolve('PaymentGateway')
01137| ));
01138| 
01139| container.register('ShipOrderHandler', (c) => new ShipOrderHandler(
01140|   c.resolve('OrderRepository'),
01141|   c.resolve('InventoryService'),
01142|   c.resolve('NotificationService')
01143| ));
01144| 
01145| // Register inbound adapters (controllers)
01146| container.register('CreateOrderController', (c) =>
01147|   new HTTPOrderController(c.resolve('CreateOrderHandler'))
01148| );
01149| 
01150| container.register('ProcessPaymentController', (c) =>
01151|   new HTTPPaymentController(c.resolve('ProcessPaymentHandler'))
01152| );
01153| 
01154| // Register domain factories
01155| container.register('OrderFactory', () => new OrderFactory(
01156|   container.resolve('ProductRepository')
01157| ));
01158| 
01159| export { container };
01160| ```
01161| 
01162| ---
01163| 
01164| ## CQRS Pattern in Layered Architecture
01165| 
01166| Command Query Responsibility Segregation (CQRS) separates operations that change state (Commands) from operations that read data (Queries). This pattern fits naturally with layered architecture.
01167| 
01168| ```typescript
01169| // application/ports/inbound/command.port.ts
01170| export interface Command<Input, Output> {
01171|   execute(input: Input): Promise<Output>;
01172| }
01173| 
01174| // application/ports/inbound/query.port.ts
01175| export interface Query<Input, Output> {
01176|   execute(input: Input): Promise<Output>;
01177| }
01178| 
01179| // application/use-cases/commands/create-order.command.ts
01180| export interface CreateOrderCommandInput {
01181|   customerId: string;
01182|   items: Array<{ productId: string; quantity: number }>;
01183|   shippingAddress: Address;
01184| }
01185| 
01186| export interface CreateOrderCommandOutput {
01187|   orderId: string;
01188|   total: number;
01189| }
01190| 
01191| export interface CreateOrderCommand extends Command<CreateOrderCommandInput, CreateOrderCommandOutput> {
01192| }
01193| 
01194| // application/use-cases/queries/get-order.query.ts
01195| export interface GetOrderQueryInput {
01196|   orderId: string;
01197|   includeItems?: boolean;
01198| }
01199| 
01200| export interface GetOrderQueryOutput {
01201|   order: OrderDto | null;
01202|   error?: string;
01203| }
01204| 
01205| export interface GetOrderQuery extends Query<GetOrderQueryInput, GetOrderQueryOutput> {
01206| }
01207| 
01208| // Different handlers for different operations
01209| export class CreateOrderCommandHandler implements CreateOrderCommand {
01210|   // Modifies state - returns new order ID and total
01211| }
01212| 
01213| export class GetOrderQueryHandler implements GetOrderQuery {
01214|   // Only reads - returns order data, never modifies state
01215| }
01216| ```
01217| 
01218| ---
01219| 
01220| ## Data Transfer Objects (DTOs) and Mappers
01221| 
01222| DTOs transfer data between layers without behavior. Mappers translate between different representations in different layers.
01223| 
01224| ```typescript
01225| // application/dtos/order.dto.ts
01226| export class OrderDto {
01227|   id: string;
01228|   customerId: string;
01229|   status: string;
01230|   total: number;
01231|   currency: string;
01232|   items: OrderLineItemDto[];
01233|   shippingAddress: AddressDto;
01234|   createdAt: Date;
01235|   updatedAt: Date;
01236| }
01237| 
01238| export class OrderLineItemDto {
01239|   productId: string;
01240|   productName: string;
01241|   quantity: number;
01242|   unitPrice: number;
01243|   subtotal: number;
01244| }
01245| 
01246| export class AddressDto {
01247|   street: string;
01248|   city: string;
01249|   state: string;
01250|   postalCode: string;
01251|   country: string;
01252| }
01253| 
01254| // infrastructure/mappers/order.mapper.ts
01255| export class OrderMapper {
01256|   // Domain → DTO
01257|   static toDto(order: Order): OrderDto {
01258|     return {
01259|       id: order.id.value,
01260|       customerId: order.customerId,
01261|       status: order.status.value,
01262|       total: order.total.amountValue,
01263|       currency: order.total.currencyCode,
01264|       items: order.lineItems.map(item => ({
01265|         productId: item.productId,
01266|         productName: item.productName,
01267|         quantity: item.quantity,
01268|         unitPrice: item.unitPrice.amountValue,
01269|         subtotal: item.subtotal.amountValue
01270|       })),
01271|       shippingAddress: {
01272|         street: order.shippingAddress.street,
01273|         city: order.shippingAddress.city,
01274|         state: order.shippingAddress.state,
01275|         postalCode: order.shippingAddress.postalCode,
01276|         country: order.shippingAddress.country
01277|       },
01278|       createdAt: order.createdAt,
01279|       updatedAt: order.updatedAt
01280|     };
01281|   }
01282| 
01283|   // Domain → Persistence Object
01284|   static toPersistence(order: Order): OrderPo {
01285|     return {
01286|       id: order.id.value,
01287|       customer_id: order.customerId,
01288|       status: order.status.value,
01289|       total_amount: order.total.amountValue,
01290|       currency: order.total.currencyCode,
01291|       shipping_address: JSON.stringify(order.shippingAddress),
01292|       created_at: order.createdAt,
01293|       updated_at: new Date()
01294|     };
01295|   }
01296| 
01297|   // Persistence Object → Domain
01298|   static toDomain(po: OrderPo): Order {
01299|     return Order.reconstitute({
01300|       id: OrderId.of(po.id),
01301|       customerId: po.customer_id,
01302|       lineItems: [], // Would need to load from separate table
01303|       status: OrderStatus.of(po.status),
01304|       shippingAddress: JSON.parse(po.shipping_address),
01305|       createdAt: po.created_at
01306|     });
01307|   }
01308| 
01309|   // DTO → Input (for use cases)
01310|   static toCreateOrderInput(dto: OrderDto): CreateOrderInput {
01311|     return {
01312|       customerId: dto.customerId,
01313|       items: dto.items.map(item => ({
01314|         productId: item.productId,
01315|         quantity: item.quantity
01316|       })),
01317|       shippingAddress: Address.of(dto.shippingAddress)
01318|     };
01319|   }
01320| }
01321| ```
01322| 
01323| ---
01324| 
01325| ## Read-Only Interface Pattern
01326| 
01327| Prevent unintended modifications by exposing read-only views of domain objects.
01328| 
01329| ```typescript
01330| // domain/interfaces/read-only-order.interface.ts
01331| export interface ReadOnlyOrder {
01332|   getId(): OrderId;
01333|   getCustomerId(): string;
01334|   getStatus(): OrderStatus;
01335|   getTotal(): Money;
01336|   getLineItems(): ReadonlyArray<ReadOnlyOrderLineItem>;
01337|   getCreatedAt(): Date;
01338| }
01339| 
01340| // infrastructure/adapters/presenters/read-only-order.adapter.ts
01341| export class ReadOnlyOrderAdapter implements ReadOnlyOrder {
01342|   constructor(private order: Order) {}
01343| 
01344|   getId(): OrderId {
01345|     return this.order.id;
01346|   }
01347| 
01348|   getCustomerId(): string {
01349|     return this.order.customerId;
01350|   }
01351| 
01352|   getStatus(): OrderStatus {
01353|     return this.order.status;
01354|   }
01355| 
01356|   getTotal(): Money {
01357|     return this.order.total;
01358|   }
01359| 
01360|   getLineItems(): ReadonlyArray<ReadOnlyOrderLineItem> {
01361|     return this.order.lineItems.map(item => new ReadOnlyOrderLineItemAdapter(item));
01362|   }
01363| 
01364|   getCreatedAt(): Date {
01365|     return this.order.createdAt;
01366|   }
01367| }
01368| 
01369| export class ReadOnlyOrderLineItemAdapter implements ReadOnlyOrderLineItem {
01370|   constructor(private item: OrderLineItem) {}
01371| 
01372|   getProductId(): string {
01373|     return this.item.productId;
01374|   }
01375| 
01376|   getQuantity(): number {
01377|     return this.item.quantity;
01378|   }
01379| 
01380|   getUnitPrice(): Money {
01381|     return this.item.unitPrice;
01382|   }
01383| 
01384|   getSubtotal(): Money {
01385|     return this.item.subtotal;
01386|   }
01387| }
01388| ```
01389| 
01390| ---
01391| 
01392| ## Testing Strategy
01393| 
01394| Because of the clear separation, testing can be done at multiple levels, each with different characteristics and purposes.
01395| 
01396| ```typescript
01397| // 1. Domain Layer - Pure unit tests, no dependencies
01398| describe('Order Entity', () => {
01399|   it('should create order with valid data', () => {
01400|     const orderId = OrderId.of('order-123');
01401|     const customerId = 'customer-456';
01402|     const lineItems = [
01403|       OrderLineItem.create({
01404|         productId: 'product-1',
01405|         quantity: 2,
01406|         unitPrice: Money.fromAmount(25.00)
01407|       })
01408|     ];
01409|     const status = OrderStatus.DRAFT;
01410| 
01411|     const order = Order.create({
01412|       id: orderId,
01413|       customerId,
01414|       lineItems,
01415|       status,
01416|       shippingAddress: TestAddresses.VALID_ADDRESS,
01417|       paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
01418|     });
01419| 
01420|     expect(order.id).toEqual(orderId);
01421|     expect(order.customerId).toBe(customerId);
01422|     expect(order.total.amountValue).toBe(50.00);
01423|   });
01424| 
01425|   it('should reject empty line items', () => {
01426|     expect(() => {
01427|       Order.create({
01428|         id: OrderId.of('order-123'),
01429|         customerId: 'customer-456',
01430|         lineItems: [],
01431|         status: OrderStatus.DRAFT,
01432|         shippingAddress: TestAddresses.VALID_ADDRESS,
01433|         paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
01434|       });
01435|     }).toThrow(DomainError);
01436|   });
01437| 
01438|   it('should calculate total correctly', () => {
01439|     const order = Order.create({
01440|       id: OrderId.of('order-123'),
01441|       customerId: 'customer-456',
01442|       lineItems: [
01443|         OrderLineItem.create({
01444|           productId: 'product-1',
01445|           quantity: 2,
01446|           unitPrice: Money.fromAmount(25.00)
01447|         }),
01448|         OrderLineItem.create({
01449|           productId: 'product-2',
01450|           quantity: 1,
01451|           unitPrice: Money.fromAmount(50.00)
01452|         })
01453|       ],
01454|       status: OrderStatus.DRAFT,
01455|       shippingAddress: TestAddresses.VALID_ADDRESS,
01456|       paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
01457|     });
01458| 
01459|     expect(order.total.amountValue).toBe(100.00);
01460|   });
01461| 
01462|   it('should transition to SUBMITTED when submitted', () => {
01463|     const order = Order.create(TestOrderData.VALID_DRAFT_ORDER);
01464| 
01465|     order.submit();
01466| 
01467|     expect(order.status).toBe(OrderStatus.SUBMITTED);
01468|   });
01469| });
01470| 
01471| // 2. Use Case Layer - Tests with in-memory repositories
01472| describe('CreateOrderHandler', () => {
01473|   let handler: CreateOrderHandler;
01474|   let orderRepository: InMemoryOrderRepository;
01475|   let inventoryService: MockInventoryService;
01476|   let paymentGateway: MockPaymentGateway;
01477|   let eventPublisher: MockEventPublisher;
01478|   let orderFactory: OrderFactory;
01479| 
01480|   beforeEach(() => {
01481|     orderRepository = new InMemoryOrderRepository();
01482|     inventoryService = new MockInventoryService();
01483|     paymentGateway = new MockPaymentGateway();
01484|     eventPublisher = new MockEventPublisher();
01485|     orderFactory = new OrderFactory();
01486| 
01487|     handler = new CreateOrderHandler(
01488|       orderRepository,
01489|       inventoryService,
01490|       paymentGateway,
01491|       eventPublisher,
01492|       orderFactory
01493|     );
01494|   });
01495| 
01496|   it('should create order successfully', async () => {
01497|     const input: CreateOrderInput = {
01498|       customerId: 'customer-123',
01499|       items: [
01500|         { productId: 'product-1', quantity: 2 }
01501|       ],
01502|       shippingAddress: TestAddresses.VALID_ADDRESS,
01503|       paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
01504|     };
01505| 
01506|     const result = await handler.execute(input);
01507| 
01508|     expect(result.isSuccess).toBe(true);
01509|     expect(result.orderId).toBeDefined();
01510|     expect(result.total).toBe(50.00);
01511|     expect(orderRepository.orders.size).toBe(1);
01512|   });
01513| 
01514|   it('should fail when inventory is unavailable', async () => {
01515|     inventoryService.setAvailabilityCheckError(
01516|       new DomainError('Product is out of stock')
01517|     );
01518| 
01519|     const input: CreateOrderInput = {
01520|       customerId: 'customer-123',
01521|       items: [
01522|         { productId: 'out-of-stock-product', quantity: 1 }
01523|       ],
01524|       shippingAddress: TestAddresses.VALID_ADDRESS,
01525|       paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
01526|     };
01527| 
01528|     const result = await handler.execute(input);
01529| 
01530|     expect(result.isSuccess).toBe(false);
01531|     expect(result.error).toBe('Product is out of stock');
01532|     expect(orderRepository.orders.size).toBe(0);
01533|   });
01534| });
01535| 
01536| // 3. Integration Tests - Full stack with test adapters
01537| describe('Order API Integration', () => {
01538|   it('should create order via HTTP API', async () => {
01539|     const app = createTestApp();
01540|     const response = await request(app)
01541|       .post('/api/orders')
01542|       .send({
01543|         customerId: 'customer-123',
01544|         items: [{ productId: 'product-1', quantity: 2 }],
01545|         shippingAddress: TestAddresses.VALID_ADDRESS,
01546|         paymentMethod: TestPaymentMethods.VALID_CREDIT_CARD
01547|       })
01548|       .expect(201);
01549| 
01550|     expect(response.body.success).toBe(true);
01551|     expect(response.body.data.orderId).toBeDefined();
01552|   });
01553| });
01554| 
01555| // 4. Contract Tests - Adapter contract verification
01556| describe('OrderRepository Contract', () => {
01557|   let repository: OrderRepository;
01558| 
01559|   beforeEach(() => {
01560|     repository = new InMemoryOrderRepository();
01561|   });
01562| 
01563|   it('should save and find order by id', async () => {
01564|     const order = Order.create(TestOrderData.VALID_DRAFT_ORDER);
01565| 
01566|     await repository.save(order);
01567|     const found = await repository.findById(order.id);
01568| 
01569|     expect(found).toEqual(order);
01570|   });
01571| 
01572|   it('should return null for non-existent order', async () => {
01573|     const found = await repository.findById(OrderId.of('non-existent'));
01574| 
01575|     expect(found).toBeNull();
01576|   });
01577| });
01578| ```
01579| 
01580| ---
01581| 
01582| ## Benefits of Layered Architecture
01583| 
01584| | Benefit | Description |
01585| |---------|-------------|
01586| | **Testability** | Business rules can be tested without UI, database, or external systems using pure unit tests |
01587| | **Maintainability** | Changes to outer layers (frameworks, databases) do not affect inner layers (domain, use cases) |
01588| | **Flexibility** | External systems and delivery mechanisms can be swapped without changing business code |
01589| | **Scalability** | Clear separation enables parallel development by multiple teams |
01590| | **Framework Independence** | Business logic survives framework changes and technology migrations |
01591| | **Adaptability** | Application can be delivered through multiple channels (HTTP, gRPC, CLI, messaging) |
01592| | **Longevity** | Core business logic outlives specific technology choices |
01593| | **Test doubles** | Easy to create mock implementations of ports for testing |
01594| | **Technology Deferral** | Technology choices can be deferred until the outer layers |
01595| | **Domain Focus** | Business logic remains pure and focused on domain rules |
01596| 
01597| ---
01598| 
01599| ## Anti-Patterns to Avoid
01600| 
01601| | Anti-Pattern | Description | Solution |
01602| |--------------|-------------|----------|
01603| | **Anemic Domain** | Entities with no behavior, logic in services | Move business logic from services to domain entities |
01604| | **Dependency Inversion Violation** | Domain depending on frameworks or infrastructure | Use dependency injection, define ports in domain/application |
01605| | **Feature Scattered Across Layers** | Related code in multiple layers | Organize by feature module, not by layer |
01606| | **God Interface** | One port/interface for everything | Separate inbound/outbound ports, single responsibility |
01607| | **Skipping Layers** | Controllers directly calling repositories | Always go through use cases for business operations |
01608| | **Leaking Infrastructure** | Domain entities with framework annotations | Keep domain pure, use adapters for persistence mapping |
01609| | **Concrete Dependencies** | Use cases depending on concrete implementations | Depend on abstractions (interfaces), inject implementations |
01610| | **Missing Ports** | Direct imports of infrastructure in core | Define ports, implement in adapters |
01611| 
01612| ### Common Violations and Fixes
01613| 
01614| ```typescript
01615| // WRONG - Domain depends on JPA/ORM annotations
01616| @Entity
01617| class Order {
01618|   @Id
01619|   private id: string;
01620|   
01621|   @Column
01622|   private status: string;
01623| }
01624| 
01625| // RIGHT - Pure domain entity
01626| export class Order {
01627|   private constructor(
01628|     private readonly id: OrderId,
01629|     private readonly status: OrderStatus
01630|   ) {}
01631| }
01632| 
01633| // WRONG - Controller directly calls repository
01634| class OrderController {
01635|   async handle() {
01636|     // Bypasses use case layer
01637|     await this.entityManager.save(order);
01638|   }
01639| }
01640| 
01641| // RIGHT - Controller calls use case
01642| class OrderController {
01643|   async handle(request: Request): Promise<Response> {
01644|     const result = await this.createOrderHandler.execute(request.body);
01645|     return Response.of(result);
01646|   }
01647| }
01648| 
01649| // WRONG - Use case depends on concrete implementation
01650| class CreateOrderHandler {
01651|   constructor(
01652|     private repository: TypeORMOrderRepository // Should be interface
01653|   ) {}
01654| }
01655| 
01656| // RIGHT - Use case depends on abstraction
01657| class CreateOrderHandler {
01658|   constructor(
01659|     private repository: OrderRepository // Interface
01660|   ) {}
01661| }
01662| ```
01663| 
01664| ---
01665| 
01666| ## Layered Architecture in AI-Assisted Development
01667| 
01668| ```markdown
01669| ## AI Prompt for Layered Architecture Implementation
01670| 
01671| Generate a feature following Layered Architecture (Ports and Adapters):
01672| 
01673| Context: E-commerce Order Management System
01674| 
01675| Requirements:
01676| 1. Create order from shopping cart items
01677| 2. Validate inventory availability
01678| 3. Process payment
01679| 4. Send order confirmation notification
01680| 5. Publish order created event
01681| 
01682| Generate:
01683| 1. Domain entities (Order, OrderLineItem)
01684| 2. Value objects (Money, Address, OrderId)
01685| 3. Domain events (OrderCreated, PaymentProcessed)
01686| 4. Use case handler (CreateOrderHandler)
01687| 5. Input/output ports (interfaces)
01688| 6. Repository interface for orders
01689| 7. HTTP controller adapter
01690| 8. TypeORM repository adapter
01691| 9. Unit tests for domain rules
01692| 10. Integration tests for use cases
01693| 
01694| Constraints:
01695| - All dependencies point inward to domain
01696| - Domain has no imports from outer layers
01697| - Use TypeScript
01698| - Include JSDoc documentation
01699| - Follow dependency inversion principle
01700| - Support multiple payment gateways (strategy pattern)
01701| ```
01702| 
01703| ---
01704| 
01705| ## Comparison with Related Patterns
01706| 
01707| ### Layered Architecture vs. Clean Architecture
01708| 
01709| Clean Architecture, introduced by Robert C. Martin, is essentially the same concept as Hexagonal Architecture with a slightly different layer naming convention. Clean Architecture typically includes four layers: Frameworks & Drivers, Interface Adapters, Application Business Rules, and Enterprise Business Rules. The core principles and dependency rules are identical.
01710| 
01711| ### Layered Architecture vs. Onion Architecture
01712| 
01713| Onion Architecture, proposed by Jeffrey Palermo, uses similar concentric layer concepts but emphasizes the "onion" metaphor with dependencies pointing toward the center. The layers are often named: Application Core (inner), Domain Services, Application Services (outer), and Infrastructure. The fundamental principles are the same as Layered Architecture.
01714| 
01715| ### Layered Architecture vs. Microservices
01716| 
01717| While both promote separation of concerns, Layered Architecture is an application-level pattern for organizing code within a single service, whereas Microservices is an architectural style for decomposing an entire system into independently deployable services. Layered Architecture principles can be applied within each microservice.
01718| 
01719| ---
01720| 
01721| ## References and Further Reading
01722| 
01723| 1. Cockburn, Alistair. "Hexagonal Architecture." 2005.
01724| 2. Martin, Robert C. "Clean Architecture: A Craftsman's Guide to Software Structure and Design." Prentice Hall, 2017.
01725| 3. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
01726| 4. Vernon, Vaughn. "Implementing Domain-Driven Design." Addison-Wesley, 2013.
01727| 5. Martin, Robert C. "The Dependency Inversion Principle." 1996.
01728| 6. Percival, Harry, and Bob Gregory. "Architecture Patterns with Python." O'Reilly, 2021.
01729| 7. Ford, Neal, et al. "Architecture: The Hard Parts." O'Reilly, 2021.
01730| 8. Zimarev, Alexey. "Hands-On Domain-Driven Design with .NET Core." Packt, 2019.
01731| 

(End of file - total 1731 lines)
</file>
```
```

---

## Assistant

_Thinking:_

Based on the blog post I fetched, the three principles of Clean Architecture are:

1. **分層原則 (Layering Principle)** - The architecture is divided into layers (Entities, Use Cases, Interface Adapters, Frameworks & Drivers). Objects should be placed in appropriate layers.

2. **相依性原則 (Dependency Principle)** - Source code dependencies must point from outer to inner layers, from low-level to high-level. Entities layer can only reference objects in the same layer. Use Cases layer can only reference Entities layer and objects in the same layer, and so on. This is achieved through Dependency Inversion Principle (DIP).

3. **跨層原則 (Cross-Layer Principle)** - When domain objects cross layers to go outside (to UI, DB, or other external systems), do not pass domain objects directly. Instead, define interfaces and data structures in the Use Cases layer and convert domain objects to DTOs before passing them outside.

The blog also mentions that while strict adherence to these principles results in a clean architecture, practical implementation may require some compromises.

Let me add this section to the layered-architecture.md file.

```
Tool: edit

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "newString": "## The Three Fundamental Principles\n\nLayered Architecture is governed by three fundamental principles that guide the organization of code and dependencies. These principles, when properly applied, create a clean separation between business logic and external concerns.\n\n### 1. Layering Principle (分層原則)\n\nThe architecture is divided into concentric layers, each with a specific responsibility and level of abstraction. The most important principle of layering is that each layer has a clear purpose and that objects within each layer know only about their own layer and the layers inside it.\n\n**The Four Layers:**\n\n- **Entities Layer (Innermost)**: Contains domain model objects including Aggregates, Entities, Value Objects, and Domain Services following Domain-Driven Design (DDD) patterns. This layer represents the core business logic and should be completely independent of other layers.\n\n- **Use Cases Layer**: Contains application-specific business logic, corresponding to DDD's Application Services. This layer orchestrates domain objects to perform specific business operations and defines input/output port interfaces.\n\n- **Interface Adapters Layer**: Contains adapters that translate between external systems and the application core. This includes controllers, gateways, presenters, and database adapters that implement the ports defined in the inner layers.\n\n- **Frameworks & Drivers Layer (Outermost)**: Contains external frameworks, tools, and delivery mechanisms such as Web frameworks, database systems, external services, and UI frameworks.\n\n**Placement Considerations:**\n\nWhile the four layers are clearly separated in principle, some objects fall into gray areas during implementation. For example, the placement of DDD Repositories and Mappers can be debated. Some practitioners place them in the Use Cases layer, while others place them closer to the Entities layer. The key is to maintain consistency within the codebase and ensure that the dependency rules are respected.\n\n```\nLayer Placement Decision Guide:\n\n┌─────────────────────────────────────────────────────────────┐\n│                  FRAMEWORKS & DRIVERS                       │\n│    Web frameworks, Database drivers, External libraries     │\n└─────────────────────────────────────────────────────────────┘\n                              │\n┌─────────────────────────────────────────────────────────────┐\n│                  INTERFACE ADAPTERS                          │\n│    Controllers, Presenters, Gateway implementations,        │\n│    Mappers, Serializers                                     │\n└─────────────────────────────────────────────────────────────┘\n                              │\n┌─────────────────────────────────────────────────────────────┐\n│                      USE CASES                              │\n│    Application services, Use case handlers,                 │\n│    Port interfaces (Input/Output), DTOs                     │\n└─────────────────────────────────────────────────────────────┘\n                              │\n┌─────────────────────────────────────────────────────────────┐\n│                       ENTITIES                               │\n│    Aggregates, Entities, Value Objects, Domain Services,    │\n│    Domain Events, Domain Factories                          │\n└─────────────────────────────────────────────────────────────┘\n```\n\n### 2. Dependency Principle (相依性原則)\n\nSource code dependencies must always point **inward** from outer layers to inner layers. This means that code in outer layers may depend on abstractions defined in inner layers, but inner layers must never depend on outer layers. This principle is the cornerstone of clean architecture and is achieved through the **Dependency Inversion Principle (DIP)**.\n\n**Dependency Rules:**\n\n- Objects in the Entities layer can only reference other objects within the Entities layer\n- Objects in the Use Cases layer can only reference the Entities layer and other objects within the Use Cases layer\n- Objects in the Interface Adapters layer can reference both Use Cases and Entities layers (through defined ports)\n- The Frameworks & Drivers layer can reference all inner layers through their public interfaces\n\n**Achieving Inward Dependencies with Dependency Inversion:**\n\nWhen the Use Cases layer needs to persist Aggregates to a database, it must reference database objects. However, this would violate the dependency rule since the database is in an outer layer. The solution is to define a port (interface) in the Use Cases layer and have the outer database layer implement it. This way, the dependency direction is reversed: the outer layer depends on the inner layer's abstraction.\n\n```typescript\n// Use Cases layer defines the abstraction (port)\ninterface OrderRepository {\n  save(order: Order): Promise<void>;\n  findById(id: OrderId): Promise<Order | null>;\n}\n\n// Interface Adapters layer implements the port\nclass SqlOrderRepository implements OrderRepository {\n  constructor(private db: DatabaseConnection) {}\n\n  async save(order: Order): Promise<void> {\n    // Implementation using SQL database\n    await this.db.query('INSERT INTO orders ...');\n  }\n\n  async findById(id: OrderId): Promise<Order | null> {\n    // Implementation using SQL database\n    const result = await this.db.query('SELECT * FROM orders WHERE id = ?', [id.value]);\n    return result ? OrderMapper.toDomain(result) : null;\n  }\n}\n\n// Dependency direction:\n// UseCases ──depends on──> OrderRepository (interface) ──implemented by──> SqlOrderRepository\n```\n\n**Practical Challenges:**\n\nCompletely adhering to the dependency principle can be challenging in practice. For example, when domain entities need to publish Domain Events using an event bus tool, using an external library directly would violate the dependency rule. The solution is to define an interface for the event bus and have the external library implement it through an adapter. While this adds complexity, it maintains the architectural integrity of the system.\n\n### 3. Cross-Layer Principle (跨層原則)\n\nWhen domain objects need to cross layer boundaries to be passed outside the Use Cases layer (to UI, database, or other external systems), they must not be passed directly. Instead, interfaces and data structures should be defined in the Use Cases layer, and domain objects should be converted to DTOs (Data Transfer Objects) before crossing layers.\n\n**Rationale:**\n\nDirectly passing domain objects outside the Use Cases layer can lead to the domain layer being affected by presentation logic or persistence requirements. For example, if a UI component needs specific data from an Order entity, the entity might be modified to accommodate UI needs, thereby contaminating the domain logic with presentation concerns. To maintain separation of concerns and single responsibility, objects crossing layers must be transformed.\n\n```typescript\n// WRONG - Domain object passed directly to UI\nclass OrderController {\n  async getOrder(req: Request, res: Response): Promise<void> {\n    const order = await this.getOrderUseCase.execute(req.params.orderId);\n    res.json(order); // Direct domain object exposure\n  }\n}\n\n// RIGHT - Domain object converted to DTO\nclass OrderController {\n  async getOrder(req: Request, res: Response): Promise<void> {\n    const result = await this.getOrderUseCase.execute(req.params.orderId);\n    const dto = OrderMapper.toDto(result.order); // Convert to DTO\n    res.json(dto);\n  }\n}\n\n// DTO defined in Use Cases layer\ninterface OrderDto {\n  id: string;\n  status: string;\n  total: number;\n  items: OrderItemDto[];\n}\n```\n\n**Benefits of the Cross-Layer Principle:**\n\n- **Separation of Concerns**: Domain objects remain focused on business logic, not presentation or persistence\n- **Flexibility**: UI and persistence can change without affecting domain objects\n- **Testability**: Domain objects can be tested in isolation without UI or database dependencies\n- **Encapsulation**: Internal domain structure is hidden from external layers\n\n### Practical Considerations: The \"Cleanliness\" Trade-off\n\nWhile strict adherence to all three principles results in a clean and maintainable architecture, practical implementation often requires balancing architectural purity with development efficiency. Consider the following guidelines:\n\n**Strict Adherence Recommended:**\n\n- The Dependency Principle should generally be followed, especially between the Entities and Use Cases layers. When external tools are needed, define abstractions and use adapters rather than direct dependencies.\n\n**Acceptable Relaxation:**\n\n- The Interface Adapters layer often couples tightly with specific frameworks (e.g., Web Controllers with Web framework classes). Attempting complete dependency inversion in this layer can be overly complex and may not provide sufficient benefit.\n\n**Practical Simplification:**\n\n- Separating Use Cases into Commands (state-modifying operations) and Queries (read operations) using the CQRS pattern can significantly simplify the design of Interface Adapters. Commands can share common presenters and view models, while only Queries need customized output handling. This approach keeps the adapter layer lean and focused.\n\n---\n\n## Benefits of Layered Architecture",
  "oldString": "## Benefits of Layered Architecture"
}
```
```

---

## Assistant

_Thinking:_

The edit was successful. Now I need to add a reference to the blog post in the References section.

```
Tool: edit

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "newString": "## References and Further Reading\n\n1. Cockburn, Alistair. \"Hexagonal Architecture.\" 2005.\n2. Martin, Robert C. \"Clean Architecture: A Craftsman's Guide to Software Structure and Design.\" Prentice Hall, 2017.\n3. Evans, Eric. \"Domain-Driven Design: Tackling Complexity in the Heart of Software.\" Addison-Wesley, 2004.\n4. Vernon, Vaughn. \"Implementing Domain-Driven Design.\" Addison-Wesley, 2013.\n5. Martin, Robert C. \"The Dependency Inversion Principle.\" 1996.\n6. Percival, Harry, and Bob Gregory. \"Architecture Patterns with Python.\" O'Reilly, 2021.\n7. Ford, Neal, et al. \"Architecture: The Hard Parts.\" O'Reilly, 2021.\n8. Zimarev, Alexey. \"Hands-On Domain-Driven Design with .NET Core.\" Packt, 2019.\n9. Chen, Teddy. \"再談Clean Architecture三原則\" (Revisiting Clean Architecture Three Principles). teddy-chen-tw.blogspot.com, 2020.\n10. Chen, Teddy. \"Clean Architecture（4）：架構三原則首部曲—分層原則\" (Architecture Three Principles Part 1: Layering Principle). teddy-chen-tw.blogspot.com, 2018.\n11. Chen, Teddy. \"Clean Architecture（5）：架構三原則二部曲—相依性原則\" (Architecture Three Principles Part 2: Dependency Principle). teddy-chen-tw.blogspot.com, 2018.\n12. Chen, Teddy. \"Clean Architecture（6）：架構三原則三部曲—跨層原則\" (Architecture Three Principles Part 3: Cross-Layer Principle). teddy-chen-tw.blogspot.com, 2018.",
  "oldString": "## References and Further Reading\n\n1. Cockburn, Alistair. \"Hexagonal Architecture.\" 2005.\n2. Martin, Robert C. \"Clean Architecture: A Craftsman's Guide to Software Structure and Design.\" Prentice Hall, 2017.\n3. Evans, Eric. \"Domain-Driven Design: Tackling Complexity in the Heart of Software.\" Addison-Wesley, 2004.\n4. Vernon, Vaughn. \"Implementing Domain-Driven Design.\" Addison-Wesley, 2013.\n5. Martin, Robert C. \"The Dependency Inversion Principle.\" 1996.\n6. Percival, Harry, and Bob Gregory. \"Architecture Patterns with Python.\" O'Reilly, 2021.\n7. Ford, Neal, et al. \"Architecture: The Hard Parts.\" O'Reilly, 2021.\n8. Zimarev, Alexey. \"Hands-On Domain-Driven Design with .NET Core.\" Packt, 2019."
}
```
```

---

## Assistant

_Thinking:_

The edit was successful. I've added the three principles in English as requested by the user. Let me verify the changes were applied correctly.

Added the three Clean Architecture principles from Teddy Chen's blog in English to `doc/layered-architecture.md`:

1. **Layering Principle (分層原則)** - The architecture is divided into four layers: Entities, Use Cases, Interface Adapters, and Frameworks & Drivers. Each layer has specific responsibilities and objects should be placed in appropriate layers.

2. **Dependency Principle (相依性原則)** - Source code dependencies must point inward from outer layers to inner layers. Achieved through Dependency Inversion Principle (DIP), where inner layers define abstractions (ports) and outer layers implement them.

3. **Cross-Layer Principle (跨層原則)** - When domain objects cross layer boundaries to external systems, they must not be passed directly. Instead, convert domain objects to DTOs in the Use Cases layer before passing them outside.

Also added a practical considerations section discussing the trade-offs between strict adherence and practical implementation, plus references to Teddy Chen's blog posts.

---

## User

look inside the webpage, you find Clean Architecture（1）to （6）. read them all and find more detail about clean architecture's principle and write them inside layered-architecture.md

---

## Assistant

_Thinking:_

The user wants me to read all 6 articles from Teddy Chen's blog about Clean Architecture principles and add more detailed information to the layered-architecture.md file. Let me fetch all 6 articles.

From the blog post, I found these links:
1. Clean Architecture（1）：軟體架構的定義與目的 - https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture1.html
2. Clean Architecture（2）：Port and Adapter Architecture - https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html
3. Clean Architecture（3）：精選參考範例 - https://teddy-chen-tw.blogspot.com/2018/06/clean-architecture.html
4. Clean Architecture（4）：架構三原則首部曲—分層原則 - https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html
5. Clean Architecture（5）：架構三原則二部曲—相依性原則 - https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html
6. Clean Architecture（6）：架構三原則三部曲—跨層原則 - https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture6.html

Let me fetch all of these articles to get more detailed information.

```
Tool: webfetch

**Input:**
```json
{
  "format": "markdown",
  "url": "https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture1.html"
}
```
**Output:**
```
                  搞笑談軟工: Clean Architecture（1）：軟體架構的定義與目的   

 

 

l

# [搞笑談軟工](https://teddy-chen-tw.blogspot.com/)

敏捷開發，設計模式，精實開發，Scrum，軟體設計，軟體架構

## 網頁

-   [首頁](https://teddy-chen-tw.blogspot.com/)
-   [泰迪軟體最新課表](http://teddysoft.tw/courses/)
-   [泰迪軟體Facebook](https://www.facebook.com/teddysoft.tw)
-   [搞笑談軟工Facebook社團](https://www.facebook.com/groups/teddy.tw/)

## 2018年3月1日 星期四

  

### Clean Architecture（1）：軟體架構的定義與目的

March 01 23:01~23:58

[![螢幕截圖 2018-03-01 23.57.25](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhyAMiLpNmJ4yFETK3D1mgvEjr0Qtc_FhXa4RFf5fFnkW12aS4sE00-qvSfeGWnEvPrykK-Cv0rFAHkaq5KxbKShBGgwQHxzj2jVJV6ap62zXNeN2-2INYyvWerCtqVwF5t3dNOPWQlCpLG/?imgmax=800 "螢幕截圖 2018-03-01 23.57.25")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhf7QN9Is2DZ5nJ0MK__s4MXIqL785thjNZBAM1bGxYB_MA7hIu6KJAQu79M8Kji6ahgB0Hg_JlWn6slGEXutI8FoR2unGqUTOnpEmIEHGTy7mcDDDUFQkg0M-b7KeYGxf1S009YID30c9i/s1600-h/Image)

▲佩納宮

**定義**

這學期開始在北科教軟體架構，使用《[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_1?ie=UTF8&qid=1519919929&sr=8-1&keywords=clean+architecture)》當作教科書，這一系列文章節錄Teddy認為的重點介紹給鄉民。

要談軟體架構，首先就要先知道軟體架構的定義。很多組織都有「架構」，例如公司組織架構、學校組織架構、政府組織架構，這些都是一般人類可以理解的架構。從這些架構著手，可以類推軟體架構的定義。

以中型公司組織架構為例，有董事會、董事長、董事會、總經理、處室、組等，以及各部門裡面的員工。由組織架構圖可以看出，所有的架構一定有三個東西：

-   **構成組織的單元**：也就是不同職能的員工，在Clean Architecture中則稱為元件（component）。
-   **人員的安排**：階層式、扁平式、矩陣式，component team還是feature team。在Clean Architecture中稱為元件的排列方式（arrangement）。
-   **人員溝通的方式**：直接溝通、透過部門主管或PM與其他部門或外部客戶與廠商溝通、禁止溝通、聖上垂詢才可開口等。在Clean Architecture中稱為（communicate）。

《[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_1?ie=UTF8&qid=1519919929&sr=8-1&keywords=clean+architecture)》的原文如下：

-   **The architecture of a software is the shape given to that system by those who build it.**
-   **The form of that shape is in the division of the system into components, the arrangement of those components, and the ways in which those components communicate with each other.**

軟體架構的定義很多，這可以算是Teddy看過最簡潔、清楚又實用的定義。

\*\*\*

**形狀與功能基本上無關**

既然軟體架構是建構軟體的人所賦予它的**形狀**，那麼這個形狀的目的是什麼？假設你要蓋一個美術館，你會給這個美術館什麼形狀（外觀）？不少人可能聽過「Form follows Function」這個說法，形狀會跟隨著功能而改變，換句話說，軟體架構的形狀應該也會跟著功能（use case或user story）而改變。但實際在軟體架構領域中，一種廣為被接受的說法則是**架構基本上與功能無關**。實際上在真實世界中也是如此，下圖是用Google搜尋圖片下「美術館」關鍵字所找到的四個美術館，達到同樣的功能，但外型卻各異。

  

[![螢幕截圖 2018-03-01 23.36.23](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhwO9KTsgO_eCIjx7cqMRbvAuyevESHkfh-sX7pf4Dk1ltohG3lu5LDBAPWGAG_nnFTrueKTpOkUU-BsZh3YWPlE9GNXU0tgih7_-VfKIRlqszMXRctDRpuW16ZKH_k83vRoePDggw-ZH7e/?imgmax=800 "螢幕截圖 2018-03-01 23.36.23")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgJwDNA_iP9D7CJlEvyX979YCzEydKEdI5Axnby2aqkeMZAlyvAwC6DP3l1V97VPuIGYa-oQnej_HoHSdEC7YiN3IE0zrD7TreiqPnH3Sl_IgFEfYLz34iBTXyWzDyofkDEO46Xy5E2yZ1t/s1600-h/Image)

  

那形狀的目的是什麼？依據《[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_1?ie=UTF8&qid=1519919929&sr=8-1&keywords=clean+architecture)》的說法，形狀的目的是為了幫助：

-   **開發（development）**
-   **佈署（deployment）**
-   **維運（operation）**
-   **維護（maintenance ）**

換句話說，決定軟體架構的因素，主要是考量如何有助於軟體開發、佈署、上線營運以及日後的維護與功能擴充。

**總的來說，軟體架構的終極目標可以簡化成以下兩點：**

-   **最小化軟體生命周期的總成本**
    
-   **最大化程式設計師的生產力**
    

\*\*\*

**好的架構要反應功能**

最後一個重要的觀念就是雖然架構的**外觀**與功能基本無關，但**好的架構**本身卻需要能夠反應系統的功能。《[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_1?ie=UTF8&qid=1519919929&sr=8-1&keywords=clean+architecture)》有個例子，你看到住宅的架構圖（平面圖），你會知道這個建築的用途是住宅，你看到圖書館的架構圖，你會知道這是一個圖書館。

回到軟體本身，試想一個Web購物網站應用系統，當開發人員打開程式，看到的是

tw.teddysoft.web

tw.teddysoft.controller

tw.teddysoft.domain

tw.teddysoft.database

類似這樣的架構組織關係，還是可以看出來系統有顯示產品列表、加入購物車、結帳、付款、選擇運送方式、追蹤訂單等功能？

**為什麼好的架構要反應系統功能？因為這樣才可以協助開發、佈署、維運與維護。**

\*\*\*  

友藏內心獨白：架構形狀與功能無關但架構內在要反應出系統功能。

張貼者： [Teddy Chen](https://www.blogger.com/profile/02066842119056439711 "author profile") 於  [晚上11:58](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture1.html "permanent link")  [![](//img1.blogblog.com/img/icon18_email.gif)](https://www.blogger.com/email-post/1298974142445162186/5357546483335560645 "電子郵件發布")[![](https://resources.blogblog.com/img/icon18_edit_allbkg.gif)](https://www.blogger.com/post-edit.g?blogID=1298974142445162186&postID=5357546483335560645&from=pencil "編輯文章")

[以電子郵件傳送這篇文章](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=5357546483335560645&target=email "以電子郵件傳送這篇文章")[BlogThis！](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=5357546483335560645&target=blog "BlogThis！")[分享至 X](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=5357546483335560645&target=twitter "分享至 X")[分享至 Facebook](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=5357546483335560645&target=facebook "分享至 Facebook")[分享到 Pinterest](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=5357546483335560645&target=pinterest "分享到 Pinterest")

標籤： [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B)

#### 沒有留言:

#### 張貼留言

[](https://www.blogger.com/comment/frame/1298974142445162186?po=5357546483335560645&hl=zh-TW&saa=85391&origin=https://teddy-chen-tw.blogspot.com)

[較新的文章](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html "較新的文章") [較舊的文章](https://teddy-chen-tw.blogspot.com/2018/03/blog-post.html "較舊的文章") [首頁](https://teddy-chen-tw.blogspot.com/)

訂閱： [張貼留言 (Atom)](https://teddy-chen-tw.blogspot.com/feeds/5357546483335560645/comments/default)

## 搜尋此網站

 

## 總網頁瀏覽量

## 關於

[![我的相片](//blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBBqKlknBrSZ3EEXiWqeZasJOxrwoUfdqqF4Es3OR0YFtwDAYNHp6bBx3PqLGVJXAJQ4INhfp0gUlRZyRoSF9vzraU_JlLOECrERAWag3VIa0JdxRhpLwyEz7w9QtX8w/s220/Teddy.jpg)](https://www.blogger.com/profile/02066842119056439711)

[Teddy Chen](https://www.blogger.com/profile/02066842119056439711)

[檢視我的完整簡介](https://www.blogger.com/profile/02066842119056439711)

## 請幫Teddy按個讚^\_^

## 標籤

-   [工商服務](https://teddy-chen-tw.blogspot.com/search/label/%E5%B7%A5%E5%95%86%E6%9C%8D%E5%8B%99) (94)
-   [生活](https://teddy-chen-tw.blogspot.com/search/label/%E7%94%9F%E6%B4%BB) (342)
-   [改行寫網路小說算了](https://teddy-chen-tw.blogspot.com/search/label/%E6%94%B9%E8%A1%8C%E5%AF%AB%E7%B6%B2%E8%B7%AF%E5%B0%8F%E8%AA%AA%E7%AE%97%E4%BA%86) (25)
-   [其他](https://teddy-chen-tw.blogspot.com/search/label/%E5%85%B6%E4%BB%96) (25)
-   [客戶教我的事](https://teddy-chen-tw.blogspot.com/search/label/%E5%AE%A2%E6%88%B6%E6%95%99%E6%88%91%E7%9A%84%E4%BA%8B) (2)
-   [持續整合](https://teddy-chen-tw.blogspot.com/search/label/%E6%8C%81%E7%BA%8C%E6%95%B4%E5%90%88) (29)
-   [哲學](https://teddy-chen-tw.blogspot.com/search/label/%E5%93%B2%E5%AD%B8) (13)
-   [旅遊](https://teddy-chen-tw.blogspot.com/search/label/%E6%97%85%E9%81%8A) (568)
-   [視力測驗](https://teddy-chen-tw.blogspot.com/search/label/%E8%A6%96%E5%8A%9B%E6%B8%AC%E9%A9%97) (3)
-   [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5) (410)
-   [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B) (110)
-   [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88) (188)
-   [最新課程](https://teddy-chen-tw.blogspot.com/search/label/%E6%9C%80%E6%96%B0%E8%AA%B2%E7%A8%8B) (81)
-   [創業](https://teddy-chen-tw.blogspot.com/search/label/%E5%89%B5%E6%A5%AD) (84)
-   [測試](https://teddy-chen-tw.blogspot.com/search/label/%E6%B8%AC%E8%A9%A6) (106)
-   [盡信書不如無書](https://teddy-chen-tw.blogspot.com/search/label/%E7%9B%A1%E4%BF%A1%E6%9B%B8%E4%B8%8D%E5%A6%82%E7%84%A1%E6%9B%B8) (2)
-   [需求](https://teddy-chen-tw.blogspot.com/search/label/%E9%9C%80%E6%B1%82) (23)
-   [課程實錄](https://teddy-chen-tw.blogspot.com/search/label/%E8%AA%B2%E7%A8%8B%E5%AF%A6%E9%8C%84) (22)
-   [學習](https://teddy-chen-tw.blogspot.com/search/label/%E5%AD%B8%E7%BF%92) (92)
-   [貓](https://teddy-chen-tw.blogspot.com/search/label/%E8%B2%93) (15)
-   [還少一本書](https://teddy-chen-tw.blogspot.com/search/label/%E9%82%84%E5%B0%91%E4%B8%80%E6%9C%AC%E6%9B%B8) (36)
-   [agile](https://teddy-chen-tw.blogspot.com/search/label/agile) (638)
-   [C. C. Agile](https://teddy-chen-tw.blogspot.com/search/label/C.%20C.%20Agile) (33)
-   [DDD](https://teddy-chen-tw.blogspot.com/search/label/DDD) (85)
-   [DevOps](https://teddy-chen-tw.blogspot.com/search/label/DevOps) (3)
-   [Eclipse](https://teddy-chen-tw.blogspot.com/search/label/Eclipse) (6)
-   [exception handling](https://teddy-chen-tw.blogspot.com/search/label/exception%20handling) (84)
-   [HCI](https://teddy-chen-tw.blogspot.com/search/label/HCI) (44)
-   [Implementation Patterns](https://teddy-chen-tw.blogspot.com/search/label/Implementation%20Patterns) (5)
-   [Kanban](https://teddy-chen-tw.blogspot.com/search/label/Kanban) (79)
-   [Mobile](https://teddy-chen-tw.blogspot.com/search/label/Mobile) (2)
-   [Patterns](https://teddy-chen-tw.blogspot.com/search/label/Patterns) (198)
-   [Programming](https://teddy-chen-tw.blogspot.com/search/label/Programming) (54)
-   [Refactoring](https://teddy-chen-tw.blogspot.com/search/label/Refactoring) (65)
-   [Scrum](https://teddy-chen-tw.blogspot.com/search/label/Scrum) (461)
-   [TDD/BDD](https://teddy-chen-tw.blogspot.com/search/label/TDD%2FBDD) (54)

## 網誌存檔

-   [►](javascript:void\(0\))  [2025](https://teddy-chen-tw.blogspot.com/2025/) (1)
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2025/06/) (1)

-   [►](javascript:void\(0\))  [2024](https://teddy-chen-tw.blogspot.com/2024/) (7)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2024/09/) (6)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2024/03/) (1)

-   [►](javascript:void\(0\))  [2023](https://teddy-chen-tw.blogspot.com/2023/) (25)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2023/09/) (5)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2023/07/) (3)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2023/06/) (5)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2023/03/) (10)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2023/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2023/01/) (1)

-   [►](javascript:void\(0\))  [2022](https://teddy-chen-tw.blogspot.com/2022/) (29)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2022/10/) (1)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2022/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2022/07/) (18)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2022/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2022/03/) (2)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2022/02/) (1)

-   [►](javascript:void\(0\))  [2021](https://teddy-chen-tw.blogspot.com/2021/) (27)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2021/12/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2021/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2021/09/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2021/08/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2021/06/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2021/04/) (1)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2021/03/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2021/01/) (8)

-   [►](javascript:void\(0\))  [2020](https://teddy-chen-tw.blogspot.com/2020/) (32)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2020/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2020/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2020/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2020/09/) (4)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2020/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2020/07/) (2)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2020/06/) (2)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2020/05/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2020/04/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2020/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2020/02/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2020/01/) (1)

-   [►](javascript:void\(0\))  [2019](https://teddy-chen-tw.blogspot.com/2019/) (50)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2019/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2019/11/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2019/10/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2019/07/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2019/06/) (1)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2019/05/) (8)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2019/04/) (2)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2019/03/) (8)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2019/02/) (8)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2019/01/) (7)

-   [▼](javascript:void\(0\))  [2018](https://teddy-chen-tw.blogspot.com/2018/) (28)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2018/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2018/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2018/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2018/08/) (6)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2018/07/) (6)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2018/06/) (3)
    
    -   [▼](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2018/03/) (3)
        -   [為什麼要寫自動化測試？](https://teddy-chen-tw.blogspot.com/2018/03/blog-post_19.html)
        -   [Clean Architecture（2）：Port and Adapter Architecture](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html)
        -   [Clean Architecture（1）：軟體架構的定義與目的](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture1.html)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2018/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2018/01/) (3)

-   [►](javascript:void\(0\))  [2017](https://teddy-chen-tw.blogspot.com/2017/) (142)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2017/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2017/11/) (8)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2017/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2017/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2017/08/) (15)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2017/07/) (10)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2017/06/) (14)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2017/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2017/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2017/03/) (23)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2017/02/) (16)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2017/01/) (25)

-   [►](javascript:void\(0\))  [2016](https://teddy-chen-tw.blogspot.com/2016/) (301)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2016/12/) (14)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2016/11/) (19)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2016/10/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2016/09/) (18)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2016/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2016/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2016/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2016/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2016/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2016/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2016/02/) (29)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2016/01/) (31)

-   [►](javascript:void\(0\))  [2015](https://teddy-chen-tw.blogspot.com/2015/) (366)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2015/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2015/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2015/10/) (32)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2015/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2015/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2015/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2015/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2015/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2015/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2015/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2015/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2015/01/) (31)

-   [►](javascript:void\(0\))  [2014](https://teddy-chen-tw.blogspot.com/2014/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2014/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2014/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2014/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2014/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2014/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2014/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2014/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2014/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2014/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2014/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2014/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2014/01/) (31)

-   [►](javascript:void\(0\))  [2013](https://teddy-chen-tw.blogspot.com/2013/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2013/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2013/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2013/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2013/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2013/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2013/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2013/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2013/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2013/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2013/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2013/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2013/01/) (31)

-   [►](javascript:void\(0\))  [2012](https://teddy-chen-tw.blogspot.com/2012/) (373)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2012/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2012/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2012/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2012/09/) (31)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2012/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2012/07/) (32)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2012/06/) (31)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2012/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2012/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2012/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2012/02/) (32)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2012/01/) (32)

-   [►](javascript:void\(0\))  [2011](https://teddy-chen-tw.blogspot.com/2011/) (120)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2011/12/) (16)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2011/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2011/10/) (4)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2011/09/) (3)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2011/08/) (9)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2011/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2011/06/) (10)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2011/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2011/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2011/03/) (16)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2011/02/) (12)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2011/01/) (11)

-   [►](javascript:void\(0\))  [2010](https://teddy-chen-tw.blogspot.com/2010/) (104)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2010/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2010/11/) (6)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2010/10/) (10)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2010/09/) (5)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2010/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2010/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2010/06/) (19)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2010/05/) (12)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2010/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2010/03/) (12)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2010/02/) (2)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2010/01/) (7)

-   [►](javascript:void\(0\))  [2009](https://teddy-chen-tw.blogspot.com/2009/) (31)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2009/12/) (5)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2009/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2009/10/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2009/08/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2009/07/) (8)

-   [►](javascript:void\(0\))  [2008](https://teddy-chen-tw.blogspot.com/2008/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2008/09/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2008/03/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2008/01/) (2)

-   [►](javascript:void\(0\))  [2007](https://teddy-chen-tw.blogspot.com/2007/) (4)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2007/12/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2007/06/) (3)

## 追蹤者

## 最新回應

[載入中…](http://teddy-chen-tw.blogspot.tw/feeds/comments/default)

## 熱門文章

-   [爭什麼，摻在一起做成撒尿牛丸啊](https://teddy-chen-tw.blogspot.com/2012/03/blog-post_20.html)
    
    March 20 13:53~14:56 (影像節錄自http://www.youtube.com/watch?v=LGV5PcM9SQo) 鄉民們都看過食神這部電影吧，電影中有一個經典的橋段，就是火鷄姐及鵝頭在爭辯誰做的撒尿蝦好吃，誰做的牛丸比較優。正當雙方吵得不可...
    
-   [擁抱改變，遠離Hard Code](https://teddy-chen-tw.blogspot.com/2018/09/hard-code.html)
    
    September 19 08:37~09:12 ▲圖片來源 在此 寫死（Hard Code） 幾天前一位朋友在Facebook上發問：「如何對非資訊人員解釋寫死（hard code）？」 維基百科 對於寫死的解釋如下： 寫死 （英語： Hard Code 或 Hard Co...
    
-   [什麼是物件導向（3）：Polymorphism](https://teddy-chen-tw.blogspot.com/2012/01/3polymorphism.html)
    
    January 26 23:01~January 27 00:23 Polymorphism，中文翻譯成「多型」，說真的這個概念當年Teddy學OO的時候搞了好久才弄懂，後來看了「課本」的定義之後，覺得寫得還滿好的，請參考Object-Oriented Software En...
    
-   [幫Macbook Pro 15” Late 2016尋找外接4K 27”螢幕](https://teddy-chen-tw.blogspot.com/2017/07/macbook-pro-15-late-20164k-27.html)
    
    June 17 21:28~23:24 先打個廣告，【 敏捷開發懶人包：物件導向技能 】確定於8月19日（週六）開課。 課程設定目標希望讓鄉民們在找工作面試的時候，如果面試官問到物件導向技術方面的問題，能夠具備輕鬆回答的能力 。 \*\*\* 動心起念 去年底買了MacBook P...
    
-   [Top-down和Bottom-up設計方法](https://teddy-chen-tw.blogspot.com/2014/03/top-downbottom-up.html)
    
    Mar. 12 08:40~10:10 Top-down（由上而下）和Bottom-up（由下而上）是兩種設計與解決問題的技巧。前者對問題先有一個整體的概念，然後再逐步加上設計細節，最後讓整體的輪廓越來越清楚。後者則是先將解決問題可能所需的基本元件、方案給準備好，然後再將這...
    
-   [羅技Logitech R500 雷射簡報筆開箱](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html)
    
    June 22 12:51~14:16 需求分析 因為工作的關係Teddy經常需要使用簡報筆，雜牌的不算，光是羅技的簡報筆Teddy就買過以下四種（R500是最新上市產品，也是今天介紹的主角）  ： ▼畫面節錄自羅技官網 Teddy心中理想的簡報筆應該有以下幾個功能： 重量輕且...
    

## 分析

 

技術提供：[Blogger](https://www.blogger.com).
```
```

```
Tool: webfetch

**Input:**
```json
{
  "format": "markdown",
  "url": "https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html"
}
```
**Output:**
```
                  搞笑談軟工: Clean Architecture（2）：Port and Adapter Architecture   

 

 

l

# [搞笑談軟工](https://teddy-chen-tw.blogspot.com/)

敏捷開發，設計模式，精實開發，Scrum，軟體設計，軟體架構

## 網頁

-   [首頁](https://teddy-chen-tw.blogspot.com/)
-   [泰迪軟體最新課表](http://teddysoft.tw/courses/)
-   [泰迪軟體Facebook](https://www.facebook.com/teddysoft.tw)
-   [搞笑談軟工Facebook社團](https://www.facebook.com/groups/teddy.tw/)

## 2018年3月15日 星期四

  

### Clean Architecture（2）：Port and Adapter Architecture

March 15 18:56~19:12；21:01~21:40

[![螢幕截圖 2018-03-15 18.58.07](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi54xc7r9n1eecFO1blw8YulKGRqfQWItwq9JMOxrWk8levNU9bVjd4Tx-7CollJ3sibDMIRgkqqFEqfPV6DNmY8xW5A9C7HAQ7H7MLUmSsAPaoQj9D8WK-R3oFGWOMN6JUD6kM6rM0z07e/?imgmax=800 "螢幕截圖 2018-03-15 18.58.07")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjnvLS0FrnnG7sGAAFH2JD3UO-X0pgJHoJD0ltshqVZSUpmB2zB_yDJdWa32rFQQt6duK1dUV8maaXe-gaIBw6KZUpkVxxscMVQb8S3vSaQAe6UaLmk17yTL9Agmqr8zQq-vPrHesqOcnOI/s1600-h/Image)

  

**一個架構各自表述?**

今天在北科上軟體架構講到**Port and Adapter Architecture**（接口與轉接器架構），這個架構又稱為**Hexagonal Architecture**（六角形架構），是Clean Architecture的核心。關於該架構的介紹可以參考Alistair Cockburn寫的〈[Hexagonal architecture](http://alistair.cockburn.us/Hexagonal+architecture)〉。

上課前一周請同學先閱讀〈[Hexagonal Architecture Is Powerful](https://dzone.com/articles/hexagonal-architecture-is-powerful)〉這篇文章，有同學發問：「**為什麼有些網路上的文章把Port畫在六角形外面，有些則是把Adapter畫在外面？**」（請參考下圖）

▼圖1：Port and Adapter的兩種不同畫法。

[![螢幕截圖 2018-03-15 17.39.22](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgqerqFadRkFjpuJMoHrwIVopXwhqk7IqL_qNpbaOqRte_7k_gvQf4UGb19LCqgm8nIFfIKUKjWKkNV0GlBpqCfYIlktIViG8FEWBGHiLD7v1wWvUaW0wf2FTtRQSP_vE7w1PwijYKKWKHk/?imgmax=800 "螢幕截圖 2018-03-15 17.39.22")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjZHXYqCEUWIrqDQqugpboTcyJBVXDboYcpW_gwtO55rl7XszWtj8tDsQIqIlAeJmr2KD5GLXhmHjPPfFNAf2Bk5H6VLINcVidXgI1JsVh80DZ8qAoTpiZXYpDumcvquWwgUsBy5PGtfnGD/s1600-h/Image)

這難道是一個架構各自表述？

\*\*\*

**畫圖的角度不同**

參考圖2，不同的文章從不同的角度來畫Port & Adapter，畫出來的圖自然不同。如果是從客戶端的角度來看整個系統架構，客戶端先接觸到Port，然後Adapter再把客戶端傳來的資料轉成函數呼叫，執行Use Case所提供的功能。

▼圖2：從不同角度來看Port and Adapter

[![螢幕截圖 2018-03-15 17.41.18](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjqZ20hu6V-LzkGIk0z2g_heq0Kb19VfwJlRUjJjyqU2vkVQSHG-dCNNLLaqcf8ombnOpe9YAvEWSjTjQTMbdZqYTw8ycyiNbwP76sBKU8IbfkjnfyGfE9DtYrm6mDnn995iGFJPmC_sAsd/?imgmax=800 "螢幕截圖 2018-03-15 17.41.18")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj3K70xVxOagZRplIyPg87bpteI6TBsjB78AhAWa0nvQsh0VEJyYXToJgzrwiKOwW0LiGeOAbOZciepwCzCF_VJ0-SXgWHJmNvFqoMpN_Xrb2KdlCpDc9YFi16Z7-jSOsZdDoSROaj2ujqy/s1600-h/Image)

  

如果是從Use Case的角度來看架構，則Use Case透過Port與外界溝通，當傳輸資料時，先把資料傳到Output Port （定義在Use Case中的Interface），然後Adapter實作這個Interface，再把資料轉給客戶端。接收資料時，客戶端透過Adapter把資料轉成Port所能接受的格式，然後傳給Use Case。

附帶說明，Adapter與通常Use Case執行於同一個記憶空間（屬於同一個Process、同一個Runtime Environment或同一個虛擬機器），轉接客戶端與Use Case之間的互動。

\*\*\*

**摻在一起做成撒尿牛丸啊**

也有人把架構畫成圖3的樣子，感覺起來有點圖1左、右兩圖的合體，但清楚標示Port與Adapter的相對位置，應該是有比較清楚一些。

▼圖3：Port在最外層，Adapter介於Port與Use Case（圖中的Mediation）之間。

[![螢幕截圖 2018-03-15 17.41.54](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg4N2JJ5d_sZnwfo14hUDE4l26bfAn3xphgPjkSrOshCxn2rHXk57ebyVcLxxb0fQ4NjAS8vNC6eiVdyp5Tooyxwh5oZ_Qb6oHOG5yJCSaxE62u5L25TWrokQbaC-zW9rTY9ODiInkwR7_S/?imgmax=800 "螢幕截圖 2018-03-15 17.41.54")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhwTyyVg7e9PAkstjgtzvrs8g-4c0l8w4CubypmGQIjNtvhZZJg0J0ccMbberWtHhLeMHa8ZEwrLO4rxUanUEzTXuON2iEVUgQkFUScFpBTYCx2b0f8ALv8gNCwfrPeINrWYaoHPzorlmHT/s1600-h/Image)

鄉民們可以參考看看各種不同的畫法，以後看資料的時候也許比較不會搞混。

\*\*\*

**結論**

嚴格來講，在《Clean Architecture》裡面，每跨一個階層邊界 （boundary）就會產生一組Input Port與Output Port，也就是說整個系統中，不同的邊界都有個別的Port and Adapter。這也是Port and Adapter的特色：「**軟體架構可以透過好多組Port and Adapter 一直不斷地串接下去**」。

類似下圖的概念。MacBook Pro的Thunderbolt 3的Port，配上HyperDrive這款Adapter，提供了好幾種不同的Port。而這些Port，又可以透過其他Adapter，繼續轉接下去。例如，HDMI轉VGA或USB Type-A轉乙太網路。

  

▼畫面節錄自[https://goo.gl/xBU3zN](https://goo.gl/xBU3zN)

[![螢幕截圖 2018-03-15 18.09.17](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgp8WAjfVRxo047_OEwDZUe_EzxZro5Eh1sLfmRINLairmH_R3PvvW_jCWwSZgHB6eCidJfba4yagoRO5CafTvAuN_9TBoUz4z_SvOITD05Chl5trMTIqJEYuVHxG7f0YBgUtbnMHzDpvDk/?imgmax=800 "螢幕截圖 2018-03-15 18.09.17")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEicVPaQ2VRJr1a7Tf7nwiC_NbWFHlkOr4g1j2GX44vux6Mwm1ePH_p1VtUNn_SpzSY95OJtkqGos06RnewlSXCqs8pe6G9P7U30HfSFxvDtIAffXM5uSa7DG3iYEZGeI51ujYcJmlWb_Dkm/s1600-h/Image)

\*\*\*

友藏內心獨白：轉接器越來越多，越來越貴。

張貼者： [Teddy Chen](https://www.blogger.com/profile/02066842119056439711 "author profile") 於  [晚上9:41](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html "permanent link")  [![](//img1.blogblog.com/img/icon18_email.gif)](https://www.blogger.com/email-post/1298974142445162186/2923682455342406480 "電子郵件發布")[![](https://resources.blogblog.com/img/icon18_edit_allbkg.gif)](https://www.blogger.com/post-edit.g?blogID=1298974142445162186&postID=2923682455342406480&from=pencil "編輯文章")

[以電子郵件傳送這篇文章](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2923682455342406480&target=email "以電子郵件傳送這篇文章")[BlogThis！](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2923682455342406480&target=blog "BlogThis！")[分享至 X](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2923682455342406480&target=twitter "分享至 X")[分享至 Facebook](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2923682455342406480&target=facebook "分享至 Facebook")[分享到 Pinterest](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=2923682455342406480&target=pinterest "分享到 Pinterest")

標籤： [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B)

#### 4 則留言:

1.  ![](//www.blogger.com/img/blogger_logo_round_35.png)
    
    [Drew](https://www.blogger.com/profile/16290801199180641771)[2018年5月16日下午1:17](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html?showComment=1526447850893#c992777702257074092)
    
    作者已經移除這則留言。
    
    回覆[刪除](https://www.blogger.com/comment/delete/1298974142445162186/992777702257074092)
    
    回覆
    
    回覆
    
2.  ![](//www.blogger.com/img/blogger_logo_round_35.png)
    
    [Drew](https://www.blogger.com/profile/16290801199180641771)[2018年5月16日下午1:26](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html?showComment=1526448363726#c7740568040686998358)
    
    Mysql connector這種jdbc driver，比較算是adapter還是port呢？  
    本來想說Mysql connector是Adapter，而裡面提供的APIs是port  
    但又覺得不對，好像要先有存取mysql的某種管道才有可能呼叫那些API吼，持續混淆中  
      
    如果以RESTful web api來說，web api本身有辦法類比成adpater或port嗎？  
    還是說這場景根本不是何這樣比喻呢？
    
    回覆[刪除](https://www.blogger.com/comment/delete/1298974142445162186/7740568040686998358)
    
    回覆
    
    -   ![](//blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBBqKlknBrSZ3EEXiWqeZasJOxrwoUfdqqF4Es3OR0YFtwDAYNHp6bBx3PqLGVJXAJQ4INhfp0gUlRZyRoSF9vzraU_JlLOECrERAWag3VIa0JdxRhpLwyEz7w9QtX8w/s45-c/Teddy.jpg)
        
        [Teddy Chen](https://www.blogger.com/profile/02066842119056439711)[2018年6月25日下午5:56](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html?showComment=1529920615650#c1993111277374100352)
        
        我覺得 jdbc driver與RESTful API進入點比較像是 port，因為有它們的的存在才可以存取（連線）資料庫/service。至於 adapter則是負責「轉換介面/資料」的物件，例如把從ReSTful API所收到的JSON物件轉成 use case Input所定義的物件，然後把它傳給 （呼叫）use case。
        
        [刪除](https://www.blogger.com/comment/delete/1298974142445162186/1993111277374100352)
        
        回覆
        
        回覆
        
    
    回覆
    
3.  ![](//www.blogger.com/img/blogger_logo_round_35.png)
    
    [Mark](https://www.blogger.com/profile/13921250437722766658)[2023年3月31日上午9:25](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html?showComment=1680225938031#c6891551101697519283)
    
    但是苹果的思想是很明确的就是简单化问题。因为大部分使用场景不需要插那么多乱七八糟的东西 port是主动设计，adapter是被动救急。 对非专业人员用port 对于有点专业的用adapter
    
    回覆[刪除](https://www.blogger.com/comment/delete/1298974142445162186/6891551101697519283)
    
    回覆
    
    回覆
    

新增留言

載入更多…

[](https://www.blogger.com/comment/frame/1298974142445162186?po=2923682455342406480&hl=zh-TW&saa=85391&origin=https://teddy-chen-tw.blogspot.com)

[較新的文章](https://teddy-chen-tw.blogspot.com/2018/03/blog-post_19.html "較新的文章") [較舊的文章](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture1.html "較舊的文章") [首頁](https://teddy-chen-tw.blogspot.com/)

訂閱： [張貼留言 (Atom)](https://teddy-chen-tw.blogspot.com/feeds/2923682455342406480/comments/default)

## 搜尋此網站

 

## 總網頁瀏覽量

## 關於

[![我的相片](//blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBBqKlknBrSZ3EEXiWqeZasJOxrwoUfdqqF4Es3OR0YFtwDAYNHp6bBx3PqLGVJXAJQ4INhfp0gUlRZyRoSF9vzraU_JlLOECrERAWag3VIa0JdxRhpLwyEz7w9QtX8w/s220/Teddy.jpg)](https://www.blogger.com/profile/02066842119056439711)

[Teddy Chen](https://www.blogger.com/profile/02066842119056439711)

[檢視我的完整簡介](https://www.blogger.com/profile/02066842119056439711)

## 請幫Teddy按個讚^\_^

## 標籤

-   [工商服務](https://teddy-chen-tw.blogspot.com/search/label/%E5%B7%A5%E5%95%86%E6%9C%8D%E5%8B%99) (94)
-   [生活](https://teddy-chen-tw.blogspot.com/search/label/%E7%94%9F%E6%B4%BB) (342)
-   [改行寫網路小說算了](https://teddy-chen-tw.blogspot.com/search/label/%E6%94%B9%E8%A1%8C%E5%AF%AB%E7%B6%B2%E8%B7%AF%E5%B0%8F%E8%AA%AA%E7%AE%97%E4%BA%86) (25)
-   [其他](https://teddy-chen-tw.blogspot.com/search/label/%E5%85%B6%E4%BB%96) (25)
-   [客戶教我的事](https://teddy-chen-tw.blogspot.com/search/label/%E5%AE%A2%E6%88%B6%E6%95%99%E6%88%91%E7%9A%84%E4%BA%8B) (2)
-   [持續整合](https://teddy-chen-tw.blogspot.com/search/label/%E6%8C%81%E7%BA%8C%E6%95%B4%E5%90%88) (29)
-   [哲學](https://teddy-chen-tw.blogspot.com/search/label/%E5%93%B2%E5%AD%B8) (13)
-   [旅遊](https://teddy-chen-tw.blogspot.com/search/label/%E6%97%85%E9%81%8A) (568)
-   [視力測驗](https://teddy-chen-tw.blogspot.com/search/label/%E8%A6%96%E5%8A%9B%E6%B8%AC%E9%A9%97) (3)
-   [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5) (410)
-   [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B) (110)
-   [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88) (188)
-   [最新課程](https://teddy-chen-tw.blogspot.com/search/label/%E6%9C%80%E6%96%B0%E8%AA%B2%E7%A8%8B) (81)
-   [創業](https://teddy-chen-tw.blogspot.com/search/label/%E5%89%B5%E6%A5%AD) (84)
-   [測試](https://teddy-chen-tw.blogspot.com/search/label/%E6%B8%AC%E8%A9%A6) (106)
-   [盡信書不如無書](https://teddy-chen-tw.blogspot.com/search/label/%E7%9B%A1%E4%BF%A1%E6%9B%B8%E4%B8%8D%E5%A6%82%E7%84%A1%E6%9B%B8) (2)
-   [需求](https://teddy-chen-tw.blogspot.com/search/label/%E9%9C%80%E6%B1%82) (23)
-   [課程實錄](https://teddy-chen-tw.blogspot.com/search/label/%E8%AA%B2%E7%A8%8B%E5%AF%A6%E9%8C%84) (22)
-   [學習](https://teddy-chen-tw.blogspot.com/search/label/%E5%AD%B8%E7%BF%92) (92)
-   [貓](https://teddy-chen-tw.blogspot.com/search/label/%E8%B2%93) (15)
-   [還少一本書](https://teddy-chen-tw.blogspot.com/search/label/%E9%82%84%E5%B0%91%E4%B8%80%E6%9C%AC%E6%9B%B8) (36)
-   [agile](https://teddy-chen-tw.blogspot.com/search/label/agile) (638)
-   [C. C. Agile](https://teddy-chen-tw.blogspot.com/search/label/C.%20C.%20Agile) (33)
-   [DDD](https://teddy-chen-tw.blogspot.com/search/label/DDD) (85)
-   [DevOps](https://teddy-chen-tw.blogspot.com/search/label/DevOps) (3)
-   [Eclipse](https://teddy-chen-tw.blogspot.com/search/label/Eclipse) (6)
-   [exception handling](https://teddy-chen-tw.blogspot.com/search/label/exception%20handling) (84)
-   [HCI](https://teddy-chen-tw.blogspot.com/search/label/HCI) (44)
-   [Implementation Patterns](https://teddy-chen-tw.blogspot.com/search/label/Implementation%20Patterns) (5)
-   [Kanban](https://teddy-chen-tw.blogspot.com/search/label/Kanban) (79)
-   [Mobile](https://teddy-chen-tw.blogspot.com/search/label/Mobile) (2)
-   [Patterns](https://teddy-chen-tw.blogspot.com/search/label/Patterns) (198)
-   [Programming](https://teddy-chen-tw.blogspot.com/search/label/Programming) (54)
-   [Refactoring](https://teddy-chen-tw.blogspot.com/search/label/Refactoring) (65)
-   [Scrum](https://teddy-chen-tw.blogspot.com/search/label/Scrum) (461)
-   [TDD/BDD](https://teddy-chen-tw.blogspot.com/search/label/TDD%2FBDD) (54)

## 網誌存檔

-   [►](javascript:void\(0\))  [2025](https://teddy-chen-tw.blogspot.com/2025/) (1)
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2025/06/) (1)

-   [►](javascript:void\(0\))  [2024](https://teddy-chen-tw.blogspot.com/2024/) (7)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2024/09/) (6)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2024/03/) (1)

-   [►](javascript:void\(0\))  [2023](https://teddy-chen-tw.blogspot.com/2023/) (25)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2023/09/) (5)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2023/07/) (3)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2023/06/) (5)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2023/03/) (10)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2023/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2023/01/) (1)

-   [►](javascript:void\(0\))  [2022](https://teddy-chen-tw.blogspot.com/2022/) (29)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2022/10/) (1)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2022/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2022/07/) (18)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2022/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2022/03/) (2)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2022/02/) (1)

-   [►](javascript:void\(0\))  [2021](https://teddy-chen-tw.blogspot.com/2021/) (27)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2021/12/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2021/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2021/09/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2021/08/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2021/06/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2021/04/) (1)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2021/03/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2021/01/) (8)

-   [►](javascript:void\(0\))  [2020](https://teddy-chen-tw.blogspot.com/2020/) (32)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2020/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2020/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2020/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2020/09/) (4)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2020/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2020/07/) (2)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2020/06/) (2)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2020/05/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2020/04/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2020/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2020/02/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2020/01/) (1)

-   [►](javascript:void\(0\))  [2019](https://teddy-chen-tw.blogspot.com/2019/) (50)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2019/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2019/11/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2019/10/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2019/07/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2019/06/) (1)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2019/05/) (8)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2019/04/) (2)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2019/03/) (8)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2019/02/) (8)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2019/01/) (7)

-   [▼](javascript:void\(0\))  [2018](https://teddy-chen-tw.blogspot.com/2018/) (28)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2018/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2018/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2018/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2018/08/) (6)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2018/07/) (6)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2018/06/) (3)
    
    -   [▼](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2018/03/) (3)
        -   [為什麼要寫自動化測試？](https://teddy-chen-tw.blogspot.com/2018/03/blog-post_19.html)
        -   [Clean Architecture（2）：Port and Adapter Architecture](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture2port-and-adapter.html)
        -   [Clean Architecture（1）：軟體架構的定義與目的](https://teddy-chen-tw.blogspot.com/2018/03/clean-architecture1.html)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2018/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2018/01/) (3)

-   [►](javascript:void\(0\))  [2017](https://teddy-chen-tw.blogspot.com/2017/) (142)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2017/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2017/11/) (8)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2017/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2017/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2017/08/) (15)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2017/07/) (10)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2017/06/) (14)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2017/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2017/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2017/03/) (23)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2017/02/) (16)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2017/01/) (25)

-   [►](javascript:void\(0\))  [2016](https://teddy-chen-tw.blogspot.com/2016/) (301)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2016/12/) (14)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2016/11/) (19)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2016/10/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2016/09/) (18)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2016/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2016/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2016/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2016/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2016/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2016/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2016/02/) (29)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2016/01/) (31)

-   [►](javascript:void\(0\))  [2015](https://teddy-chen-tw.blogspot.com/2015/) (366)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2015/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2015/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2015/10/) (32)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2015/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2015/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2015/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2015/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2015/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2015/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2015/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2015/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2015/01/) (31)

-   [►](javascript:void\(0\))  [2014](https://teddy-chen-tw.blogspot.com/2014/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2014/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2014/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2014/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2014/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2014/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2014/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2014/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2014/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2014/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2014/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2014/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2014/01/) (31)

-   [►](javascript:void\(0\))  [2013](https://teddy-chen-tw.blogspot.com/2013/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2013/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2013/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2013/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2013/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2013/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2013/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2013/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2013/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2013/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2013/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2013/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2013/01/) (31)

-   [►](javascript:void\(0\))  [2012](https://teddy-chen-tw.blogspot.com/2012/) (373)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2012/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2012/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2012/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2012/09/) (31)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2012/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2012/07/) (32)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2012/06/) (31)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2012/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2012/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2012/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2012/02/) (32)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2012/01/) (32)

-   [►](javascript:void\(0\))  [2011](https://teddy-chen-tw.blogspot.com/2011/) (120)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2011/12/) (16)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2011/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2011/10/) (4)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2011/09/) (3)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2011/08/) (9)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2011/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2011/06/) (10)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2011/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2011/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2011/03/) (16)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2011/02/) (12)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2011/01/) (11)

-   [►](javascript:void\(0\))  [2010](https://teddy-chen-tw.blogspot.com/2010/) (104)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2010/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2010/11/) (6)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2010/10/) (10)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2010/09/) (5)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2010/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2010/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2010/06/) (19)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2010/05/) (12)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2010/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2010/03/) (12)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2010/02/) (2)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2010/01/) (7)

-   [►](javascript:void\(0\))  [2009](https://teddy-chen-tw.blogspot.com/2009/) (31)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2009/12/) (5)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2009/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2009/10/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2009/08/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2009/07/) (8)

-   [►](javascript:void\(0\))  [2008](https://teddy-chen-tw.blogspot.com/2008/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2008/09/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2008/03/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2008/01/) (2)

-   [►](javascript:void\(0\))  [2007](https://teddy-chen-tw.blogspot.com/2007/) (4)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2007/12/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2007/06/) (3)

## 追蹤者

## 最新回應

[載入中…](http://teddy-chen-tw.blogspot.tw/feeds/comments/default)

## 熱門文章

-   [爭什麼，摻在一起做成撒尿牛丸啊](https://teddy-chen-tw.blogspot.com/2012/03/blog-post_20.html)
    
    March 20 13:53~14:56 (影像節錄自http://www.youtube.com/watch?v=LGV5PcM9SQo) 鄉民們都看過食神這部電影吧，電影中有一個經典的橋段，就是火鷄姐及鵝頭在爭辯誰做的撒尿蝦好吃，誰做的牛丸比較優。正當雙方吵得不可...
    
-   [擁抱改變，遠離Hard Code](https://teddy-chen-tw.blogspot.com/2018/09/hard-code.html)
    
    September 19 08:37~09:12 ▲圖片來源 在此 寫死（Hard Code） 幾天前一位朋友在Facebook上發問：「如何對非資訊人員解釋寫死（hard code）？」 維基百科 對於寫死的解釋如下： 寫死 （英語： Hard Code 或 Hard Co...
    
-   [什麼是物件導向（3）：Polymorphism](https://teddy-chen-tw.blogspot.com/2012/01/3polymorphism.html)
    
    January 26 23:01~January 27 00:23 Polymorphism，中文翻譯成「多型」，說真的這個概念當年Teddy學OO的時候搞了好久才弄懂，後來看了「課本」的定義之後，覺得寫得還滿好的，請參考Object-Oriented Software En...
    
-   [幫Macbook Pro 15” Late 2016尋找外接4K 27”螢幕](https://teddy-chen-tw.blogspot.com/2017/07/macbook-pro-15-late-20164k-27.html)
    
    June 17 21:28~23:24 先打個廣告，【 敏捷開發懶人包：物件導向技能 】確定於8月19日（週六）開課。 課程設定目標希望讓鄉民們在找工作面試的時候，如果面試官問到物件導向技術方面的問題，能夠具備輕鬆回答的能力 。 \*\*\* 動心起念 去年底買了MacBook P...
    
-   [Top-down和Bottom-up設計方法](https://teddy-chen-tw.blogspot.com/2014/03/top-downbottom-up.html)
    
    Mar. 12 08:40~10:10 Top-down（由上而下）和Bottom-up（由下而上）是兩種設計與解決問題的技巧。前者對問題先有一個整體的概念，然後再逐步加上設計細節，最後讓整體的輪廓越來越清楚。後者則是先將解決問題可能所需的基本元件、方案給準備好，然後再將這...
    
-   [羅技Logitech R500 雷射簡報筆開箱](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html)
    
    June 22 12:51~14:16 需求分析 因為工作的關係Teddy經常需要使用簡報筆，雜牌的不算，光是羅技的簡報筆Teddy就買過以下四種（R500是最新上市產品，也是今天介紹的主角）  ： ▼畫面節錄自羅技官網 Teddy心中理想的簡報筆應該有以下幾個功能： 重量輕且...
    

## 分析

 

技術提供：[Blogger](https://www.blogger.com).
```
```

```
Tool: webfetch

**Input:**
```json
{
  "format": "markdown",
  "url": "https://teddy-chen-tw.blogspot.com/2018/06/clean-architecture.html"
}
```
**Output:**
```
                  搞笑談軟工: Clean Architecture（3）：精選參考範例   

 

 

l

# [搞笑談軟工](https://teddy-chen-tw.blogspot.com/)

敏捷開發，設計模式，精實開發，Scrum，軟體設計，軟體架構

## 網頁

-   [首頁](https://teddy-chen-tw.blogspot.com/)
-   [泰迪軟體最新課表](http://teddysoft.tw/courses/)
-   [泰迪軟體Facebook](https://www.facebook.com/teddysoft.tw)
-   [搞笑談軟工Facebook社團](https://www.facebook.com/groups/teddy.tw/)

## 2018年6月22日 星期五

  

### Clean Architecture（3）：精選參考範例

June 22 10:01~11:14

[![螢幕截圖 2018-06-22 11.08.47](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhwyEIvNzo8wQxXdR_1eUAe9_4znZpUWIokIwRxoUCCm0RLqRcpXyN_ufwzgnZSYqa0iVmoVHCm3JTCxap7VKOI9Pg_54zTwhQa_MXWO-cXlOwmijWNt8jPuu6OT_ZxGnUyqAkFNHg8rmum/?imgmax=800 "螢幕截圖 2018-06-22 11.08.47")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjKOfbAgcTU_TSHbHHODL4m2h3WeM6BzWKZh7cJUXfsh8JOhNAc59cnfUnuRhpch1LoU3ReGvS6wBXkU1fKiMq7kp_Dse5Ft7AABK2VXSgN9hOrx0G3wbgB3q8zUGVjtu-RvF2_ZUPl31gj/s1600-h/Image)

這學期Teddy在北科大資工所兼任軟體架構課程，轉眼下禮拜就要學期結束。Teddy在網路上找了幾個外國鄉民實作的Clean Architecture程式範例，請學生當作期末報告。聽完學生報告，今天選了四個相對簡單易懂又可以反應Clean Architecture精神的範例推薦給鄉民們參考。

\*\*\*

**範例介紹**

**範例1：Real Life Clean Architecture**

-   參考網址：[https://goo.gl/nZgbTT](https://goo.gl/nZgbTT)
-   程式碼：[https://goo.gl/BdPngq](https://goo.gl/BdPngq)
-   語言：Java
-   範例用途：取得網路設備相關資訊。

**範例2：A complete idiot’s guide to Clean**

-   參考網址： [https://goo.gl/Be8RGN](https://goo.gl/Be8RGN)
-   程式碼：[https://goo.gl/EPkjhD](https://goo.gl/EPkjhD)
-   語言：Android/Java
-   範例用途：餐廳訂位App，可顯示地區清單，從選定的地區取得餐廳資料並預訂座位。

**範例3：A Clean Architecture in .Net**

-   參考網址：[https://goo.gl/W6Tp4F](https://goo.gl/W6Tp4F)
-   程式碼：[https://goo.gl/9KWdVu](https://goo.gl/9KWdVu)
-   語言：C#
-   範例用途：線上房地產平台，請客戶留下資料讓房地產代理人可聯繫有興趣的客戶。

**範例4：Architecting Android...The clean way?**

-   參考網址：[https://goo.gl/VwuQC1](https://goo.gl/VwuQC1)
-   程式碼：[https://goo.gl/M8Qike](https://goo.gl/M8Qike)
-   語言：Android/Java
-   範例用途：顯示使用者與其朋友列表，並可查看他們的詳細資料。

\*\*\*

**學習重點**

在觀看範例程式時，可留意以下幾個重點：

-   專案結構：一個專案還是多個專案？如果是多個專案，分了那些專案，為什麼？專案之間的相依性為何？
    
-   有哪些Entity物件？
    
-   如何實作Use Case雙向介面？Use Case的Input與Output介面以及資料結構如何定義？由誰實作？
    
-   Use Case是否採用Command方式實作？
    
-   如何處理物件跨層的問題？
    
-   Interface Adapter這層有哪些物件？如何跟 Use Case溝通？
    
-   是否如《[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_sc_1?s=books&ie=UTF8&qid=1510580792&sr=1-1-spell&keywords=cClean+Architecture%3A+A+Craftsman%E2%80%99s+Guide+to+Software+Structure+and+Design)》書中所示套用MVP？
    
-   是否有使用View Model？
    
-   範例是否遵循dependency rule？
    
-   是否有外層(low level)物件跨層參考內層(high level)的物件？例如 controller直接參考entity。
    
-   如何實作Repository？
    
-   如何實作View？
    

\*\*\*

**結論**

《[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_sc_1?s=books&ie=UTF8&qid=1510580792&sr=1-1-spell&keywords=cClean+Architecture%3A+A+Craftsman%E2%80%99s+Guide+to+Software+Structure+and+Design)》這本書讀起來不難，但有很多實作細節需要注意，如果自己不動手做過一次不容易發現箇中奇妙之處。一邊自己動手練習，同時參考別人的範例，學習效果可收加倍之效。

\*\*\*

友藏內心獨白：實作方式有很多種。

張貼者： [Teddy Chen](https://draft.blogger.com/profile/02066842119056439711 "author profile") 於  [上午11:15](https://teddy-chen-tw.blogspot.com/2018/06/clean-architecture.html "permanent link")  [![](//img1.blogblog.com/img/icon18_email.gif)](https://draft.blogger.com/email-post/1298974142445162186/5433529944263822736 "電子郵件發布")[![](https://resources.blogblog.com/img/icon18_edit_allbkg.gif)](https://draft.blogger.com/post-edit.g?blogID=1298974142445162186&postID=5433529944263822736&from=pencil "編輯文章")

[以電子郵件傳送這篇文章](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=5433529944263822736&target=email "以電子郵件傳送這篇文章")[BlogThis！](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=5433529944263822736&target=blog "BlogThis！")[分享至 X](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=5433529944263822736&target=twitter "分享至 X")[分享至 Facebook](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=5433529944263822736&target=facebook "分享至 Facebook")[分享到 Pinterest](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=5433529944263822736&target=pinterest "分享到 Pinterest")

標籤： [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B), [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88)

#### 沒有留言:

#### 張貼留言

[](https://draft.blogger.com/comment/frame/1298974142445162186?po=5433529944263822736&hl=zh-TW&saa=85391&origin=https://teddy-chen-tw.blogspot.com)

[較新的文章](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html "較新的文章") [較舊的文章](https://teddy-chen-tw.blogspot.com/2018/06/macbook-pro-15-late-20162innergiepowerg.html "較舊的文章") [首頁](https://teddy-chen-tw.blogspot.com/)

訂閱： [張貼留言 (Atom)](https://teddy-chen-tw.blogspot.com/feeds/5433529944263822736/comments/default)

## 搜尋此網站

 

## 總網頁瀏覽量

## 關於

[![我的相片](//blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBBqKlknBrSZ3EEXiWqeZasJOxrwoUfdqqF4Es3OR0YFtwDAYNHp6bBx3PqLGVJXAJQ4INhfp0gUlRZyRoSF9vzraU_JlLOECrERAWag3VIa0JdxRhpLwyEz7w9QtX8w/s220/Teddy.jpg)](https://draft.blogger.com/profile/02066842119056439711)

[Teddy Chen](https://draft.blogger.com/profile/02066842119056439711)

[檢視我的完整簡介](https://draft.blogger.com/profile/02066842119056439711)

## 請幫Teddy按個讚^\_^

## 標籤

-   [工商服務](https://teddy-chen-tw.blogspot.com/search/label/%E5%B7%A5%E5%95%86%E6%9C%8D%E5%8B%99) (94)
-   [生活](https://teddy-chen-tw.blogspot.com/search/label/%E7%94%9F%E6%B4%BB) (342)
-   [改行寫網路小說算了](https://teddy-chen-tw.blogspot.com/search/label/%E6%94%B9%E8%A1%8C%E5%AF%AB%E7%B6%B2%E8%B7%AF%E5%B0%8F%E8%AA%AA%E7%AE%97%E4%BA%86) (25)
-   [其他](https://teddy-chen-tw.blogspot.com/search/label/%E5%85%B6%E4%BB%96) (25)
-   [客戶教我的事](https://teddy-chen-tw.blogspot.com/search/label/%E5%AE%A2%E6%88%B6%E6%95%99%E6%88%91%E7%9A%84%E4%BA%8B) (2)
-   [持續整合](https://teddy-chen-tw.blogspot.com/search/label/%E6%8C%81%E7%BA%8C%E6%95%B4%E5%90%88) (29)
-   [哲學](https://teddy-chen-tw.blogspot.com/search/label/%E5%93%B2%E5%AD%B8) (13)
-   [旅遊](https://teddy-chen-tw.blogspot.com/search/label/%E6%97%85%E9%81%8A) (568)
-   [視力測驗](https://teddy-chen-tw.blogspot.com/search/label/%E8%A6%96%E5%8A%9B%E6%B8%AC%E9%A9%97) (3)
-   [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5) (410)
-   [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B) (110)
-   [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88) (188)
-   [最新課程](https://teddy-chen-tw.blogspot.com/search/label/%E6%9C%80%E6%96%B0%E8%AA%B2%E7%A8%8B) (81)
-   [創業](https://teddy-chen-tw.blogspot.com/search/label/%E5%89%B5%E6%A5%AD) (84)
-   [測試](https://teddy-chen-tw.blogspot.com/search/label/%E6%B8%AC%E8%A9%A6) (106)
-   [盡信書不如無書](https://teddy-chen-tw.blogspot.com/search/label/%E7%9B%A1%E4%BF%A1%E6%9B%B8%E4%B8%8D%E5%A6%82%E7%84%A1%E6%9B%B8) (2)
-   [需求](https://teddy-chen-tw.blogspot.com/search/label/%E9%9C%80%E6%B1%82) (23)
-   [課程實錄](https://teddy-chen-tw.blogspot.com/search/label/%E8%AA%B2%E7%A8%8B%E5%AF%A6%E9%8C%84) (22)
-   [學習](https://teddy-chen-tw.blogspot.com/search/label/%E5%AD%B8%E7%BF%92) (92)
-   [貓](https://teddy-chen-tw.blogspot.com/search/label/%E8%B2%93) (15)
-   [還少一本書](https://teddy-chen-tw.blogspot.com/search/label/%E9%82%84%E5%B0%91%E4%B8%80%E6%9C%AC%E6%9B%B8) (36)
-   [agile](https://teddy-chen-tw.blogspot.com/search/label/agile) (638)
-   [C. C. Agile](https://teddy-chen-tw.blogspot.com/search/label/C.%20C.%20Agile) (33)
-   [DDD](https://teddy-chen-tw.blogspot.com/search/label/DDD) (85)
-   [DevOps](https://teddy-chen-tw.blogspot.com/search/label/DevOps) (3)
-   [Eclipse](https://teddy-chen-tw.blogspot.com/search/label/Eclipse) (6)
-   [exception handling](https://teddy-chen-tw.blogspot.com/search/label/exception%20handling) (84)
-   [HCI](https://teddy-chen-tw.blogspot.com/search/label/HCI) (44)
-   [Implementation Patterns](https://teddy-chen-tw.blogspot.com/search/label/Implementation%20Patterns) (5)
-   [Kanban](https://teddy-chen-tw.blogspot.com/search/label/Kanban) (79)
-   [Mobile](https://teddy-chen-tw.blogspot.com/search/label/Mobile) (2)
-   [Patterns](https://teddy-chen-tw.blogspot.com/search/label/Patterns) (198)
-   [Programming](https://teddy-chen-tw.blogspot.com/search/label/Programming) (54)
-   [Refactoring](https://teddy-chen-tw.blogspot.com/search/label/Refactoring) (65)
-   [Scrum](https://teddy-chen-tw.blogspot.com/search/label/Scrum) (461)
-   [TDD/BDD](https://teddy-chen-tw.blogspot.com/search/label/TDD%2FBDD) (54)

## 網誌存檔

-   [►](javascript:void\(0\))  [2025](https://teddy-chen-tw.blogspot.com/2025/) (1)
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2025/06/) (1)

-   [►](javascript:void\(0\))  [2024](https://teddy-chen-tw.blogspot.com/2024/) (7)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2024/09/) (6)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2024/03/) (1)

-   [►](javascript:void\(0\))  [2023](https://teddy-chen-tw.blogspot.com/2023/) (25)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2023/09/) (5)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2023/07/) (3)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2023/06/) (5)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2023/03/) (10)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2023/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2023/01/) (1)

-   [►](javascript:void\(0\))  [2022](https://teddy-chen-tw.blogspot.com/2022/) (29)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2022/10/) (1)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2022/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2022/07/) (18)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2022/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2022/03/) (2)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2022/02/) (1)

-   [►](javascript:void\(0\))  [2021](https://teddy-chen-tw.blogspot.com/2021/) (27)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2021/12/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2021/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2021/09/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2021/08/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2021/06/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2021/04/) (1)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2021/03/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2021/01/) (8)

-   [►](javascript:void\(0\))  [2020](https://teddy-chen-tw.blogspot.com/2020/) (32)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2020/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2020/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2020/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2020/09/) (4)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2020/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2020/07/) (2)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2020/06/) (2)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2020/05/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2020/04/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2020/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2020/02/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2020/01/) (1)

-   [►](javascript:void\(0\))  [2019](https://teddy-chen-tw.blogspot.com/2019/) (50)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2019/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2019/11/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2019/10/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2019/07/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2019/06/) (1)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2019/05/) (8)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2019/04/) (2)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2019/03/) (8)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2019/02/) (8)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2019/01/) (7)

-   [▼](javascript:void\(0\))  [2018](https://teddy-chen-tw.blogspot.com/2018/) (28)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2018/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2018/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2018/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2018/08/) (6)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2018/07/) (6)
    
    -   [▼](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2018/06/) (3)
        -   [羅技Logitech R500 雷射簡報筆開箱](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html)
        -   [Clean Architecture（3）：精選參考範例](https://teddy-chen-tw.blogspot.com/2018/06/clean-architecture.html)
        -   [幫Macbook Pro 15” Late 2016尋找外出電源轉接器（2）：InnergiePow...](https://teddy-chen-tw.blogspot.com/2018/06/macbook-pro-15-late-20162innergiepowerg.html)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2018/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2018/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2018/01/) (3)

-   [►](javascript:void\(0\))  [2017](https://teddy-chen-tw.blogspot.com/2017/) (142)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2017/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2017/11/) (8)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2017/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2017/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2017/08/) (15)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2017/07/) (10)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2017/06/) (14)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2017/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2017/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2017/03/) (23)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2017/02/) (16)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2017/01/) (25)

-   [►](javascript:void\(0\))  [2016](https://teddy-chen-tw.blogspot.com/2016/) (301)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2016/12/) (14)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2016/11/) (19)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2016/10/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2016/09/) (18)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2016/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2016/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2016/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2016/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2016/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2016/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2016/02/) (29)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2016/01/) (31)

-   [►](javascript:void\(0\))  [2015](https://teddy-chen-tw.blogspot.com/2015/) (366)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2015/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2015/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2015/10/) (32)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2015/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2015/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2015/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2015/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2015/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2015/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2015/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2015/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2015/01/) (31)

-   [►](javascript:void\(0\))  [2014](https://teddy-chen-tw.blogspot.com/2014/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2014/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2014/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2014/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2014/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2014/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2014/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2014/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2014/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2014/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2014/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2014/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2014/01/) (31)

-   [►](javascript:void\(0\))  [2013](https://teddy-chen-tw.blogspot.com/2013/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2013/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2013/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2013/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2013/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2013/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2013/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2013/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2013/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2013/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2013/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2013/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2013/01/) (31)

-   [►](javascript:void\(0\))  [2012](https://teddy-chen-tw.blogspot.com/2012/) (373)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2012/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2012/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2012/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2012/09/) (31)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2012/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2012/07/) (32)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2012/06/) (31)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2012/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2012/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2012/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2012/02/) (32)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2012/01/) (32)

-   [►](javascript:void\(0\))  [2011](https://teddy-chen-tw.blogspot.com/2011/) (120)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2011/12/) (16)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2011/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2011/10/) (4)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2011/09/) (3)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2011/08/) (9)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2011/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2011/06/) (10)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2011/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2011/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2011/03/) (16)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2011/02/) (12)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2011/01/) (11)

-   [►](javascript:void\(0\))  [2010](https://teddy-chen-tw.blogspot.com/2010/) (104)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2010/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2010/11/) (6)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2010/10/) (10)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2010/09/) (5)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2010/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2010/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2010/06/) (19)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2010/05/) (12)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2010/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2010/03/) (12)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2010/02/) (2)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2010/01/) (7)

-   [►](javascript:void\(0\))  [2009](https://teddy-chen-tw.blogspot.com/2009/) (31)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2009/12/) (5)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2009/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2009/10/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2009/08/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2009/07/) (8)

-   [►](javascript:void\(0\))  [2008](https://teddy-chen-tw.blogspot.com/2008/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2008/09/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2008/03/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2008/01/) (2)

-   [►](javascript:void\(0\))  [2007](https://teddy-chen-tw.blogspot.com/2007/) (4)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2007/12/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2007/06/) (3)

## 追蹤者

## 最新回應

[載入中…](http://teddy-chen-tw.blogspot.tw/feeds/comments/default)

## 熱門文章

-   [爭什麼，摻在一起做成撒尿牛丸啊](https://teddy-chen-tw.blogspot.com/2012/03/blog-post_20.html)
    
    March 20 13:53~14:56 (影像節錄自http://www.youtube.com/watch?v=LGV5PcM9SQo) 鄉民們都看過食神這部電影吧，電影中有一個經典的橋段，就是火鷄姐及鵝頭在爭辯誰做的撒尿蝦好吃，誰做的牛丸比較優。正當雙方吵得不可...
    
-   [擁抱改變，遠離Hard Code](https://teddy-chen-tw.blogspot.com/2018/09/hard-code.html)
    
    September 19 08:37~09:12 ▲圖片來源 在此 寫死（Hard Code） 幾天前一位朋友在Facebook上發問：「如何對非資訊人員解釋寫死（hard code）？」 維基百科 對於寫死的解釋如下： 寫死 （英語： Hard Code 或 Hard Co...
    
-   [什麼是物件導向（3）：Polymorphism](https://teddy-chen-tw.blogspot.com/2012/01/3polymorphism.html)
    
    January 26 23:01~January 27 00:23 Polymorphism，中文翻譯成「多型」，說真的這個概念當年Teddy學OO的時候搞了好久才弄懂，後來看了「課本」的定義之後，覺得寫得還滿好的，請參考Object-Oriented Software En...
    
-   [幫Macbook Pro 15” Late 2016尋找外接4K 27”螢幕](https://teddy-chen-tw.blogspot.com/2017/07/macbook-pro-15-late-20164k-27.html)
    
    June 17 21:28~23:24 先打個廣告，【 敏捷開發懶人包：物件導向技能 】確定於8月19日（週六）開課。 課程設定目標希望讓鄉民們在找工作面試的時候，如果面試官問到物件導向技術方面的問題，能夠具備輕鬆回答的能力 。 \*\*\* 動心起念 去年底買了MacBook P...
    
-   [Top-down和Bottom-up設計方法](https://teddy-chen-tw.blogspot.com/2014/03/top-downbottom-up.html)
    
    Mar. 12 08:40~10:10 Top-down（由上而下）和Bottom-up（由下而上）是兩種設計與解決問題的技巧。前者對問題先有一個整體的概念，然後再逐步加上設計細節，最後讓整體的輪廓越來越清楚。後者則是先將解決問題可能所需的基本元件、方案給準備好，然後再將這...
    
-   [羅技Logitech R500 雷射簡報筆開箱](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html)
    
    June 22 12:51~14:16 需求分析 因為工作的關係Teddy經常需要使用簡報筆，雜牌的不算，光是羅技的簡報筆Teddy就買過以下四種（R500是最新上市產品，也是今天介紹的主角）  ： ▼畫面節錄自羅技官網 Teddy心中理想的簡報筆應該有以下幾個功能： 重量輕且...
    

## 分析

 

技術提供：[Blogger](https://draft.blogger.com).
```
```

```
Tool: webfetch

**Input:**
```json
{
  "format": "markdown",
  "url": "https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html"
}
```
**Output:**
```
                  搞笑談軟工: Clean Architecture（4）：架構三原則首部曲—分層原則   

 

 

l

# [搞笑談軟工](https://teddy-chen-tw.blogspot.com/)

敏捷開發，設計模式，精實開發，Scrum，軟體設計，軟體架構

## 網頁

-   [首頁](https://teddy-chen-tw.blogspot.com/)
-   [泰迪軟體最新課表](http://teddysoft.tw/courses/)
-   [泰迪軟體Facebook](https://www.facebook.com/teddysoft.tw)
-   [搞笑談軟工Facebook社團](https://www.facebook.com/groups/teddy.tw/)

## 2018年7月4日 星期三

  

### Clean Architecture（4）：架構三原則首部曲—分層原則

July 3 14:38~15:58

[![螢幕截圖 2018-07-03 16.08.32](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh2XbZGWpXo7Cvbz5h1qo6JjBaqMLu6OEhJLh2dwndlWmlUQHtMOfvREa70rhVi_aio0_PXyW6Qb-ndO_J6RYq43N6Eud50rzrxhuEDdkSU9d05E6dW9pk-GUgUvEn97B-ax06oH72RWXLU/?imgmax=800 "螢幕截圖 2018-07-03 16.08.32")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgpQ3jGtkyOQnwJFr3aLQuS1svaZlA9d7cYQ0bPmzTMnvkYfbb0roKIEgRyQwcywc8w3VKjr3eVtIKbZVbCcAS_3gCJObIFnvUzptr-KkQEiIL_s4IK9PMBP_drShu8PnsjkDr9pKLZ7arL/s1600-h/Image)

  

**前言**

《Clean Architecture》這本書一共有34章，全部讀完也是有點累。今天Teddy幫鄉民們整理一下重點，總地來講Teddy認為Clean Architecture有三個最重要的原則，將分三集逐一介紹這三個原則。今天先談「**分層原則**」。

\*\*\*

**階層式架構**

軟體架構就是軟體的形狀（shape），而形狀是由多個元件及其互動所構成。這些元件通常會依據其功能加以分類，以便於管理、擴充與維護。

  

▼**階層式架構**就是一種很常見的分類方法，常見的階層式架構長成下面這樣 （詳細說明可參考[https://goo.gl/Tjq75M](https://goo.gl/Tjq75M)）：

[![螢幕截圖 2018-07-03 14.55.57](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhj4UUqI-2Tvcx0lDA6b9i6F8CPvkPYW1jkd7eZlxAEL0OSo5nSKx6IsbvKTqRt0yo0kH6Gwy2V3PJJc5OT7KrhHsmMxJNLkpta0E7wdkG1XCwwIedQXcK_8CSIer5z6VByyR-X01dm1cHk/?imgmax=800 "螢幕截圖 2018-07-03 14.55.57")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjdajEjZiyHhLJXE9pgI8P6zflKuGyxJBSmH3BVIJabiBz6jg8Yj2EP3Cej_1w8kO8bpBXF6nXiZ_66oG3KSKj1VE7pSJNELSi34HkVMQoCaJfbEChyphenhyphenC1QDaC7_KTkSrTkgegtWF5lE5ImF/s1600-h/Image)

  

既然稱為階層式架構，一定有上、下層之分。**請問上圖階層式架構中，誰是上層、誰是下層？**

一般直覺的看法：Presentation Layer畫在架構的最上方，所以它是最上層，而Database Layer在最下方，所以它是最下層。**這種解讀有一個問題，因為在階層式架構中，越上層的人（元件）通常擁有越核心的商業邏輯（business logic）。如果Presentation Layer是最上層，那豈不是代表它擁有最核心的商業邏輯？**這也不對啊，因為大家都知道，儘量不要在Presentation Layer放置商業邏輯，因為那將使系統擴充、維護與測試變得更加困難。

\*\*\*

**定義高層與低層**

《Clean Architecture》書中關於分層有一個非常明確的定義：**「離I/O（輸入、輸出）越遠的元件層級越高，離I/O越近的層級越低」**。這其實也很容易理解，在公司中，櫃台小姐離I/O（接電話、接待訪客）最近，通常層級比較低。董事長、總經理離I/O很遠，層級最高（通常看高階主管的辦公室位置離大門遠近就可猜出層級高低XD）。

▼Port and Adapter不把階層式架構畫成常見的垂直形狀，而是畫成圖1中的六角型。

[![螢幕截圖 2018-03-15 17.39.22](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgqerqFadRkFjpuJMoHrwIVopXwhqk7IqL_qNpbaOqRte_7k_gvQf4UGb19LCqgm8nIFfIKUKjWKkNV0GlBpqCfYIlktIViG8FEWBGHiLD7v1wWvUaW0wf2FTtRQSP_vE7w1PwijYKKWKHk/?imgmax=800 "螢幕截圖 2018-03-15 17.39.22")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjZHXYqCEUWIrqDQqugpboTcyJBVXDboYcpW_gwtO55rl7XszWtj8tDsQIqIlAeJmr2KD5GLXhmHjPPfFNAf2Bk5H6VLINcVidXgI1JsVh80DZ8qAoTpiZXYpDumcvquWwgUsBy5PGtfnGD/s1600-h/Image)

  

▼《Clean Architecture》書中則是畫成同心圓（圖片來源[在此](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)）。

[![CleanArchitecture](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiofyVn5BrjrUx64_FhXuCslg8jX7HZiX5Z_MW-0QvcYw-qS_h0XUIY-iNXk2yfV1LNPnumnP22_6zwdGhOMnllqbbMwMiXJJVVtrC2f2FLirNXBppQPGySWqwuPg77xG1lNwMQ6NwyKbvF/?imgmax=800 "CleanArchitecture")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgHOugVUfASncB4KxEx7TI050IHQP4FCrNY1Ghx0bL5KS_5orpSd4ATCjoaW0016KCRhAbg4GCyN2pJdvKZ0c4GkVh5cDGazHG8VaMD3Wr6uc3Ak_OZfikBja_MjQcQNSpucYDX1RUsBd52/s1600-h/CleanArchitecture%255B4%255D)

  

▲上圖中，蛋黃區（Entities）是架構中層級最高的一層，而最外圈屬於I/O層，是層級最低的。

《Clean Architecture》書中有四個預設的階層，分別是：

-   **Entities**：也就是傳統物件導向分析與設計所說的domain model object。
-   **Use Cases**：Entities這一層存放著**核心商業邏輯**，也就是在這個領域中，不同應用程式都用得到的物件。而Use Cases則代表應用程式邏輯，也就是應用程式的功能。Use Cases扮演著controller的角色，呼叫Entities或是Repository物件提供應用程式對外的服務。
-   **Interface Adapters**：將外部資料與呼叫介面透過此層轉呼叫Use Cases，如此一來Use Cases就可以與I/O或是應用程式框架無關。
-   **Frameworks and Drivers**：此層包含了應用程式框架，例如如果Java程式使用了Spring Framework，則Spring Framework就位於這一層。資料庫，或常見的MVC Framework也都位於這一層。通常大家在這一層所寫的程式都只是為了把應用程式框架與內部的Interface Adapters或Use Cases串起來的膠水程式，鮮少有複雜的商業邏輯會位於這一層。

\*\*\*

**結論**

依據「元件距離I/O遠近」分好層級之後，高、低層之間的界線就非常的清楚。層級分明之後，接下來衍生出另一個問題：**每一層之間的相依性要怎麼管理？**如果層與層之間的相依性沒有好好管理，僅僅是分層你的架構還是很可能會一團亂。

下一集「相依性原則」將說明這個問題，下次見。

\*\*\*

友藏內心獨白：真正的高層都是坐在離I/O很近的大門口XD。

張貼者： [Teddy Chen](https://www.blogger.com/profile/02066842119056439711 "author profile") 於  [凌晨12:00](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html "permanent link")  [![](//img1.blogblog.com/img/icon18_email.gif)](https://www.blogger.com/email-post/1298974142445162186/3653120718563553641 "電子郵件發布")[![](https://resources.blogblog.com/img/icon18_edit_allbkg.gif)](https://www.blogger.com/post-edit.g?blogID=1298974142445162186&postID=3653120718563553641&from=pencil "編輯文章")

[以電子郵件傳送這篇文章](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=3653120718563553641&target=email "以電子郵件傳送這篇文章")[BlogThis！](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=3653120718563553641&target=blog "BlogThis！")[分享至 X](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=3653120718563553641&target=twitter "分享至 X")[分享至 Facebook](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=3653120718563553641&target=facebook "分享至 Facebook")[分享到 Pinterest](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=3653120718563553641&target=pinterest "分享到 Pinterest")

標籤： [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5), [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B), [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88)

#### 2 則留言:

1.  ![](//www.blogger.com/img/blogger_logo_round_35.png)
    
    [Unknown](https://www.blogger.com/profile/17241260491348584074)[2018年9月3日下午4:10](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html?showComment=1535962211960#c7448628713071845572)
    
    “真正的高層都是坐在離I/O很近的大門口”--》 所以老闆娘只要顧在櫃檯收錢
    
    回覆[刪除](https://www.blogger.com/comment/delete/1298974142445162186/7448628713071845572)
    
    回覆
    
    回覆
    
2.  ![](//www.blogger.com/img/blogger_logo_round_35.png)
    
    [Piece](https://www.blogger.com/profile/04042500623696208344)[2024年2月22日晚上8:58](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html?showComment=1708606692426#c3425767132027646686)
    
    Persistence Layer
    
    回覆[刪除](https://www.blogger.com/comment/delete/1298974142445162186/3425767132027646686)
    
    回覆
    
    回覆
    

新增留言

載入更多…

[](https://www.blogger.com/comment/frame/1298974142445162186?po=3653120718563553641&hl=zh-TW&saa=85391&origin=https://teddy-chen-tw.blogspot.com)

[較新的文章](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html "較新的文章") [較舊的文章](https://teddy-chen-tw.blogspot.com/2018/07/lg-gram-15z980-raas9u1-2018-amazon_3.html "較舊的文章") [首頁](https://teddy-chen-tw.blogspot.com/)

訂閱： [張貼留言 (Atom)](https://teddy-chen-tw.blogspot.com/feeds/3653120718563553641/comments/default)

## 搜尋此網站

 

## 總網頁瀏覽量

## 關於

[![我的相片](//blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBBqKlknBrSZ3EEXiWqeZasJOxrwoUfdqqF4Es3OR0YFtwDAYNHp6bBx3PqLGVJXAJQ4INhfp0gUlRZyRoSF9vzraU_JlLOECrERAWag3VIa0JdxRhpLwyEz7w9QtX8w/s220/Teddy.jpg)](https://www.blogger.com/profile/02066842119056439711)

[Teddy Chen](https://www.blogger.com/profile/02066842119056439711)

[檢視我的完整簡介](https://www.blogger.com/profile/02066842119056439711)

## 請幫Teddy按個讚^\_^

## 標籤

-   [工商服務](https://teddy-chen-tw.blogspot.com/search/label/%E5%B7%A5%E5%95%86%E6%9C%8D%E5%8B%99) (94)
-   [生活](https://teddy-chen-tw.blogspot.com/search/label/%E7%94%9F%E6%B4%BB) (342)
-   [改行寫網路小說算了](https://teddy-chen-tw.blogspot.com/search/label/%E6%94%B9%E8%A1%8C%E5%AF%AB%E7%B6%B2%E8%B7%AF%E5%B0%8F%E8%AA%AA%E7%AE%97%E4%BA%86) (25)
-   [其他](https://teddy-chen-tw.blogspot.com/search/label/%E5%85%B6%E4%BB%96) (25)
-   [客戶教我的事](https://teddy-chen-tw.blogspot.com/search/label/%E5%AE%A2%E6%88%B6%E6%95%99%E6%88%91%E7%9A%84%E4%BA%8B) (2)
-   [持續整合](https://teddy-chen-tw.blogspot.com/search/label/%E6%8C%81%E7%BA%8C%E6%95%B4%E5%90%88) (29)
-   [哲學](https://teddy-chen-tw.blogspot.com/search/label/%E5%93%B2%E5%AD%B8) (13)
-   [旅遊](https://teddy-chen-tw.blogspot.com/search/label/%E6%97%85%E9%81%8A) (568)
-   [視力測驗](https://teddy-chen-tw.blogspot.com/search/label/%E8%A6%96%E5%8A%9B%E6%B8%AC%E9%A9%97) (3)
-   [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5) (410)
-   [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B) (110)
-   [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88) (188)
-   [最新課程](https://teddy-chen-tw.blogspot.com/search/label/%E6%9C%80%E6%96%B0%E8%AA%B2%E7%A8%8B) (81)
-   [創業](https://teddy-chen-tw.blogspot.com/search/label/%E5%89%B5%E6%A5%AD) (84)
-   [測試](https://teddy-chen-tw.blogspot.com/search/label/%E6%B8%AC%E8%A9%A6) (106)
-   [盡信書不如無書](https://teddy-chen-tw.blogspot.com/search/label/%E7%9B%A1%E4%BF%A1%E6%9B%B8%E4%B8%8D%E5%A6%82%E7%84%A1%E6%9B%B8) (2)
-   [需求](https://teddy-chen-tw.blogspot.com/search/label/%E9%9C%80%E6%B1%82) (23)
-   [課程實錄](https://teddy-chen-tw.blogspot.com/search/label/%E8%AA%B2%E7%A8%8B%E5%AF%A6%E9%8C%84) (22)
-   [學習](https://teddy-chen-tw.blogspot.com/search/label/%E5%AD%B8%E7%BF%92) (92)
-   [貓](https://teddy-chen-tw.blogspot.com/search/label/%E8%B2%93) (15)
-   [還少一本書](https://teddy-chen-tw.blogspot.com/search/label/%E9%82%84%E5%B0%91%E4%B8%80%E6%9C%AC%E6%9B%B8) (36)
-   [agile](https://teddy-chen-tw.blogspot.com/search/label/agile) (638)
-   [C. C. Agile](https://teddy-chen-tw.blogspot.com/search/label/C.%20C.%20Agile) (33)
-   [DDD](https://teddy-chen-tw.blogspot.com/search/label/DDD) (85)
-   [DevOps](https://teddy-chen-tw.blogspot.com/search/label/DevOps) (3)
-   [Eclipse](https://teddy-chen-tw.blogspot.com/search/label/Eclipse) (6)
-   [exception handling](https://teddy-chen-tw.blogspot.com/search/label/exception%20handling) (84)
-   [HCI](https://teddy-chen-tw.blogspot.com/search/label/HCI) (44)
-   [Implementation Patterns](https://teddy-chen-tw.blogspot.com/search/label/Implementation%20Patterns) (5)
-   [Kanban](https://teddy-chen-tw.blogspot.com/search/label/Kanban) (79)
-   [Mobile](https://teddy-chen-tw.blogspot.com/search/label/Mobile) (2)
-   [Patterns](https://teddy-chen-tw.blogspot.com/search/label/Patterns) (198)
-   [Programming](https://teddy-chen-tw.blogspot.com/search/label/Programming) (54)
-   [Refactoring](https://teddy-chen-tw.blogspot.com/search/label/Refactoring) (65)
-   [Scrum](https://teddy-chen-tw.blogspot.com/search/label/Scrum) (461)
-   [TDD/BDD](https://teddy-chen-tw.blogspot.com/search/label/TDD%2FBDD) (54)

## 網誌存檔

-   [►](javascript:void\(0\))  [2025](https://teddy-chen-tw.blogspot.com/2025/) (1)
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2025/06/) (1)

-   [►](javascript:void\(0\))  [2024](https://teddy-chen-tw.blogspot.com/2024/) (7)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2024/09/) (6)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2024/03/) (1)

-   [►](javascript:void\(0\))  [2023](https://teddy-chen-tw.blogspot.com/2023/) (25)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2023/09/) (5)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2023/07/) (3)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2023/06/) (5)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2023/03/) (10)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2023/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2023/01/) (1)

-   [►](javascript:void\(0\))  [2022](https://teddy-chen-tw.blogspot.com/2022/) (29)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2022/10/) (1)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2022/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2022/07/) (18)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2022/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2022/03/) (2)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2022/02/) (1)

-   [►](javascript:void\(0\))  [2021](https://teddy-chen-tw.blogspot.com/2021/) (27)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2021/12/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2021/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2021/09/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2021/08/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2021/06/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2021/04/) (1)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2021/03/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2021/01/) (8)

-   [►](javascript:void\(0\))  [2020](https://teddy-chen-tw.blogspot.com/2020/) (32)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2020/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2020/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2020/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2020/09/) (4)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2020/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2020/07/) (2)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2020/06/) (2)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2020/05/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2020/04/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2020/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2020/02/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2020/01/) (1)

-   [►](javascript:void\(0\))  [2019](https://teddy-chen-tw.blogspot.com/2019/) (50)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2019/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2019/11/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2019/10/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2019/07/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2019/06/) (1)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2019/05/) (8)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2019/04/) (2)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2019/03/) (8)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2019/02/) (8)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2019/01/) (7)

-   [▼](javascript:void\(0\))  [2018](https://teddy-chen-tw.blogspot.com/2018/) (28)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2018/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2018/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2018/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2018/08/) (6)
    
    -   [▼](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2018/07/) (6)
        -   [阿嬤覺得你冷](https://teddy-chen-tw.blogspot.com/2018/07/blog-post.html)
        -   [Clean Architecture（6）：架構三原則三部曲—跨層原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture6.html)
        -   [Clean Architecture（5）：架構三原則二部曲—相依性原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html)
        -   [Clean Architecture（4）：架構三原則首部曲—分層原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html)
        -   [LG gram 15Z980-R.AAS9U1 (2018) Amazon 購買與使用心得（下）](https://teddy-chen-tw.blogspot.com/2018/07/lg-gram-15z980-raas9u1-2018-amazon_3.html)
        -   [LG gram 15Z980-R.AAS9U1 (2018) Amazon 購買與使用心得（上）](https://teddy-chen-tw.blogspot.com/2018/07/lg-gram-15z980-raas9u1-2018-amazon.html)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2018/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2018/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2018/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2018/01/) (3)

-   [►](javascript:void\(0\))  [2017](https://teddy-chen-tw.blogspot.com/2017/) (142)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2017/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2017/11/) (8)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2017/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2017/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2017/08/) (15)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2017/07/) (10)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2017/06/) (14)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2017/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2017/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2017/03/) (23)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2017/02/) (16)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2017/01/) (25)

-   [►](javascript:void\(0\))  [2016](https://teddy-chen-tw.blogspot.com/2016/) (301)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2016/12/) (14)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2016/11/) (19)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2016/10/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2016/09/) (18)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2016/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2016/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2016/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2016/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2016/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2016/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2016/02/) (29)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2016/01/) (31)

-   [►](javascript:void\(0\))  [2015](https://teddy-chen-tw.blogspot.com/2015/) (366)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2015/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2015/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2015/10/) (32)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2015/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2015/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2015/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2015/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2015/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2015/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2015/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2015/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2015/01/) (31)

-   [►](javascript:void\(0\))  [2014](https://teddy-chen-tw.blogspot.com/2014/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2014/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2014/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2014/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2014/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2014/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2014/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2014/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2014/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2014/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2014/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2014/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2014/01/) (31)

-   [►](javascript:void\(0\))  [2013](https://teddy-chen-tw.blogspot.com/2013/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2013/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2013/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2013/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2013/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2013/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2013/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2013/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2013/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2013/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2013/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2013/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2013/01/) (31)

-   [►](javascript:void\(0\))  [2012](https://teddy-chen-tw.blogspot.com/2012/) (373)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2012/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2012/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2012/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2012/09/) (31)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2012/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2012/07/) (32)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2012/06/) (31)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2012/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2012/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2012/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2012/02/) (32)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2012/01/) (32)

-   [►](javascript:void\(0\))  [2011](https://teddy-chen-tw.blogspot.com/2011/) (120)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2011/12/) (16)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2011/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2011/10/) (4)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2011/09/) (3)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2011/08/) (9)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2011/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2011/06/) (10)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2011/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2011/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2011/03/) (16)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2011/02/) (12)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2011/01/) (11)

-   [►](javascript:void\(0\))  [2010](https://teddy-chen-tw.blogspot.com/2010/) (104)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2010/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2010/11/) (6)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2010/10/) (10)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2010/09/) (5)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2010/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2010/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2010/06/) (19)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2010/05/) (12)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2010/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2010/03/) (12)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2010/02/) (2)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2010/01/) (7)

-   [►](javascript:void\(0\))  [2009](https://teddy-chen-tw.blogspot.com/2009/) (31)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2009/12/) (5)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2009/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2009/10/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2009/08/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2009/07/) (8)

-   [►](javascript:void\(0\))  [2008](https://teddy-chen-tw.blogspot.com/2008/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2008/09/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2008/03/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2008/01/) (2)

-   [►](javascript:void\(0\))  [2007](https://teddy-chen-tw.blogspot.com/2007/) (4)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2007/12/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2007/06/) (3)

## 追蹤者

## 最新回應

[載入中…](http://teddy-chen-tw.blogspot.tw/feeds/comments/default)

## 熱門文章

-   [爭什麼，摻在一起做成撒尿牛丸啊](https://teddy-chen-tw.blogspot.com/2012/03/blog-post_20.html)
    
    March 20 13:53~14:56 (影像節錄自http://www.youtube.com/watch?v=LGV5PcM9SQo) 鄉民們都看過食神這部電影吧，電影中有一個經典的橋段，就是火鷄姐及鵝頭在爭辯誰做的撒尿蝦好吃，誰做的牛丸比較優。正當雙方吵得不可...
    
-   [擁抱改變，遠離Hard Code](https://teddy-chen-tw.blogspot.com/2018/09/hard-code.html)
    
    September 19 08:37~09:12 ▲圖片來源 在此 寫死（Hard Code） 幾天前一位朋友在Facebook上發問：「如何對非資訊人員解釋寫死（hard code）？」 維基百科 對於寫死的解釋如下： 寫死 （英語： Hard Code 或 Hard Co...
    
-   [什麼是物件導向（3）：Polymorphism](https://teddy-chen-tw.blogspot.com/2012/01/3polymorphism.html)
    
    January 26 23:01~January 27 00:23 Polymorphism，中文翻譯成「多型」，說真的這個概念當年Teddy學OO的時候搞了好久才弄懂，後來看了「課本」的定義之後，覺得寫得還滿好的，請參考Object-Oriented Software En...
    
-   [幫Macbook Pro 15” Late 2016尋找外接4K 27”螢幕](https://teddy-chen-tw.blogspot.com/2017/07/macbook-pro-15-late-20164k-27.html)
    
    June 17 21:28~23:24 先打個廣告，【 敏捷開發懶人包：物件導向技能 】確定於8月19日（週六）開課。 課程設定目標希望讓鄉民們在找工作面試的時候，如果面試官問到物件導向技術方面的問題，能夠具備輕鬆回答的能力 。 \*\*\* 動心起念 去年底買了MacBook P...
    
-   [Top-down和Bottom-up設計方法](https://teddy-chen-tw.blogspot.com/2014/03/top-downbottom-up.html)
    
    Mar. 12 08:40~10:10 Top-down（由上而下）和Bottom-up（由下而上）是兩種設計與解決問題的技巧。前者對問題先有一個整體的概念，然後再逐步加上設計細節，最後讓整體的輪廓越來越清楚。後者則是先將解決問題可能所需的基本元件、方案給準備好，然後再將這...
    
-   [羅技Logitech R500 雷射簡報筆開箱](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html)
    
    June 22 12:51~14:16 需求分析 因為工作的關係Teddy經常需要使用簡報筆，雜牌的不算，光是羅技的簡報筆Teddy就買過以下四種（R500是最新上市產品，也是今天介紹的主角）  ： ▼畫面節錄自羅技官網 Teddy心中理想的簡報筆應該有以下幾個功能： 重量輕且...
    

## 分析

 

技術提供：[Blogger](https://www.blogger.com).
```
```

```
Tool: webfetch

**Input:**
```json
{
  "format": "markdown",
  "url": "https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html"
}
```
**Output:**
```
                  搞笑談軟工: Clean Architecture（5）：架構三原則二部曲—相依性原則   

 

 

l

# [搞笑談軟工](https://teddy-chen-tw.blogspot.com/)

敏捷開發，設計模式，精實開發，Scrum，軟體設計，軟體架構

## 網頁

-   [首頁](https://teddy-chen-tw.blogspot.com/)
-   [泰迪軟體最新課表](http://teddysoft.tw/courses/)
-   [泰迪軟體Facebook](https://www.facebook.com/teddysoft.tw)
-   [搞笑談軟工Facebook社團](https://www.facebook.com/groups/teddy.tw/)

## 2018年7月5日 星期四

  

### Clean Architecture（5）：架構三原則二部曲—相依性原則

July 5 16:17~17:23

[![螢幕截圖 2018-07-05 17.21.14](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEglcVhOgxThrRKkZdG53fW_l7KFFBz86XhTLq190JaVjDlgwk_JptKlDqWL-aoNoobEKT1Yc0UDMYUauujEXJo8xNWGDDS80G1E6opVislWK94bHdL7h0KIFLX6_4LP9qfT5RHfzwVPNZo5/?imgmax=800 "螢幕截圖 2018-07-05 17.21.14")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjiaSzpYgaD89U3W84eRtQwaOBJMHob0h6EWDYdQTbPNyHGfMKQT5viZaXBWTYCnGgpbyEoPm4xUoOWgXI9pn59udQajaEoqhxSNbq_CgfqDmWIEEuPE5PVx0CGiczn9nWSzQfhU5b6mtp-/s1600-h/Image)

  

**前言**

軟體架構依據「距離I/O遠近」分層之後，接下來遇到一個問題：「如何管理各層之間的相依性？」。在《Clean Architecture》書中，把這個規則稱為**相依性原則（Dependency Rule）。**

\*\*\*

**低層相依於高層**

《Clean Architecture》書中所定義的相依性原則只有如下的短短一句話：

***Source code dependencies must point only inward, toward higher-level policies. （程式碼相依性必須只能往內，指向更高層級的策略。）***

▼如下圖所示（圖片來源[在此](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)），內圈（較高層級）的元件不能知道（相依）任何外層的元件。也就是說，外層的函數、類別、變數或任何有名字的軟體實體，都不能出現在內層的元件之中。相依性只有一個方向，就是由外往內，由低層往高層。

[![CleanArchitecture](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiofyVn5BrjrUx64_FhXuCslg8jX7HZiX5Z_MW-0QvcYw-qS_h0XUIY-iNXk2yfV1LNPnumnP22_6zwdGhOMnllqbbMwMiXJJVVtrC2f2FLirNXBppQPGySWqwuPg77xG1lNwMQ6NwyKbvF/?imgmax=800 "CleanArchitecture")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgHOugVUfASncB4KxEx7TI050IHQP4FCrNY1Ghx0bL5KS_5orpSd4ATCjoaW0016KCRhAbg4GCyN2pJdvKZ0c4GkVh5cDGazHG8VaMD3Wr6uc3Ak_OZfikBja_MjQcQNSpucYDX1RUsBd52/s1600-h/CleanArchitecture%255B4%255D)

\*\*\*

**煩人的實作細節**

相依性原則本身非常簡單，但實務上在系統中要完全落實此原則有許多細節要注意。

▼例如下圖中位於Entities層（最高層）的Host類別，依據相依性原則它不能相依於任何位於此層以外的「任何東西」。為了將Host物件儲存於資料庫中，Teddy使用了javax.persistence package裡面的annotation（@Entity、@Table、@EmbeddedId、@Column等）。

[![螢幕截圖 2018-07-05 16.34.14](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjebRmE4eJb69PPOZoO_aaiYcBVB7RWNDeqoQDProFay547QRhVDd_0nbxHApx1Hk_onAeKvL4_QpBfMLQjuj_XAnsSrApgLvIP-nYAAW2OXVd95teP7sVY03uTfBfCNy5gqE4S8nObx1CA/?imgmax=800 "螢幕截圖 2018-07-05 16.34.14")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg5yJN2yykR2ZLWVgIKhL_qthzWNQLe-Smi5ZhVP-R2FfTRNK2qbZACiKIJg6iHH8ncL_dODfSn0jy8DSp71rQLfqaCeZDCb4U_N-24NPso1jR3PgjAGd1rB7JSLNpOYfFQxQ-MG6r6JSrC/s1600-h/Image)

雖然這些annotation是 Java Persistence API（JPA）標準的一部分，但它們的實作是由Hibernate所提供。嚴格來說，上面的程式違反了《Clean Architecture》的相依性原則。

如果不用annotation，也可以用XML檔案來註記，把對hibernate-jpa-2.1-api的相依性從原始碼中移除。但廣義的來講，雖然在原始碼中Host不直接相依於hibernate-jpa-2.1-api，但透過XML設定檔案還是間接的相依於hibernate-jpa-2.1-api，Teddy認為兩者效用實際上是差不多，只是把「顯性相依」轉成「隱性相依」，並沒有好到哪裡去，甚至更糟糕。

\*\*\*

要完全把Host對hibernate-jpa-2.1-api的相依性拿掉，一個常見的作法是定義另一組針對持久化（Persistence ）所需的物件，例如針對原本的Host物件的持久化需求另外定義一個HostEntity類別，如此一來原本Host類別中JPA annotation就可以移到HostEntity之中，然後再透過Hibernate把HostEntity直接存到資料庫。

但這種做法又衍生另其他問題，首先，必須另外撰寫物件負責把Host轉成HostEntity，以及把HostEntity轉成Host。這種物件稱之為**Mapper**。

其次，HostEntity要擺在哪一層也是需要考慮一下，很顯然不會擺在Entities這一層。因為Repository通常被Use Case所使用，所以放在Use Cases層似乎很合理：當要儲存Host時，Use Case先產生或是獲得一個Host物件，然後呼叫Mapper把Host轉成HostEntity，在透過Repository將HostEntity存到資料庫。反之，要讀取Host時，Use Case透過Repository從資料庫讀取HostEntity，再把它轉成Host。

但是這麼做其實還是違反了相依性原則，因為Use Case相依於外部的hibernate-jpa-2.1-api。忙了大半天其實是白忙一場XD。

\*\*\*

另一種作法是，不透過任何JPA annotation來簡化物件永久化的問題，只定義Repository介面，然後透過main元件注入Repository介面的實作。Repository介面的實作程式碼位於Interface Adapter層，也就是《Clean Architecture》架構圖裡面的Gateways。它再透過相依性注入的方式，獲得不同持久化框架的實作，如此一來便符合相依性原則。

但是，原本只要貼一貼annotation就可以把透過持久化框架（例如Hibernate）達到持久化的目的，現在為了嚴格遵守相依性原則，程式碼變得麻煩好多。這就是在實作上必須注意與取捨的地方。

\*\*\*

**結論**

滿足相依性原則可以獲得兩大好處，首先，因為高層的物件不再相依於框架等低層的物件，所以測試碼可以單獨測試，大幅簡化測試的工作。

其次，系統的核心功能位於Entities與Use Cases這兩層（蛋黃區與蛋白區），要更換I/O或是應用程式框架「理論上」變得簡單很多。例如，如果要將Web-Based UI換成Android App，只需異動最外面兩層（Framework & Drivers與Interface Adapters），便可直接銜接原本的Use Cases與Entities。

\*\*\*

友藏內心獨白：跟有潔癖的人住在一起家裡肯定很乾淨，但你可能會被對方煩死。

張貼者： [Teddy Chen](https://www.blogger.com/profile/02066842119056439711 "author profile") 於  [下午5:24](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html "permanent link")  [![](//img1.blogblog.com/img/icon18_email.gif)](https://www.blogger.com/email-post/1298974142445162186/8993242928289211911 "電子郵件發布")[![](https://resources.blogblog.com/img/icon18_edit_allbkg.gif)](https://www.blogger.com/post-edit.g?blogID=1298974142445162186&postID=8993242928289211911&from=pencil "編輯文章")

[以電子郵件傳送這篇文章](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=8993242928289211911&target=email "以電子郵件傳送這篇文章")[BlogThis！](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=8993242928289211911&target=blog "BlogThis！")[分享至 X](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=8993242928289211911&target=twitter "分享至 X")[分享至 Facebook](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=8993242928289211911&target=facebook "分享至 Facebook")[分享到 Pinterest](https://www.blogger.com/share-post.g?blogID=1298974142445162186&postID=8993242928289211911&target=pinterest "分享到 Pinterest")

標籤： [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5), [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B), [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88)

#### 沒有留言:

#### 張貼留言

[](https://www.blogger.com/comment/frame/1298974142445162186?po=8993242928289211911&hl=zh-TW&saa=85391&origin=https://teddy-chen-tw.blogspot.com)

[較新的文章](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture6.html "較新的文章") [較舊的文章](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html "較舊的文章") [首頁](https://teddy-chen-tw.blogspot.com/)

訂閱： [張貼留言 (Atom)](https://teddy-chen-tw.blogspot.com/feeds/8993242928289211911/comments/default)

## 搜尋此網站

 

## 總網頁瀏覽量

## 關於

[![我的相片](//blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBBqKlknBrSZ3EEXiWqeZasJOxrwoUfdqqF4Es3OR0YFtwDAYNHp6bBx3PqLGVJXAJQ4INhfp0gUlRZyRoSF9vzraU_JlLOECrERAWag3VIa0JdxRhpLwyEz7w9QtX8w/s220/Teddy.jpg)](https://www.blogger.com/profile/02066842119056439711)

[Teddy Chen](https://www.blogger.com/profile/02066842119056439711)

[檢視我的完整簡介](https://www.blogger.com/profile/02066842119056439711)

## 請幫Teddy按個讚^\_^

## 標籤

-   [工商服務](https://teddy-chen-tw.blogspot.com/search/label/%E5%B7%A5%E5%95%86%E6%9C%8D%E5%8B%99) (94)
-   [生活](https://teddy-chen-tw.blogspot.com/search/label/%E7%94%9F%E6%B4%BB) (342)
-   [改行寫網路小說算了](https://teddy-chen-tw.blogspot.com/search/label/%E6%94%B9%E8%A1%8C%E5%AF%AB%E7%B6%B2%E8%B7%AF%E5%B0%8F%E8%AA%AA%E7%AE%97%E4%BA%86) (25)
-   [其他](https://teddy-chen-tw.blogspot.com/search/label/%E5%85%B6%E4%BB%96) (25)
-   [客戶教我的事](https://teddy-chen-tw.blogspot.com/search/label/%E5%AE%A2%E6%88%B6%E6%95%99%E6%88%91%E7%9A%84%E4%BA%8B) (2)
-   [持續整合](https://teddy-chen-tw.blogspot.com/search/label/%E6%8C%81%E7%BA%8C%E6%95%B4%E5%90%88) (29)
-   [哲學](https://teddy-chen-tw.blogspot.com/search/label/%E5%93%B2%E5%AD%B8) (13)
-   [旅遊](https://teddy-chen-tw.blogspot.com/search/label/%E6%97%85%E9%81%8A) (568)
-   [視力測驗](https://teddy-chen-tw.blogspot.com/search/label/%E8%A6%96%E5%8A%9B%E6%B8%AC%E9%A9%97) (3)
-   [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5) (410)
-   [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B) (110)
-   [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88) (188)
-   [最新課程](https://teddy-chen-tw.blogspot.com/search/label/%E6%9C%80%E6%96%B0%E8%AA%B2%E7%A8%8B) (81)
-   [創業](https://teddy-chen-tw.blogspot.com/search/label/%E5%89%B5%E6%A5%AD) (84)
-   [測試](https://teddy-chen-tw.blogspot.com/search/label/%E6%B8%AC%E8%A9%A6) (106)
-   [盡信書不如無書](https://teddy-chen-tw.blogspot.com/search/label/%E7%9B%A1%E4%BF%A1%E6%9B%B8%E4%B8%8D%E5%A6%82%E7%84%A1%E6%9B%B8) (2)
-   [需求](https://teddy-chen-tw.blogspot.com/search/label/%E9%9C%80%E6%B1%82) (23)
-   [課程實錄](https://teddy-chen-tw.blogspot.com/search/label/%E8%AA%B2%E7%A8%8B%E5%AF%A6%E9%8C%84) (22)
-   [學習](https://teddy-chen-tw.blogspot.com/search/label/%E5%AD%B8%E7%BF%92) (92)
-   [貓](https://teddy-chen-tw.blogspot.com/search/label/%E8%B2%93) (15)
-   [還少一本書](https://teddy-chen-tw.blogspot.com/search/label/%E9%82%84%E5%B0%91%E4%B8%80%E6%9C%AC%E6%9B%B8) (36)
-   [agile](https://teddy-chen-tw.blogspot.com/search/label/agile) (638)
-   [C. C. Agile](https://teddy-chen-tw.blogspot.com/search/label/C.%20C.%20Agile) (33)
-   [DDD](https://teddy-chen-tw.blogspot.com/search/label/DDD) (85)
-   [DevOps](https://teddy-chen-tw.blogspot.com/search/label/DevOps) (3)
-   [Eclipse](https://teddy-chen-tw.blogspot.com/search/label/Eclipse) (6)
-   [exception handling](https://teddy-chen-tw.blogspot.com/search/label/exception%20handling) (84)
-   [HCI](https://teddy-chen-tw.blogspot.com/search/label/HCI) (44)
-   [Implementation Patterns](https://teddy-chen-tw.blogspot.com/search/label/Implementation%20Patterns) (5)
-   [Kanban](https://teddy-chen-tw.blogspot.com/search/label/Kanban) (79)
-   [Mobile](https://teddy-chen-tw.blogspot.com/search/label/Mobile) (2)
-   [Patterns](https://teddy-chen-tw.blogspot.com/search/label/Patterns) (198)
-   [Programming](https://teddy-chen-tw.blogspot.com/search/label/Programming) (54)
-   [Refactoring](https://teddy-chen-tw.blogspot.com/search/label/Refactoring) (65)
-   [Scrum](https://teddy-chen-tw.blogspot.com/search/label/Scrum) (461)
-   [TDD/BDD](https://teddy-chen-tw.blogspot.com/search/label/TDD%2FBDD) (54)

## 網誌存檔

-   [►](javascript:void\(0\))  [2025](https://teddy-chen-tw.blogspot.com/2025/) (1)
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2025/06/) (1)

-   [►](javascript:void\(0\))  [2024](https://teddy-chen-tw.blogspot.com/2024/) (7)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2024/09/) (6)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2024/03/) (1)

-   [►](javascript:void\(0\))  [2023](https://teddy-chen-tw.blogspot.com/2023/) (25)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2023/09/) (5)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2023/07/) (3)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2023/06/) (5)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2023/03/) (10)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2023/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2023/01/) (1)

-   [►](javascript:void\(0\))  [2022](https://teddy-chen-tw.blogspot.com/2022/) (29)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2022/10/) (1)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2022/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2022/07/) (18)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2022/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2022/03/) (2)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2022/02/) (1)

-   [►](javascript:void\(0\))  [2021](https://teddy-chen-tw.blogspot.com/2021/) (27)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2021/12/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2021/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2021/09/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2021/08/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2021/06/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2021/04/) (1)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2021/03/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2021/01/) (8)

-   [►](javascript:void\(0\))  [2020](https://teddy-chen-tw.blogspot.com/2020/) (32)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2020/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2020/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2020/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2020/09/) (4)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2020/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2020/07/) (2)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2020/06/) (2)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2020/05/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2020/04/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2020/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2020/02/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2020/01/) (1)

-   [►](javascript:void\(0\))  [2019](https://teddy-chen-tw.blogspot.com/2019/) (50)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2019/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2019/11/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2019/10/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2019/07/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2019/06/) (1)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2019/05/) (8)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2019/04/) (2)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2019/03/) (8)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2019/02/) (8)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2019/01/) (7)

-   [▼](javascript:void\(0\))  [2018](https://teddy-chen-tw.blogspot.com/2018/) (28)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2018/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2018/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2018/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2018/08/) (6)
    
    -   [▼](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2018/07/) (6)
        -   [阿嬤覺得你冷](https://teddy-chen-tw.blogspot.com/2018/07/blog-post.html)
        -   [Clean Architecture（6）：架構三原則三部曲—跨層原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture6.html)
        -   [Clean Architecture（5）：架構三原則二部曲—相依性原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html)
        -   [Clean Architecture（4）：架構三原則首部曲—分層原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html)
        -   [LG gram 15Z980-R.AAS9U1 (2018) Amazon 購買與使用心得（下）](https://teddy-chen-tw.blogspot.com/2018/07/lg-gram-15z980-raas9u1-2018-amazon_3.html)
        -   [LG gram 15Z980-R.AAS9U1 (2018) Amazon 購買與使用心得（上）](https://teddy-chen-tw.blogspot.com/2018/07/lg-gram-15z980-raas9u1-2018-amazon.html)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2018/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2018/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2018/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2018/01/) (3)

-   [►](javascript:void\(0\))  [2017](https://teddy-chen-tw.blogspot.com/2017/) (142)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2017/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2017/11/) (8)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2017/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2017/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2017/08/) (15)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2017/07/) (10)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2017/06/) (14)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2017/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2017/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2017/03/) (23)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2017/02/) (16)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2017/01/) (25)

-   [►](javascript:void\(0\))  [2016](https://teddy-chen-tw.blogspot.com/2016/) (301)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2016/12/) (14)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2016/11/) (19)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2016/10/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2016/09/) (18)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2016/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2016/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2016/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2016/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2016/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2016/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2016/02/) (29)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2016/01/) (31)

-   [►](javascript:void\(0\))  [2015](https://teddy-chen-tw.blogspot.com/2015/) (366)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2015/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2015/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2015/10/) (32)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2015/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2015/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2015/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2015/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2015/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2015/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2015/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2015/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2015/01/) (31)

-   [►](javascript:void\(0\))  [2014](https://teddy-chen-tw.blogspot.com/2014/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2014/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2014/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2014/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2014/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2014/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2014/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2014/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2014/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2014/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2014/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2014/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2014/01/) (31)

-   [►](javascript:void\(0\))  [2013](https://teddy-chen-tw.blogspot.com/2013/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2013/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2013/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2013/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2013/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2013/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2013/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2013/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2013/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2013/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2013/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2013/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2013/01/) (31)

-   [►](javascript:void\(0\))  [2012](https://teddy-chen-tw.blogspot.com/2012/) (373)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2012/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2012/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2012/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2012/09/) (31)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2012/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2012/07/) (32)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2012/06/) (31)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2012/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2012/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2012/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2012/02/) (32)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2012/01/) (32)

-   [►](javascript:void\(0\))  [2011](https://teddy-chen-tw.blogspot.com/2011/) (120)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2011/12/) (16)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2011/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2011/10/) (4)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2011/09/) (3)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2011/08/) (9)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2011/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2011/06/) (10)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2011/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2011/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2011/03/) (16)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2011/02/) (12)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2011/01/) (11)

-   [►](javascript:void\(0\))  [2010](https://teddy-chen-tw.blogspot.com/2010/) (104)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2010/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2010/11/) (6)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2010/10/) (10)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2010/09/) (5)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2010/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2010/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2010/06/) (19)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2010/05/) (12)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2010/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2010/03/) (12)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2010/02/) (2)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2010/01/) (7)

-   [►](javascript:void\(0\))  [2009](https://teddy-chen-tw.blogspot.com/2009/) (31)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2009/12/) (5)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2009/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2009/10/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2009/08/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2009/07/) (8)

-   [►](javascript:void\(0\))  [2008](https://teddy-chen-tw.blogspot.com/2008/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2008/09/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2008/03/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2008/01/) (2)

-   [►](javascript:void\(0\))  [2007](https://teddy-chen-tw.blogspot.com/2007/) (4)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2007/12/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2007/06/) (3)

## 追蹤者

## 最新回應

[載入中…](http://teddy-chen-tw.blogspot.tw/feeds/comments/default)

## 熱門文章

-   [爭什麼，摻在一起做成撒尿牛丸啊](https://teddy-chen-tw.blogspot.com/2012/03/blog-post_20.html)
    
    March 20 13:53~14:56 (影像節錄自http://www.youtube.com/watch?v=LGV5PcM9SQo) 鄉民們都看過食神這部電影吧，電影中有一個經典的橋段，就是火鷄姐及鵝頭在爭辯誰做的撒尿蝦好吃，誰做的牛丸比較優。正當雙方吵得不可...
    
-   [擁抱改變，遠離Hard Code](https://teddy-chen-tw.blogspot.com/2018/09/hard-code.html)
    
    September 19 08:37~09:12 ▲圖片來源 在此 寫死（Hard Code） 幾天前一位朋友在Facebook上發問：「如何對非資訊人員解釋寫死（hard code）？」 維基百科 對於寫死的解釋如下： 寫死 （英語： Hard Code 或 Hard Co...
    
-   [什麼是物件導向（3）：Polymorphism](https://teddy-chen-tw.blogspot.com/2012/01/3polymorphism.html)
    
    January 26 23:01~January 27 00:23 Polymorphism，中文翻譯成「多型」，說真的這個概念當年Teddy學OO的時候搞了好久才弄懂，後來看了「課本」的定義之後，覺得寫得還滿好的，請參考Object-Oriented Software En...
    
-   [幫Macbook Pro 15” Late 2016尋找外接4K 27”螢幕](https://teddy-chen-tw.blogspot.com/2017/07/macbook-pro-15-late-20164k-27.html)
    
    June 17 21:28~23:24 先打個廣告，【 敏捷開發懶人包：物件導向技能 】確定於8月19日（週六）開課。 課程設定目標希望讓鄉民們在找工作面試的時候，如果面試官問到物件導向技術方面的問題，能夠具備輕鬆回答的能力 。 \*\*\* 動心起念 去年底買了MacBook P...
    
-   [Top-down和Bottom-up設計方法](https://teddy-chen-tw.blogspot.com/2014/03/top-downbottom-up.html)
    
    Mar. 12 08:40~10:10 Top-down（由上而下）和Bottom-up（由下而上）是兩種設計與解決問題的技巧。前者對問題先有一個整體的概念，然後再逐步加上設計細節，最後讓整體的輪廓越來越清楚。後者則是先將解決問題可能所需的基本元件、方案給準備好，然後再將這...
    
-   [羅技Logitech R500 雷射簡報筆開箱](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html)
    
    June 22 12:51~14:16 需求分析 因為工作的關係Teddy經常需要使用簡報筆，雜牌的不算，光是羅技的簡報筆Teddy就買過以下四種（R500是最新上市產品，也是今天介紹的主角）  ： ▼畫面節錄自羅技官網 Teddy心中理想的簡報筆應該有以下幾個功能： 重量輕且...
    

## 分析

 

技術提供：[Blogger](https://www.blogger.com).
```
```

```
Tool: webfetch

**Input:**
```json
{
  "format": "markdown",
  "url": "https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture6.html"
}
```
**Output:**
```
                  搞笑談軟工: Clean Architecture（6）：架構三原則三部曲—跨層原則   

 

 

l

# [搞笑談軟工](https://teddy-chen-tw.blogspot.com/)

敏捷開發，設計模式，精實開發，Scrum，軟體設計，軟體架構

## 網頁

-   [首頁](https://teddy-chen-tw.blogspot.com/)
-   [泰迪軟體最新課表](http://teddysoft.tw/courses/)
-   [泰迪軟體Facebook](https://www.facebook.com/teddysoft.tw)
-   [搞笑談軟工Facebook社團](https://www.facebook.com/groups/teddy.tw/)

## 2018年7月6日 星期五

  

### Clean Architecture（6）：架構三原則三部曲—跨層原則

July 6 09:52~10:58

[![螢幕截圖 2018-07-06 10.58.15](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi4JkxJdKwVURlb4UblwmovnDDQxlSLdypmbTekmV_LJy8-RH_ATVAzhsdxZwxjxo_kZE_WvAzGhzc6Q6XT-vtYT7H3GBEnUP9gB_Oz4XL7EReJLbuqEMBP6coaJtvQWgSyf8hc3jb-toez/?imgmax=800 "螢幕截圖 2018-07-06 10.58.15")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgPC0zn2wx75ZmRadVHMaj6nFH1ckuLZSKHgwuPQTprgNHpFilukGpg5LGjSRTqvSXeaxZ_6-BgXkknjl5ykgscVxda4cQ7jMB39Pv1bUb7enBuxUUrJz0sMfhE45n03wdCLkC8UzOWlgSM/s1600-h/Image)

▲直接跨層XD

  

**前言**

將軟體架構分層並且確認相依性嚴格遵守由外往內（由低層往高層）的這兩個原則之後，接下來會面臨到另一個問題：「物件跨層時該怎麼辦？」。

\*\*\*

**跨層範例**

▼請參考下圖，AddHostController呼叫AddHost Use Case，它再呼叫Entity層的Host物件，並獲得一個Host物件。接著AddHost Use Case直接把Host物件往外傳給AddHostController。

[![螢幕截圖 2018-07-06 10.05.10](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi_SMKPlyJvlVfCGxSnwLni3ZU7CUS8T5p0Y-2mTpYuLGRuABRTdG9e6bVvicL-IHRxZjEl8XcvR0EPGiYH9Ea6bjtkwflsMyOPfLlNo1CjERmR91zIIobJvjimLhYGMJXI-ow2KL4Z3KC7/?imgmax=800 "螢幕截圖 2018-07-06 10.05.10")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhL_HySUcf9KnlcqMHW7i1DhYOMVMvGxLfwBVa_QEK78ziU3xbo_qMwS53GUUerFu0EVfGnZXwclxeiXdBnGbRSiAzNWUst4dBJvz7zzBaOR5YtZ2WlKs0EGf9HDE0syZTHoC_0uXZT0X5z/s1600-h/Image)

  

上面這個例子有兩件事情需要討論：

-   **內層物件可否跨層往外傳**：舉例來說，位於Entity層的Host物件可以直接跨層（跨過Use Case層）傳給AddHostController嗎？依據相依性原則，上述情況還是有遵守「外層相依於內層」的規定，只不過AddHostController跨過Use Case這層相依於Entity層的Host物件。Teddy並未在《Clean Architecture》書中看到可否「跨層相依」的描述，而網路上國外鄉民所實作的範例，有些直接跨層相依，有些則是會將Host物件在AddHost Use Case轉過一次再傳給AddHostController（請參考稍後討論的Use Case雙向介面）。
-   **外層如何將資料傳給內層**：假設AddHostController從瀏覽器收到使用者所填寫的「新增Host表單」，並將其內容轉成一個JSON物件。如果AddHostController直接將這個JSON物件傳給AddHost Use Case，將會破壞相依性原則，因為如此一來AddHost Use Case就相依於屬於應用程式框架的JSON物件。

▼直接把JSON從Controller往Use Case傳遞違反了相依性原則（假設JSON函式庫是由網頁應用程式框架所提供）

[![螢幕截圖 2018-07-06 10.29.56](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjtDiEhue58mTpn7P_AmR9kbl2_hasd2JEOM5qXvLa6kxmhtd33L46l1UfLTIOAxa05CGz4QouCSo-AldpwYdiK_TML6Q4O9Sj3BnkGg64xLWvUXm_xyYCxgm5EwHhsHXfpowScvNCZuJ2D/?imgmax=800 "螢幕截圖 2018-07-06 10.29.56")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj1scggLLik_yCug0jswYnIELW82yfKwEY0y4iDw98d76KU3K_bHK8WFBKsD-ysujx6fuRhU5RdYi-4fwlOYJgwr40bMLMnUiSCOVkpewwlebqyFXWG6bU4P9vq0H82LuzD6VV4I32VtxVr/s1600-h/Image)

\*\*\*

**定義雙向介面**

▼如下圖所示，《Clean Architecture》書中建議Use Case需要定義「雙向介面」，簡單來說就是定義Use Case的Input與Output介面與參數的資料型態。

[![螢幕截圖 2018-07-06 10.32.55](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhcsWqVdkUTTt4pQp_tWp9cK1h3-I-cdVleboYxr9KrAzP9fVZ8aX1ND2Sy8pJkZZ_JdMwJA1gf7qH9Et3qvo5mlGq2HPn9P5V79mTq8k4s_R1BUaAqnJ-21VlSyYL6HOGaSw6i-wdNnBEQ/?imgmax=800 "螢幕截圖 2018-07-06 10.32.55")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhWyZGDHfKdbq9o5WBhheXRd3aA3abp4N5LlQ1mBViMt3IXvwFgBQEmHxnxmDWLG8ZYFp5Rdv9gnbWK_veMpNU6gIdllD0vvuhxOaDVfzlFjBGIpsJJK1CaQ7zVV1QTc8OZNHohd9A9KjfQ/s1600-h/Image)

Use Case本身提供Input介面與資料型別實作，外層的Controller要將參數傳給Use Case時，必須負責把參數型別轉成Use Case所定義的Input介面。

Use Case的Output介面則由Presenter所實作（Presenter是Model-View-Presenter設計模式裡面的那個Presenter），呼叫Use Case時Controller會傳入Presenter物件，Use Case則將透過Presenter物件輸出資料。

  

▼看一個程式範例片段，HostController的getHosts函數呼叫FetchHosts Use Case得到所有host的列表，它首先產生HostListPresenter物件與FetchHostsInput物件，並將它們傳給FetchHosts Use Case。FetchHosts Use Case從FetchHostsInput物件獲得搜尋物件的參數資料，並將找到的資料傳給HostListPresenter。

[![螢幕截圖 2018-07-06 10.40.21](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgL8EedBh6Q8cMW9WYirOAl7Zpas2K4IMXOVs-XroO7xU3Cse56ZhIGKLhqT7odCBlBZWIuiPCA8S4qhLTcKUdTT8vklcKgYG3kLf2v1LkW6DRsvoDYk0ENvyHj3cSQVrvCExO0P_FJ52rB/?imgmax=800 "螢幕截圖 2018-07-06 10.40.21")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhHpvp5LvJsX_Qo0AC6TYuxoMzgIZqPmHQNqd_h1cDFjnMdh7MLKI3c3VXrbt6s8DeAyoy_n4MgaOFTB-VM-6C1s2nXJahnSRO-oGK_nmlFGSWSzaf8cq3Ify_hL619lK2zxdF4xfWKhdri/s1600-h/Image)

  

▼FetchHostsUseCase介面定義

[![螢幕截圖 2018-07-06 10.45.04](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhfA03ZpnLodQl1eMxawgJooAa_klmXqf7OvF8lbtJ04N43-n2Rxv0673p7VbPd4gARcL-PQ1cSHJmBukQTq9v54aDLXoEn1YlkaTwVqZxRSEWpAzG0n4SfS8LpVx5XNru7sAZpPh_YJDSq/?imgmax=800 "螢幕截圖 2018-07-06 10.45.04")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjF3cVfVf_g1amYoHNWVKQKtW1AufsOqB8r-JRgR1BTOseTPyh6Twk4XBiHiyHOrW83IaCTSmgJ2NQ1ZXQ67Tz0RnIY8wxVmc7rzzp9Wzd_JnlI1Vi1j6rKfS5eEO971eTy-P_zgvZ6hPYw/s1600-h/Image)

  

▼FetchHostsInput類別。**請注意，如果嚴格依據《Clean Architecture》書中的圖示，應該先將FetchHostsInput定義成一個介面，然後再讓FetchHostsUseCase提供這個介面的實作。在此Teddy偷懶簡化這個步驟，直接將FetchHostsInput設計成一個具體類別。**

[![螢幕截圖 2018-07-06 10.45.49](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgky4k0-rgABi-Zi-V4_qq-ns5SOjI3wj3UYk1MjR8N-hgYAmyWZh9Bq3T4j7VPoGHTRfpV-4h4Qdu20H25rrhOx6XxwwPJUbZ9hZRHaMzxIpwg81kB_oyBXhCvCau1lAwJINGzN6zo7zUg/?imgmax=800 "螢幕截圖 2018-07-06 10.45.49")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgScPxxS7It-i_gZhVLLErOivgL1hKEAI0cTpnkNia6pI2uGjACNSs_SEYfWZCH_aMhCuCutyIQXgnTAaFZFrMkqC1BcKzocjD-lbZFfYS3gr6gfQoK6fYn2rkDHWKqlf4_73LKp1u_lzY_/s1600-h/Image)

▼FetchHostsOutput介面

[![螢幕截圖 2018-07-06 10.48.20](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjQXYYH881HRetlXE5Tx8fSJkvl7VmmLPlmyN564lRA-fZ9MBTSwkKie8DlbcMF2qvKXGh2gpONOqi7KRkkt3BgX60iGv7iVCsDt10bmtaSGbuJjdfPwg3h8PvE9OWaFwH3wMGaA3j-BHWi/?imgmax=800 "螢幕截圖 2018-07-06 10.48.20")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgPLwoqsFSoDBWjdEHUX5AV-7S66kN9FnNsQsjJ-p0hwcbeICgQked6igs1MMtwV5bLDXOtVVkl-l8VSl7fUXlOOPoISSI6Ggga4jlFlMk2sRzlKhXDQHtVd8wAtP24um2GtWsRsOYNYM1Z/s1600-h/Image)

  

▼HostListPresenter

[](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEghTXy7N8AJUBSlypCRxUa18LxTDUFxqTln42eOzOKgvWbusA7j9EV2wf65giTIctcVjIzbsE7tezY3WMMQHIxHDPSqRz2XLSGSRT2tg4poYOsLV3nHt8yKFSXb8yL0tl4ozrQBtRQ9CYaY/s1600-h/Image)[![螢幕截圖 2018-07-06 10.49.15](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh9memOlEn36BxSOzFtXDTPfw5W1CArp6S6Q961x_eANPywjk3MqhwYqgj84gipGmYpJ2r8hBSqKEG00QqIl4kouG6qHknDRX6omGEH7jnKYmzdu9xcFJR0eMSUtCAsZCTCLkmXJ2Q3jm4x/?imgmax=800 "螢幕截圖 2018-07-06 10.49.15")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj0VOnrfrczHnvxMxw6kXYaMBGu-8LRu-gAlpQAbdMfB_J5287rvdXaNxdOjTSZ6kjB2BOOTc3XYh7d-qIHbnN2i7F7n56t6TMI38mG5654DWyt2qRJrHkIny2VlW3ULOxlJpTYQyC17wKz/s1600-h/Image)

  

▼FetchHostsUseCase實做

[![螢幕截圖 2018-07-06 10.52.54](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi0GOvbroxi94z1sWGopqY0adOBySU1RlfmDR6iKUQ7aWG1gtLp-LQot8TF5LTug4P4D9QoE84l5sgyFpwcDPrhmBCVwZwy5-Dacjc5SzvWIphRVMni5zSABJnXaZY02i66PhdtO_QxRt6b/?imgmax=800 "螢幕截圖 2018-07-06 10.52.54")](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEipjmhFtYEohTQ3KyC5C_4U58SRvbTOFmiqI-FhwfMtBD8AHktlm5PCMeJxBkBeEe_bB1x26GGApDxhkdelQyvF5R7k8zX86oEcS7iNjiLI1FIIxFe57SW1m1r5xMaWrG8kuZzpgsdwacjn/s1600-h/Image)

\*\*\*

**結論**

《Clean Architecture》整本書雖然有多達34章，如果可以掌握**分層、相依性與跨層**這三個重點，在實作Clean Architecture時應該就沒什麼困難之處。

\*\*\*

友藏內心獨白：還有一個Missing Chapter沒討論…

張貼者： [Teddy Chen](https://draft.blogger.com/profile/02066842119056439711 "author profile") 於  [上午11:06](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture6.html "permanent link")  [![](//img1.blogblog.com/img/icon18_email.gif)](https://draft.blogger.com/email-post/1298974142445162186/8694405946706501839 "電子郵件發布")[![](https://resources.blogblog.com/img/icon18_edit_allbkg.gif)](https://draft.blogger.com/post-edit.g?blogID=1298974142445162186&postID=8694405946706501839&from=pencil "編輯文章")

[以電子郵件傳送這篇文章](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=8694405946706501839&target=email "以電子郵件傳送這篇文章")[BlogThis！](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=8694405946706501839&target=blog "BlogThis！")[分享至 X](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=8694405946706501839&target=twitter "分享至 X")[分享至 Facebook](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=8694405946706501839&target=facebook "分享至 Facebook")[分享到 Pinterest](https://draft.blogger.com/share-post.g?blogID=1298974142445162186&postID=8694405946706501839&target=pinterest "分享到 Pinterest")

標籤： [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5), [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B), [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88)

#### 沒有留言:

#### 張貼留言

[](https://draft.blogger.com/comment/frame/1298974142445162186?po=8694405946706501839&hl=zh-TW&saa=85391&origin=https://teddy-chen-tw.blogspot.com)

[較新的文章](https://teddy-chen-tw.blogspot.com/2018/07/blog-post.html "較新的文章") [較舊的文章](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html "較舊的文章") [首頁](https://teddy-chen-tw.blogspot.com/)

訂閱： [張貼留言 (Atom)](https://teddy-chen-tw.blogspot.com/feeds/8694405946706501839/comments/default)

## 搜尋此網站

 

## 總網頁瀏覽量

## 關於

[![我的相片](//blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBBqKlknBrSZ3EEXiWqeZasJOxrwoUfdqqF4Es3OR0YFtwDAYNHp6bBx3PqLGVJXAJQ4INhfp0gUlRZyRoSF9vzraU_JlLOECrERAWag3VIa0JdxRhpLwyEz7w9QtX8w/s220/Teddy.jpg)](https://draft.blogger.com/profile/02066842119056439711)

[Teddy Chen](https://draft.blogger.com/profile/02066842119056439711)

[檢視我的完整簡介](https://draft.blogger.com/profile/02066842119056439711)

## 請幫Teddy按個讚^\_^

## 標籤

-   [工商服務](https://teddy-chen-tw.blogspot.com/search/label/%E5%B7%A5%E5%95%86%E6%9C%8D%E5%8B%99) (94)
-   [生活](https://teddy-chen-tw.blogspot.com/search/label/%E7%94%9F%E6%B4%BB) (342)
-   [改行寫網路小說算了](https://teddy-chen-tw.blogspot.com/search/label/%E6%94%B9%E8%A1%8C%E5%AF%AB%E7%B6%B2%E8%B7%AF%E5%B0%8F%E8%AA%AA%E7%AE%97%E4%BA%86) (25)
-   [其他](https://teddy-chen-tw.blogspot.com/search/label/%E5%85%B6%E4%BB%96) (25)
-   [客戶教我的事](https://teddy-chen-tw.blogspot.com/search/label/%E5%AE%A2%E6%88%B6%E6%95%99%E6%88%91%E7%9A%84%E4%BA%8B) (2)
-   [持續整合](https://teddy-chen-tw.blogspot.com/search/label/%E6%8C%81%E7%BA%8C%E6%95%B4%E5%90%88) (29)
-   [哲學](https://teddy-chen-tw.blogspot.com/search/label/%E5%93%B2%E5%AD%B8) (13)
-   [旅遊](https://teddy-chen-tw.blogspot.com/search/label/%E6%97%85%E9%81%8A) (568)
-   [視力測驗](https://teddy-chen-tw.blogspot.com/search/label/%E8%A6%96%E5%8A%9B%E6%B8%AC%E9%A9%97) (3)
-   [軟工](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E5%B7%A5) (410)
-   [軟體架構](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E6%9E%B6%E6%A7%8B) (110)
-   [軟體設計](https://teddy-chen-tw.blogspot.com/search/label/%E8%BB%9F%E9%AB%94%E8%A8%AD%E8%A8%88) (188)
-   [最新課程](https://teddy-chen-tw.blogspot.com/search/label/%E6%9C%80%E6%96%B0%E8%AA%B2%E7%A8%8B) (81)
-   [創業](https://teddy-chen-tw.blogspot.com/search/label/%E5%89%B5%E6%A5%AD) (84)
-   [測試](https://teddy-chen-tw.blogspot.com/search/label/%E6%B8%AC%E8%A9%A6) (106)
-   [盡信書不如無書](https://teddy-chen-tw.blogspot.com/search/label/%E7%9B%A1%E4%BF%A1%E6%9B%B8%E4%B8%8D%E5%A6%82%E7%84%A1%E6%9B%B8) (2)
-   [需求](https://teddy-chen-tw.blogspot.com/search/label/%E9%9C%80%E6%B1%82) (23)
-   [課程實錄](https://teddy-chen-tw.blogspot.com/search/label/%E8%AA%B2%E7%A8%8B%E5%AF%A6%E9%8C%84) (22)
-   [學習](https://teddy-chen-tw.blogspot.com/search/label/%E5%AD%B8%E7%BF%92) (92)
-   [貓](https://teddy-chen-tw.blogspot.com/search/label/%E8%B2%93) (15)
-   [還少一本書](https://teddy-chen-tw.blogspot.com/search/label/%E9%82%84%E5%B0%91%E4%B8%80%E6%9C%AC%E6%9B%B8) (36)
-   [agile](https://teddy-chen-tw.blogspot.com/search/label/agile) (638)
-   [C. C. Agile](https://teddy-chen-tw.blogspot.com/search/label/C.%20C.%20Agile) (33)
-   [DDD](https://teddy-chen-tw.blogspot.com/search/label/DDD) (85)
-   [DevOps](https://teddy-chen-tw.blogspot.com/search/label/DevOps) (3)
-   [Eclipse](https://teddy-chen-tw.blogspot.com/search/label/Eclipse) (6)
-   [exception handling](https://teddy-chen-tw.blogspot.com/search/label/exception%20handling) (84)
-   [HCI](https://teddy-chen-tw.blogspot.com/search/label/HCI) (44)
-   [Implementation Patterns](https://teddy-chen-tw.blogspot.com/search/label/Implementation%20Patterns) (5)
-   [Kanban](https://teddy-chen-tw.blogspot.com/search/label/Kanban) (79)
-   [Mobile](https://teddy-chen-tw.blogspot.com/search/label/Mobile) (2)
-   [Patterns](https://teddy-chen-tw.blogspot.com/search/label/Patterns) (198)
-   [Programming](https://teddy-chen-tw.blogspot.com/search/label/Programming) (54)
-   [Refactoring](https://teddy-chen-tw.blogspot.com/search/label/Refactoring) (65)
-   [Scrum](https://teddy-chen-tw.blogspot.com/search/label/Scrum) (461)
-   [TDD/BDD](https://teddy-chen-tw.blogspot.com/search/label/TDD%2FBDD) (54)

## 網誌存檔

-   [►](javascript:void\(0\))  [2025](https://teddy-chen-tw.blogspot.com/2025/) (1)
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2025/06/) (1)

-   [►](javascript:void\(0\))  [2024](https://teddy-chen-tw.blogspot.com/2024/) (7)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2024/09/) (6)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2024/03/) (1)

-   [►](javascript:void\(0\))  [2023](https://teddy-chen-tw.blogspot.com/2023/) (25)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2023/09/) (5)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2023/07/) (3)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2023/06/) (5)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2023/03/) (10)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2023/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2023/01/) (1)

-   [►](javascript:void\(0\))  [2022](https://teddy-chen-tw.blogspot.com/2022/) (29)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2022/10/) (1)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2022/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2022/07/) (18)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2022/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2022/03/) (2)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2022/02/) (1)

-   [►](javascript:void\(0\))  [2021](https://teddy-chen-tw.blogspot.com/2021/) (27)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2021/12/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2021/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2021/09/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2021/08/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2021/06/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2021/04/) (1)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2021/03/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2021/01/) (8)

-   [►](javascript:void\(0\))  [2020](https://teddy-chen-tw.blogspot.com/2020/) (32)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2020/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2020/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2020/10/) (1)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2020/09/) (4)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2020/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2020/07/) (2)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2020/06/) (2)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2020/05/) (4)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2020/04/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2020/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2020/02/) (3)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2020/01/) (1)

-   [►](javascript:void\(0\))  [2019](https://teddy-chen-tw.blogspot.com/2019/) (50)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2019/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2019/11/) (2)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2019/10/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2019/07/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2019/06/) (1)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2019/05/) (8)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2019/04/) (2)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2019/03/) (8)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2019/02/) (8)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2019/01/) (7)

-   [▼](javascript:void\(0\))  [2018](https://teddy-chen-tw.blogspot.com/2018/) (28)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2018/11/) (1)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2018/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2018/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2018/08/) (6)
    
    -   [▼](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2018/07/) (6)
        -   [阿嬤覺得你冷](https://teddy-chen-tw.blogspot.com/2018/07/blog-post.html)
        -   [Clean Architecture（6）：架構三原則三部曲—跨層原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture6.html)
        -   [Clean Architecture（5）：架構三原則二部曲—相依性原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture5.html)
        -   [Clean Architecture（4）：架構三原則首部曲—分層原則](https://teddy-chen-tw.blogspot.com/2018/07/clean-architecture4.html)
        -   [LG gram 15Z980-R.AAS9U1 (2018) Amazon 購買與使用心得（下）](https://teddy-chen-tw.blogspot.com/2018/07/lg-gram-15z980-raas9u1-2018-amazon_3.html)
        -   [LG gram 15Z980-R.AAS9U1 (2018) Amazon 購買與使用心得（上）](https://teddy-chen-tw.blogspot.com/2018/07/lg-gram-15z980-raas9u1-2018-amazon.html)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2018/06/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2018/03/) (3)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2018/02/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2018/01/) (3)

-   [►](javascript:void\(0\))  [2017](https://teddy-chen-tw.blogspot.com/2017/) (142)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2017/12/) (12)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2017/11/) (8)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2017/10/) (3)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2017/09/) (2)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2017/08/) (15)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2017/07/) (10)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2017/06/) (14)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2017/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2017/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2017/03/) (23)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2017/02/) (16)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2017/01/) (25)

-   [►](javascript:void\(0\))  [2016](https://teddy-chen-tw.blogspot.com/2016/) (301)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2016/12/) (14)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2016/11/) (19)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2016/10/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2016/09/) (18)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2016/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2016/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2016/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2016/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2016/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2016/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2016/02/) (29)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2016/01/) (31)

-   [►](javascript:void\(0\))  [2015](https://teddy-chen-tw.blogspot.com/2015/) (366)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2015/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2015/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2015/10/) (32)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2015/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2015/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2015/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2015/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2015/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2015/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2015/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2015/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2015/01/) (31)

-   [►](javascript:void\(0\))  [2014](https://teddy-chen-tw.blogspot.com/2014/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2014/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2014/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2014/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2014/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2014/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2014/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2014/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2014/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2014/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2014/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2014/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2014/01/) (31)

-   [►](javascript:void\(0\))  [2013](https://teddy-chen-tw.blogspot.com/2013/) (365)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2013/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2013/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2013/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2013/09/) (30)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2013/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2013/07/) (31)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2013/06/) (30)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2013/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2013/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2013/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2013/02/) (28)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2013/01/) (31)

-   [►](javascript:void\(0\))  [2012](https://teddy-chen-tw.blogspot.com/2012/) (373)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2012/12/) (31)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2012/11/) (30)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2012/10/) (31)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2012/09/) (31)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2012/08/) (31)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2012/07/) (32)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2012/06/) (31)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2012/05/) (31)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2012/04/) (30)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2012/03/) (31)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2012/02/) (32)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2012/01/) (32)

-   [►](javascript:void\(0\))  [2011](https://teddy-chen-tw.blogspot.com/2011/) (120)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2011/12/) (16)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2011/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2011/10/) (4)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2011/09/) (3)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2011/08/) (9)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2011/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2011/06/) (10)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2011/05/) (6)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2011/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2011/03/) (16)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2011/02/) (12)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2011/01/) (11)

-   [►](javascript:void\(0\))  [2010](https://teddy-chen-tw.blogspot.com/2010/) (104)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2010/12/) (4)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2010/11/) (6)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2010/10/) (10)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2010/09/) (5)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2010/08/) (4)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2010/07/) (15)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2010/06/) (19)
    
    -   [►](javascript:void\(0\))  [5月](https://teddy-chen-tw.blogspot.com/2010/05/) (12)
    
    -   [►](javascript:void\(0\))  [4月](https://teddy-chen-tw.blogspot.com/2010/04/) (8)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2010/03/) (12)
    
    -   [►](javascript:void\(0\))  [2月](https://teddy-chen-tw.blogspot.com/2010/02/) (2)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2010/01/) (7)

-   [►](javascript:void\(0\))  [2009](https://teddy-chen-tw.blogspot.com/2009/) (31)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2009/12/) (5)
    
    -   [►](javascript:void\(0\))  [11月](https://teddy-chen-tw.blogspot.com/2009/11/) (10)
    
    -   [►](javascript:void\(0\))  [10月](https://teddy-chen-tw.blogspot.com/2009/10/) (7)
    
    -   [►](javascript:void\(0\))  [8月](https://teddy-chen-tw.blogspot.com/2009/08/) (1)
    
    -   [►](javascript:void\(0\))  [7月](https://teddy-chen-tw.blogspot.com/2009/07/) (8)

-   [►](javascript:void\(0\))  [2008](https://teddy-chen-tw.blogspot.com/2008/) (6)
    
    -   [►](javascript:void\(0\))  [9月](https://teddy-chen-tw.blogspot.com/2008/09/) (3)
    
    -   [►](javascript:void\(0\))  [3月](https://teddy-chen-tw.blogspot.com/2008/03/) (1)
    
    -   [►](javascript:void\(0\))  [1月](https://teddy-chen-tw.blogspot.com/2008/01/) (2)

-   [►](javascript:void\(0\))  [2007](https://teddy-chen-tw.blogspot.com/2007/) (4)
    
    -   [►](javascript:void\(0\))  [12月](https://teddy-chen-tw.blogspot.com/2007/12/) (1)
    
    -   [►](javascript:void\(0\))  [6月](https://teddy-chen-tw.blogspot.com/2007/06/) (3)

## 追蹤者

## 最新回應

[載入中…](http://teddy-chen-tw.blogspot.tw/feeds/comments/default)

## 熱門文章

-   [爭什麼，摻在一起做成撒尿牛丸啊](https://teddy-chen-tw.blogspot.com/2012/03/blog-post_20.html)
    
    March 20 13:53~14:56 (影像節錄自http://www.youtube.com/watch?v=LGV5PcM9SQo) 鄉民們都看過食神這部電影吧，電影中有一個經典的橋段，就是火鷄姐及鵝頭在爭辯誰做的撒尿蝦好吃，誰做的牛丸比較優。正當雙方吵得不可...
    
-   [擁抱改變，遠離Hard Code](https://teddy-chen-tw.blogspot.com/2018/09/hard-code.html)
    
    September 19 08:37~09:12 ▲圖片來源 在此 寫死（Hard Code） 幾天前一位朋友在Facebook上發問：「如何對非資訊人員解釋寫死（hard code）？」 維基百科 對於寫死的解釋如下： 寫死 （英語： Hard Code 或 Hard Co...
    
-   [什麼是物件導向（3）：Polymorphism](https://teddy-chen-tw.blogspot.com/2012/01/3polymorphism.html)
    
    January 26 23:01~January 27 00:23 Polymorphism，中文翻譯成「多型」，說真的這個概念當年Teddy學OO的時候搞了好久才弄懂，後來看了「課本」的定義之後，覺得寫得還滿好的，請參考Object-Oriented Software En...
    
-   [幫Macbook Pro 15” Late 2016尋找外接4K 27”螢幕](https://teddy-chen-tw.blogspot.com/2017/07/macbook-pro-15-late-20164k-27.html)
    
    June 17 21:28~23:24 先打個廣告，【 敏捷開發懶人包：物件導向技能 】確定於8月19日（週六）開課。 課程設定目標希望讓鄉民們在找工作面試的時候，如果面試官問到物件導向技術方面的問題，能夠具備輕鬆回答的能力 。 \*\*\* 動心起念 去年底買了MacBook P...
    
-   [Top-down和Bottom-up設計方法](https://teddy-chen-tw.blogspot.com/2014/03/top-downbottom-up.html)
    
    Mar. 12 08:40~10:10 Top-down（由上而下）和Bottom-up（由下而上）是兩種設計與解決問題的技巧。前者對問題先有一個整體的概念，然後再逐步加上設計細節，最後讓整體的輪廓越來越清楚。後者則是先將解決問題可能所需的基本元件、方案給準備好，然後再將這...
    
-   [羅技Logitech R500 雷射簡報筆開箱](https://teddy-chen-tw.blogspot.com/2018/06/logitech-r500.html)
    
    June 22 12:51~14:16 需求分析 因為工作的關係Teddy經常需要使用簡報筆，雜牌的不算，光是羅技的簡報筆Teddy就買過以下四種（R500是最新上市產品，也是今天介紹的主角）  ： ▼畫面節錄自羅技官網 Teddy心中理想的簡報筆應該有以下幾個功能： 重量輕且...
    

## 分析

 

技術提供：[Blogger](https://draft.blogger.com).
```
```

---

## Assistant

_Thinking:_

Now I have read all 6 articles from Teddy Chen's blog. Let me summarize the key insights I found and update the layered-architecture.md file with more detailed information.

Key insights from the articles:

1. **Clean Architecture (1): Software Architecture Definition and Purpose**
   - Software architecture is the "shape" given to a system by those who build it
   - Architecture consists of three elements: components, arrangement, and communication
   - The purpose of architecture is to help: development, deployment, operation, and maintenance
   - Ultimate goals: minimize total software lifecycle cost, maximize programmer productivity
   - Good architecture should reflect the system's functionality

2. **Clean Architecture (2): Port and Adapter Architecture**
   - Port and Adapter is also known as Hexagonal Architecture
   - Different diagrams show Port/Adapter positions differently based on perspective
   - From client perspective: client → Port → Adapter → Use Case
   - From Use Case perspective: Use Case → Output Port → Adapter → External System
   - JDBC drivers and REST API entry points are more like "ports" - they enable connection
   - Adapters are responsible for "converting interfaces/data"
   - The software architecture can be continuously chained with multiple Port and Adapter sets

3. **Clean Architecture (3): Selected Reference Examples**
   - Learning points when reviewing examples:
     - Project structure: single project or multiple projects?
     - What entities exist?
     - How are use case bidirectional interfaces implemented?
     - Does use case use Command pattern?
     - How to handle cross-layer objects?
     - Does it follow dependency rule?
     - Any cross-layer references from outer to inner?
     - How is Repository implemented?

4. **Clean Architecture (4): Layering Principle**
   - The key definition is: "The further a component layer is from I/O, the higher its level; the closer to I/O, the lower its level"
   - Port and Adapter draws architecture as a hexagon, not vertical
   - Clean Architecture draws it as concentric circles
   - Entities = highest level (yolk), Frameworks & Drivers = lowest level (outermost)
   - Entities: core business logic (domain model objects)
   - Use Cases: application logic, controllers calling Entities or Repository
   - Interface Adapters: converts external data and call interfaces to call Use Cases
   - Frameworks and Drivers: contains application frameworks (Spring), databases, MVC frameworks

I see a systematic approach to architectural design that emphasizes separation of concerns, modularity, and clear communication between different system components. The layered principle ensures that high-level business logic remains independent of low-level implementation details, creating a flexible and maintainable software architecture.

Dependency management requires careful navigation of technical constraints. I'll strategically handle framework interactions by defining abstract interfaces and using design patterns like Repository to create clean boundaries between architectural layers. This approach allows for easier testing, swapping implementations, and maintaining system flexibility.

The key challenges involve managing cross-layer dependencies, particularly when persistence frameworks introduce coupling. By creating additional mapping layers and defining clear interface contracts, I can mitigate direct dependencies and maintain architectural integrity.

When transferring objects between layers, I must ensure proper type conversion and avoid violating separation of concerns. The goal is to create clean interfaces that prevent unintended cross-layer interactions while maintaining clear communication pathways.

Specific challenges emerge when transferring data structures like JSON between application layers. Direct transmission risks introducing framework-specific dependencies, which compromises the architectural design. I'll implement an input port mechanism that abstracts away framework-specific details and enables clean, decoupled object transformation.

The presenter pattern plays a critical role in managing output interfaces, providing a structured approach to transferring results from use cases to the appropriate presentation layer components.

By defining clear input and output interfaces, I can ensure type safety, reduce coupling, and maintain a clean architectural boundary between different system layers.

Based on my analysis of all 6 articles from Teddy Chen's blog, I'll now add the detailed insights to the layered-architecture.md file.

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "limit": 300,
  "offset": 1
}
```
**Output:**
```
<file>
00002| 
00003| ## Overview
00004| 
00005| Layered Architecture, also known as **Hexagonal Architecture** (Ports and Adapters) or **Clean Architecture**, is a software design philosophy that emphasizes the separation of concerns through architectural layers. The core principle is that business rules and application logic should remain independent of external frameworks, databases, delivery mechanisms, and user interfaces.
00006| 
00007| This architectural pattern was formalized by Alistair Cockburn as Hexagonal Architecture and further developed by Robert C. Martin (Uncle Bob) as Clean Architecture. Both approaches share the same fundamental insight: business logic should not depend on databases, web frameworks, or any external system. Instead, external systems depend on abstractions defined by the business logic. This inversion of dependencies ensures that the core application can be tested independently of its delivery mechanisms and that external systems can be replaced without changing business code.
00008| 
00009| The architecture creates clear boundaries between what the application does (core/domain) and how it integrates with the outside world (adapters). This separation enables teams to work on different parts independently, allows technology choices to be deferred, and ensures that core business logic outlives specific technology choices.
00010| 
00011| ---
00012| 
00013| ## Core Principles
00014| 
00015| ### The Dependency Rule
00016| 
00017| The most critical rule in layered architectures is that **dependencies always point inward** toward the domain. This is the Dependency Inversion Principle applied at the architectural level. Inner layers do not know about outer layers, and all source code dependencies point toward the more abstract inner layers.
00018| 
00019| ```
00020|                     ┌───────────────────────────────────────┐
00021|                     │          External Systems             │  ← Outermost Layer
00022|                     │  (Web, DB, UI, External Services)    │
00023|                     └───────────────────┬───────────────────┘
00024|                                         │
00025|                     ┌───────────────────▼───────────────────┐
00026|                     │            Adapters Layer             │  ← Interface Adapters
00027|                     │  (Controllers, Gateways, Presenters,  │
00028|                     │   Database Adapters, External Clients)│
00029|                     └───────────────────┬───────────────────┘
00030|                                         │
00031|                     ┌───────────────────▼───────────────────┐
00032|                     │         Application Layer             │  ← Use Cases / Services
00033|                     │  (Interactors, Use Case Coordinators, │
00034|                     │   Application Services)               │
00035|                     └───────────────────┬───────────────────┘
00036|                                         │
00037|                     ┌───────────────────▼───────────────────┐
00038|                     │            Domain Layer               │  ← Innermost Layer
00039|                     │  (Entities, Value Objects, Aggregates,│
00040|                     │   Domain Services, Domain Events)     │
00041|                     └───────────────────────────────────────┘
00042| 
00043|      Source code dependencies point INWARD toward the domain.
00044|      Inner layers do NOT know about outer layers.
00045| ```
00046| 
00047| ### Key Principles
00048| 
00049| 1. **Independence of Frameworks**: Business logic remains independent of frameworks and can be tested without them
00050| 2. **Testability**: Business rules are testable without UI, database, or external systems
00051| 3. **Independence of UI**: User interfaces can change without affecting business logic
00052| 4. **Independence of Database**: Business logic is decoupled from database implementation details
00053| 5. **Independence of External Agencies**: Business rules have no knowledge of the outside world
00054| 6. **Framework Independence**: Business logic survives framework changes and upgrades
00055| 
00056| ---
00057| 
00058| ## Architecture Overview
00059| 
00060| ```
00061| ┌─────────────────────────────────────────────────────────────────────────┐
00062| │                    LAYERED ARCHITECTURE                                 │
00063| ├─────────────────────────────────────────────────────────────────────────┤
00064| │                                                                         │
00065| │                         ┌─────────────────────┐                         │
00066| │                         │                     │                         │
00067| │                         │        DOMAIN       │                         │
00068| │                         │        LAYER        │                         │
00069| │                         │                     │                         │
00070| │                         │  ┌───────────────┐  │                         │
00071| │                         │  │   Entities    │  │                         │
00072| │                         │  │  Value Objects│  │                         │
00073| │                         │  │  Aggregates   │  │                         │
00074| │                         │  │  Domain Events│  │                         │
00075| │                         │  └───────────────┘  │                         │
00076| │                         │                     │                         │
00077| │                         └──────────┬──────────┘                         │
00078| │                                    │                                    │
00079| │                                    ▼                                    │
00080| │                         ┌─────────────────────┐                         │
00081| │                         │     APPLICATION     │                         │
00082| │                         │        LAYER        │                         │
00083| │                         │                     │                         │
00084| │                         │  ┌───────────────┐  │                         │
00085| │                         │  │   Use Cases   │  │                         │
00086| │                         │  │   Services    │  │                         │
00087| │                         │  │   Ports       │  │                         │
00088| │                         │  └───────────────┘  │                         │
00089| │                         │                     │                         │
00090| │                         └──────────┬──────────┘                         │
00091| │                                    │                                    │
00092| │              ┌─────────────────────┼─────────────────────┐              │
00093| │              │                     │                     │              │
00094| │              ▼                     ▼                     ▼              │
00095| │    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │
00096| │    │     PORTS       │  │                 │  │     PORTS       │       │
00097| │    │   (Interfaces)  │  │                 │  │   (Interfaces)  │       │
00098| │    │                 │  │                 │  │                 │       │
00099| │    │  • Input Ports  │  │                 │  │  • Output Ports │       │
00100| │    │    (Use Cases)  │  │                 │  │  • Repositories │       │
00101| │    │                 │  │                 │  │  • Presenters   │       │
00102| │    └────────┬────────┘  │                 │  └────────┬────────┘       │
00103| │             │           │                 │           │                │
00104| │             │           │                 │           │                │
00105| │             ▼           │                 │           ▼                │
00106| │    ┌─────────────────┐  │                 │  ┌─────────────────┐       │
00107| │    │   ADAPTERS      │  │                 │  │   ADAPTERS      │       │
00108| │    │   (Inbound)     │  │                 │  │   (Outbound)    │       │
00109| │    │                 │  │                 │  │                 │       │
00110| │    │  • Web/HTTP     │  │                 │  │  • Database     │       │
00111| │    │  • Console/CLI  │  │                 │  │  • File System  │       │
00112| │    │  • API/Gateway  │  │                 │  │  • External APIs│       │
00113| │    │  • Messaging    │  │                 │  │  • Messaging    │       │
00114| │    │  • Mobile       │  │                 │  │  • Event Bus    │       │
00115| │    └─────────────────┘  │                 │  └─────────────────┘       │
00116| │                         │                 │                            │
00117| └─────────────────────────┼─────────────────┼────────────────────────────┘
00118|                           │                 │
00119|                           ▼                 ▼
00120|               ┌───────────────────┐ ┌───────────────────┐
00121|               │    EXTERNAL       │ │    EXTERNAL       │
00122|               │    SYSTEMS        │ │    SYSTEMS        │
00123|               │                   │ │                   │
00124|               │  • Web Browsers   │ │  • Databases      │
00125|               │  • Mobile Apps    │ │  • File Systems   │
00126|               │  • CLI Users      │ │  • Web Services   │
00127|               │  • API Clients    │ │  • Message Queues │
00128|               └───────────────────┘ └───────────────────┘
00129| ```
00130| 
00131| ---
00132| 
00133| ## The Three Core Layers
00134| 
00135| ### Domain Layer (Core)
00136| 
00137| The domain layer contains the business logic of the application at its purest form. It consists of entities, value objects, aggregates, domain services, and domain events. This layer has **zero dependencies** on any other layer. It defines interfaces (ports) that describe what it needs from the outside world. The domain layer represents the heart of the application and should be completely isolated from infrastructure concerns.
00138| 
00139| The domain layer is where the most stable code resides because it contains the core business rules that rarely change, regardless of how the application is delivered or what technologies are used. This layer can be thoroughly tested without any external dependencies, making it the most reliable and valuable part of the codebase.
00140| 
00141| ### Application Layer (Use Cases)
00142| 
00143| The application layer contains application-specific business logic that orchestrates operations. It coordinates between the domain layer and external systems by using domain objects and calling through ports to interact with external concerns. This layer depends only on domain abstractions and contains use cases, application services, and input/output port definitions.
00144| 
00145| The application layer defines what the system does without caring about how it communicates with the outside world. Each use case represents a single business operation and follows a consistent pattern of validating input, performing business operations, persisting changes, and returning results.
00146| 
00147| ### Adapter Layer (External Integration)
00148| 
00149| The adapter layer contains implementations of ports that integrate with external systems. Adapters are divided into two categories: inbound adapters handle user input and external requests, while outbound adapters handle communication with external systems. Adapters depend on ports (interfaces), never the reverse.
00150| 
00151| This layer translates between external protocols and internal abstractions, allowing the core application to remain unchanged regardless of how it is accessed or what external systems it integrates with.
00152| 
00153| ---
00154| 
00155| ## Ports and Adapters Explained
00156| 
00157| ### Ports (Interfaces)
00158| 
00159| Ports define the boundary between the application core and external systems. They are abstract contracts that specify what the application expects from and provides to the outside world. Ports create a stable interface that shields the core from changes in external systems.
00160| 
00161| **Input Ports (Use Case Interfaces)**
00162| 
00163| Input ports define operations that external systems can perform on the application. They represent the application's API for incoming operations. These ports are implemented by application services and called by inbound adapters.
00164| 
00165| ```typescript
00166| // Input Port - What the outside world can DO
00167| interface CreateOrderUseCase {
00168|   execute(input: CreateOrderInput): Promise<CreateOrderOutput>;
00169| }
00170| 
00171| interface GetOrderUseCase {
00172|   execute(input: GetOrderInput): Promise<GetOrderOutput>;
00173| }
00174| 
00175| interface ProcessPaymentUseCase {
00176|   execute(input: ProcessPaymentInput): Promise<ProcessPaymentOutput>;
00177| }
00178| 
00179| interface ShipOrderUseCase {
00180|   execute(input: ShipOrderInput): Promise<ShipOrderOutput>;
00181| }
00182| ```
00183| 
00184| **Output Ports (Repository and Service Interfaces)**
00185| 
00186| Output ports define operations that the application needs to perform on external systems. These ports are implemented by outbound adapters and called by application services and domain logic. Output ports decouple the core from specific implementations of persistence, external services, and other infrastructure concerns.
00187| 
00188| ```typescript
00189| // Output Port - What the application NEEDS from external systems
00190| interface OrderRepository {
00191|   findById(id: OrderId): Promise<Order | null>;
00192|   findByCustomerId(customerId: string): Promise<Order[]>;
00193|   save(order: Order): Promise<void>;
00194|   delete(id: OrderId): Promise<void>;
00195| }
00196| 
00197| interface InventoryService {
00198|   checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00199|   reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00200|   releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00201| }
00202| 
00203| interface PaymentGateway {
00204|   processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;
00205|   refundPayment(transactionId: string): Promise<RefundResult>;
00206| }
00207| 
00208| interface EventPublisher {
00209|   publish(event: DomainEvent): Promise<void>;
00210|   publishAll(events: DomainEvent[]): Promise<void>;
00211| }
00212| 
00213| interface NotificationService {
00214|   sendNotification(customerId: string, message: string): Promise<void>;
00215| }
00216| ```
00217| 
00218| ### Adapters (Implementations)
00219| 
00220| Adapters implement ports and handle the specifics of interacting with external systems. They translate between external protocols and internal abstractions, keeping the core completely unaware of how external communication happens.
00221| 
00222| **Inbound Adapters (Delivery Mechanisms)**
00223| 
00224| Inbound adapters translate external input into calls to input ports. They handle the specifics of receiving requests from various delivery mechanisms and translating them into a format the application can understand.
00225| 
00226| ```typescript
00227| // HTTP Controller (Inbound Adapter)
00228| class CreateOrderController {
00229|   constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}
00230| 
00231|   async handleRequest(request: HttpRequest): Promise<HttpResponse> {
00232|     try {
00233|       const input: CreateOrderInput = {
00234|         customerId: request.body.customerId,
00235|         items: request.body.items.map((item: any) => ({
00236|           productId: item.productId,
00237|           quantity: item.quantity
00238|         })),
00239|         shippingAddress: request.body.shippingAddress,
00240|         paymentMethod: request.body.paymentMethod
00241|       };
00242| 
00243|       const result = await this.createOrderUseCase.execute(input);
00244| 
00245|       return new HttpResponse(
00246|         result.isSuccess ? 201 : 400,
00247|         {
00248|           success: result.isSuccess,
00249|           data: result.isSuccess ? result.orderId : null,
00250|           error: result.isSuccess ? null : result.error
00251|         }
00252|       );
00253|     } catch (error) {
00254|       return new HttpResponse(
00255|         500,
00256|         {
00257|           success: false,
00258|           error: error instanceof Error ? error.message : 'Unknown error'
00259|         }
00260|       );
00261|     }
00262|   }
00263| }
00264| 
00265| // gRPC Controller (Inbound Adapter)
00266| class OrderGrpcController {
00267|   constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}
00268| 
00269|   async createOrder(call: ServerUnaryCall<CreateOrderRequest>): Promise<CreateOrderResponse> {
00270|     const input: CreateOrderInput = {
00271|       customerId: call.request.customerId,
00272|       items: call.request.items.map(item => ({
00273|         productId: item.productId,
00274|         quantity: item.quantity
00275|       }))
00276|     };
00277| 
00278|     const result = await this.createOrderUseCase.execute(input);
00279| 
00280|     return {
00281|       success: result.isSuccess,
00282|       orderId: result.isSuccess ? result.orderId : '',
00283|       error: result.error || ''
00284|     };
00285|   }
00286| }
00287| 
00288| // Message Queue Consumer (Inbound Adapter)
00289| class OrderMessageHandler {
00290|   constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}
00291| 
00292|   async handleMessage(message: OrderMessage): Promise<void> {
00293|     const input: CreateOrderInput = {
00294|       customerId: message.customerId,
00295|       items: message.items
00296|     };
00297| 
00298|     const result = await this.createOrderUseCase.execute(input);
00299| 
00300|     if (!result.isSuccess) {
00301|       throw new Error(`Failed to process order: ${result.error}`);

(File has more lines. Use 'offset' parameter to read beyond line 301)
</file>
```
```

---

## Assistant

_Thinking:_

The file is quite long. Let me find the section where I added the "Three Fundamental Principles" section and enhance it with more detailed information from all 6 articles. I should edit the file to add more comprehensive content about:

1. Software architecture definition and purpose
2. Port and Adapter architecture perspectives
3. Reference examples learning points
4. Detailed layer explanations
5. Dependency principle implementation challenges and solutions
6. Cross-layer principle with bidirectional interfaces
7. The relationship between all these concepts

Let me find the right section to edit.

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "limit": 150,
  "offset": 130
}
```
**Output:**
```
<file>
00131| ---
00132| 
00133| ## The Three Core Layers
00134| 
00135| ### Domain Layer (Core)
00136| 
00137| The domain layer contains the business logic of the application at its purest form. It consists of entities, value objects, aggregates, domain services, and domain events. This layer has **zero dependencies** on any other layer. It defines interfaces (ports) that describe what it needs from the outside world. The domain layer represents the heart of the application and should be completely isolated from infrastructure concerns.
00138| 
00139| The domain layer is where the most stable code resides because it contains the core business rules that rarely change, regardless of how the application is delivered or what technologies are used. This layer can be thoroughly tested without any external dependencies, making it the most reliable and valuable part of the codebase.
00140| 
00141| ### Application Layer (Use Cases)
00142| 
00143| The application layer contains application-specific business logic that orchestrates operations. It coordinates between the domain layer and external systems by using domain objects and calling through ports to interact with external concerns. This layer depends only on domain abstractions and contains use cases, application services, and input/output port definitions.
00144| 
00145| The application layer defines what the system does without caring about how it communicates with the outside world. Each use case represents a single business operation and follows a consistent pattern of validating input, performing business operations, persisting changes, and returning results.
00146| 
00147| ### Adapter Layer (External Integration)
00148| 
00149| The adapter layer contains implementations of ports that integrate with external systems. Adapters are divided into two categories: inbound adapters handle user input and external requests, while outbound adapters handle communication with external systems. Adapters depend on ports (interfaces), never the reverse.
00150| 
00151| This layer translates between external protocols and internal abstractions, allowing the core application to remain unchanged regardless of how it is accessed or what external systems it integrates with.
00152| 
00153| ---
00154| 
00155| ## Ports and Adapters Explained
00156| 
00157| ### Ports (Interfaces)
00158| 
00159| Ports define the boundary between the application core and external systems. They are abstract contracts that specify what the application expects from and provides to the outside world. Ports create a stable interface that shields the core from changes in external systems.
00160| 
00161| **Input Ports (Use Case Interfaces)**
00162| 
00163| Input ports define operations that external systems can perform on the application. They represent the application's API for incoming operations. These ports are implemented by application services and called by inbound adapters.
00164| 
00165| ```typescript
00166| // Input Port - What the outside world can DO
00167| interface CreateOrderUseCase {
00168|   execute(input: CreateOrderInput): Promise<CreateOrderOutput>;
00169| }
00170| 
00171| interface GetOrderUseCase {
00172|   execute(input: GetOrderInput): Promise<GetOrderOutput>;
00173| }
00174| 
00175| interface ProcessPaymentUseCase {
00176|   execute(input: ProcessPaymentInput): Promise<ProcessPaymentOutput>;
00177| }
00178| 
00179| interface ShipOrderUseCase {
00180|   execute(input: ShipOrderInput): Promise<ShipOrderOutput>;
00181| }
00182| ```
00183| 
00184| **Output Ports (Repository and Service Interfaces)**
00185| 
00186| Output ports define operations that the application needs to perform on external systems. These ports are implemented by outbound adapters and called by application services and domain logic. Output ports decouple the core from specific implementations of persistence, external services, and other infrastructure concerns.
00187| 
00188| ```typescript
00189| // Output Port - What the application NEEDS from external systems
00190| interface OrderRepository {
00191|   findById(id: OrderId): Promise<Order | null>;
00192|   findByCustomerId(customerId: string): Promise<Order[]>;
00193|   save(order: Order): Promise<void>;
00194|   delete(id: OrderId): Promise<void>;
00195| }
00196| 
00197| interface InventoryService {
00198|   checkAvailability(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00199|   reserveItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00200|   releaseItems(items: Array<{ productId: string; quantity: number }>): Promise<void>;
00201| }
00202| 
00203| interface PaymentGateway {
00204|   processPayment(amount: Money, paymentMethod: PaymentMethod): Promise<PaymentResult>;
00205|   refundPayment(transactionId: string): Promise<RefundResult>;
00206| }
00207| 
00208| interface EventPublisher {
00209|   publish(event: DomainEvent): Promise<void>;
00210|   publishAll(events: DomainEvent[]): Promise<void>;
00211| }
00212| 
00213| interface NotificationService {
00214|   sendNotification(customerId: string, message: string): Promise<void>;
00215| }
00216| ```
00217| 
00218| ### Adapters (Implementations)
00219| 
00220| Adapters implement ports and handle the specifics of interacting with external systems. They translate between external protocols and internal abstractions, keeping the core completely unaware of how external communication happens.
00221| 
00222| **Inbound Adapters (Delivery Mechanisms)**
00223| 
00224| Inbound adapters translate external input into calls to input ports. They handle the specifics of receiving requests from various delivery mechanisms and translating them into a format the application can understand.
00225| 
00226| ```typescript
00227| // HTTP Controller (Inbound Adapter)
00228| class CreateOrderController {
00229|   constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}
00230| 
00231|   async handleRequest(request: HttpRequest): Promise<HttpResponse> {
00232|     try {
00233|       const input: CreateOrderInput = {
00234|         customerId: request.body.customerId,
00235|         items: request.body.items.map((item: any) => ({
00236|           productId: item.productId,
00237|           quantity: item.quantity
00238|         })),
00239|         shippingAddress: request.body.shippingAddress,
00240|         paymentMethod: request.body.paymentMethod
00241|       };
00242| 
00243|       const result = await this.createOrderUseCase.execute(input);
00244| 
00245|       return new HttpResponse(
00246|         result.isSuccess ? 201 : 400,
00247|         {
00248|           success: result.isSuccess,
00249|           data: result.isSuccess ? result.orderId : null,
00250|           error: result.isSuccess ? null : result.error
00251|         }
00252|       );
00253|     } catch (error) {
00254|       return new HttpResponse(
00255|         500,
00256|         {
00257|           success: false,
00258|           error: error instanceof Error ? error.message : 'Unknown error'
00259|         }
00260|       );
00261|     }
00262|   }
00263| }
00264| 
00265| // gRPC Controller (Inbound Adapter)
00266| class OrderGrpcController {
00267|   constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}
00268| 
00269|   async createOrder(call: ServerUnaryCall<CreateOrderRequest>): Promise<CreateOrderResponse> {
00270|     const input: CreateOrderInput = {
00271|       customerId: call.request.customerId,
00272|       items: call.request.items.map(item => ({
00273|         productId: item.productId,
00274|         quantity: item.quantity
00275|       }))
00276|     };
00277| 
00278|     const result = await this.createOrderUseCase.execute(input);
00279| 
00280|     return {

(File has more lines. Use 'offset' parameter to read beyond line 280)
</file>
```
```

---

## Assistant

_Thinking:_

Let me find the section I added earlier about the "Three Fundamental Principles" to enhance it with more detailed content from all 6 articles.

```
Tool: grep

**Input:**
```json
{
  "path": "E:\\AI-Learn\\ai-coding-prompts\\doc",
  "pattern": "The Three Fundamental Principles"
}
```
**Output:**
```
Found 1 matches
E:\AI-Learn\ai-coding-prompts\doc\layered-architecture.md:
  Line 1582: ## The Three Fundamental Principles
```
```

---

## Assistant

_Thinking:_

Let me read the section starting from line 1582 to see what I added earlier and then enhance it with more detailed information from all 6 articles.

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "limit": 200,
  "offset": 1580
}
```
**Output:**
```
<file>
01581| 
01582| ## The Three Fundamental Principles
01583| 
01584| Layered Architecture is governed by three fundamental principles that guide the organization of code and dependencies. These principles, when properly applied, create a clean separation between business logic and external concerns.
01585| 
01586| ### 1. Layering Principle (分層原則)
01587| 
01588| The architecture is divided into concentric layers, each with a specific responsibility and level of abstraction. The most important principle of layering is that each layer has a clear purpose and that objects within each layer know only about their own layer and the layers inside it.
01589| 
01590| **The Four Layers:**
01591| 
01592| - **Entities Layer (Innermost)**: Contains domain model objects including Aggregates, Entities, Value Objects, and Domain Services following Domain-Driven Design (DDD) patterns. This layer represents the core business logic and should be completely independent of other layers.
01593| 
01594| - **Use Cases Layer**: Contains application-specific business logic, corresponding to DDD's Application Services. This layer orchestrates domain objects to perform specific business operations and defines input/output port interfaces.
01595| 
01596| - **Interface Adapters Layer**: Contains adapters that translate between external systems and the application core. This includes controllers, gateways, presenters, and database adapters that implement the ports defined in the inner layers.
01597| 
01598| - **Frameworks & Drivers Layer (Outermost)**: Contains external frameworks, tools, and delivery mechanisms such as Web frameworks, database systems, external services, and UI frameworks.
01599| 
01600| **Placement Considerations:**
01601| 
01602| While the four layers are clearly separated in principle, some objects fall into gray areas during implementation. For example, the placement of DDD Repositories and Mappers can be debated. Some practitioners place them in the Use Cases layer, while others place them closer to the Entities layer. The key is to maintain consistency within the codebase and ensure that the dependency rules are respected.
01603| 
01604| ```
01605| Layer Placement Decision Guide:
01606| 
01607| ┌─────────────────────────────────────────────────────────────┐
01608| │                  FRAMEWORKS & DRIVERS                       │
01609| │    Web frameworks, Database drivers, External libraries     │
01610| └─────────────────────────────────────────────────────────────┘
01611|                               │
01612| ┌─────────────────────────────────────────────────────────────┐
01613| │                  INTERFACE ADAPTERS                          │
01614| │    Controllers, Presenters, Gateway implementations,        │
01615| │    Mappers, Serializers                                     │
01616| └─────────────────────────────────────────────────────────────┘
01617|                               │
01618| ┌─────────────────────────────────────────────────────────────┐
01619| │                      USE CASES                              │
01620| │    Application services, Use case handlers,                 │
01621| │    Port interfaces (Input/Output), DTOs                     │
01622| └─────────────────────────────────────────────────────────────┘
01623|                               │
01624| ┌─────────────────────────────────────────────────────────────┐
01625| │                       ENTITIES                               │
01626| │    Aggregates, Entities, Value Objects, Domain Services,    │
01627| │    Domain Events, Domain Factories                          │
01628| └─────────────────────────────────────────────────────────────┘
01629| ```
01630| 
01631| ### 2. Dependency Principle (相依性原則)
01632| 
01633| Source code dependencies must always point **inward** from outer layers to inner layers. This means that code in outer layers may depend on abstractions defined in inner layers, but inner layers must never depend on outer layers. This principle is the cornerstone of clean architecture and is achieved through the **Dependency Inversion Principle (DIP)**.
01634| 
01635| **Dependency Rules:**
01636| 
01637| - Objects in the Entities layer can only reference other objects within the Entities layer
01638| - Objects in the Use Cases layer can only reference the Entities layer and other objects within the Use Cases layer
01639| - Objects in the Interface Adapters layer can reference both Use Cases and Entities layers (through defined ports)
01640| - The Frameworks & Drivers layer can reference all inner layers through their public interfaces
01641| 
01642| **Achieving Inward Dependencies with Dependency Inversion:**
01643| 
01644| When the Use Cases layer needs to persist Aggregates to a database, it must reference database objects. However, this would violate the dependency rule since the database is in an outer layer. The solution is to define a port (interface) in the Use Cases layer and have the outer database layer implement it. This way, the dependency direction is reversed: the outer layer depends on the inner layer's abstraction.
01645| 
01646| ```typescript
01647| // Use Cases layer defines the abstraction (port)
01648| interface OrderRepository {
01649|   save(order: Order): Promise<void>;
01650|   findById(id: OrderId): Promise<Order | null>;
01651| }
01652| 
01653| // Interface Adapters layer implements the port
01654| class SqlOrderRepository implements OrderRepository {
01655|   constructor(private db: DatabaseConnection) {}
01656| 
01657|   async save(order: Order): Promise<void> {
01658|     // Implementation using SQL database
01659|     await this.db.query('INSERT INTO orders ...');
01660|   }
01661| 
01662|   async findById(id: OrderId): Promise<Order | null> {
01663|     // Implementation using SQL database
01664|     const result = await this.db.query('SELECT * FROM orders WHERE id = ?', [id.value]);
01665|     return result ? OrderMapper.toDomain(result) : null;
01666|   }
01667| }
01668| 
01669| // Dependency direction:
01670| // UseCases ──depends on──> OrderRepository (interface) ──implemented by──> SqlOrderRepository
01671| ```
01672| 
01673| **Practical Challenges:**
01674| 
01675| Completely adhering to the dependency principle can be challenging in practice. For example, when domain entities need to publish Domain Events using an event bus tool, using an external library directly would violate the dependency rule. The solution is to define an interface for the event bus and have the external library implement it through an adapter. While this adds complexity, it maintains the architectural integrity of the system.
01676| 
01677| ### 3. Cross-Layer Principle (跨層原則)
01678| 
01679| When domain objects need to cross layer boundaries to be passed outside the Use Cases layer (to UI, database, or other external systems), they must not be passed directly. Instead, interfaces and data structures should be defined in the Use Cases layer, and domain objects should be converted to DTOs (Data Transfer Objects) before crossing layers.
01680| 
01681| **Rationale:**
01682| 
01683| Directly passing domain objects outside the Use Cases layer can lead to the domain layer being affected by presentation logic or persistence requirements. For example, if a UI component needs specific data from an Order entity, the entity might be modified to accommodate UI needs, thereby contaminating the domain logic with presentation concerns. To maintain separation of concerns and single responsibility, objects crossing layers must be transformed.
01684| 
01685| ```typescript
01686| // WRONG - Domain object passed directly to UI
01687| class OrderController {
01688|   async getOrder(req: Request, res: Response): Promise<void> {
01689|     const order = await this.getOrderUseCase.execute(req.params.orderId);
01690|     res.json(order); // Direct domain object exposure
01691|   }
01692| }
01693| 
01694| // RIGHT - Domain object converted to DTO
01695| class OrderController {
01696|   async getOrder(req: Request, res: Response): Promise<void> {
01697|     const result = await this.getOrderUseCase.execute(req.params.orderId);
01698|     const dto = OrderMapper.toDto(result.order); // Convert to DTO
01699|     res.json(dto);
01700|   }
01701| }
01702| 
01703| // DTO defined in Use Cases layer
01704| interface OrderDto {
01705|   id: string;
01706|   status: string;
01707|   total: number;
01708|   items: OrderItemDto[];
01709| }
01710| ```
01711| 
01712| **Benefits of the Cross-Layer Principle:**
01713| 
01714| - **Separation of Concerns**: Domain objects remain focused on business logic, not presentation or persistence
01715| - **Flexibility**: UI and persistence can change without affecting domain objects
01716| - **Testability**: Domain objects can be tested in isolation without UI or database dependencies
01717| - **Encapsulation**: Internal domain structure is hidden from external layers
01718| 
01719| ### Practical Considerations: The "Cleanliness" Trade-off
01720| 
01721| While strict adherence to all three principles results in a clean and maintainable architecture, practical implementation often requires balancing architectural purity with development efficiency. Consider the following guidelines:
01722| 
01723| **Strict Adherence Recommended:**
01724| 
01725| - The Dependency Principle should generally be followed, especially between the Entities and Use Cases layers. When external tools are needed, define abstractions and use adapters rather than direct dependencies.
01726| 
01727| **Acceptable Relaxation:**
01728| 
01729| - The Interface Adapters layer often couples tightly with specific frameworks (e.g., Web Controllers with Web framework classes). Attempting complete dependency inversion in this layer can be overly complex and may not provide sufficient benefit.
01730| 
01731| **Practical Simplification:**
01732| 
01733| - Separating Use Cases into Commands (state-modifying operations) and Queries (read operations) using the CQRS pattern can significantly simplify the design of Interface Adapters. Commands can share common presenters and view models, while only Queries need customized output handling. This approach keeps the adapter layer lean and focused.
01734| 
01735| ---
01736| 
01737| ## Benefits of Layered Architecture
01738| 
01739| | Benefit | Description |
01740| |---------|-------------|
01741| | **Testability** | Business rules can be tested without UI, database, or external systems using pure unit tests |
01742| | **Maintainability** | Changes to outer layers (frameworks, databases) do not affect inner layers (domain, use cases) |
01743| | **Flexibility** | External systems and delivery mechanisms can be swapped without changing business code |
01744| | **Scalability** | Clear separation enables parallel development by multiple teams |
01745| | **Framework Independence** | Business logic survives framework changes and technology migrations |
01746| | **Adaptability** | Application can be delivered through multiple channels (HTTP, gRPC, CLI, messaging) |
01747| | **Longevity** | Core business logic outlives specific technology choices |
01748| | **Test doubles** | Easy to create mock implementations of ports for testing |
01749| | **Technology Deferral** | Technology choices can be deferred until the outer layers |
01750| | **Domain Focus** | Business logic remains pure and focused on domain rules |
01751| 
01752| ---
01753| 
01754| ## Anti-Patterns to Avoid
01755| 
01756| | Anti-Pattern | Description | Solution |
01757| |--------------|-------------|----------|
01758| | **Anemic Domain** | Entities with no behavior, logic in services | Move business logic from services to domain entities |
01759| | **Dependency Inversion Violation** | Domain depending on frameworks or infrastructure | Use dependency injection, define ports in domain/application |
01760| | **Feature Scattered Across Layers** | Related code in multiple layers | Organize by feature module, not by layer |
01761| | **God Interface** | One port/interface for everything | Separate inbound/outbound ports, single responsibility |
01762| | **Skipping Layers** | Controllers directly calling repositories | Always go through use cases for business operations |
01763| | **Leaking Infrastructure** | Domain entities with framework annotations | Keep domain pure, use adapters for persistence mapping |
01764| | **Concrete Dependencies** | Use cases depending on concrete implementations | Depend on abstractions (interfaces), inject implementations |
01765| | **Missing Ports** | Direct imports of infrastructure in core | Define ports, implement in adapters |
01766| 
01767| ### Common Violations and Fixes
01768| 
01769| ```typescript
01770| // WRONG - Domain depends on JPA/ORM annotations
01771| @Entity
01772| class Order {
01773|   @Id
01774|   private id: string;
01775|   
01776|   @Column
01777|   private status: string;
01778| }
01779| 
01780| // RIGHT - Pure domain entity

(File has more lines. Use 'offset' parameter to read beyond line 1780)
</file>
```
```

---

## Assistant

_Thinking:_

Now I can see the section I added earlier. I need to enhance this section with more detailed information from all 6 articles. Let me replace the "Three Fundamental Principles" section with a more comprehensive version that includes:

1. Software architecture definition and purpose (from article 1)
2. Port and Adapter perspectives (from article 2)
3. Reference examples learning points (from article 3)
4. More detailed layer explanations with I/O distance concept (from article 4)
5. Dependency principle challenges with JPA annotations example (from article 5)
6. Cross-layer principle with bidirectional interfaces and Presenters (from article 6)

Let me edit the file to replace the existing section with a more comprehensive version.

```
Tool: invalid

**Input:**
```json
{
  "tool": "edit",
  "error": "Invalid input for tool edit: JSON parsing failed: Text: {\"filePath\": \"E:\\\\AI-Learn\\\\ai-coding-prompts\\\\doc\\\\layered-architecture.md\", \"newString\": \"## The Three Fundamental Principles\\n\\nLayered Architecture is governed by three fundamental principles that guide the organization of code and dependencies. These principles, when properly applied, create a clean separation between business logic and external concerns. Understanding these principles deeply is essential for implementing a truly clean architecture.\\n\\n---\\n\\n### 0. Software Architecture: Definition and Purpose\\n\\nBefore diving into the three principles, it is important to understand what software architecture is and why it matters. Software architecture is the **shape** given to a system by those who build it. This shape is determined by three fundamental elements:\\n\\n1. **Components**: The building blocks that constitute the system\\n2. **Arrangement**: How components are organized and structured\\n3. **Communication**: How components interact with each other and with external systems\\n\\nThe purpose of architecture is to serve four key objectives:\\n\\n- **Development**: Enabling efficient and maintainable software development\\n- **Deployment**: Supporting smooth and reliable system deployment\\n- **Operation**: Facilitating system operation and monitoring\\n- **Maintenance**: Enabling easy modification and enhancement over time\\n\\n**The ultimate goals of good architecture are:**\\n\\n- **Minimize the total cost of the software lifecycle**\\n- **Maximize programmer productivity**\\n\\nA well-designed architecture reflects the system's functionality clearly. When developers examine a properly structured codebase, they should be able to understand what the system does simply by looking at its organization. For example, a web shopping system should clearly show components for displaying product lists, shopping cart management, checkout processing, payment handling, shipping selection, and order tracking.\\n\\n---\\n\\n### 1. Layering Principle (分層原則)\\n\\nThe layering principle organizes code into concentric layers based on their distance from Input/Output (I/O). This is the foundation of Clean Architecture and determines the hierarchy of components.\\n\\n**The Key Definition:**\\n\\n> \\\"The further a component layer is from I/O, the higher its level; the closer to I/O, the lower its level.\\\"\\n\\nThis definition is intuitive when compared to organizational hierarchies. In a company, receptionists who handle phone calls and visitor接待 (reception) are close to I/O and typically have lower levels. Executives who are far from daily operational I/O have higher levels. Similarly, in software architecture, components that directly handle user input, database access, or network communication are at lower levels, while pure business logic resides at higher levels.\\n\\n**The Four Layers (from outermost to innermost):**\\n\\n```\\n┌─────────────────────────────────────────────────────────────────────────┐\\n│                    FRAMEWORKS & DRIVERS                                 │\\n│  (Outermost Layer - Closest to I/O - Lowest Level)                      │\\n│                                                                         │\\n│  Contains:                                                              │\\n│  - Web frameworks (Express, Fastify, Spring MVC)                        │\\n│  - Database drivers (JDBC, TypeORM, Prisma)                             │\\n│  - UI frameworks (React, Angular, Android)                              │\\n│  - External service SDKs (Stripe, AWS SDK)                              │\\n│  - Message queue clients (Kafka, RabbitMQ)                              │\\n│                                                                         │\\n│  These are typically \\\"glue code\\\" that connects the application          │\\n│  to external frameworks and tools. Minimal business logic resides here. │\\n└─────────────────────────────────────────────────────────────────────────┘\\n                                    │\\n                                    ▼\\n┌─────────────────────────────────────────────────────────────────────────┐\\n│                    INTERFACE ADAPTERS                                    │\\n│  (Converts between external and internal interfaces)                    │\\n│                                                                         │\\n│  Contains:                                                              │\\n│  - Controllers (HTTP, gRPC, CLI handlers)                               │\\n│  - Presenters (View Models, Response formatters)                        │\\n│  - Gateway implementations (Repository implementations)                 │\\n│  - Mappers (Entity to DTO, DTO to Entity transformations)               │\\n│  - Serializers (JSON, XML, Protocol Buffers)                            │\\n│                                                                         │\\n│  Adapters translate external protocols and data formats into internal   │\\n│  abstractions that the inner layers can understand.                     │\\n└─────────────────────────────────────────────────────────────────────────┘\\n                                    │\\n                                    ▼\\n┌─────────────────────────────────────────────────────────────────────────┐\\n│                         USE CASES                                        │\\n│  (Application Business Rules - High Level)                              │\\n│                                                                         │\\n│  Contains:                                                              │\\n│  - Application services (Use case handlers)                             │\\n│  - Input port interfaces (Use case contracts)                           │\\n│  - Output port interfaces (Repository, Service contracts)               │\\n│  - Input/Output DTOs (Data Transfer Objects)                            │\\n│  - Business rule orchestrators                                          │\\n│                                                                         │\\n│  This layer defines WHAT the system does without caring HOW external    │\\n│  communication happens. Each use case represents a single business      │\\n│  operation.                                                             │\\n└─────────────────────────────────────────────────────────────────────────┘\\n                                    │\\n                                    ▼\\n┌─────────────────────────────────────────────────────────────────────────┐\\n│                         ENTITIES                                         │\\n│  (Innermost Layer - Furthest from I/O - Highest Level)                  │\\n│                                                                         │\\n│  Contains:                                                              │\\n│  - Domain entities (Aggregate Roots, Entities)                          │\\n│  - Value Objects                                                        │\\n│  - Domain Services (for complex business logic)                         │\\n│  - Domain Events                                                       │\\n│  - Domain Factories                                                    │\\n│                                                                         │\\n│  This layer contains the CORE business logic that is reusable across    │\\n│  different applications. It has ZERO dependencies on any other layer.   │\\n└─────────────────────────────────────────────────────────────────────────┘\\n```\\n\\n**Layer Responsibilities:**\\n\\n| Layer | Responsibility | I/O Distance | Level |\\n|-------|---------------|--------------|-------|\\n| Entities | Core business rules | Farthest | Highest |\\n| Use Cases | Application logic | Far | High |\\n| Interface Adapters | Translation | Near | Low |\\n| Frameworks & Drivers | External tools | Closest | Lowest |\\n\\n**Placement Considerations and Gray Areas:**\\n\\nWhile the four layers are clearly separated in principle, practical implementation reveals some gray areas. One common debate is where to place DDD Repositories and Mappers:\\n\\n- Some practitioners place Repositories in the Use Cases layer because they are used by application services\\n- Others place them closer to the Entities layer as they deal with aggregate persistence\\n- The key is to maintain consistency within the codebase and ensure dependency rules are respected\\n\\nAnother consideration is the placement of persistence annotations. If domain entities contain JPA/Hibernate annotations, they technically violate the dependency principle since they depend on the persistence framework. The solution is to either accept this practical relaxation or create separate persistence entities (often called POJOs or Data Transfer Objects) that are mapped to/from domain entities.\\n\\n---\\n\\n### 2. Dependency Principle (相依性原則)\\n\\nThe dependency principle is the cornerstone of Clean Architecture. It states:\\n\\n> **\\\"Source code dependencies must point only inward, toward higher-level policies.\\\"**\\n\\nThis means that inner layers (higher-level, more abstract) should not know about outer layers (lower-level, more concrete). Outer layers may depend on inner layer abstractions, but never the reverse.\\n\\n**Dependency Rules by Layer:**\\n\\n```\\nDependency Flow (Correct):\\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\\n\\nFrameworks & Drivers (outermost)\\n        │\\n        │ depends on\\n        ▼\\nInterface Adapters\\n        │\\n        │ depends on (through ports)\\n        ▼\\nUse Cases\\n        │\\n        │ depends on\\n        ▼\\nEntities (innermost)\\n\\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\\nDependency Flow (Incorrect - Violations):\\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\\n\\n✗ Entities depending on Frameworks & Drivers\\n✗ Use Cases importing framework classes directly\\n✗ Domain entities with ORM annotations\\n✗ Controllers calling repositories directly (skipping use cases)\\n✗ Application services using HttpServletRequest directly\\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\\n```\\n\\n**Achieving Inward Dependencies with Dependency Inversion:**\\n\\nWhen inner layers need functionality from outer layers (e.g., persisting entities to a database), the dependency inversion principle is used. Instead of the inner layer depending on the outer layer, both depend on an abstraction defined by the inner layer.\\n\\n```typescript\\n// Step 1: Use Cases layer defines the abstraction (Output Port)\\ninterface OrderRepository {\\n  save(order: Order): Promise<void>;\\n  findById(id: OrderId): Promise<Order | null>;\\n  findByCustomerId(customerId: string): Promise<Order[]>;\\n}\\n\\n// Step 2: Interface Adapters layer implements the abstraction\\nclass TypeORMOrderRepository implements OrderRepository {\\n  constructor(\\n    private entityManager: EntityManager,\\n    private eventPublisher: EventPublisher\\n  ) {}\\n\\n  async save(order: Order): Promise<void> {\\n    const entity = OrderMapper.toPersistence(order);\\n    await this.entityManager.save(entity);\\n\\n    for (const event of order.getUncommittedEvents()) {\\n      await this.eventPublisher.publish(event);\\n  }\\n\\n  async findById(id: OrderId): Promise<Order | null> {\\n    const entity = await this.entityManager.findOne(OrderEntity, {\\n      where: { id: id.value }\\n    });\\n    return entity ? OrderMapper.toDomain(entity) : null;\\n  }\\n\\n  async findByCustomerId(customerId: string): Promise<Order[]> {\\n    const entities = await this.entityManager.find(OrderEntity, {\\n      where: { customerId }\\n    });\\n    return entities.map(OrderMapper.toDomain);\\n  }\\n}\\n\\n// Dependency direction:\\n// UseCases ──depends on──> OrderRepository (interface) ◄──implemented by── TypeORMOrderRepository\\n```\\n\\n**Practical Challenges in Implementing the Dependency Principle:**\\n\\nFully adhering to the dependency principle in practice presents several challenges:\\n\\n**Challenge 1: Framework Annotations in Domain Entities**\\n\\nAdding JPA/Hibernate annotations to domain entities creates a dependency on the persistence framework:\\n\\n```java\\n// Violates dependency principle - entity depends on JPA\\n@Entity\\n@Table(name = \\\"hosts\\\")\\npublic class Host {\\n  @Id\\n  @GeneratedValue(strategy = GenerationType.IDENTITY)\\n  private Long id;\\n\\n  @Column(nullable = false)\\n  private String name;\\n\\n  @Embedded\\n  private Address address;\\n}\\n```\\n\\n**Solutions:**\\n\\n1. **Accept the practical relaxation**: Accept that entities will have framework annotations for simpler code\\n2. **Create separate persistence entities**: Define separate POJOs for persistence and use mappers to convert between domain entities and persistence objects\\n3. **Use XML mappings instead of annotations**: Move metadata to XML configuration files (though this just changes implicit to explicit dependency)\\n\\n**Challenge 2: External Library Dependencies**\\n\\nWhen domain entities need to use external tools (e.g., an event bus for domain events), directly using the library violates the dependency principle.\\n\\n**Solution:** Define an abstraction (interface) in the domain/application layer and have the external library implement it through an adapter.\\n\\n**Challenge 3: Testing Without Dependencies**\\n\\nOne major benefit of the dependency principle is testability. High-level policies (entities and use cases) can be tested without depending on low-level details (databases, frameworks).\\n\\n**Benefits of Strictly Following the Dependency Principle:**\\n\\n1. **Enhanced Testability**: Core business logic can be tested without UI, database, or external systems\\n2. **Framework Independence**: Business logic survives framework changes and upgrades\\n3. **Technology Flexibility**: Easy to swap databases, web frameworks, or external services\\n4. **Maintainability**: Changes to outer layers do not affect inner layers\\n5. **Parallel Development**: Teams can work on different layers independently\\n\\n---\\n\\n### 3. Cross-Layer Principle (跨層原則)\\n\\nWhen objects need to cross layer boundaries, special care must be taken to maintain the separation of concerns. This principle governs how data and objects are passed between layers.\\n\\n**The Problem with Direct Cross-Layer Passing:**\\n\\nConsider a scenario where a Use Case retrieves a domain entity and passes it directly to an outer layer:\\n\\n```typescript\\n// PROBLEM: Domain entity crosses layers directly\\nclass AddHostController {\\n  constructor(private addHostUseCase: AddHostUseCase) {}\\n\\n  async handleRequest(request: HttpRequest): Promise<HttpResponse> {\\n    const result = await this.addHostUseCase.execute(request.body);\\n\\n    // Direct domain object exposure to outer layer\\n    res.json(result.host); // Host entity exposed to HTTP layer\\n  }\\n}\\n```\\n\\nThis direct passing creates several issues:\\n\\n1. **Presentation logic contamination**: UI requirements might influence domain entity design\\n2. **Serialization problems**: Domain entities might contain circular references or unwante.\nError message: JSON Parse error: Unterminated string"
}
```
**Output:**
```
The arguments provided to the tool are invalid: Invalid input for tool edit: JSON parsing failed: Text: {"filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md", "newString": "## The Three Fundamental Principles\n\nLayered Architecture is governed by three fundamental principles that guide the organization of code and dependencies. These principles, when properly applied, create a clean separation between business logic and external concerns. Understanding these principles deeply is essential for implementing a truly clean architecture.\n\n---\n\n### 0. Software Architecture: Definition and Purpose\n\nBefore diving into the three principles, it is important to understand what software architecture is and why it matters. Software architecture is the **shape** given to a system by those who build it. This shape is determined by three fundamental elements:\n\n1. **Components**: The building blocks that constitute the system\n2. **Arrangement**: How components are organized and structured\n3. **Communication**: How components interact with each other and with external systems\n\nThe purpose of architecture is to serve four key objectives:\n\n- **Development**: Enabling efficient and maintainable software development\n- **Deployment**: Supporting smooth and reliable system deployment\n- **Operation**: Facilitating system operation and monitoring\n- **Maintenance**: Enabling easy modification and enhancement over time\n\n**The ultimate goals of good architecture are:**\n\n- **Minimize the total cost of the software lifecycle**\n- **Maximize programmer productivity**\n\nA well-designed architecture reflects the system's functionality clearly. When developers examine a properly structured codebase, they should be able to understand what the system does simply by looking at its organization. For example, a web shopping system should clearly show components for displaying product lists, shopping cart management, checkout processing, payment handling, shipping selection, and order tracking.\n\n---\n\n### 1. Layering Principle (分層原則)\n\nThe layering principle organizes code into concentric layers based on their distance from Input/Output (I/O). This is the foundation of Clean Architecture and determines the hierarchy of components.\n\n**The Key Definition:**\n\n> \"The further a component layer is from I/O, the higher its level; the closer to I/O, the lower its level.\"\n\nThis definition is intuitive when compared to organizational hierarchies. In a company, receptionists who handle phone calls and visitor接待 (reception) are close to I/O and typically have lower levels. Executives who are far from daily operational I/O have higher levels. Similarly, in software architecture, components that directly handle user input, database access, or network communication are at lower levels, while pure business logic resides at higher levels.\n\n**The Four Layers (from outermost to innermost):**\n\n```\n┌─────────────────────────────────────────────────────────────────────────┐\n│                    FRAMEWORKS & DRIVERS                                 │\n│  (Outermost Layer - Closest to I/O - Lowest Level)                      │\n│                                                                         │\n│  Contains:                                                              │\n│  - Web frameworks (Express, Fastify, Spring MVC)                        │\n│  - Database drivers (JDBC, TypeORM, Prisma)                             │\n│  - UI frameworks (React, Angular, Android)                              │\n│  - External service SDKs (Stripe, AWS SDK)                              │\n│  - Message queue clients (Kafka, RabbitMQ)                              │\n│                                                                         │\n│  These are typically \"glue code\" that connects the application          │\n│  to external frameworks and tools. Minimal business logic resides here. │\n└─────────────────────────────────────────────────────────────────────────┘\n                                    │\n                                    ▼\n┌─────────────────────────────────────────────────────────────────────────┐\n│                    INTERFACE ADAPTERS                                    │\n│  (Converts between external and internal interfaces)                    │\n│                                                                         │\n│  Contains:                                                              │\n│  - Controllers (HTTP, gRPC, CLI handlers)                               │\n│  - Presenters (View Models, Response formatters)                        │\n│  - Gateway implementations (Repository implementations)                 │\n│  - Mappers (Entity to DTO, DTO to Entity transformations)               │\n│  - Serializers (JSON, XML, Protocol Buffers)                            │\n│                                                                         │\n│  Adapters translate external protocols and data formats into internal   │\n│  abstractions that the inner layers can understand.                     │\n└─────────────────────────────────────────────────────────────────────────┘\n                                    │\n                                    ▼\n┌─────────────────────────────────────────────────────────────────────────┐\n│                         USE CASES                                        │\n│  (Application Business Rules - High Level)                              │\n│                                                                         │\n│  Contains:                                                              │\n│  - Application services (Use case handlers)                             │\n│  - Input port interfaces (Use case contracts)                           │\n│  - Output port interfaces (Repository, Service contracts)               │\n│  - Input/Output DTOs (Data Transfer Objects)                            │\n│  - Business rule orchestrators                                          │\n│                                                                         │\n│  This layer defines WHAT the system does without caring HOW external    │\n│  communication happens. Each use case represents a single business      │\n│  operation.                                                             │\n└─────────────────────────────────────────────────────────────────────────┘\n                                    │\n                                    ▼\n┌─────────────────────────────────────────────────────────────────────────┐\n│                         ENTITIES                                         │\n│  (Innermost Layer - Furthest from I/O - Highest Level)                  │\n│                                                                         │\n│  Contains:                                                              │\n│  - Domain entities (Aggregate Roots, Entities)                          │\n│  - Value Objects                                                        │\n│  - Domain Services (for complex business logic)                         │\n│  - Domain Events                                                       │\n│  - Domain Factories                                                    │\n│                                                                         │\n│  This layer contains the CORE business logic that is reusable across    │\n│  different applications. It has ZERO dependencies on any other layer.   │\n└─────────────────────────────────────────────────────────────────────────┘\n```\n\n**Layer Responsibilities:**\n\n| Layer | Responsibility | I/O Distance | Level |\n|-------|---------------|--------------|-------|\n| Entities | Core business rules | Farthest | Highest |\n| Use Cases | Application logic | Far | High |\n| Interface Adapters | Translation | Near | Low |\n| Frameworks & Drivers | External tools | Closest | Lowest |\n\n**Placement Considerations and Gray Areas:**\n\nWhile the four layers are clearly separated in principle, practical implementation reveals some gray areas. One common debate is where to place DDD Repositories and Mappers:\n\n- Some practitioners place Repositories in the Use Cases layer because they are used by application services\n- Others place them closer to the Entities layer as they deal with aggregate persistence\n- The key is to maintain consistency within the codebase and ensure dependency rules are respected\n\nAnother consideration is the placement of persistence annotations. If domain entities contain JPA/Hibernate annotations, they technically violate the dependency principle since they depend on the persistence framework. The solution is to either accept this practical relaxation or create separate persistence entities (often called POJOs or Data Transfer Objects) that are mapped to/from domain entities.\n\n---\n\n### 2. Dependency Principle (相依性原則)\n\nThe dependency principle is the cornerstone of Clean Architecture. It states:\n\n> **\"Source code dependencies must point only inward, toward higher-level policies.\"**\n\nThis means that inner layers (higher-level, more abstract) should not know about outer layers (lower-level, more concrete). Outer layers may depend on inner layer abstractions, but never the reverse.\n\n**Dependency Rules by Layer:**\n\n```\nDependency Flow (Correct):\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\nFrameworks & Drivers (outermost)\n        │\n        │ depends on\n        ▼\nInterface Adapters\n        │\n        │ depends on (through ports)\n        ▼\nUse Cases\n        │\n        │ depends on\n        ▼\nEntities (innermost)\n\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\nDependency Flow (Incorrect - Violations):\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\n✗ Entities depending on Frameworks & Drivers\n✗ Use Cases importing framework classes directly\n✗ Domain entities with ORM annotations\n✗ Controllers calling repositories directly (skipping use cases)\n✗ Application services using HttpServletRequest directly\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\n```\n\n**Achieving Inward Dependencies with Dependency Inversion:**\n\nWhen inner layers need functionality from outer layers (e.g., persisting entities to a database), the dependency inversion principle is used. Instead of the inner layer depending on the outer layer, both depend on an abstraction defined by the inner layer.\n\n```typescript\n// Step 1: Use Cases layer defines the abstraction (Output Port)\ninterface OrderRepository {\n  save(order: Order): Promise<void>;\n  findById(id: OrderId): Promise<Order | null>;\n  findByCustomerId(customerId: string): Promise<Order[]>;\n}\n\n// Step 2: Interface Adapters layer implements the abstraction\nclass TypeORMOrderRepository implements OrderRepository {\n  constructor(\n    private entityManager: EntityManager,\n    private eventPublisher: EventPublisher\n  ) {}\n\n  async save(order: Order): Promise<void> {\n    const entity = OrderMapper.toPersistence(order);\n    await this.entityManager.save(entity);\n\n    for (const event of order.getUncommittedEvents()) {\n      await this.eventPublisher.publish(event);\n  }\n\n  async findById(id: OrderId): Promise<Order | null> {\n    const entity = await this.entityManager.findOne(OrderEntity, {\n      where: { id: id.value }\n    });\n    return entity ? OrderMapper.toDomain(entity) : null;\n  }\n\n  async findByCustomerId(customerId: string): Promise<Order[]> {\n    const entities = await this.entityManager.find(OrderEntity, {\n      where: { customerId }\n    });\n    return entities.map(OrderMapper.toDomain);\n  }\n}\n\n// Dependency direction:\n// UseCases ──depends on──> OrderRepository (interface) ◄──implemented by── TypeORMOrderRepository\n```\n\n**Practical Challenges in Implementing the Dependency Principle:**\n\nFully adhering to the dependency principle in practice presents several challenges:\n\n**Challenge 1: Framework Annotations in Domain Entities**\n\nAdding JPA/Hibernate annotations to domain entities creates a dependency on the persistence framework:\n\n```java\n// Violates dependency principle - entity depends on JPA\n@Entity\n@Table(name = \"hosts\")\npublic class Host {\n  @Id\n  @GeneratedValue(strategy = GenerationType.IDENTITY)\n  private Long id;\n\n  @Column(nullable = false)\n  private String name;\n\n  @Embedded\n  private Address address;\n}\n```\n\n**Solutions:**\n\n1. **Accept the practical relaxation**: Accept that entities will have framework annotations for simpler code\n2. **Create separate persistence entities**: Define separate POJOs for persistence and use mappers to convert between domain entities and persistence objects\n3. **Use XML mappings instead of annotations**: Move metadata to XML configuration files (though this just changes implicit to explicit dependency)\n\n**Challenge 2: External Library Dependencies**\n\nWhen domain entities need to use external tools (e.g., an event bus for domain events), directly using the library violates the dependency principle.\n\n**Solution:** Define an abstraction (interface) in the domain/application layer and have the external library implement it through an adapter.\n\n**Challenge 3: Testing Without Dependencies**\n\nOne major benefit of the dependency principle is testability. High-level policies (entities and use cases) can be tested without depending on low-level details (databases, frameworks).\n\n**Benefits of Strictly Following the Dependency Principle:**\n\n1. **Enhanced Testability**: Core business logic can be tested without UI, database, or external systems\n2. **Framework Independence**: Business logic survives framework changes and upgrades\n3. **Technology Flexibility**: Easy to swap databases, web frameworks, or external services\n4. **Maintainability**: Changes to outer layers do not affect inner layers\n5. **Parallel Development**: Teams can work on different layers independently\n\n---\n\n### 3. Cross-Layer Principle (跨層原則)\n\nWhen objects need to cross layer boundaries, special care must be taken to maintain the separation of concerns. This principle governs how data and objects are passed between layers.\n\n**The Problem with Direct Cross-Layer Passing:**\n\nConsider a scenario where a Use Case retrieves a domain entity and passes it directly to an outer layer:\n\n```typescript\n// PROBLEM: Domain entity crosses layers directly\nclass AddHostController {\n  constructor(private addHostUseCase: AddHostUseCase) {}\n\n  async handleRequest(request: HttpRequest): Promise<HttpResponse> {\n    const result = await this.addHostUseCase.execute(request.body);\n\n    // Direct domain object exposure to outer layer\n    res.json(result.host); // Host entity exposed to HTTP layer\n  }\n}\n```\n\nThis direct passing creates several issues:\n\n1. **Presentation logic contamination**: UI requirements might influence domain entity design\n2. **Serialization problems**: Domain entities might contain circular references or unwante.
Error message: JSON Parse error: Unterminated string
```
```

---

## Assistant

_Thinking:_

The JSON string got truncated due to special characters or formatting issues. Let me try a smaller edit that focuses on enhancing specific sections with content from the blog posts. I'll use a simpler approach.

Let me try a different approach - I'll add a new comprehensive section to the file:

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "limit": 50,
  "offset": 1
}
```
**Output:**
```
<file>
00002| 
00003| ## Overview
00004| 
00005| Layered Architecture, also known as **Hexagonal Architecture** (Ports and Adapters) or **Clean Architecture**, is a software design philosophy that emphasizes the separation of concerns through architectural layers. The core principle is that business rules and application logic should remain independent of external frameworks, databases, delivery mechanisms, and user interfaces.
00006| 
00007| This architectural pattern was formalized by Alistair Cockburn as Hexagonal Architecture and further developed by Robert C. Martin (Uncle Bob) as Clean Architecture. Both approaches share the same fundamental insight: business logic should not depend on databases, web frameworks, or any external system. Instead, external systems depend on abstractions defined by the business logic. This inversion of dependencies ensures that the core application can be tested independently of its delivery mechanisms and that external systems can be replaced without changing business code.
00008| 
00009| The architecture creates clear boundaries between what the application does (core/domain) and how it integrates with the outside world (adapters). This separation enables teams to work on different parts independently, allows technology choices to be deferred, and ensures that core business logic outlives specific technology choices.
00010| 
00011| ---
00012| 
00013| ## Core Principles
00014| 
00015| ### The Dependency Rule
00016| 
00017| The most critical rule in layered architectures is that **dependencies always point inward** toward the domain. This is the Dependency Inversion Principle applied at the architectural level. Inner layers do not know about outer layers, and all source code dependencies point toward the more abstract inner layers.
00018| 
00019| ```
00020|                     ┌───────────────────────────────────────┐
00021|                     │          External Systems             │  ← Outermost Layer
00022|                     │  (Web, DB, UI, External Services)    │
00023|                     └───────────────────┬───────────────────┘
00024|                                         │
00025|                     ┌───────────────────▼───────────────────┐
00026|                     │            Adapters Layer             │  ← Interface Adapters
00027|                     │  (Controllers, Gateways, Presenters,  │
00028|                     │   Database Adapters, External Clients)│
00029|                     └───────────────────┬───────────────────┘
00030|                                         │
00031|                     ┌───────────────────▼───────────────────┐
00032|                     │         Application Layer             │  ← Use Cases / Services
00033|                     │  (Interactors, Use Case Coordinators, │
00034|                     │   Application Services)               │
00035|                     └───────────────────┬───────────────────┘
00036|                                         │
00037|                     ┌───────────────────▼───────────────────┐
00038|                     │            Domain Layer               │  ← Innermost Layer
00039|                     │  (Entities, Value Objects, Aggregates,│
00040|                     │   Domain Services, Domain Events)     │
00041|                     └───────────────────────────────────────┘
00042| 
00043|      Source code dependencies point INWARD toward the domain.
00044|      Inner layers do NOT know about outer layers.
00045| ```
00046| 
00047| ### Key Principles
00048| 
00049| 1. **Independence of Frameworks**: Business logic remains independent of frameworks and can be tested without them
00050| 2. **Testability**: Business rules are testable without UI, database, or external systems
00051| 3. **Independence of UI**: User interfaces can change without affecting business logic

(File has more lines. Use 'offset' parameter to read beyond line 51)
</file>
```
```

---

## Assistant

_Thinking:_

Let me add a new section after the "Core Principles" section with detailed information from all 6 blog posts. I'll use the write tool to append a new comprehensive section to the file.

```
Tool: bash

**Input:**
```json
{
  "command": "wc -l \"E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md\"",
  "description": "Count total lines in the file"
}
```
**Output:**
```
1889 E:\AI-Learn\ai-coding-prompts\doc\layered-architecture.md

```
```

---

## Assistant

_Thinking:_

The file has 1889 lines. Let me read the end of the file to see where I can add the new content.

```
Tool: read

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "limit": 50,
  "offset": 1840
}
```
**Output:**
```
<file>
01841| 4. Use case handler (CreateOrderHandler)
01842| 5. Input/output ports (interfaces)
01843| 6. Repository interface for orders
01844| 7. HTTP controller adapter
01845| 8. TypeORM repository adapter
01846| 9. Unit tests for domain rules
01847| 10. Integration tests for use cases
01848| 
01849| Constraints:
01850| - All dependencies point inward to domain
01851| - Domain has no imports from outer layers
01852| - Use TypeScript
01853| - Include JSDoc documentation
01854| - Follow dependency inversion principle
01855| - Support multiple payment gateways (strategy pattern)
01856| ```
01857| 
01858| ---
01859| 
01860| ## Comparison with Related Patterns
01861| 
01862| ### Layered Architecture vs. Clean Architecture
01863| 
01864| Clean Architecture, introduced by Robert C. Martin, is essentially the same concept as Hexagonal Architecture with a slightly different layer naming convention. Clean Architecture typically includes four layers: Frameworks & Drivers, Interface Adapters, Application Business Rules, and Enterprise Business Rules. The core principles and dependency rules are identical.
01865| 
01866| ### Layered Architecture vs. Onion Architecture
01867| 
01868| Onion Architecture, proposed by Jeffrey Palermo, uses similar concentric layer concepts but emphasizes the "onion" metaphor with dependencies pointing toward the center. The layers are often named: Application Core (inner), Domain Services, Application Services (outer), and Infrastructure. The fundamental principles are the same as Layered Architecture.
01869| 
01870| ### Layered Architecture vs. Microservices
01871| 
01872| While both promote separation of concerns, Layered Architecture is an application-level pattern for organizing code within a single service, whereas Microservices is an architectural style for decomposing an entire system into independently deployable services. Layered Architecture principles can be applied within each microservice.
01873| 
01874| ---
01875| 
01876| ## References and Further Reading
01877| 
01878| 1. Cockburn, Alistair. "Hexagonal Architecture." 2005.
01879| 2. Martin, Robert C. "Clean Architecture: A Craftsman's Guide to Software Structure and Design." Prentice Hall, 2017.
01880| 3. Evans, Eric. "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley, 2004.
01881| 4. Vernon, Vaughn. "Implementing Domain-Driven Design." Addison-Wesley, 2013.
01882| 5. Martin, Robert C. "The Dependency Inversion Principle." 1996.
01883| 6. Percival, Harry, and Bob Gregory. "Architecture Patterns with Python." O'Reilly, 2021.
01884| 7. Ford, Neal, et al. "Architecture: The Hard Parts." O'Reilly, 2021.
01885| 8. Zimarev, Alexey. "Hands-On Domain-Driven Design with .NET Core." Packt, 2019.
01886| 9. Chen, Teddy. "再談Clean Architecture三原則" (Revisiting Clean Architecture Three Principles). teddy-chen-tw.blogspot.com, 2020.
01887| 10. Chen, Teddy. "Clean Architecture（4）：架構三原則首部曲—分層原則" (Architecture Three Principles Part 1: Layering Principle). teddy-chen-tw.blogspot.com, 2018.
01888| 11. Chen, Teddy. "Clean Architecture（5）：架構三原則二部曲—相依性原則" (Architecture Three Principles Part 2: Dependency Principle). teddy-chen-tw.blogspot.com, 2018.
01889| 12. Chen, Teddy. "Clean Architecture（6）：架構三原則三部曲—跨層原則" (Architecture Three Principles Part 3: Cross-Layer Principle). teddy-chen-tw.blogspot.com, 2018.
01890| 

(End of file - total 1890 lines)
</file>
```
```

---

## Assistant

_Thinking:_

Now I'll add a comprehensive new section that includes all the detailed information from all 6 blog posts. I'll append it before the References section.

```
Tool: edit

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "newString": "---\n\n## Detailed Architecture Principles (From Clean Architecture Literature)\n\n### Software Architecture: Definition and Purpose\n\nUnderstanding the foundational concepts of software architecture is essential before diving into implementation details. Software architecture is fundamentally the **shape** given to a system by those who build it. This shape is determined by three fundamental elements:\n\n1. **Components**: The building blocks that constitute the system\n2. **Arrangement**: How components are organized and structured\n3. **Communication**: How components interact with each other and with external systems\n\nThe purpose of architecture is to serve four key objectives:\n\n- **Development**: Enabling efficient and maintainable software development\n- **Deployment**: Supporting smooth and reliable system deployment\n- **Operation**: Facilitating system operation and monitoring\n- **Maintenance**: Enabling easy modification and enhancement over time\n\n**The ultimate goals of good architecture are:**\n\n- **Minimize the total cost of the software lifecycle**\n- **Maximize programmer productivity**\n\nA well-designed architecture reflects the system's functionality clearly. When developers examine a properly structured codebase, they should be able to understand what the system does simply by looking at its organization. For example, a web shopping system should clearly show components for displaying product lists, shopping cart management, checkout processing, payment handling, shipping selection, and order tracking.\n\n---\n\n### Port and Adapter Architecture: Different Perspectives\n\nPort and Adapter Architecture, also known as Hexagonal Architecture, can be visualized from different perspectives, leading to different but equivalent diagrams.\n\n**From the Client Perspective:**\n\n```\nClient → Port → Adapter → Use Case\n```\n\nThe client first interacts with a Port (interface), which is then handled by an Adapter that converts the request and calls the Use Case. This perspective emphasizes how external systems access the application.\n\n**From the Use Case Perspective:**\n\n```\nUse Case → Output Port → Adapter → External System\n```\n\nThe Use Case communicates through Output Ports to external systems. The Adapter implements these ports and handles the translation between internal and external formats.\n\n**Port vs. Adapter: What is What?**\n\nA common point of confusion is distinguishing between Ports and Adapters:\n\n- **Ports** are the interfaces or contracts that define what the application can do or what it needs. Examples include:\n  - JDBC drivers (enable connection to databases)\n  - REST API entry points (enable connection to services)\n  - Repository interfaces (define data access contracts)\n\n- **Adapters** are implementations that convert between external formats and internal interfaces. Examples include:\n  - Controllers that convert HTTP requests to use case inputs\n  - Repositories that convert database results to domain objects\n  - Presenters that format use case outputs for display\n\n**Key Insight:** Software architecture can be continuously chained with multiple Port and Adapter sets. Just as a laptop's Thunderbolt port can connect through multiple adapters (HDMI to VGA, USB to Ethernet), an application's ports can connect to various external systems through appropriate adapters.\n\n---\n\n### Reference Examples: Learning Points\n\nWhen studying Clean Architecture implementations, consider the following evaluation criteria:\n\n**1. Project Structure:**\n- Is it organized as a single project or multiple projects?\n- If multiple projects, what are they and why were they separated?\n- What are the dependencies between projects?\n\n**2. Entity Design:**\n- What entities exist in the domain layer?\n- Are they rich with behavior or anemic (pure data)?\n- Do entities contain any framework dependencies?\n\n**3. Use Case Implementation:**\n- How are bidirectional interfaces implemented (Input/Output ports)?\n- How are input and output data structures defined?\n- Does the use case follow the Command pattern?\n\n**4. Cross-Layer Object Handling:**\n- How are objects transformed when crossing layer boundaries?\n- Are DTOs or View Models used?\n- How are mappers implemented?\n\n**5. Dependency Compliance:**\n- Does the architecture follow the dependency rule?\n- Are there any cross-layer references from outer to inner layers?\n- How are framework dependencies isolated?\n\n**6. Repository Pattern:**\n- Where are repositories defined (which layer)?\n- Are repository interfaces in the domain/application layer?\n- How is persistence technology isolated?\n\n**7. Presenter and View Model:**\n- How are results presented to the user interface?\n- Are View Models used to separate presentation concerns?\n- How does the use case communicate results?\n\n---\n\n### Layering Principle: Distance from I/O\n\nThe layering principle is defined by a key concept: **the further a component layer is from I/O, the higher its level; the closer to I/O, the lower its level**.\n\nThis concept aligns with organizational hierarchies. In a company, receptionists handling phone calls and visitor reception are close to I/O and have lower organizational levels. Executives who are far from daily operational I/O have higher levels. Similarly, in software:\n\n- **Entities (Innermost, Highest Level)**: Core business logic, farthest from I/O, most abstract\n- **Use Cases (High Level)**: Application-specific business logic, defines what the system does\n- **Interface Adapters (Low Level)**: Translates between external protocols and internal abstractions\n- **Frameworks & Drivers (Outermost, Lowest Level)**: Directly handles I/O, uses external tools and frameworks\n\n**The Four Layers in Detail:**\n\n**Entities Layer:**\n- Contains domain model objects (Aggregates, Entities, Value Objects)\n- Domain Services for complex business logic\n- Domain Events for modeling state changes\n- Domain Factories for creating complex objects\n- **Zero dependencies** on any other layer\n- Represents the core business rules that should be reusable across applications\n\n**Use Cases Layer:**\n- Contains application services (Use Case handlers)\n- Defines Input Port interfaces (what external systems can do)\n- Defines Output Port interfaces (what the application needs)\n- Contains Input/Output DTOs for data transfer\n- Orchestrates domain objects to perform business operations\n- Does not contain business rules, only application logic\n\n**Interface Adapters Layer:**\n- Controllers (HTTP, gRPC, CLI) that receive external requests\n- Presenters that format output for presentation\n- Gateway implementations (Repository implementations)\n- Mappers that transform between layers\n- Translates external protocols to internal abstractions\n\n**Frameworks & Drivers Layer:**\n- Web frameworks (Express, Spring, Fastify)\n- Database drivers (JDBC, TypeORM, Prisma)\n- UI frameworks (React, Angular)\n- External service SDKs (Stripe, AWS)\n- Message queue clients (Kafka, RabbitMQ)\n- Contains \"glue code\" connecting the application to external tools\n\n---\n\n### Dependency Principle: Implementation Challenges\n\nThe dependency principle states: **\"Source code dependencies must point only inward, toward higher-level policies.\"** While conceptually simple, practical implementation presents challenges.\n\n**Challenge 1: Framework Annotations in Domain Entities**\n\nA common violation is placing JPA/Hibernate annotations directly onjava\n@Entity\n domain entities:\n\n```@Table(name = \"hosts\")\npublic class Host {\n    @Id\n    @GeneratedValue(strategy = GenerationType.IDENTITY)\n    private Long id;\n    \n    @Column(nullable = false)\n    private String name;\n    \n    @Embedded\n    private Address address;\n}\n```\n\nThis creates a dependency between the Entities layer and the persistence framework. While convenient, this couples the domain model to a specific technology.\n\n**Solutions:**\n\n1. **Accept Practical Relaxation**: Accept the coupling for simpler code, understanding the trade-off\n2. **Create Separate Persistence Objects**: Define POJOs for persistence and use mappers to convert between domain entities and persistence objects\n3. **Use XML Mappings**: Move metadata to configuration files (though this just changes implicit to explicit dependency)\n\n**Challenge 2: External Library Dependencies**\n\nWhen domain logic needs external functionality (e.g., an event bus for domain events), using external libraries directly violates the dependency principle.\n\n**Solution:** Define an abstraction (interface) in the domain/application layer. Have the external library implement this interface through an adapter. This maintains architectural integrity at the cost of additional code.\n\n**Challenge 3: Database Access in Use Cases**\n\nUse Cases need to persist aggregates but cannot depend on the database directly. The solution is dependency inversion:\n\n```typescript\n// Domain/Application layer defines the abstraction\ninterface HostRepository {\n    save(host: Host): Promise<void>;\n    findById(id: HostId): Promise<Host | null>;\n    findAll(): Promise<Host[]>;\n    delete(id: HostId): Promise<void>;\n}\n\n// Infrastructure layer implements the abstraction\nclass JPAHostRepository implements HostRepository {\n    constructor(private entityManager: EntityManager) {}\n    \n    async save(host: Host): Promise<void> {\n        const entity = HostMapper.toPersistence(host);\n        await this.entityManager.save(entity);\n    }\n    \n    async findById(id: HostId): Promise<Host | null> {\n        const entity = await this.entityManager.findOne(HostEntity, {\n            where: { id: id.value }\n        });\n        return entity ? HostMapper.toDomain(entity) : null;\n    }\n    \n    async findAll(): Promise<Host[]> {\n        const entities = await this.entityManager.find(HostEntity, {});\n        return entities.map(HostMapper.toDomain);\n    }\n    \n    async delete(id: HostId): Promise<void> {\n        await this.entityManager.delete(HostEntity, { where: { id: id.value } });\n    }\n}\n```\n\n**Benefits of Strictly Following the Dependency Principle:**\n\n1. **Enhanced Testability**: Core business logic can be tested without UI, database, or external systems\n2. **Framework Independence**: Business logic survives framework changes\n3. **Technology Flexibility**: Easy to swap databases, web frameworks, or external services\n4. **Maintainability**: Changes to outer layers do not affect inner layers\n5. **Parallel Development**: Teams can work on different layers independently\n\n---\n\n### Cross-Layer Principle: Bidirectional Interfaces\n\nWhen objects need to cross layer boundaries, the cross-layer principle governs how this should be done to maintain separation of concerns.\n\n**The Problem with Direct Passing:**\n\nIf a Use Case directly passes domain entities to the presentation layer, several issues arise:\n\n1. Presentation requirements might influence domain entity design\n2. Domain entities might be modified to accommodate UI needs\n3. Serialization issues (circular references, lazy loading proxies)\n4. Coupling between domain logic and presentation concerns\n\n**The Solution: Bidirectional Interfaces**\n\nClean Architecture recommends defining bidirectional interfaces for Use Cases:\n\n1. **Input Interface**: Defines the data the Use Case accepts (Input Port)\n2. **Output Interface**: Defines how the Use Case returns results (Output Port, implemented by Presenter)\n\n**Implementation Pattern:**\n\n```typescript\n// Use Case Input Interface\ninterface FetchHostsInput {\n    getSearchCriteria(): HostSearchCriteria;\n}\n\n// Use Case Output Interface (Presenter)\ninterface FetchHostsOutput {\n    presentHosts(hosts: Host[]): void;\n    presentError(error: string): void;\n}\n\n// Concrete Use Case Input Data\nclass FetchHostsInputData implements FetchHostsInput {\n    private criteria: HostSearchCriteria;\n    \n    constructor(criteria: HostSearchCriteria) {\n        this.criteria = criteria;\n    }\n    \n    getSearchCriteria(): HostSearchCriteria {\n        return this.criteria;\n    }\n}\n\n// Concrete Presenter\nclass HostListPresenter implements FetchHostsOutput {\n    private viewModel: HostListViewModel;\n    \n    presentHosts(hosts: Host[]): void {\n        this.viewModel = {\n            hosts: hosts.map(host => ({\n                id: host.getId().value,\n                name: host.getName().value,\n                status: host.getStatus().value\n            })),\n            isEmpty: hosts.length === 0\n        };\n    }\n    \n    presentError(error: string): void {\n        this.viewModel = {\n            error,\n            hosts: [],\n            isEmpty: true\n        };\n    }\n    \n    getViewModel(): HostListViewModel {\n        return this.viewModel;\n    }\n}\n\n// Use Case Implementation\nclass FetchHostsUseCase implements FetchHostsInput {\n    constructor(\n        private hostRepository: HostRepository,\n        private outputPort: FetchHostsOutput\n    ) {}\n    \n    execute(input: FetchHostsInput): void {\n        try {\n            const criteria = input.getSearchCriteria();\n            const hosts = this.hostRepository.findByCriteria(criteria);\n            this.outputPort.presentHosts(hosts);\n        } catch (error) {\n            this.outputPort.presentError(error.message);\n        }\n    }\n}\n\n// Controller (Inbound Adapter)\nclass HostController {\n    constructor(private fetchHostsUseCase: FetchHostsUseCase) {}\n    \n    getHosts(request: HttpRequest): HttpResponse {\n        const criteria = HostSearchCriteria.fromRequest(request);\n        const input = new FetchHostsInputData(criteria);\n        const presenter = new HostListPresenter();\n        \n        this.fetchHostsUseCase.execute(input);\n        \n        return HttpResponse.ok(presenter.getViewModel());\n    }\n}\n```\n\n**Benefits of the Cross-Layer Principle:**\n\n1. **Separation of Concerns**: Domain objects remain focused on business logic\n2. **Flexibility**: UI and persistence can change without affecting domain objects\n3. **Testability**: Each layer can be tested independently\n4. **Encapsulation**: Internal domain structure is hidden from external layers\n5. **Adaptability**: Easy to add new presentation formats or data sources\n\n---\n\n### The Cleanliness Trade-off\n\nWhile strict adherence to all three principles results in a clean and maintainable architecture, practical implementation often requires balancing architectural purity with development efficiency.\n\n**When to Strictly Follow:**\n\n- The Dependency Principle should generally be followed, especially between Entities and Use Cases layers\n- When external tools are needed, define abstractions and use adapters\n- Critical business logic that needs to be stable and testable\n\n**When to Accept Relaxation:**\n\n- The Interface Adapters layer often couples tightly with specific frameworks\n- Web Controllers with Web framework classes\n- Attempting complete dependency inversion in this layer can be overly complex\n\n**Practical Simplifications:**\n\n- Separating Use Cases into Commands (state-modifying operations) and Queries (read operations) using the CQRS pattern simplifies adapter design\n- Commands can share common presenters and view models\n- Only Queries need customized output handling\n- This approach keeps the adapter layer lean and focused\n\n**Key Insight:** There is no absolute cleanliness—every clean room still has dust. The goal is to achieve the right balance for your specific context and requirements.\n\n---\n\n## Comparison with Related Patterns",
  "oldString": "---\n\n## Comparison with Related Patterns"
}
```
```

---

## Assistant

_Thinking:_

The edit was successful. Now let me update the References section to include all the blog posts I read.

```
Tool: edit

**Input:**
```json
{
  "filePath": "E:\\AI-Learn\\ai-coding-prompts\\doc\\layered-architecture.md",
  "newString": "## References and Further Reading\n\n1. Cockburn, Alistair. \"Hexagonal Architecture.\" 2005.\n2. Martin, Robert C. \"Clean Architecture: A Craftsman's Guide to Software Structure and Design.\" Prentice Hall, 2017.\n3. Evans, Eric. \"Domain-Driven Design: Tackling Complexity in the Heart of Software.\" Addison-Wesley, 2004.\n4. Vernon, Vaughn. \"Implementing Domain-Driven Design.\" Addison-Wesley, 2013.\n5. Martin, Robert C. \"The Dependency Inversion Principle.\" 1996.\n6. Percival, Harry, and Bob Gregory. \"Architecture Patterns with Python.\" O'Reilly, 2021.\n7. Ford, Neal, et al. \"Architecture: The Hard Parts.\" O'Reilly, 2021.\n8. Zimarev, Alexey. \"Hands-On Domain-Driven Design with .NET Core.\" Packt, 2019.\n9. Chen, Teddy. \"Clean Architecture（1）：軟體架構的定義與目的\" (Clean Architecture (1): Software Architecture Definition and Purpose). teddy-chen-tw.blogspot.com, 2018.\n10. Chen, Teddy. \"Clean Architecture（2）：Port and Adapter Architecture\" (Clean Architecture (2): Port and Adapter Architecture). teddy-chen-tw.blogspot.com, 2018.\n11. Chen, Teddy. \"Clean Architecture（3）：精選參考範例\" (Clean Architecture (3): Selected Reference Examples). teddy-chen-tw.blogspot.com, 2018.\n12. Chen, Teddy. \"Clean Architecture（4）：架構三原則首部曲—分層原則\" (Clean Architecture (4): Architecture Three Principles Part 1 - Layering Principle). teddy-chen-tw.blogspot.com, 2018.\n13. Chen, Teddy. \"Clean Architecture（5）：架構三原則二部曲—相依性原則\" (Clean Architecture (5): Architecture Three Principles Part 2 - Dependency Principle). teddy-chen-tw.blogspot.com, 2018.\n14. Chen, Teddy. \"Clean Architecture（6）：架構三原則三部曲—跨層原則\" (Clean Architecture (6): Architecture Three Principles Part 3 - Cross-Layer Principle). teddy-chen-tw.blogspot.com, 2018.\n15. Chen, Teddy. \"再談Clean Architecture三原則\" (Revisiting Clean Architecture Three Principles). teddy-chen-tw.blogspot.com, 2020.\n16. Cockburn, Alistair. \"Hexagonal Architecture.\" http://alistair.cockburn.us/Hexagonal+architecture\n17. 8th Light. \"The Clean Architecture.\" https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html",
  "oldString": "## References and Further Reading\n\n1. Cockburn, Alistair. \"Hexagonal Architecture.\" 2005.\n2. Martin, Robert C. \"Clean Architecture: A Craftsman's Guide to Software Structure and Design.\" Prentice Hall, 2017.\n3. Evans, Eric. \"Domain-Driven Design: Tackling Complexity in the Heart of Software.\" Addison-Wesley, 2004.\n4. Vernon, Vaughn. \"Implementing Domain-Driven Design.\" Addison-Wesley, 2013.\n5. Martin, Robert C. \"The Dependency Inversion Principle.\" 1996.\n6. Percival, Harry, and Bob Gregory. \"Architecture Patterns with Python.\" O'Reilly, 2021.\n7. Ford, Neal, et al. \"Architecture: The Hard Parts.\" O'Reilly, 2021.\n8. Zimarev, Alexey. \"Hands-On Domain-Driven Design with .NET Core.\" Packt, 2019.\n9. Chen, Teddy. \"再談Clean Architecture三原則\" (Revisiting Clean Architecture Three Principles). teddy-chen-tw.blogspot.com, 2020.\n10. Chen, Teddy. \"Clean Architecture（4）：架構三原則首部曲—分層原則\" (Architecture Three Principles Part 1: Layering Principle). teddy-chen-tw.blogspot.com, 2018.\n11. Chen, Teddy. \"Clean Architecture（5）：架構三原則二部曲—相依性原則\" (Architecture Three Principles Part 2: Dependency Principle). teddy-chen-tw.blogspot.com, 2018.\n12. Chen, Teddy. \"Clean Architecture（6）：架構三原則三部曲—跨層原則\" (Architecture Three Principles Part 3: Cross-Layer Principle). teddy-chen-tw.blogspot.com, 2018."
}
```
```

---

## Assistant

_Thinking:_

I've successfully added the comprehensive content from all 6 blog posts to the layered-architecture.md file. The new sections include:

1. Software Architecture: Definition and Purpose - from article 1
2. Port and Adapter Architecture: Different Perspectives - from article 2
3. Reference Examples: Learning Points - from article 3
4. Layering Principle: Distance from I/O - from article 4 (detailed)
5. Dependency Principle: Implementation Challenges - from article 5 (detailed with JPA example)
6. Cross-Layer Principle: Bidirectional Interfaces - from article 6 (detailed with code examples)
7. The Cleanliness Trade-off - practical considerations

I've also updated the References section to include all the blog posts.

I've successfully added comprehensive content from all 6 of Teddy Chen's blog posts to `doc/layered-architecture.md`. The new sections include:

1. **Software Architecture: Definition and Purpose** - Explains that architecture is the "shape" given to a system by its builders, with three elements: components, arrangement, and communication. Covers the four goals: development, deployment, operation, and maintenance.

2. **Port and Adapter Architecture: Different Perspectives** - Clarifies the confusion about Port/Adapter positioning from different viewpoints. Explains that Ports are interfaces/contracts (like JDBC drivers, REST APIs), while Adapters are implementations that convert between formats.

3. **Reference Examples: Learning Points** - Provides a checklist for evaluating Clean Architecture implementations: project structure, entity design, use case implementation, cross-layer handling, dependency compliance, repository patterns, and presenter patterns.

4. **Layering Principle: Distance from I/O** - Detailed explanation of how layers are defined by their distance from I/O, with Entities being highest (farthest from I/O) and Frameworks & Drivers being lowest (closest to I/O). Includes the four-layer diagram with responsibilities.

5. **Dependency Principle: Implementation Challenges** - Covers practical challenges like JPA annotations in entities, external library dependencies, and database access. Provides solutions including mappers, separate persistence objects, and dependency inversion with code examples.

6. **Cross-Layer Principle: Bidirectional Interfaces** - Explains the problem with direct object passing and provides a complete implementation pattern with Input/Output interfaces, Presenters, and Controllers, including code examples.

7. **The Cleanliness Trade-off** - Practical guidance on when to strictly follow principles and when to accept relaxation, including CQRS simplification benefits.

Also updated References to include all 6 original blog posts plus the Hexagonal Architecture original source.

---

