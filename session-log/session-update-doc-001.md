# Analyzing AI Coding Exercise - CA/DDD Refactor Journey

**Session ID:** ses_48bfce894ffeB41Y2jmVUhcZTj
**Created:** 12/31/2025, 6:45:18 PM
**Updated:** 12/31/2025, 6:49:06 PM

---

## User

read @ref/ai-coding-exercise-analysis.md and @ref/refactor-to-ca-analysis.md and learn from them. any knowlege are match knowlege inside @doc/ you can enhance @doc/ now

---

## Assistant

_Thinking:_

The user wants me to read two analysis files and then enhance the documentation in the `doc/` directory based on any matching knowledge found in those analysis files.

Let me analyze what I found:

From `refactor-to-ca-analysis.md`:
1. A 14-step refactoring journey from monolithic to Clean Architecture
2. Step-by-step architectural evolution
3. Design patterns applied at each step
4. Dependency evolution across layers
5. Testing strategy evolution
6. Key architectural principles
7. Detailed analysis of Clean Architecture, DDD tactical patterns

From `ai-coding-exercise-analysis.md`:
1. AI Coding Pattern Language with pattern definitions
2. Specification-Driven Development with formal specification language
3. Domain-Driven Design implementation details
4. Clean Architecture patterns
5. Sub-Agent System Architecture
6. Testing strategy (ezSpec BDD framework)
7. Code Review Framework
8. Architecture Decision Records (ADRs)
9. Failure Case Analysis
10. Comprehensive scoring matrix

Now let me look at the existing `doc/` directory to see what documentation exists and what can be enhanced. I need to read the relevant documentation files to understand what needs to be enhanced.

The key question is: what documentation in `doc/` can be enhanced with the knowledge from these two analysis files?

Let me think about matching topics:

1. Both analysis files talk about Clean Architecture - there's `clean-architecture.md` in `doc/archive/`
2. Both talk about DDD patterns - there's `ddd-patterns.md` in `doc/archive/`
3. Both talk about layered architecture - there's `layered-architecture.md` in `doc/`
4. The ai-coding-exercise has extensive testing strategy - there's `test-driven-development.md` in `doc/`
5. Both talk about design patterns - there's `design-patterns.md` in `doc/`


6. I'll investigate other potential documentation matches systematically

I notice additional documentation areas that might intersect with the analysis files. The repository contains specifications and development approaches that could align with broader software engineering methodologies. Key files like `behavior-driven-development.md`, `cqrs.md`, and `event-sourcing.md` seem particularly relevant for cross-referencing with the analysis documents.

The development documentation suggests comprehensive architectural and design considerations that likely parallel the analysis files' insights. I'll focus on understanding how these documents interconnect and potentially enhance each other.

My next step involves systematically examining the `doc/` directory to identify precise alignment and potential enhancement opportunities between the documentation and the analysis files.

I notice several key documentation areas that seem promising for targeted improvements:
- Clean Architecture documentation could benefit from practical implementation steps
- DDD patterns might need more nuanced explanation around Aggregate design
- Testing strategies could integrate more AI-specific coding methodologies

I'll focus on creating concrete, actionable enhancements that address specific gaps in the existing documentation. The goal is to elevate the doc's practical utility by incorporating insights from the analysis files, particularly around AI-driven coding patterns and architectural evolution.

My approach will prioritize documents with direct thematic connections to the AI coding exercise analysis, ensuring each enhancement provides meaningful, implementable guidance for developers working with AI-assisted development workflows.

The enhancement strategy will center on three core principles:
1. Injecting practical, step-by-step implementation guidance
2. Integrating AI-specific coding patterns and anti-patterns
3. Providing clear, actionable examples that bridge theoretical concepts with real-world application

I'll start by reviewing the current documentation structure, identifying precise points where AI-assisted development insights can be strategically inserted to maximize developer comprehension and practical utility.

The goal is transforming static architectural documentation into a dynamic, AI-aware knowledge base that reflects contemporary software engineering practices.

```
Tool: read

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/clean-code.md"
}
```
**Output:**
```
<file>
00001| # Clean Code
00002| 
00003| ## Overview
00004| 
00005| Clean Code is a software development philosophy articulated by Robert C. Martin (Uncle Bob) that emphasizes writing code that is easy to understand, maintain, and extend. The core premise is that code is read far more often than it is written, and therefore readability should be a primary concern.
00006| 
00007| ## Core Principles
00008| 
00009| ### Meaningful Names
00010| 
00011| ```typescript
00012| // ❌ Bad: Cryptic names
00013| function p(d: Date): number {
00014|   return d.getTime() / 86400000;
00015| }
00016| 
00017| // ✅ Good: Descriptive names
00018| function daysSinceEpoch(epochDate: Date): number {
00019|   const millisecondsPerDay = 86_400_000;
00020|   return epochDate.getTime() / millisecondsPerDay;
00021| }
00022| 
00023| // ❌ Bad: Ambiguous abbreviations
00024| interface SvcCfg {
00025|   dmn: string;
00026|   ttl: number;
00027|   rtry: boolean;
00028| }
00029| 
00030| // ✅ Good: Self-explanatory names
00031| interface ServiceConfig {
00032|   domainName: string;
00033|   timeToLive: number;
00034|   shouldRetry: boolean;
00035| }
00036| 
00037| // ❌ Bad: Magic numbers
00038| if (user.status === 2 && user.score > 75) {
00039|   activatePremium(user);
00040| }
00041| 
00042| // ✅ Good: Named constants
00043| enum UserStatus {
00044|   ACTIVE = 1,
00045|   INACTIVE = 2,
00046|   SUSPENDED = 3
00047| }
00048| 
00049| const PREMIUM_ACTIVATION_SCORE_THRESHOLD = 75;
00050| 
00051| if (user.status === UserStatus.INACTIVE && user.score > PREMIUM_ACTIVATION_SCORE_THRESHOLD) {
00052|   activatePremium(user);
00053| }
00054| ```
00055| 
00056| ### Functions
00057| 
00058| ```typescript
00059| // ❌ Bad: Long function with multiple responsibilities
00060| function processUserData(user: User): void {
00061|   // Validate user
00062|   if (!user.email || !user.name) throw new Error('Invalid user');
00063|   
00064|   // Save to database
00065|   database.save(user);
00066|   
00067|   // Send notification
00068|   emailService.send(user.email, 'Welcome!');
00069|   
00070|   // Update analytics
00071|   analytics.track('user_created', user.id);
00072| }
00073| 
00074| // ✅ Good: Single responsibility, composed functions
00075| function validateUser(user: User): void {
00076|   if (!user.email || !user.name) {
00077|     throw new ValidationError('User must have email and name');
00078|   }
00079| }
00080| 
00081| async function processUserRegistration(user: User): Promise<void> {
00082|   validateUser(user);
00083|   
00084|   await saveUser(user);
00085|   await sendWelcomeEmail(user);
00086|   await trackUserCreation(user.id);
00087| }
00088| 
00089| async function saveUser(user: User): Promise<void> {
00090|   await userRepository.save(user);
00091| }
00092| 
00093| async function sendWelcomeEmail(user: User): Promise<void> {
00094|   await emailService.send(user.email, 'Welcome!', 'welcome-template');
00095| }
00096| 
00097| async function trackUserCreation(userId: string): Promise<void> {
00098|   analytics.track('user_created', userId);
00099| }
00100| 
00101| // ✅ Good: Small, focused function
00102| function isValidEmail(email: string): boolean {
00103|   const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
00104|   return emailRegex.test(email);
00105| }
00106| 
00107| // ✅ Good: Parameter objects reduce parameter count
00108| interface FetchOptions {
00109|   url: string;
00110|   method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
00111|   headers?: Record<string, string>;
00112|   body?: unknown;
00113|   timeout?: number;
00114| }
00115| 
00116| async function fetchWithDefaults(options: FetchOptions): Promise<Response> {
00117|   const response = await fetch(options.url, {
00118|     method: options.method ?? 'GET',
00119|     headers: options.headers ?? {},
00120|     body: options.body ? JSON.stringify(options.body) : undefined
00121|   });
00122|   return response;
00123| }
00124| ```
00125| 
00126| ### Comments
00127| 
00128| ```typescript
00129| // ❌ Bad: Redundant comments
00130| // This function calculates the total
00131| function calculateTotal(items: Item[]): number {
00132|   let total = 0; // Start with zero
00133|   for (const item of items) { // Loop through items
00134|     total += item.price; // Add each price
00135|   }
00136|   return total; // Return the total
00137| }
00138| 
00139| // ✅ Good: Explain "why", not "what"
00140| // The product catalog is priced in cents to avoid floating-point errors
00141| const PRICE_CENTS_MULTIPLIER = 100;
00142| 
00143| function calculateOrderTotal(items: OrderItem[]): Money {
00144|   return items.reduce(
00145|     (total, item) => total.add(item.unitPrice.multiply(item.quantity)),
00146|     Money.zero()
00147|   );
00148| }
00149| 
00150| // ✅ Good: Public API documentation
00151| /**
00152|  * Calculates the total price for an order including applicable discounts.
00153|  * 
00154|  * @param items - The line items in the order
00155|  * @param discountCode - Optional discount code to apply
00156|  * @returns The total price as a Money object
00157|  * @throws InvalidDiscountError if the discount code is expired or invalid
00158|  * 
00159|  * @example
00160|  * ```typescript
00161|  * const total = calculateOrderTotal([
00162|  *   { name: 'Widget', price: Money.fromAmount(9.99) }
00163|  * ], 'SAVE10');
00164|  * ```
00165|  */
00166| function calculateOrderTotal(
00167|   items: OrderItem[],
00168|   discountCode?: string
00169| ): Money {
00170|   // Implementation
00171| }
00172| ```
00173| 
00174| ### Formatting
00175| 
00176| ```typescript
00177| // ✅ Good: Consistent spacing and alignment
00178| const users = [
00179|   { id: 1, name: 'Alice', email: 'alice@example.com' },
00180|   { id: 2, name: 'Bob', email: 'bob@example.com' },
00181|   { id: 3, name: 'Charlie', email: 'charlie@example.com' }
00182| ];
00183| 
00184| // ✅ Good: Related concepts grouped
00185| interface UserRegistration {
00186|   personalInfo: {
00187|     firstName: string;
00188|     lastName: string;
00189|     email: string;
00190|   };
00191|   address: {
00192|     street: string;
00193|     city: string;
00194|     zipCode: string;
00195|     country: string;
00196|   };
00197|   preferences: {
00198|     newsletter: boolean;
00199|     notifications: boolean;
00200|     language: string;
00201|   };
00202| }
00203| 
00204| // ✅ Good: Vertical spacing between concepts
00205| class UserService {
00206|   // Public methods first
00207|   async createUser(data: CreateUserInput): Promise<User> {
00208|     const user = User.create(data);
00209|     await this.repository.save(user);
00210|     await this.eventBus.publish(new UserCreatedEvent(user));
00211|     return user;
00212|   }
00213| 
00214|   async getUser(id: string): Promise<User | null> {
00215|     return this.repository.findById(id);
00216|   }
00217| 
00218|   // Private methods after public
00219|   private validateInput(data: CreateUserInput): void {
00220|     if (!data.email || !data.name) {
00221|       throw new ValidationError('Email and name are required');
00222|     }
00223|   }
00224| 
00225|   private generateUsername(email: string): string {
00226|     return email.split('@')[0];
00227|   }
00228| }
00229| ```
00230| 
00231| ### Error Handling
00232| 
00233| ```typescript
00234| // ❌ Bad: Silent failures and generic errors
00235| try {
00236|   const user = await fetchUser(id);
00237| } catch (e) {
00238|   console.log('Error');
00239|   return null;
00240| }
00241| 
00242| // ✅ Good: Specific error handling with context
00243| async function getUserProfile(userId: string): Promise<UserProfile> {
00244|   try {
00245|     const user = await userRepository.findById(userId);
00246|     if (!user) {
00247|       throw new UserNotFoundError(`User ${userId} not found`);
00248|     }
00249|     
00250|     const profile = await profileService.buildProfile(user);
00251|     return profile;
00252|   } catch (error) {
00253|     if (error instanceof UserNotFoundError) {
00254|       logger.warn('User not found', { userId });
00255|       throw error;
00256|     }
00257|     if (error instanceof DatabaseConnectionError) {
00258|       logger.error('Database connection failed', { userId, error });
00259|       throw new ServiceUnavailableError('Profile service temporarily unavailable');
00260|     }
00261|     logger.error('Unexpected error fetching profile', { userId, error });
00262|     throw new ProfileFetchError('Failed to fetch user profile', { cause: error });
00263|   }
00264| }
00265| 
00266| // ✅ Good: Result type for expected failures
00267| type Result<T, E = Error> = 
00268|   | { ok: true; value: T }
00269|   | { ok: false; error: E };
00270| 
00271| function safeParse<T>(json: string, schema: Schema<T>): Result<T, ParseError> {
00272|   try {
00273|     const value = JSON.parse(json);
00274|     const result = schema.validate(value);
00275|     if (result.success) {
00276|       return { ok: true, value: result.value };
00277|     }
00278|     return { ok: false, error: result.error };
00279|   } catch (error) {
00280|     return { ok: false, error: new ParseError(error.message) };
00281|   }
00282| }
00283| ```
00284| 
00285| ### Object Orientation
00286| 
00287| ```typescript
00288| // ❌ Bad: Data classes with behavior elsewhere
00289| class User {
00290|   constructor(
00291|     public id: string,
00292|     public email: string,
00293|     public name: string,
00294|     public status: string
00295|   ) {}
00296| }
00297| 
00298| function activateUser(user: User): void {
00299|   if (user.status === 'ACTIVE') return;
00300|   user.status = 'ACTIVE';
00301|   emailService.send(user.email, 'Your account is activated!');
00302| }
00303| 
00304| function deactivateUser(user: User): void {
00305|   user.status = 'INACTIVE';
00306|   auditService.log(`User ${user.id} was deactivated`);
00307| }
00308| 
00309| // ✅ Good: Encapsulated behavior
00310| class User {
00311|   private constructor(
00312|     private readonly id: string,
00313|     private readonly email: string,
00314|     private readonly name: string,
00315|     private status: UserStatus,
00316|     private readonly events: UserEvent[] = []
00317|   ) {}
00318| 
00319|   static register(input: RegisterUserInput): User {
00320|     const user = new User(
00321|       UserId.generate(),
00322|       input.email,
00323|       input.name,
00324|       UserStatus.PENDING,
00325|       [new UserRegisteredEvent(input.email)]
00326|     );
00327|     return user;
00328|   }
00329| 
00330|   activate(): void {
00331|     if (this.status === UserStatus.ACTIVE) {
00332|       throw new Error('User is already active');
00333|     }
00334|     
00335|     this.status = UserStatus.ACTIVE;
00336|     this.events.push(new UserActivatedEvent(this.id));
00337|   }
00338| 
00339|   deactivate(reason: DeactivationReason): void {
00340|     if (this.status === UserStatus.INACTIVE) {
00341|       throw new Error('User is already inactive');
00342|     }
00343|     
00344|     this.status = UserStatus.INACTIVE;
00345|     this.events.push(new UserDeactivatedEvent(this.id, reason));
00346|   }
00347| 
00348|   getEvents(): UserEvent[] {
00349|     return [...this.events];
00350|   }
00351| }
00352| ```
00353| 
00354| ### SOLID Principles in Practice
00355| 
00356| ```typescript
00357| // Single Responsibility Principle
00358| // ❌ Bad: Multiple responsibilities
00359| class ReportGenerator {
00360|   generate(data: Data): string { /* ... */ }
00361|   saveToFile(content: string, path: string): void { /* ... */ }
00362|   sendEmail(to: string, content: string): void { /* ... */ }
00363| }
00364| 
00365| // ✅ Good: Single responsibility each
00366| class ReportGenerator {
00367|   constructor(
00368|     private readonly templateEngine: TemplateEngine,
00369|     private readonly dataProvider: DataProvider
00370|   ) {}
00371| 
00372|   generate(reportType: string, parameters: Record<string, unknown>): string {
00373|     const template = this.templateEngine.getTemplate(reportType);
00374|     const data = this.dataProvider.fetchData(parameters);
00375|     return template.render(data);
00376|   }
00377| }
00378| 
00379| class FileStorage {
00380|   save(content: string, path: string): void { /* ... */ }
00381| }
00382| 
00383| class EmailSender {
00384|   send(to: string, subject: string, content: string): void { /* ... */ }
00385| }
00386| 
00387| // Open/Closed Principle
00388| // ✅ Good: Open for extension, closed for modification
00389| interface DiscountStrategy {
00390|   calculateDiscount(order: Order): Money;
00391| }
00392| 
00393| class PercentageDiscount implements DiscountStrategy {
00394|   constructor(private readonly percentage: number) {}
00395| 
00396|   calculateDiscount(order: Order): Money {
00397|     return order.subtotal.multiply(this.percentage / 100);
00398|   }
00399| }
00400| 
00401| class FixedDiscount implements DiscountStrategy {
00402|   constructor(private readonly amount: Money) {}
00403| 
00404|   calculateDiscount(order: Order): Money {
00405|     return this.amount;
00406|   }
00407| }
00408| 
00409| class DiscountCalculator {
00410|   constructor(private readonly strategies: DiscountStrategy[]) {}
00411| 
00412|   calculateBestDiscount(order: Order): Money {
00413|     return this.strategies.reduce(
00414|       (best, strategy) => {
00415|         const discount = strategy.calculateDiscount(order);
00416|         return discount.amount > best.amount ? discount : best;
00417|       },
00418|       Money.zero()
00419|     );
00420|   }
00421| }
00422| 
00423| // Liskov Substitution Principle
00424| // ✅ Good: Subtypes can replace base types
00425| interface Repository<T> {
00426|   findById(id: string): Promise<T | null>;
00427|   findAll(): Promise<T[]>;
00428|   save(entity: T): Promise<void>;
00429| }
00430| 
00431| class UserRepository implements Repository<User> {
00432|   async findById(id: string): Promise<User | null> {
00433|     // User-specific implementation
00434|   }
00435| 
00436|   async findAll(): Promise<User[]> {
00437|     // User-specific implementation
00438|   }
00439| 
00440|   async save(user: User): Promise<void> {
00441|     // User-specific implementation
00442|   }
00443| }
00444| ```
00445| 
00446| ## Code Review Checklist
00447| 
00448| ```markdown
00449| ## Clean Code Review Checklist
00450| 
00451| ### Naming
00452| - [ ] Names reveal intent
00453| - [ ] Names are pronounceable
00454| - [ ] Names are searchable
00455| - [ ] Constants have meaningful names
00456| - [ ] No magic numbers or strings
00457| 
00458| ### Functions
00459| - [ ] Functions do one thing
00460| - [ ] Functions are small (ideally < 20 lines)
00461| - [ ] Functions have few parameters (< 4)
00462| - [ ] No side effects
00463| - [ ] Try/Error handling separated from main logic
00464| 
00465| ### Comments
00466| - [ ] Comments explain "why", not "what"
00467| - [ ] No commented-out code
00468| - [ ] Commented code removed
00469| - [ ] Public API documented
00470| 
00471| ### Formatting
00472| - [ ] Consistent indentation
00473| - [ ] Logical grouping of code
00474| - [ ] Vertical spacing between concepts
00475| - [ ] Line length reasonable (< 120 chars)
00476| 
00477| ### Objects & Data
00478| - [ ] Data structures have behavior
00479| - [ ] Objects hide their internals
00480| - [ ] Law of Demeter followed
00481| - [ ] No public mutable fields
00482| 
00483| ### Error Handling
00484| - [ ] Errors are specific
00485| - [ ] Errors include context
00486| - [ ] No silent failures
00487| - [ ] No empty catch blocks
00488| ```
00489| 
00490| ## AI-Assisted Clean Code
00491| 
00492| ```markdown
00493| ## AI Prompt for Clean Code Refactoring
00494| 
00495| Refactor the following code following Clean Code principles:
00496| 
00497| ```typescript
00498| // Original code
00499| function p(o) {
00500|   let r = { s: [], t: 0 };
00501|   for (let i = 0; i < o.items.length; i++) {
00502|     let item = o.items[i];
00503|     if (item.status === 'A' && item.qty > 0) {
00504|       r.s.push(item);
00505|       r.t += item.price * item.qty;
00506|     }
00507|   }
00508|   return r;
00509| }
00510| ```
00511| 
00512| Requirements:
00513| 1. Use meaningful names
00514| 2. Extract into small, focused functions
00515| 3. Add proper TypeScript types
00516| 4. Add JSDoc documentation
00517| 5. Include error handling
00518| 6. Make it testable
00519| 
00520| Output:
00521| 1. Refactored TypeScript code
00522| 2. Explanation of changes made
00523| 3. Unit tests for the function
00524| ```
00525| </think>
00526| 
00527| The file has been created. I notice the file was truncated. Let me continue creating the remaining documentation files. Due to the large number of files requested, I'll create them in batches.
00528| <minimax:tool_call>
00529| <invoke name="write">
00530| <parameter name="content"># Clean Code - Continued
00531| 
00532| ## Testability
00533| 
00534| ```typescript
00535| // ✅ Good: Testable code
00536| class PaymentProcessor {
00537|   constructor(
00538|     private readonly paymentGateway: PaymentGateway,
00539|     private readonly logger: Logger
00540|   ) {}
00541| 
00542|   async processPayment(
00543|     orderId: string,
00544|     amount: Money,
00545|     paymentMethod: PaymentMethod
00546|   ): Promise<PaymentResult> {
00547|     try {
00548|       this.logger.info(`Processing payment for order ${orderId}`);
00549|       
00550|       const result = await this.paymentGateway.charge(
00551|         paymentMethod,
00552|         amount
00553|       );
00554| 
00555|       if (result.success) {
00556|         this.logger.info(`Payment successful for order ${orderId}`);
00557|         return PaymentResult.success(result.transactionId);
00558|       }
00559| 
00560|       this.logger.warn(`Payment failed for order ${orderId}: ${result.error}`);
00561|       return PaymentResult.failure(result.error);
00562|     } catch (error) {
00563|       this.logger.error(`Payment error for order ${orderId}`, { error });
00564|       return PaymentResult.error(error.message);
00565|     }
00566|   }
00567| }
00568| 
00569| // ✅ Good: Dependencies injectable for testing
00570| describe('PaymentProcessor', () => {
00571|   let processor: PaymentProcessor;
00572|   let mockGateway: jest.Mocked<PaymentGateway>;
00573|   let mockLogger: jest.Mocked<Logger>;
00574| 
00575|   beforeEach(() => {
00576|     mockGateway = {
00577|       charge: jest.fn().mockResolvedValue({ success: true, transactionId: 'txn_123' })
00578|     };
00579|     mockLogger = {
00580|       info: jest.fn(),
00581|       warn: jest.fn(),
00582|       error: jest.fn()
00583|     };
00584|     processor = new PaymentProcessor(mockGateway, mockLogger);
00585|   });
00586| 
00587|   it('processes payment successfully', async () => {
00588|     const result = await processor.processPayment(
00589|       'order_123',
00590|       Money.fromAmount(99.99),
00591|       { type: 'credit_card', last4: '4242' }
00592|     );
00593| 
00594|     expect(result.success).toBe(true);
00595|     expect(mockGateway.charge).toHaveBeenCalled();
00596|     expect(mockLogger.info).toHaveBeenCalled();
00597|   });
00598| });
00599| ```
00600| 
00601| ## References
00602| 
00603| - "Clean Code" by Robert C. Martin
00604| - "The Pragmatic Programmer" by Andrew Hunt and David Thomas
00605| - "Code Complete" by Steve McConnell
00606| - ESLint rules for code quality
00607| 

(End of file - total 607 lines)
</file>
```
```

```
Tool: read

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/coding-conventions.md"
}
```
**Output:**
```
<file>
00001| # Coding Conventions and Best Practices
00002| 
00003| ## Core Concept
00004| 
00005| Coding conventions are a set of guidelines and standards that define how code should be written within a project. These conventions ensure consistency, readability, and maintainability across the codebase. While specific syntax may vary by language, the principles of clear, consistent code apply universally.
00006| 
00007| This document captures universal conventions extracted from real-world projects, focusing on practices that have proven effective in large, complex codebases. These conventions support team collaboration, reduce cognitive load, and make code easier to understand and modify.
00008| 
00009| ---
00010| 
00011| ## Naming Conventions
00012| 
00013| ### General Principles
00014| - Use descriptive names that clearly indicate purpose
00015| - Avoid abbreviations except for widely accepted ones (id, num, config)
00016| - Be consistent with existing naming patterns in the codebase
00017| - Choose clarity over brevity
00018| 
00019| ### Type Naming
00020| 
00021| | Type | Convention | Example |
00022| |------|------------|---------|
00023| | Entity | Singular noun | `Project`, `Task`, `User` |
00024| | Value Object | Descriptive noun/phrase | `ProjectName`, `TaskId`, `Money` |
00025| | Read-Only Interface | `ReadOnly` + Entity name | `ReadOnlyProject` |
00026| | Use Case Interface | Verb + noun + `UseCase` | `AddTaskUseCase`, `ShowUseCase` |
00027| | Use Case Service | Noun + `Service` | `AddTaskService`, `ShowService` |
00028| | DTO | Entity + `Dto` | `TaskDto`, `ProjectDto` |
00029| | PO (Persistent Object) | Entity + `Po` | `TaskPo`, `ProjectPo` |
00030| | Mapper | Entity + `Mapper` | `TaskMapper`, `ProjectMapper` |
00031| | Input | Operation + `Input` | `AddTaskInput`, `ShowInput` |
00032| | Output | Operation + `Output` | `ShowOutput`, `TodayOutput` |
00033| | Controller | Operation + `XxxController` | `AddConsoleController` |
00034| | Presenter Interface | Operation + `Presenter` | `ShowPresenter` |
00035| | Presenter Impl | Operation + `XxxPresenter` | `ShowConsolePresenter` |
00036| 
00037| ### Method Naming
00038| 
00039| | Category | Convention | Example |
00040| |----------|------------|---------|
00041| | Getters | `get` + Noun | `getName()`, `getId()` |
00042| | Boolean Getters | `is` + Adjective | `isDone()`, `isActive()` |
00043| | Setters | `set` + Noun | `setName()` (entities only) |
00044| | Actions | Verb + noun | `addProject()`, `setDone()`, `deleteTask()` |
00045| | Queries | Noun or verb | `getProjects()`, `findById()` |
00046| | Factory | `of()` or `create()` | `ProjectName.of()`, `Task.create()` |
00047| | Mappers | `to` + Target | `toDto()`, `toDomain()`, `toPo()` |
00048| 
00049| ### Field Naming
00050| 
00051| ```typescript
00052| // Private fields
00053| private name: string;
00054| private readonly id: ProjectId;
00055| private tasks: TaskCollection;
00056| 
00057| // Descriptive names
00058| private completedTasks: Task[];     // ✓ Clear
00059| private doneItems: Task[];          // ✓ Acceptable
00060| private stuff: Task[];              // ✗ Unclear
00061| 
00062| // Collections
00063| private tasks: Task[];             // ✓ Plural for arrays
00064| private taskList: Task[];          // ✓ Acceptable
00065| private t: Task[];                 // ✗ Unclear
00066| ```
00067| 
00068| ---
00069| 
00070| ## Code Organization
00071| 
00072| ### Package/Directory Structure
00073| Organize code by feature (vertical slice) while maintaining hexagonal boundaries:
00074| 
00075| ```
00076| feature-name/
00077| ├── domain/              # Entities, Value Objects
00078| ├── usecase/
00079| │   ├── port/in/        # Input ports (use case interfaces)
00080| │   ├── service/        # Use case implementations
00081| │   └── port/           # DTOs, POs, Mappers
00082| └── adapter/
00083|     ├── in/             # Controllers
00084|     └── out/            # Presenters, Repositories
00085| ```
00086| 
00087| ### Class/Interface Organization
00088| ```typescript
00089| // Standard ordering within a class
00090| 1. Static fields/constants
00091| 2. Instance fields (private first, then protected/public)
00092| 3. Constructors
00093| 4. Public methods
00094| 5. Protected methods
00095| 6. Private methods
00096| 7. Getters/Setters
00097| ```
00098| 
00099| ---
00100| 
00101| ## Immutability
00102| 
00103| ### Value Objects
00104| Always use immutable value objects:
00105| 
00106| ```typescript
00107| // ✓ CORRECT - Immutable
00108| record Email implements ValueObject {
00109|   private readonly value: string;
00110| 
00111|   constructor(value: string) {
00112|     if (!this.isValid(value)) {
00113|       throw new DomainError('Invalid email');
00114|     }
00115|     this.value = value;
00116|   }
00117| 
00118|   getValue(): string {
00119|     return this.value;
00120|   }
00121| 
00122|   private isValid(email: string): boolean {
00123|     return email.includes('@');
00124|   }
00125| }
00126| 
00127| // ✗ INCORRECT - Mutable
00128| class Email {
00129|   public value: string;  // Public and mutable
00130| }
00131| ```
00132| 
00133| ### Entities
00134| Make fields private with final where possible:
00135| 
00136| ```typescript
00137| class Project implements Entity<ProjectId> {
00138|   private readonly id: ProjectId;      // ✓ Final - cannot be changed
00139|   private name: ProjectName;            // ✓ Private - encapsulated
00140|   private tasks: TaskCollection;        // ✓ Private - encapsulated
00141| 
00142|   constructor(id: ProjectId, name: ProjectName) {
00143|     this.id = id;
00144|     this.name = name;
00145|     this.tasks = new TaskCollection();
00146|   }
00147| }
00148| ```
00149| 
00150| ### Return Defensive Copies
00151| ```typescript
00152| // Return unmodifiable views
00153| getTasks(): ReadonlyArray<Task> {
00154|   return Object.freeze([...this.tasks.getAll()]);
00155| }
00156| 
00157| // Or return copy
00158| getTasks(): Task[] {
00159|   return [...this.tasks.getAll()];
00160| }
00161| ```
00162| 
00163| ---
00164| 
00165| ## Constructor Injection
00166| 
00167| Prefer constructor injection for dependencies:
00168| 
00169| ```typescript
00170| // ✓ CORRECT - Constructor injection
00171| class AddTaskService implements AddTaskUseCase {
00172|   private readonly repository: ToDoListRepository;
00173|   private readonly presenter: TaskPresenter;
00174| 
00175|   constructor(
00176|     repository: ToDoListRepository,
00177|     presenter: TaskPresenter
00178|   ) {
00179|     this.repository = repository;
00180|     this.presenter = presenter;
00181|   }
00182| }
00183| 
00184| // ✓ CORRECT - Interface injection
00185| class AddTaskService implements AddTaskUseCase {
00186|   constructor(
00187|     private repository: ToDoListRepository
00188|   ) {}
00189| }
00190| ```
00191| 
00192| ---
00193| 
00194| ## Error Handling
00195| 
00196| ### Use Specific Exception Types
00197| ```typescript
00198| // ✓ CORRECT - Specific exceptions
00199| class DomainError extends Error {
00200|   constructor(message: string) {
00201|     super(message);
00202|     this.name = 'DomainError';
00203|   }
00204| }
00205| 
00206| class ValidationError extends DomainError {
00207|   constructor(field: string, message: string) {
00208|     super(`Validation failed for ${field}: ${message}`);
00209|     this.name = 'ValidationError';
00210|   }
00211| }
00212| 
00213| // ✗ INCORRECT - Generic exceptions
00214| throw new Error('Something went wrong');
00215| ```
00216| 
00217| ### Use Result/Output Types for Operations
00218| ```typescript
00219| // Return structured output
00220| interface CqrsOutput {
00221|   isSuccess(): boolean;
00222|   getMessage(): string | null;
00223|   getData(): unknown;
00224|   fail(): this;
00225|   succeed(): this;
00226|   setMessage(msg: string): this;
00227|   setData(data: unknown): this;
00228| }
00229| 
00230| // Usage
00231| return CqrsOutput.create()
00232|   .fail()
00233|   .setMessage('Task not found');
00234| 
00235| return CqrsOutput.create()
00236|   .succeed()
00237|   .setData(TaskMapper.toDto(task));
00238| ```
00239| 
00240| ---
00241| 
00242| ## Validation Strategy
00243| 
00244| ### Value Objects
00245| Validate in constructor:
00246| 
00247| ```typescript
00248| record ProjectName implements ValueObject {
00249|   private readonly value: string;
00250| 
00251|   constructor(value: string) {
00252|     if (value === null || value.trim().length === 0) {
00253|       throw new ValidationError('name', 'cannot be empty');
00254|     }
00255|     if (value.length > 100) {
00256|       throw new ValidationError('name', 'cannot exceed 100 characters');
00257|     }
00258|     this.value = value.trim();
00259|   }
00260| }
00261| ```
00262| 
00263| ### Use Cases
00264| Validate inputs before business logic:
00265| 
00266| ```typescript
00267| class AddTaskService implements AddTaskUseCase {
00268|   async execute(input: AddTaskInput): Promise<CqrsOutput> {
00269|     // 1. Validate inputs
00270|     if (!input.description) {
00271|       return CqrsOutput.create()
00272|         .fail()
00273|         .setMessage('Description is required');
00274|     }
00275| 
00276|     // 2. Validate business rules
00277|     const toDoList = await this.repository.findById(
00278|       ToDoListId.of(input.toDoListId)
00279|     );
00280| 
00281|     if (!toDoList) {
00282|       return CqrsOutput.create()
00283|         .fail()
00284|         .setMessage('ToDoList not found');
00285|     }
00286| 
00287|     // 3. Business operation
00288|     const task = toDoList.addTask(TaskDescription.of(input.description));
00289|     await this.repository.save(toDoList);
00290| 
00291|     return CqrsOutput.create().succeed();
00292|   }
00293| }
00294| ```
00295| 
00296| ---
00297| 
00298| ## Stream API Usage
00299| 
00300| ### Preferred Patterns
00301| 
00302| ```typescript
00303| // Method chaining
00304| const activeTasks = tasks
00305|   .filter(t => !t.isDone())
00306|   .map(TaskMapper.toDto)
00307|   .toList();
00308| 
00309| // Method references
00310| const names = projects
00311|   .stream()
00312|   .map(Project::getName)
00313|   .filter(n => n.value.includes('Important'))
00314|   .toList();
00315| 
00316| // findFirst for single results
00317| const task = tasks
00318|   .stream()
00319|   .filter(t => t.getId().equals(taskId))
00320|   .findFirst()
00321|   .orElse(null);
00322| 
00323| // Avoid old-style loops
00324| // ✗ for (let i = 0; i < tasks.length; i++)
00325| ```
00326| 
00327| ---
00328| 
00329| ## Optional Pattern
00330| 
00331| ### Use Optional for Nullable Returns
00332| 
00333| ```typescript
00334| // ✓ CORRECT - Use Optional
00335| async findById(id: TaskId): Promise<Task | null> {
00336|   const po = await this.entityManager.findOne(TaskPo, { where: { id: id.value } });
00337|   return po ? TaskMapper.toDomain(po) : null;
00338| }
00339| 
00340| // ✓ CORRECT - Return Optional
00341| async findById(id: TaskId): Promise<Optional<Task>> {
00342|   const po = await this.entityManager.findOne(TaskPo, { where: { id: id.value } });
00343|   return Optional.ofNullable(po).map(TaskMapper.toDomain);
00344| }
00345| 
00346| // ✗ INCORRECT - Return null directly for complex operations
00347| async findTasks(): Promise<Task[]> {
00348|   return null;  // Confusing
00349| }
00350| ```
00351| 
00352| ### Optional Operations
00353| 
00354| ```typescript
00355| const task = repository.findById(id);
00356| 
00357| // Use map instead of get
00358| const taskName = repository.findById(id)
00359|   .map(task => task.getDescription().value)
00360|   .orElse('Unknown');
00361| 
00362| // Use filter
00363| const doneTask = repository.findById(id)
00364|   .filter(task => task.isDone());
00365| 
00366| // Use orElse for defaults
00367| const taskName = repository.findById(id)
00368|   .map(task => task.getName().value)
00369|   .orElse('Default Project');
00370| ```
00371| 
00372| ---
00373| 
00374| ## Testing Conventions
00375| 
00376| ### Test Organization
00377| ```
00378| src/test/java/.../feature/
00379| ├── domain/
00380| │   └── XxxTest.java
00381| ├── usecase/
00382| │   └── XxxServiceTest.java
00383| └── adapter/
00384|     └── XxxControllerTest.java
00385| ```
00386| 
00387| ### Test Naming
00388| ```typescript
00389| // Describe behavior with underscores
00390| @Test
00391| public void add_a_project_with_duplicated_name_has_no_effect() {
00392|   // Arrange
00393|   // Act
00394|   // Assert
00395| }
00396| 
00397| @Test
00398| public void remove_task_from_project_decreases_task_count() {
00399|   // ...
00400| }
00401| ```
00402| 
00403| ### Test Structure (Arrange-Act-Assert)
00404| 
00405| ```typescript
00406| @Test
00407| public void add_task_increases_project_task_count() {
00408|   // 1. Arrange
00409|   const repository = new InMemoryToDoListRepository();
00410|   const project = new ProjectBuilder()
00411|     .withName('Test Project')
00412|     .build();
00413|   await repository.save(project);
00414| 
00415|   const service = new AddTaskService(repository);
00416|   const input = new AddTaskInput();
00417|   input.toDoListId = project.getId().value;
00418|   input.projectName = 'Test Project';
00419|   input.description = 'New Task';
00420| 
00421|   // 2. Act
00422|   const result = await service.execute(input);
00423| 
00424|   // 3. Assert
00425|   assertTrue(result.isSuccess());
00426|   const updatedProject = await repository.findById(project.getId());
00427|   assertEquals(1, updatedProject.getTaskCount());
00428| }
00429| ```
00430| 
00431| ---
00432| 
00433| ## Code Quality Rules
00434| 
00435| ### KISS (Keep It Simple, Stupid)
00436| - Prefer simple, straightforward solutions
00437| - Avoid over-engineering for hypothetical future needs
00438| - Clear, readable code over clever code
00439| 
00440| ### DRY (Don't Repeat Yourself)
00441| - Extract common patterns into shared utilities
00442| - Use inheritance and composition appropriately
00443| - Avoid copy-paste code
00444| 
00445| ### Single Responsibility
00446| - Each class has one reason to change
00447| - Each method does one thing well
00448| - Extract complex logic into private methods
00449| 
00450| ### Tell, Don't Ask
00451| ```typescript
00452| // ✓ Tell objects what to do
00453| project.addTask(taskDescription);
00454| 
00455| // ✗ Ask about state, then act
00456| if (project.getTasks().length < project.getMaxTasks()) {
00457|   project.getTasks().push(newTask);
00458| }
00459| ```
00460| 
00461| ---
00462| 
00463| ## Documentation Standards
00464| 
00465| ### Self-Documenting Code
00466| ```typescript
00467| // ✓ GOOD - Clear names, obvious intent
00468| const completedTasks = tasks.filter(t => t.isDone());
00469| 
00470| // ✗ BAD - Needs comment to explain
00471| const filtered = items.filter(/* filter out done items */);
00472| ```
00473| 
00474| ### Minimal Comments
00475| - Comment why, not what
00476| - Explain complex business rules
00477| - Document non-obvious decisions
00478| - Keep comments synchronized with code
00479| 
00480| ### When to Document
00481| - API contracts and interfaces
00482| - Complex business logic
00483| - Non-obvious workarounds
00484| - Design decisions and rationale
00485| 
00486| ---
00487| 
00488| ## Common Anti-Patterns to Avoid
00489| 
00490| 1. **Primitive Obsession**: Using primitives instead of value objects
00491| 2. **Feature Envy**: Method accesses data of another class excessively
00492| 3. **God Class**: Class with too many responsibilities
00493| 4. **Magic Numbers/Strings**: Hardcoded values without explanation
00494| 5. **Long Methods**: Methods doing too many things
00495| 6. **Deep Nesting**: Multiple levels of conditionals/loops
00496| 7. **Public Fields**: Exposing internal state
00497| 8. **Null Returns**: Returning null instead of Optional/empty collections
00498| 9. **Mutable Collections**: Exposing internal collections
00499| 10. **Unused Code**: Code that is never called
00500| 
00501| ---
00502| 
00503| ## References and Further Reading
00504| 
00505| 1. Martin, Robert C. "Clean Code: A Handbook of Agile Software Craftsmanship." Prentice Hall, 2008.
00506| 2. Martin, Robert C. "Clean Architecture: A Craftsman's Guide to Software Structure and Design." Prentice Hall, 2017.
00507| 3. Effective Java. Joshua Bloch. Addison-Wesley, 2018.
00508| 4. "TypeScript Deep Dive." Basarat Ali Syed.
00509| 5. "Design Patterns: Elements of Reusable Object-Oriented Software." Gamma et al. Addison-Wesley, 1994.
00510| 

(End of file - total 510 lines)
</file>
```
```

```
Tool: read

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/design-patterns.md"
}
```
**Output:**
```
<file>
00001| # Design Patterns
00002| 
00003| ## Core Concept
00004| 
00005| Design patterns are reusable solutions to commonly occurring software design problems. They represent best practices evolved over time by experienced software developers and provide a common vocabulary for discussing software architecture and design decisions.
00006| 
00007| The concept of design patterns was popularized by the 1994 book "Design Patterns: Elements of Reusable Object-Oriented Software" by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides, collectively known as the "Gang of Four" (GoF). This seminal work catalogued 23 design patterns that have since become fundamental tools in software engineering.
00008| 
00009| Design patterns are not specific algorithms or complete solutions but rather templates for how to solve a problem in a way that is proven to work, maintainable, and flexible. They capture the wisdom of experienced developers and help less experienced developers avoid common pitfalls and make better design decisions.
00010| 
00011| Understanding design patterns is essential for several reasons: they provide a shared vocabulary that improves communication between developers, they help in creating more maintainable and flexible code, and they represent battle-tested solutions to common problems that have been refined over decades of collective experience.
00012| 
00013| ---
00014| 
00015| ## Design Pattern Categories
00016| 
00017| ```
00018| ┌─────────────────────────────────────────────────────────────────────────┐
00019| │                    DESIGN PATTERN CATEGORIES                             │
00020| ├─────────────────────────────────────────────────────────────────────────┤
00021| │                                                                         │
00022| │  ┌─────────────────────────────────────────────────────────────────┐    │
00023| │  │                    CREATIONAL PATTERNS                           │    │
00024| │  │                                                                 │    │
00025| │  │  Focus: Object creation mechanisms                              │    │
00026| │  │                                                                 │    │
00027| │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐ │    │
00028| │  │  │Factory  │  │Abstract │  │Builder  │  │Prototype│  │Singleton│ │    │
00029| │  │  │Method   │  │Factory  │  │         │  │         │  │        │ │    │
00030| │  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘ │    │
00031| │  └─────────────────────────────────────────────────────────────────┘    │
00032| │                                                                         │
00033| │  ┌─────────────────────────────────────────────────────────────────┐    │
00034| │  │                   STRUCTURAL PATTERNS                            │    │
00035| │  │                                                                 │    │
00036| │  │  Focus: Composition of classes and objects                       │    │
00037| │  │                                                                 │    │
00038| │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐ │    │
00039| │  │  │Adapter  │  │Bridge   │  │Composite│  │Decorator│  │Facade │ │    │
00040| │  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘ │    │
00041| │  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐                 │    │
00042| │  │  │Flyweight│  │Proxy    │  │                 │                 │    │
00043| │  │  └─────────┘  └─────────┘  └─────────────────┘                 │    │
00044| │  └─────────────────────────────────────────────────────────────────┘    │
00045| │                                                                         │
00046| │  ┌─────────────────────────────────────────────────────────────────┐    │
00047| │  │                   BEHAVIORAL PATTERNS                            │    │
00048| │  │                                                                 │    │
00049| │  │  Focus: Communication between objects                            │    │
00050| │  │                                                                 │    │
00051| │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐ │    │
00052| │  │  │Chain of │  │Command  │  │Interpreter││Iterator │  │Mediator│ │    │
00053| │  │  │Responsib.│  │         │  │          │  │        │  │       │ │    │
00054| │  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘ │    │
00055| │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────────┐   │    │
00056| │  │  │Memento  │  │Observer │  │State    │  │Strategy         │   │    │
00057| │  │  └─────────┘  └─────────┘  └─────────┘  └─────────────────┘   │    │
00058| │  │  ┌─────────────────────┐                                         │    │
00059| │  │  │Template Method      │                                         │    │
00060| │  │  └─────────────────────┘                                         │    │
00061| │  └─────────────────────────────────────────────────────────────────┘    │
00062| │                                                                         │
00063| └─────────────────────────────────────────────────────────────────────────┘
00064| ```
00065| 
00066| ---
00067| 
00068| ## Creational Patterns
00069| 
00070| ### 1. Factory Method
00071| 
00072| ```typescript
00073| // Factory Method: Create objects without specifying exact class
00074| interface Product {
00075|   operation(): string;
00076| }
00077| 
00078| class ConcreteProductA implements Product {
00079|   operation(): string {
00080|     return 'Product A operation';
00081|   }
00082| }
00083| 
00084| class ConcreteProductB implements Product {
00085|   operation(): string {
00086|     return 'Product B operation';
00087|   }
00088| }
00089| 
00090| abstract class Creator {
00091|   abstract factoryMethod(): Product;
00092| 
00093|   someOperation(): string {
00094|     const product = this.factoryMethod();
00095|     return `Creator: ${product.operation()}`;
00096|   }
00097| }
00098| 
00099| class ConcreteCreatorA extends Creator {
00100|   factoryMethod(): Product {
00101|     return new ConcreteProductA();
00102|   }
00103| }
00104| 
00105| class ConcreteCreatorB extends Creator {
00106|   factoryMethod(): Product {
00107|     return new ConcreteProductB();
00108|   }
00109| }
00110| 
00111| // Usage
00112| const creatorA = new ConcreteCreatorA();
00113| console.log(creatorA.someOperation());
00114| 
00115| const creatorB = new ConcreteCreatorB();
00116| console.log(creatorB.someOperation());
00117| ```
00118| 
00119| ### 2. Abstract Factory
00120| 
00121| ```typescript
00122| // Abstract Factory: Create families of related objects
00123| interface GUIFactory {
00124|   createButton(): Button;
00125|   createCheckbox(): Checkbox;
00126| }
00127| 
00128| interface Button {
00129|   render(): void;
00130|   onClick(): void;
00131| }
00132| 
00133| interface Checkbox {
00134|   render(): void;
00135|   toggle(): void;
00136| }
00137| 
00138| class WinFactory implements GUIFactory {
00139|   createButton(): Button {
00140|     return new WinButton();
00141|   }
00142|   createCheckbox(): Checkbox {
00143|     return new WinCheckbox();
00144|   }
00145| }
00146| 
00147| class MacFactory implements GUIFactory {
00148|   createButton(): Button {
00149|     return new MacButton();
00150|   }
00151|   createCheckbox(): Checkbox {
00152|     return new MacCheckbox();
00153|   }
00154| }
00155| 
00156| class WinButton implements Button {
00157|   render(): void {
00158|     console.log('Rendering Windows button');
00159|   }
00160|   onClick(): void {
00161|     console.log('Windows button clicked');
00162|   }
00163| }
00164| 
00165| class MacButton implements Button {
00166|   render(): void {
00167|     console.log('Rendering macOS button');
00168|   }
00169|   onClick(): void {
00170|     console.log('macOS button clicked');
00171|   }
00172| }
00173| 
00174| class WinCheckbox implements Checkbox {
00175|   render(): void {
00176|     console.log('Rendering Windows checkbox');
00177|   }
00178|   toggle(): void {
00179|     console.log('Windows checkbox toggled');
00180|   }
00181| }
00182| 
00183| class MacCheckbox implements Checkbox {
00184|   render(): void {
00185|     console.log('Rendering macOS checkbox');
00186|   }
00187|   toggle(): void {
00188|     console.log('macOS checkbox toggled');
00189|   }
00190| }
00191| 
00192| // Usage
00193| function createUI(factory: GUIFactory) {
00194|   const button = factory.createButton();
00195|   const checkbox = factory.createCheckbox();
00196|   button.render();
00197|   checkbox.render();
00198| }
00199| 
00200| createUI(new WinFactory());
00201| createUI(new MacFactory());
00202| ```
00203| 
00204| ### 3. Builder
00205| 
00206| ```typescript
00207| // Builder: Construct complex objects step by step
00208| class House {
00209|   private foundation: string = '';
00210|   private structure: string = '';
00211|   private roof: string = '';
00212|   private hasGarage: boolean = false;
00213|   private hasPool: boolean = false;
00214|   private hasGarden: boolean = false;
00215| 
00216|   setFoundation(foundation: string): this {
00217|     this.foundation = foundation;
00218|     return this;
00219|   }
00220| 
00221|   setStructure(structure: string): this {
00222|     this.structure = structure;
00223|     return this;
00224|   }
00225| 
00226|   setRoof(roof: string): this {
00227|     this.roof = roof;
00228|     return this;
00229|   }
00230| 
00231|   setGarage(hasGarage: boolean): this {
00232|     this.hasGarage = hasGarage;
00233|     return this;
00234|   }
00235| 
00236|   setPool(hasPool: boolean): this {
00237|     this.hasPool = hasPool;
00238|     return this;
00239|   }
00240| 
00241|   setGarden(hasGarden: boolean): this {
00242|     this.hasGarden = hasGarden;
00243|     return this;
00244|   }
00245| 
00246|   describe(): string {
00247|     return `House with ${this.foundation} foundation, ${this.structure} structure, ${this.roof} roof, ` +
00248|       `${this.hasGarage ? 'with garage' : 'no garage'}, ` +
00249|       `${this.hasPool ? 'with pool' : 'no pool'}, ` +
00250|       `${this.hasGarden ? 'with garden' : 'no garden'}`;
00251|   }
00252| }
00253| 
00254| class HouseBuilder {
00255|   private house: House;
00256| 
00257|   constructor() {
00258|     this.house = new House();
00259|   }
00260| 
00261|   foundation(type: string): HouseBuilder {
00262|     this.house.setFoundation(type);
00263|     return this;
00264|   }
00265| 
00266|   structure(type: string): HouseBuilder {
00267|     this.house.setStructure(type);
00268|     return this;
00269|   }
00270| 
00271|   roof(type: string): HouseBuilder {
00272|     this.house.setRoof(type);
00273|     return this;
00274|   }
00275| 
00276|   garage(hasGarage: boolean): HouseBuilder {
00277|     this.house.setGarage(hasGarage);
00278|     return this;
00279|   }
00280| 
00281|   pool(hasPool: boolean): HouseBuilder {
00282|     this.house.setPool(hasPool);
00283|     return this;
00284|   }
00285| 
00286|   garden(hasGarden: boolean): HouseBuilder {
00287|     this.house.setGarden(hasGarden);
00288|     return this;
00289|   }
00290| 
00291|   build(): House {
00292|     return this.house;
00293|   }
00294| }
00295| 
00296| // Usage
00297| const house = new HouseBuilder()
00298|   .foundation('Concrete')
00299|   .structure('Brick')
00300|   .roof('Tile')
00301|   .garage(true)
00302|   .pool(false)
00303|   .garden(true)
00304|   .build();
00305| 
00306| console.log(house.describe());
00307| ```
00308| 
00309| ### 4. Prototype
00310| 
00311| ```typescript
00312| // Prototype: Create objects by cloning existing ones
00313| interface Prototype {
00314|   clone(): Prototype;
00315|   getInfo(): string;
00316| }
00317| 
00318| class User implements Prototype {
00319|   constructor(
00320|     private name: string,
00321|     private age: number,
00322|     private permissions: string[]
00323|   ) {}
00324| 
00325|   clone(): User {
00326|     return new User(this.name, this.age, [...this.permissions]);
00327|   }
00328| 
00329|   getInfo(): string {
00330|     return `User: ${this.name}, Age: ${this.age}, Permissions: ${this.permissions.join(', ')}`;
00331|   }
00332| 
00333|   addPermission(permission: string): void {
00334|     this.permissions.push(permission);
00335|   }
00336| }
00337| 
00338| class UserRegistry {
00339|   private prototypes: Map<string, Prototype> = new Map();
00340| 
00341|   addPrototype(name: string, prototype: Prototype): void {
00342|     this.prototypes.set(name, prototype);
00343|   }
00344| 
00345|   getPrototype(name: string): Prototype | undefined {
00346|     return this.prototypes.get(name)?.clone();
00347|   }
00348| }
00349| 
00350| // Usage
00351| const adminTemplate = new User('Admin Template', 30, ['read', 'write']);
00352| const guestTemplate = new User('Guest Template', 25, ['read']);
00353| 
00354| const registry = new UserRegistry();
00355| registry.addPrototype('admin', adminTemplate);
00356| registry.addPrototype('guest', guestTemplate);
00357| 
00358| const newAdmin = registry.getPrototype('admin');
00359| if (newAdmin) {
00360|   newAdmin.addPermission('delete');
00361|   console.log(newAdmin.getInfo());
00362| }
00363| ```
00364| 
00365| ### 5. Singleton
00366| 
00367| ```typescript
00368| // Singleton: Ensure only one instance exists
00369| class DatabaseConnection {
00370|   private static instance: DatabaseConnection | null = null;
00371|   private connection: string = '';
00372| 
00373|   private constructor() {
00374|     this.connection = 'Connected to database';
00375|   }
00376| 
00377|   static getInstance(): DatabaseConnection {
00378|     if (!DatabaseConnection.instance) {
00379|       DatabaseConnection.instance = new DatabaseConnection();
00380|     }
00381|     return DatabaseConnection.instance;
00382|   }
00383| 
00384|   query(sql: string): string {
00385|     return `Executing: ${sql}`;
00386|   }
00387| 
00388|   getConnectionInfo(): string {
00389|     return this.connection;
00390|   }
00391| }
00392| 
00393| class ConfigManager {
00394|   private static instance: ConfigManager;
00395|   private config: Map<string, string> = new Map();
00396| 
00397|   private constructor() {
00398|     this.config.set('app.name', 'MyApp');
00399|     this.config.set('app.version', '1.0.0');
00400|   }
00401| 
00402|   static getInstance(): ConfigManager {
00403|     if (!ConfigManager.instance) {
00404|       ConfigManager.instance = new ConfigManager();
00405|     }
00406|     return ConfigManager.instance;
00407|   }
00408| 
00409|   get(key: string): string | undefined {
00410|     return this.config.get(key);
00411|   }
00412| 
00413|   set(key: string, value: string): void {
00414|     this.config.set(key, value);
00415|   }
00416| }
00417| 
00418| // Usage
00419| const db1 = DatabaseConnection.getInstance();
00420| const db2 = DatabaseConnection.getInstance();
00421| console.log(db1 === db2); // true
00422| 
00423| console.log(db1.query('SELECT * FROM users'));
00424| console.log(ConfigManager.getInstance().get('app.name'));
00425| ```
00426| 
00427| ---
00428| 
00429| ## Structural Patterns
00430| 
00431| ### 1. Adapter
00432| 
00433| ```typescript
00434| // Adapter: Make incompatible interfaces compatible
00435| interface ModernPaymentProcessor {
00436|   processPayment(amount: number): Promise<{ success: boolean; transactionId: string }>;
00437| }
00438| 
00439| class LegacyPaymentSystem {
00440|   legacyProcess(amount: number, currency: string): string {
00441|     return `Legacy processing: ${amount} ${currency}`;
00442|   }
00443| }
00444| 
00445| class PaymentAdapter implements ModernPaymentProcessor {
00446|   constructor(private legacySystem: LegacyPaymentSystem) {}
00447| 
00448|   async processPayment(amount: number): Promise<{ success: boolean; transactionId: string }> {
00449|     const result = this.legacySystem.legacyProcess(amount, 'USD');
00450|     return {
00451|       success: true,
00452|       transactionId: `TXN-${Date.now()}`,
00453|     };
00454|   }
00455| }
00456| 
00457| // Third-party library with different interface
00458| class ExternalPaymentService {
00459|   charge(cardToken: string, amountInCents: number): { id: string; status: string } {
00460|     return {
00461|       id: `EXT-${Date.now()}`,
00462|       status: 'charged',
00463|     };
00464|   }
00465| }
00466| 
00467| class ExternalPaymentAdapter implements ModernPaymentProcessor {
00468|   constructor(private externalService: ExternalPaymentService) {}
00469| 
00470|   async processPayment(amount: number): Promise<{ success: boolean; transactionId: string }> {
00471|     const result = this.externalService.charge('token', Math.round(amount * 100));
00472|     return {
00473|       success: result.status === 'charged',
00474|       transactionId: result.id,
00475|     };
00476|   }
00477| }
00478| 
00479| // Usage
00480| const legacySystem = new LegacyPaymentSystem();
00481| const adapter = new PaymentAdapter(legacySystem);
00482| await adapter.processPayment(99.99);
00483| 
00484| const externalService = new ExternalPaymentService();
00485| const externalAdapter = new ExternalPaymentAdapter(externalService);
00486| await externalAdapter.processPayment(149.99);
00487| ```
00488| 
00489| ### 2. Bridge
00490| 
00491| ```typescript
00492| // Bridge: Decouple abstraction from implementation
00493| interface Device {
00494|   turnOn(): void;
00495|   turnOff(): void;
00496|   setVolume(percent: number): void;
00497| }
00498| 
00499| class TV implements Device {
00500|   private volume: number = 0;
00501|   private isOn: boolean = false;
00502| 
00503|   turnOn(): void {
00504|     this.isOn = true;
00505|     console.log('TV is on');
00506|   }
00507| 
00508|   turnOff(): void {
00509|     this.isOn = false;
00510|     console.log('TV is off');
00511|   }
00512| 
00513|   setVolume(percent: number): void {
00514|     this.volume = percent;
00515|     console.log(`TV volume set to ${percent}%`);
00516|   }
00517| }
00518| 
00519| class Radio implements Device {
00520|   private volume: number = 0;
00521|   private isOn: boolean = false;
00522| 
00523|   turnOn(): void {
00524|     this.isOn = true;
00525|     console.log('Radio is on');
00526|   }
00527| 
00528|   turnOff(): void {
00529|     this.isOn = false;
00530|     console.log('Radio is off');
00531|   }
00532| 
00533|   setVolume(percent: number): void {
00534|     this.volume = percent;
00535|     console.log(`Radio volume set to ${percent}%`);
00536|   }
00537| }
00538| 
00539| abstract class RemoteControl {
00540|   protected device: Device;
00541| 
00542|   constructor(device: Device) {
00543|     this.device = device;
00544|   }
00545| 
00546|   abstract power(): void;
00547|   abstract volumeUp(): void;
00548|   abstract volumeDown(): void;
00549| }
00550| 
00551| class BasicRemote extends RemoteControl {
00552|   power(): void {
00553|     this.device.turnOn();
00554|   }
00555| 
00556|   volumeUp(): void {
00557|     this.device.setVolume(100);
00558|   }
00559| 
00560|   volumeDown(): void {
00561|     this.device.setVolume(0);
00562|   }
00563| }
00564| 
00565| class AdvancedRemote extends RemoteControl {
00566|   power(): void {
00567|     if (this.device instanceof TV) {
00568|       this.device.turnOn();
00569|     } else {
00570|       this.device.turnOff();
00571|     }
00572|   }
00573| 
00574|   volumeUp(): void {
00575|     this.device.setVolume(80);
00576|   }
00577| 
00578|   volumeDown(): void {
00579|     this.device.setVolume(20);
00580|   }
00581| 
00582|   mute(): void {
00583|     this.device.setVolume(0);
00584|   }
00585| }
00586| 
00587| // Usage
00588| const tv = new TV();
00589| const basicRemote = new BasicRemote(tv);
00590| basicRemote.power();
00591| 
00592| const radio = new Radio();
00593| const advancedRemote = new AdvancedRemote(radio);
00594| advancedRemote.power();
00595| advancedRemote.mute();
00596| ```
00597| 
00598| ### 3. Composite
00599| 
00600| ```typescript
00601| // Composite: Compose objects into tree structures
00602| interface FileSystemComponent {
00603|   getName(): string;
00604|   getSize(): number;
00605|   list(depth?: number): string;
00606| }
00607| 
00608| class File implements FileSystemComponent {
00609|   constructor(
00610|     private name: string,
00611|     private size: number
00612|   ) {}
00613| 
00614|   getName(): string {
00615|     return this.name;
00616|   }
00617| 
00618|   getSize(): number {
00619|     return this.size;
00620|   }
00621| 
00622|   list(depth: number = 0): string {
00623|     return `${'  '.repeat(depth)}📄 ${this.name} (${this.size} bytes)`;
00624|   }
00625| }
00626| 
00627| class Directory implements FileSystemComponent {
00628|   private children: FileSystemComponent[] = [];
00629| 
00630|   constructor(private name: string) {}
00631| 
00632|   add(component: FileSystemComponent): void {
00633|     this.children.push(component);
00634|   }
00635| 
00636|   remove(component: FileSystemComponent): void {
00637|     const index = this.children.indexOf(component);
00638|     if (index > -1) {
00639|       this.children.splice(index, 1);
00640|     }
00641|   }
00642| 
00643|   getName(): string {
00644|     return this.name;
00645|   }
00646| 
00647|   getSize(): number {
00648|     return this.children.reduce((total, child) => total + child.getSize(), 0);
00649|   }
00650| 
00651|   list(depth: number = 0): string {
00652|     const indent = '  '.repeat(depth);
00653|     const lines = [`${indent}📁 ${this.name}/`];
00654|     for (const child of this.children) {
00655|       lines.push(child.list(depth + 1));
00656|     }
00657|     return lines.join('\n');
00658|   }
00659| }
00660| 
00661| // Usage
00662| const root = new Directory('root');
00663| const docs = new Directory('documents');
00664| const pictures = new Directory('pictures');
00665| 
00666| docs.add(new File('resume.pdf', 1024));
00667| docs.add(new File('budget.xlsx', 2048));
00668| pictures.add(new File('vacation.jpg', 4096));
00669| pictures.add(new File('profile.png', 1536));
00670| 
00671| root.add(docs);
00672| root.add(pictures);
00673| 
00674| console.log(root.list());
00675| console.log(`\nTotal size: ${root.getSize()} bytes`);
00676| ```
00677| 
00678| ### 4. Decorator
00679| 
00680| ```typescript
00681| // Decorator: Add behavior dynamically
00682| interface Coffee {
00683|   getDescription(): string;
00684|   getCost(): number;
00685| }
00686| 
00687| class Espresso implements Coffee {
00688|   getDescription(): string {
00689|     return 'Espresso';
00690|   }
00691| 
00692|   getCost(): number {
00693|     return 3.00;
00694|   }
00695| }
00696| 
00697| abstract class CoffeeDecorator implements Coffee {
00698|   protected coffee: Coffee;
00699| 
00700|   constructor(coffee: Coffee) {
00701|     this.coffee = coffee;
00702|   }
00703| 
00704|   abstract getDescription(): string;
00705|   abstract getCost(): number;
00706| }
00707| 
00708| class MilkDecorator extends CoffeeDecorator {
00709|   getDescription(): string {
00710|     return this.coffee.getDescription() + ', Milk';
00711|   }
00712| 
00713|   getCost(): number {
00714|     return this.coffee.getCost() + 0.50;
00715|   }
00716| }
00717| 
00718| class SugarDecorator extends CoffeeDecorator {
00719|   getDescription(): string {
00720|     return this.coffee.getDescription() + ', Sugar';
00721|   }
00722| 
00723|   getCost(): number {
00724|     return this.coffee.getCost() + 0.25;
00725|   }
00726| }
00727| 
00728| class WhipDecorator extends CoffeeDecorator {
00729|   getDescription(): string {
00730|     return this.coffee.getDescription() + ', Whip';
00731|   }
00732| 
00733|   getCost(): number {
00734|     return this.coffee.getCost() + 0.75;
00735|   }
00736| }
00737| 
00738| class CaramelDecorator extends CoffeeDecorator {
00739|   getDescription(): string {
00740|     return this.coffee.getDescription() + ', Caramel';
00741|   }
00742| 
00743|   getCost(): number {
00744|     return this.coffee.getCost() + 0.60;
00745|   }
00746| }
00747| 
00748| // Usage
00749| let coffee: Coffee = new Espresso();
00750| console.log(`${coffee.getDescription()}: $${coffee.getCost().toFixed(2)}`);
00751| 
00752| coffee = new MilkDecorator(coffee);
00753| console.log(`${coffee.getDescription()}: $${coffee.getCost().toFixed(2)}`);
00754| 
00755| coffee = new WhipDecorator(coffee);
00756| console.log(`${coffee.getDescription()}: $${coffee.getCost().toFixed(2)}`);
00757| ```
00758| 
00759| ### 5. Facade
00760| 
00761| ```typescript
00762| // Facade: Provide simplified interface to complex subsystem
00763| class CPU {
00764|   startProcess(): void {
00765|     console.log('CPU: Starting process...');
00766|   }
00767| 
00768|   executeInstruction(): void {
00769|     console.log('CPU: Executing instruction...');
00770|   }
00771| }
00772| 
00773| class Memory {
00774|   allocate(size: number): void {
00775|     console.log(`Memory: Allocating ${size} bytes...`);
00776|   }
00777| 
00778|   load(address: number, data: string): void {
00779|     console.log(`Memory: Loading ${data} at address ${address}`);
00780|   }
00781| }
00782| 
00783| class HardDrive {
00784|   readSector(sector: number): string {
00785|     console.log(`HardDrive: Reading sector ${sector}`);
00786|     return '0101010';
00787|   }
00788| }
00789| 
00790| class ComputerFacade {
00791|   private cpu = new CPU();
00792|   private memory = new Memory();
00793|   private hardDrive = new HardDrive();
00794| 
00795|   startComputer(): void {
00796|     console.log('=== Computer Startup ===');
00797|     this.cpu.startProcess();
00798|     this.memory.allocate(1024);
00799|     const data = this.hardDrive.readSector(100);
00800|     this.memory.load(0x1000, data);
00801|     this.cpu.executeInstruction();
00802|     console.log('=== Computer Ready ===\n');
00803|   }
00804| }
00805| 
00806| // Usage
00807| const computer = new ComputerFacade();
00808| computer.startComputer();
00809| ```
00810| 
00811| ### 6. Flyweight
00812| 
00813| ```typescript
00814| // Flyweight: Share common state to support large numbers of objects
00815| interface TreeType {
00816|   render(x: number, y: number, color: string): void;
00817| }
00818| 
00819| class PineTree implements TreeType {
00820|   render(x: number, y: number, color: string): void {
00821|     console.log(`Rendering PINE tree at (${x}, ${y}) with color ${color}`);
00822|   }
00823| }
00824| 
00825| class OakTree implements TreeType {
00826|   render(x: number, y: number, color: string): void {
00827|     console.log(`Rendering OAK tree at (${x}, ${y}) with color ${color}`);
00828|   }
00829| }
00830| 
00831| class TreeFactory {
00832|   private static treeTypes: Map<string, TreeType> = new Map();
00833| 
00834|   static getTreeType(type: string): TreeType {
00835|     if (!this.treeTypes.has(type)) {
00836|       switch (type) {
00837|         case 'pine':
00838|           this.treeTypes.set(type, new PineTree());
00839|           break;
00840|         case 'oak':
00841|           this.treeTypes.set(type, new OakTree());
00842|           break;
00843|       }
00844|     }
00845|     return this.treeTypes.get(type)!;
00846|   }
00847| }
00848| 
00849| class Tree {
00850|   constructor(
00851|     private x: number,
00852|     private y: number,
00853|     private type: string,
00854|     private color: string
00855|   ) {}
00856| 
00857|   render(): void {
00858|     const treeType = TreeFactory.getTreeType(this.type);
00859|     treeType.render(this.x, this.y, this.color);
00860|   }
00861| }
00862| 
00863| class Forest {
00864|   private trees: Tree[] = [];
00865| 
00866|   plantTree(x: number, y: number, type: string, color: string): void {
00867|     this.trees.push(new Tree(x, y, type, color));
00868|   }
00869| 
00870|   renderAll(): void {
00871|     console.log('=== Rendering Forest ===');
00872|     this.trees.forEach((tree) => tree.render());
00873|   }
00874| }
00875| 
00876| // Usage
00877| const forest = new Forest();
00878| forest.plantTree(10, 20, 'pine', 'darkgreen');
00879| forest.plantTree(15, 25, 'oak', 'lightgreen');
00880| forest.plantTree(12, 22, 'pine', 'green');
00881| // ... plant thousands of trees with only 2 TreeType instances
00882| forest.renderAll();
00883| ```
00884| 
00885| ### 7. Proxy
00886| 
00887| ```typescript
00888| // Proxy: Provide a surrogate for another object
00889| interface Database {
00890|   query(sql: string): Promise<any[]>;
00891| }
00892| 
00893| class RealDatabase implements Database {
00894|   async query(sql: string): Promise<any[]> {
00895|     console.log(`Executing query: ${sql}`);
00896|     await new Promise((resolve) => setTimeout(resolve, 1000));
00897|     return [{ id: 1, name: 'John' }];
00898|   }
00899| }
00900| 
00901| class CachedDatabase implements Database {
00902|   private cache: Map<string, any[]> = new Map();
00903|   private database: RealDatabase;
00904| 
00905|   constructor(database: RealDatabase) {
00906|     this.database = database;
00907|   }
00908| 
00909|   async query(sql: string): Promise<any[]> {
00910|     if (this.cache.has(sql)) {
00911|       console.log('Returning cached result');
00912|       return this.cache.get(sql)!;
00913|     }
00914| 
00915|     const result = await this.database.query(sql);
00916|     this.cache.set(sql, result);
00917|     return result;
00918|   }
00919| }
00920| 
00921| class LoggingDatabase implements Database {
00922|   private database: Database;
00923| 
00924|   constructor(database: Database) {
00925|     this.database = database;
00926|   }
00927| 
00928|   async query(sql: string): Promise<any[]> {
00929|     const start = Date.now();
00930|     const result = await this.database.query(sql);
00931|     const duration = Date.now() - start;
00932|     console.log(`Query executed in ${duration}ms, returned ${result.length} rows`);
00933|     return result;
00934|   }
00935| }
00936| 
00937| // Usage
00938| const realDb = new RealDatabase();
00939| const cachedDb = new CachedDatabase(realDb);
00940| const loggedDb = new LoggingDatabase(cachedDb);
00941| 
00942| await loggedDb.query('SELECT * FROM users');
00943| await loggedDb.query('SELECT * FROM users'); // Uses cache
00944| ```
00945| 
00946| ---
00947| 
00948| ## Behavioral Patterns
00949| 
00950| ### 1. Chain of Responsibility
00951| 
00952| ```typescript
00953| // Chain of Responsibility: Pass request along a chain of handlers
00954| abstract class Handler {
00955|   protected next: Handler | null = null;
00956| 
00957|   setNext(handler: Handler): Handler {
00958|     this.next = handler;
00959|     return handler;
00960|   }
00961| 
00962|   abstract handle(request: string): string;
00963| }
00964| 
00965| class AuthenticationHandler extends Handler {
00966|   handle(request: string): string {
00967|     if (request.includes('token')) {
00968|       return `AuthenticationHandler: Authenticated ${request}`;
00969|     }
00970|     if (this.next) {
00971|       return this.next.handle(request);
00972|     }
00973|     return 'Request rejected: No authentication';
00974|   }
00975| }
00976| 
00977| class ValidationHandler extends Handler {
00978|   handle(request: string): string {
00979|     if (request.includes('valid')) {
00980|       return `ValidationHandler: Validated ${request}`;
00981|     }
00982|     if (this.next) {
00983|       return this.next.handle(request);
00984|     }
00985|     return 'Request rejected: Validation failed';
00986|   }
00987| }
00988| 
00989| class LoggingHandler extends Handler {
00990|   handle(request: string): string {
00991|     if (request.includes('log')) {
00992|       return `LoggingHandler: Logged ${request}`;
00993|     }
00994|     if (this.next) {
00995|       return this.next.handle(request);
00996|     }
00997|     return 'Request rejected: Logging failed';
00998|   }
00999| }
01000| 
01001| // Usage
01002| const auth = new AuthenticationHandler();
01003| const validation = new ValidationHandler();
01004| const logging = new LoggingHandler();
01005| 
01006| auth.setNext(validation).setNext(logging);
01007| 
01008| console.log(auth.handle('request with token'));
01009| console.log(auth.handle('request valid'));
01010| console.log(auth.handle('request log'));
01011| ```
01012| 
01013| ### 2. Command
01014| 
01015| ```typescript
01016| // Command: Encapsulate request as an object
01017| interface Command {
01018|   execute(): void;
01019|   undo(): void;
01020| }
01021| 
01022| class Light {
01023|   private isOn: boolean = false;
01024| 
01025|   on(): void {
01026|     this.isOn = true;
01027|     console.log('Light is ON');
01028|   }
01029| 
01030|   off(): void {
01031|     this.isOn = false;
01032|     console.log('Light is OFF');
01033|   }
01034| 
01035|   getState(): boolean {
01036|     return this.isOn;
01037|   }
01038| }
01039| 
01040| class LightOnCommand implements Command {
01041|   constructor(private light: Light) {}
01042| 
01043|   execute(): void {
01044|     this.light.on();
01045|   }
01046| 
01047|   undo(): void {
01048|     this.light.off();
01049|   }
01050| }
01051| 
01052| class LightOffCommand implements Command {
01053|   constructor(private light: Light) {}
01054| 
01055|   execute(): void {
01056|     this.light.off();
01057|   }
01058| 
01059|   undo(): void {
01060|     this.light.on();
01061|   }
01062| }
01063| 
01064| class RemoteControl {
01065|   private commands: Map<number, Command> = new Map();
01066|   private history: Command[] = [];
01067| 
01068|   setCommand(slot: number, command: Command): void {
01069|     this.commands.set(slot, command);
01070|   }
01071| 
01072|   pressButton(slot: number): void {
01073|     const command = this.commands.get(slot);
01074|     if (command) {
01075|       command.execute();
01076|       this.history.push(command);
01077|     }
01078|   }
01079| 
01080|   pressUndo(): void {
01081|     const command = this.history.pop();
01082|     if (command) {
01083|       command.undo();
01084|     }
01085|   }
01086| }
01087| 
01088| // Usage
01089| const light = new Light();
01090| const lightOn = new LightOnCommand(light);
01091| const lightOff = new LightOffCommand(light);
01092| 
01093| const remote = new RemoteControl();
01094| remote.setCommand(1, lightOn);
01095| remote.setCommand(2, lightOff);
01096| 
01097| remote.pressButton(1);
01098| remote.pressButton(2);
01099| remote.pressUndo();
01100| ```
01101| 
01102| ### 3. Iterator
01103| 
01104| ```typescript
01105| // Iterator: Traverse collections without exposing underlying structure
01106| interface Iterator<T> {
01107|   hasNext(): boolean;
01108|   next(): T;
01109| }
01110| 
01111| interface Aggregator<T> {
01112|   createIterator(): Iterator<T>;
01113| }
01114| 
01115| class Node<T> {
01116|   constructor(public value: T, public left: Node<T> | null = null, public right: Node<T> | null = null) {}
01117| }
01118| 
01119| class BinaryTreeIterator<T> implements Iterator<T> {
01120|   private stack: Node<T>[] = [];
01121|   private current: Node<T> | null;
01122| 
01123|   constructor(root: Node<T> | null) {
01124|     this.current = root;
01125|   }
01126| 
01127|   hasNext(): boolean {
01128|     return this.current !== null || this.stack.length > 0;
01129|   }
01130| 
01131|   next(): T {
01132|     while (this.current) {
01133|       this.stack.push(this.current);
01134|       this.current = this.current.left;
01135|     }
01136|     this.current = this.stack.pop()!;
01137|     const value = this.current.value;
01138|     this.current = this.current.right;
01139|     return value;
01140|   }
01141| }
01142| 
01143| class BinaryTree<T> implements Aggregator<T> {
01144|   constructor(public root: Node<T> | null) {}
01145| 
01146|   createIterator(): Iterator<T> {
01147|     return new BinaryTreeIterator(this.root);
01148|   }
01149| }
01150| 
01151| // Usage
01152| const root = new Node(1);
01153| root.left = new Node(2);
01154| root.right = new Node(3);
01155| root.left.left = new Node(4);
01156| root.left.right = new Node(5);
01157| 
01158| const tree = new BinaryTree(root);
01159| const iterator = tree.createIterator();
01160| 
01161| console.log('In-order traversal:');
01162| while (iterator.hasNext()) {
01163|   console.log(iterator.next());
01164| }
01165| ```
01166| 
01167| ### 4. Mediator
01168| 
01169| ```typescript
01170| // Mediator: Define object that encapsulates interaction
01171| interface ChatMediator {
01172|   sendMessage(msg: string, user: User): void;
01173|   addUser(user: User): void;
01174| }
01175| 
01176| class ChatRoom implements ChatMediator {
01177|   private users: User[] = [];
01178| 
01179|   addUser(user: User): void {
01180|     this.users.push(user);
01181|     user.setMediator(this);
01182|   }
01183| 
01184|   sendMessage(msg: string, sender: User): void {
01185|     for (const user of this.users) {
01186|       if (user !== sender) {
01187|         user.receive(msg);
01188|       }
01189|     }
01190|   }
01191| }
01192| 
01193| abstract class User {
01194|   protected mediator: ChatMediator | null = null;
01195|   protected name: string;
01196| 
01197|   constructor(name: string) {
01198|     this.name = name;
01199|   }
01200| 
01201|   setMediator(mediator: ChatMediator): void {
01202|     this.mediator = mediator;
01203|   }
01204| 
01205|   abstract send(msg: string): void;
01206|   abstract receive(msg: string): void;
01207| }
01208| 
01209| class ConcreteUser extends User {
01210|   constructor(name: string) {
01211|     super(name);
01212|   }
01213| 
01214|   send(msg: string): void {
01215|     console.log(`${this.name} sends: ${msg}`);
01216|     this.mediator?.sendMessage(msg, this);
01217|   }
01218| 
01219|   receive(msg: string): void {
01220|     console.log(`${this.name} receives: ${msg}`);
01221|   }
01222| }
01223| 
01224| // Usage
01225| const chatRoom = new ChatRoom();
01226| 
01227| const user1 = new ConcreteUser('Alice');
01228| const user2 = new ConcreteUser('Bob');
01229| const user3 = new ConcreteUser('Charlie');
01230| 
01231| chatRoom.addUser(user1);
01232| chatRoom.addUser(user2);
01233| chatRoom.addUser(user3);
01234| 
01235| user1.send('Hello everyone!');
01236| user2.send('Hi Alice!');
01237| ```
01238| 
01239| ### 5. Observer
01240| 
01241| ```typescript
01242| // Observer: Notify multiple objects about state changes
01243| interface Observer {
01244|   update(subject: Subject): void;
01245| }
01246| 
01247| interface Subject {
01248|   attach(observer: Observer): void;
01249|   detach(observer: Observer): void;
01250|   notify(): void;
01251| }
01252| 
01253| class WeatherStation implements Subject {
01254|   private observers: Observer[] = [];
01255|   private temperature: number = 0;
01256|   private humidity: number = 0;
01257| 
01258|   attach(observer: Observer): void {
01259|     this.observers.push(observer);
01260|   }
01261| 
01262|   detach(observer: Observer): void {
01263|     const index = this.observers.indexOf(observer);
01264|     if (index > -1) {
01265|       this.observers.splice(index, 1);
01266|     }
01267|   }
01268| 
01269|   notify(): void {
01270|     for (const observer of this.observers) {
01271|       observer.update(this);
01272|     }
01273|   }
01274| 
01275|   setMeasurements(temp: number, humidity: number): void {
01276|     this.temperature = temp;
01277|     this.humidity = humidity;
01278|     this.notify();
01279|   }
01280| 
01281|   getTemperature(): number {
01282|     return this.temperature;
01283|   }
01284| 
01285|   getHumidity(): number {
01286|     return this.humidity;
01287|   }
01288| }
01289| 
01290| class PhoneDisplay implements Observer {
01291|   update(subject: Subject): void {
01292|     if (subject instanceof WeatherStation) {
01293|       console.log(`Phone Display: Temp=${subject.getTemperature()}°C, Humidity=${subject.getHumidity()}%`);
01294|     }
01295|   }
01296| }
01297| 
01298| class WebDisplay implements Observer {
01299|   update(subject: Subject): void {
01300|     if (subject instanceof WeatherStation) {
01301|       console.log(`Web Display: Updated weather - ${subject.getTemperature()}°C`);
01302|     }
01303|   }
01304| }
01305| 
01306| // Usage
01307| const weatherStation = new WeatherStation();
01308| const phone = new PhoneDisplay();
01309| const web = new WebDisplay();
01310| 
01311| weatherStation.attach(phone);
01312| weatherStation.attach(web);
01313| 
01314| weatherStation.setMeasurements(25, 60);
01315| weatherStation.setMeasurements(28, 55);
01316| ```
01317| 
01318| ### 6. State
01319| 
01320| ```typescript
01321| // State: Alter behavior based on internal state
01322| interface State {
01323|   handle(context: Context): void;
01324| }
01325| 
01326| class Context {
01327|   private state: State;
01328| 
01329|   constructor(state: State) {
01330|     this.state = state;
01331|   }
01332| 
01333|   setState(state: State): void {
01334|     this.state = state;
01335|   }
01336| 
01337|   request(): void {
01338|     this.state.handle(this);
01339|   }
01340| }
01341| 
01342| class PendingState implements State {
01343|   handle(context: Context): void {
01344|     console.log('Request is pending...');
01345|     context.setState(new ProcessingState());
01346|   }
01347| }
01348| 
01349| class ProcessingState implements State {
01350|   handle(context: Context): void {
01351|     console.log('Request is processing...');
01352|     context.setState(new CompletedState());
01353|   }
01354| }
01355| 
01356| class CompletedState implements State {
01357|   handle(context: Context): void {
01358|     console.log('Request is already completed!');
01359|   }
01360| }
01361| 
01362| class RejectedState implements State {
01363|   handle(context: Context): void {
01364|     console.log('Request has been rejected');
01365|   }
01366| }
01367| 
01368| // Usage
01369| const context = new Context(new PendingState());
01370| context.request();
01371| context.request();
01372| context.request();
01373| ```
01374| 
01375| ### 7. Strategy
01376| 
01377| ```typescript
01378| // Strategy: Define family of algorithms and make them interchangeable
01379| interface PaymentStrategy {
01380|   pay(amount: number): boolean;
01381| }
01382| 
01383| class CreditCardStrategy implements PaymentStrategy {
01384|   constructor(
01385|     private cardNumber: string,
01386|     private cvv: string,
01387|     private expiry: string
01388|   ) {}
01389| 
01390|   pay(amount: number): boolean {
01391|     console.log(`Paying $${amount} with credit card ${this.cardNumber}`);
01392|     return true;
01393|   }
01394| }
01395| 
01396| class PayPalStrategy implements PaymentStrategy {
01397|   constructor(private email: string) {}
01398| 
01399|   pay(amount: number): boolean {
01400|     console.log(`Paying $${amount} with PayPal account ${this.email}`);
01401|     return true;
01402|   }
01403| }
01404| 
01405| class CryptoStrategy implements PaymentStrategy {
01406|   constructor(private walletAddress: string) {}
01407| 
01408|   pay(amount: number): boolean {
01409|     console.log(`Paying $${amount} with crypto wallet ${this.walletAddress}`);
01410|     return true;
01411|   }
01412| }
01413| 
01414| class ShoppingCart {
01415|   private items: { name: string; price: number }[] = [];
01416|   private paymentStrategy: PaymentStrategy | null = null;
01417| 
01418|   addItem(name: string, price: number): void {
01419|     this.items.push({ name, price });
01420|   }
01421| 
01422|   getTotal(): number {
01423|     return this.items.reduce((sum, item) => sum + item.price, 0);
01424|   }
01425| 
01426|   setPaymentStrategy(strategy: PaymentStrategy): void {
01427|     this.paymentStrategy = strategy;
01428|   }
01429| 
01430|   checkout(): boolean {
01431|     if (!this.paymentStrategy) {
01432|       console.log('No payment strategy set');
01433|       return false;
01434|     }
01435|     return this.paymentStrategy.pay(this.getTotal());
01436|   }
01437| }
01438| 
01439| // Usage
01440| const cart = new ShoppingCart();
01441| cart.addItem('Laptop', 999);
01442| cart.addItem('Mouse', 29);
01443| 
01444| cart.setPaymentStrategy(new CreditCardStrategy('4111', '123', '12/25'));
01445| cart.checkout();
01446| 
01447| cart.setPaymentStrategy(new PayPalStrategy('user@example.com'));
01448| cart.checkout();
01449| ```
01450| 
01451| ### 8. Template Method
01452| 
01453| ```typescript
01454| // Template Method: Define skeleton of algorithm in a method
01455| abstract class Game {
01456|   protected abstract initialize(): void;
01457|   protected abstract startPlay(): void;
01458|   protected abstract endPlay(): void;
01459| 
01460|   play(): void {
01461|     this.initialize();
01462|     this.startPlay();
01463|     this.endPlay();
01464|   }
01465| }
01466| 
01467| class Cricket extends Game {
01468|   protected initialize(): void {
01469|     console.log('Cricket game initialized');
01470|   }
01471| 
01472|   protected startPlay(): void {
01473|     console.log('Cricket game started');
01474|   }
01475| 
01476|   protected endPlay(): void {
01477|     console.log('Cricket game ended');
01478|   }
01479| }
01480| 
01481| class Football extends Game {
01482|   protected initialize(): void {
01483|     console.log('Football game initialized');
01484|   }
01485| 
01486|   protected startPlay(): void {
01487|     console.log('Football game started');
01488|   }
01489| 
01490|   protected endPlay(): void {
01491|     console.log('Football game ended');
01492|   }
01493| }
01494| 
01495| // Usage
01496| const cricket = new Cricket();
01497| cricket.play();
01498| 
01499| const football = new Football();
01500| football.play();
01501| ```
01502| 
01503| ---
01504| 
01505| ## Design Pattern Selection Guide
01506| 
01507| | Problem | Recommended Patterns |
01508| |---------|---------------------|
01509| | Creating objects with complex construction | Builder, Factory Method, Abstract Factory |
01510| | Ensuring single instance | Singleton |
01511| | Adapting incompatible interfaces | Adapter |
01512| | Decoupling abstraction from implementation | Bridge |
01513| | Treating individual and composite objects uniformly | Composite |
01514| | Adding responsibilities dynamically | Decorator |
01515| | Providing simplified interface to complex system | Facade |
01516| | Sharing common state efficiently | Flyweight |
01517| | Controlling access to objects | Proxy |
01518| | Handling requests through a chain | Chain of Responsibility |
01519| | Encapsulating requests as objects | Command |
01520| | Traversing collections | Iterator |
01521| | Centralizing communication between objects | Mediator |
01522| | Notifying objects about state changes | Observer |
01523| | Changing behavior with state | State |
01524| | Selecting algorithm at runtime | Strategy |
01525| | Defining algorithm skeleton | Template Method |
01526| 
01527| ---
01528| 
01529| ## References and Further Reading
01530| 
01531| 1. Gamma, Erich, et al. "Design Patterns: Elements of Reusable Object-Oriented Software." Addison-Wesley, 1994.
01532| 2. Freeman, Eric, and Elisabeth Robson. "Head First Design Patterns." O'Reilly, 2004.
01533| 3. Refactoring Guru. "Design Patterns." https://refactoring.guru/design-patterns
01534| 4. "AntiPatterns: Refactoring Software, Architectures, and Projects in Crisis." Brown et al., Wiley, 1998.
01535| 

(End of file - total 1535 lines)
</file>
```
```

```
Tool: read

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/index.md"
}
```
**Output:**
```
<file>
00001| # Software Engineering Concepts and Methodologies
00002| 
00003| A comprehensive documentation of modern software engineering concepts, methodologies, and practices relevant to AI-assisted development.
00004| 
00005| ## Documentation Index
00006| 
00007| ### Requirements & Analysis
00008| - [Requirements and Specification](requirements-and-specification.md) - Understanding software requirements and specification documents
00009| - [Specification Driven Development](specification-driven-development.md) - Development approach centered around formal specifications
00010| - [Problem Frames Approach](problem-frames-approach.md) - Michael Jackson's framework for requirements analysis
00011| 
00012| ### Domain-Driven Design
00013| - [Domain-Driven Design](domain-driven-design-megred.md) - Strategic and tactical DDD patterns
00014| - [Domain Event Storming](domain-event-storming.md) - Collaborative event modeling technique
00015| - [Domain Event Storytelling](domain-event-storytelling.md) - Narrative approach to domain events
00016| 
00017| ### Architecture & Code Quality
00018| - [Layered Architecture](layered-architecture.md) - Ports and Adapters / Clean Architecture principles
00019| - [Clean Code](clean-code.md) - Writing maintainable and expressive code
00020| - [Vertical Slice Architecture](vertical-slice-architecture-megred.md) - Feature-based project organization
00021| 
00022| ### Specification & Testing
00023| - [Specification by Example](specification-by-example.md) - Requirements through concrete examples
00024| - [Design by Contract](design-by-contract.md) - Formal verification through contracts
00025| - [Executable Specifications](executable-specifications.md) - Living documentation as tests
00026| - [Living Documentation](living-documentation.md) - Self-updating project documentation
00027| 
00028| ### Testing Methodologies
00029| - [Test-Driven Development](test-driven-development.md) - TDD principles and practices
00030| - [Behavior-Driven Development](behavior-driven-development.md) - BDD with Gherkin and Cucumber
00031| 
00032| ### Advanced Patterns
00033| - [Event Sourcing](event-sourcing.md) - Pattern for storing state changes
00034| - [CQRS](cqrs.md) - Command Query Responsibility Segregation
00035| - [Design Patterns](design-patterns.md) - Gang of Four patterns catalog
00036| - [Pattern Language](pattern-language.md) - Christopher Alexander's pattern language concept
00037| 
00038| ### Collaborative Planning
00039| - [Impact Mapping](impact-mapping.md) - Gojko Adzic's strategic planning technique
00040| - [Example Mapping](example-mapping.md) - BDD example discovery technique
00041| 
00042| ### Coding Standards
00043| - [Coding Conventions](coding-conventions.md) - Universal naming, organization, and style guidelines
00044| 
00045| ### AI-Assisted Development
00046| - [AGENTS.md and CLI AI Agent Tools](agents-md-cli-ai-agent-tools.md) - The AGENTS.md standard and supporting tools
00047| - [Prompt Engineering for AI Agents](prompt-engineering-ai-agents.md) - Techniques for effective agent instructions
00048| - [AI Agent Limitations](ai-agent-limitations.md) - Why agents skip instructions and fundamental constraints
00049| - [On-Demand Rule Loading](on-demand-rule-loading.md) - Techniques to load rules only when needed
00050| 
00051| ## Quick Links
00052| 
00053| ### For AI Coding Assistants
00054| All documentation is optimized for AI-assisted development, including:
00055| - Practical implementation guidance
00056| - Code examples and patterns
00057| - Tool recommendations
00058| - Best practices for each methodology
00059| - Anti-patterns to avoid
00060| 
00061| ### How to Use This Documentation
00062| 1. **New Projects**: Start with Requirements and Specification, then explore Architecture patterns
00063| 2. **Refactoring**: Reference Layered Architecture and Clean Code for guidance
00064| 3. **Team Collaboration**: Use Impact Mapping and Example Mapping for planning
00065| 4. **Quality Assurance**: Implement TDD, BDD, and Executable Specifications
00066| 5. **AI-Assisted Development**: Use AGENTS.md and Prompt Engineering for effective agent collaboration
00067| 
00068| ### AI Agent Development
00069| When working with AI agents:
00070| 1. Read relevant AGENTS.md files before coding
00071| 2. Reference pattern documents for implementation guidance
00072| 3. Use the Domain-Driven Design document for domain modeling
00073| 4. Follow Coding Conventions for consistent code
00074| 5. Use On-Demand Rule Loading to manage complex rule sets
00075| 
00076| ## Documentation Structure
00077| 
00078| | Category | Topics Covered | Best For |
00079| |----------|---------------|----------|
00080| | **Requirements** | Problem Frames, Spec-by-Example | Understanding what to build |
00081| | **Architecture** | Layered Architecture, Vertical Slices | Structuring the system |
00082| | **DDD** | Strategic/Tactical Patterns, Entity/VO/Aggregate | Complex domain modeling |
00083| | **Testing** | TDD, BDD, Executable Specs | Quality assurance |
00084| | **Patterns** | Design Patterns, Pattern Languages | Reusable solutions |
00085| | **Collaboration** | Impact Mapping, Example Mapping | Team alignment |
00086| | **AI Development** | AGENTS.md, Prompt Engineering, On-Demand Loading | Effective AI collaboration |
00087| 
00088| ## Contributing
00089| 
00090| These documents are maintained as living documentation. Each file includes:
00091| - Theoretical foundations
00092| - Practical implementation guidance
00093| - Code examples
00094| - Common pitfalls and best practices
00095| - References to authoritative sources
00096| 
00097| ### Adding New Documentation
00098| When adding new documentation files:
00099| 1. Follow the established structure and format
00100| 2. Include TypeScript/JavaScript code examples
00101| 3. Add AI-assisted development guidance sections
00102| 4. Document best practices and anti-patterns
00103| 5. Include references to authoritative sources
00104| 
00105| ### Maintenance Guidelines
00106| - Review and update documentation with each major release
00107| - Verify code examples compile and work correctly
00108| - Keep references and links current
00109| - Incorporate feedback from AI-assisted development usage
00110| 

(End of file - total 110 lines)
</file>
```
```

---

## Assistant

_Thinking:_

Looking at the two analysis files I read and the existing documentation, I can identify several key areas where the documentation can be enhanced with knowledge from these analysis files:

From `refactor-to-ca-analysis.md`:
1. **14-step refactoring journey** - This is a concrete example of evolving from monolith to Clean Architecture. Can be referenced in `layered-architecture.md`
2. **Detailed step-by-step analysis** - Shows practical implementation of DDD tactical patterns
3. **Dependency evolution** - Shows how dependencies flow across layers
4. **Testing strategy evolution** - Shows how testing changes as architecture evolves

From `ai-coding-exercise-analysis.md`:
1. **AI Coding Pattern Language** - Formal pattern definition language for AI agents
2. **Specification-Driven Development** - Formal specification structure and organization
3. **Sub-Agent System Architecture** - How AI agents can be specialized
4. **Failure Case Analysis** - Documented AI mistakes and corrections
5. **Code Review Framework** - Checklist-based code review for AI-generated code
6. **Architecture Decision Records (ADRs)** - Documentation format for architectural decisions

Now let me identify which documentation files can be enhanced:

1. **layered-architecture.md** - Can incorporate the 14-step refactoring journey from refactor-to-ca as a practical example
2. **test-driven-development.md** - Can incorporate the testing pyramid and evolution from both analysis files
3. **coding-conventions.md** - Can incorporate the pattern anti-patterns and AI coding patterns from ai-coding-exercise
4. **design-patterns.md** - Can incorporate additional patterns from the analysis (like Input/Output Port pattern, Command/Query Separation)
5. **index.md** - Can add references to new enhanced content

Let me start enhancing these files systematically. I'll begin with the most impactful changes:

1. First, I'll enhance `layered-architecture.md` to include the 14-step refactoring journey


2. Next, I'll update `test-driven-development.md` to incorporate the testing pyramid insights
3. I'll then modify `coding-conventions.md` to include AI-specific coding patterns
4. Finally, I'll expand `design-patterns.md` with additional architectural patterns

I'll proceed with each documentation enhancement systematically, focusing on integrating the insights from the analysis files to provide comprehensive guidance for AI-assisted software development.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/clean-code.md",
  "newString": "## AI-Assisted Clean Code\n\n### Common AI Coding Anti-Patterns\n\nBased on analysis of AI-generated code patterns, these anti-patterns frequently occur and should be explicitly avoided:\n\n```typescript\n// ANTI-PATTERN: Nested Input/Output Classes\n// ❌ AI creates: Separate Input.java, Output.java files\n// ✅ CORRECT: Inner classes within UseCase interface\ninterface CreateProductUseCase {\n    class CreateProductInput implements Input {\n        public productId: string;\n        public name: string;\n        public userId: string;\n    }\n}\n\n// ANTI-PATTERN: if-else instanceof Chain\n// ❌ AI writes: Multiple if-else checks with instanceof\n// ✅ CORRECT: Switch expression with pattern matching\nwhen(event: ProductEvent): void {\n    switch (event) {\n        case ProductEvents.ProductCreated e -> {\n            this.id = e.productId();\n            this.name = e.name();\n        }\n        case ProductEvents.ProductRenamed e -> {\n            this.name = e.newName();\n        }\n    }\n}\n\n// ANTI-PATTERN: Automatic Repository Implementation\n// ❌ AI generates: Custom repository interface\n// ✅ CORRECT: Use framework GenericInMemoryRepository\ninterface ProductRepository extends Repository<Product, ProductId> {\n    List<Product> findBySprintId(SprintId id);  // WRONG: Custom query\n}\n\n// ANTI-PATTERN: Contract Validation in Value Objects\n// ❌ AI uses: Contract.requireNotNull() in Value Object\n// ✅ CORRECT: Use Objects.requireNonNull()\nrecord ProductId(String value) implements ValueObject {\n    public ProductId {\n        Objects.requireNonNull(value, \"Product ID cannot be null\");\n    }\n}\n```\n\n### AI Prompt for Clean Code Refactoring",
  "oldString": "## AI-Assisted Clean Code\n\n```markdown\n## AI Prompt for Clean Code Refactoring"
}
```
```

---

## Assistant

_Thinking:_

The file is incomplete. I need to complete the edit with the rest of the AI-Assisted Clean Code section.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/clean-code.md",
  "newString": "Refactor the following code following Clean Code principles:\n\n```typescript\n// Original code\nfunction p(o) {\n  let r = { s: [], t: 0 };\n  for (let i = 0; i < o.items.length; i++) {\n    let item = o.items[i];\n    if (item.status === 'A' && item.qty > 0) {\n      r.s.push(item);\n      r.t += item.price * item.qty;\n    }\n  }\n  return r;\n}\n```\n\nRequirements:\n1. Use meaningful names\n2. Extract into small, focused functions\n3. Add proper TypeScript types\n4. Add JSDoc documentation\n5. Include error handling\n6. Make it testable\n\nOutput:\n1. Refactored TypeScript code\n2. Explanation of changes made\n3. Unit tests for the function\n```",
  "oldString": "Refactor the following code following Clean Code principles:\n\n```typescript\n// Original code\nfunction p(o) {\n  let r = { s: [], t: 0 };\n  for (let i = 0; i < o.items.length; i++) {\n    let item = o.items[i];\n    if (item.status === 'A' && item.qty > 0) {\n      r.s.push(item);\n      r.t += item.price * item.qty;\n    }\n  }\n  return r;\n}\n```\n\nRequirements:\n1. Use meaningful names\n2. Extract into small, focused functions\n3. Add proper TypeScript types\n4. Add JSDoc documentation\n5. Include error handling\n6. Make it testable\n\nOutput:\n1. Refactored TypeScript code\n2. Explanation of changes made\n3. Unit tests for the function\n```"
}
```
**Error:**
```
Error: oldString and newString must be different
```
```

---

## Assistant

_Thinking:_

The oldString and newString are the same. I need to add more content to the new version to make it different. Let me create a proper enhancement with additional content.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/clean-code.md",
  "newString": "### AI Code Review Checklist\n\nWhen reviewing AI-generated code, use this enhanced checklist:\n\n```markdown\n## AI Code Review Checklist\n\n### 1. Architecture Compliance\n- [ ] Package location follows layer structure\n- [ ] Clean Architecture boundaries respected\n- [ ] Dependency direction is correct (inward)\n\n### 2. Coding Standards\n- [ ] Input class is inner class of UseCase\n- [ ] Value Objects use Objects.requireNonNull()\n- [ ] Repository pattern follows framework conventions\n- [ ] No nested Input/Output files\n\n### 3. Business Logic\n- [ ] Contract validation in constructors\n- [ ] Domain events properly applied\n- [ ] Aggregate invariants enforced\n- [ ] Soft delete pattern implemented\n\n### 4. Testability\n- [ ] Dependencies injected via constructor\n- [ ] No static references to framework\n- [ ] Can be unit tested in isolation\n```\n\n### AI Prompt for Clean Code Refactoring",
  "oldString": "## AI-Assisted Clean Code\n\n```markdown\n## AI Prompt for Clean Code Refactoring"
}
```
**Error:**
```
Error: oldString not found in content
```
```

---

## Assistant

_Thinking:_

The oldString doesn't match. I need to read the file again to see its current content and find the right section to edit.

```
Tool: read

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/clean-code.md",
  "limit": 100,
  "offset": 490
}
```
**Output:**
```
<file>
00491| 
00492| ### Common AI Coding Anti-Patterns
00493| 
00494| Based on analysis of AI-generated code patterns, these anti-patterns frequently occur and should be explicitly avoided:
00495| 
00496| ```typescript
00497| // ANTI-PATTERN: Nested Input/Output Classes
00498| // ❌ AI creates: Separate Input.java, Output.java files
00499| // ✅ CORRECT: Inner classes within UseCase interface
00500| interface CreateProductUseCase {
00501|     class CreateProductInput implements Input {
00502|         public productId: string;
00503|         public name: string;
00504|         public userId: string;
00505|     }
00506| }
00507| 
00508| // ANTI-PATTERN: if-else instanceof Chain
00509| // ❌ AI writes: Multiple if-else checks with instanceof
00510| // ✅ CORRECT: Switch expression with pattern matching
00511| when(event: ProductEvent): void {
00512|     switch (event) {
00513|         case ProductEvents.ProductCreated e -> {
00514|             this.id = e.productId();
00515|             this.name = e.name();
00516|         }
00517|         case ProductEvents.ProductRenamed e -> {
00518|             this.name = e.newName();
00519|         }
00520|     }
00521| }
00522| 
00523| // ANTI-PATTERN: Automatic Repository Implementation
00524| // ❌ AI generates: Custom repository interface
00525| // ✅ CORRECT: Use framework GenericInMemoryRepository
00526| interface ProductRepository extends Repository<Product, ProductId> {
00527|     List<Product> findBySprintId(SprintId id);  // WRONG: Custom query
00528| }
00529| 
00530| // ANTI-PATTERN: Contract Validation in Value Objects
00531| // ❌ AI uses: Contract.requireNotNull() in Value Object
00532| // ✅ CORRECT: Use Objects.requireNonNull()
00533| record ProductId(String value) implements ValueObject {
00534|     public ProductId {
00535|         Objects.requireNonNull(value, "Product ID cannot be null");
00536|     }
00537| }
00538| ```
00539| 
00540| ### AI Prompt for Clean Code Refactoring
00541| 
00542| Refactor the following code following Clean Code principles:
00543| 
00544| ```typescript
00545| // Original code
00546| function p(o) {
00547|   let r = { s: [], t: 0 };
00548|   for (let i = 0; i < o.items.length; i++) {
00549|     let item = o.items[i];
00550|     if (item.status === 'A' && item.qty > 0) {
00551|       r.s.push(item);
00552|       r.t += item.price * item.qty;
00553|     }
00554|   }
00555|   return r;
00556| }
00557| ```
00558| 
00559| Requirements:
00560| 1. Use meaningful names
00561| 2. Extract into small, focused functions
00562| 3. Add proper TypeScript types
00563| 4. Add JSDoc documentation
00564| 5. Include error handling
00565| 6. Make it testable
00566| 
00567| Output:
00568| 1. Refactored TypeScript code
00569| 2. Explanation of changes made
00570| 3. Unit tests for the function
00571| ```
00572| </think>
00573| 
00574| The file has been created. I notice the file was truncated. Let me continue creating the remaining documentation files. Due to the large number of files requested, I'll create them in batches.
00575| <minimax:tool_call>
00576| <invoke name="write">
00577| <parameter name="content"># Clean Code - Continued
00578| 
00579| ## Testability
00580| 
00581| ```typescript
00582| // ✅ Good: Testable code
00583| class PaymentProcessor {
00584|   constructor(
00585|     private readonly paymentGateway: PaymentGateway,
00586|     private readonly logger: Logger
00587|   ) {}
00588| 
00589|   async processPayment(
00590|     orderId: string,

(File has more lines. Use 'offset' parameter to read beyond line 590)
</file>
```
```

---

## Assistant

_Thinking:_

I see the file already has the Common AI Coding Anti-Patterns section I wanted to add. It seems my earlier edit was partially successful or there was already content in the file. Let me now add the AI Code Review Checklist section after the AI Prompt section.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/clean-code.md",
  "newString": "Output:\n1. Refactored TypeScript code\n2. Explanation of changes made\n3. Unit tests for the function\n```\n\n### AI Code Review Checklist\n\nWhen reviewing AI-generated code, use this enhanced checklist:\n\n```markdown\n## AI Code Review Checklist\n\n### 1. Architecture Compliance\n- [ ] Package location follows layer structure\n- [ ] Clean Architecture boundaries respected\n- [ ] Dependency direction is correct (inward)\n\n### 2. Coding Standards\n- [ ] Input class is inner class of UseCase\n- [ ] Value Objects use Objects.requireNonNull()\n- [ ] Repository pattern follows framework conventions\n- [ ] No nested Input/Output files\n\n### 3. Business Logic\n- [ ] Contract validation in constructors\n- [ ] Domain events properly applied\n- [ ] Aggregate invariants enforced\n- [ ] Soft delete pattern implemented\n\n### 4. Testability\n- [ ] Dependencies injected via constructor\n- [ ] No static references to framework\n- [ ] Can be unit tested in isolation\n```\n\n## Testability",
  "oldString": "Output:\n1. Refactored TypeScript code\n2. Explanation of changes made\n3. Unit tests for the function\n```\n\n## Testability"
}
```
```

---

## Assistant

_Thinking:_

Good, the edit was successful. Now let me enhance the coding-conventions.md file with more content from the analysis files, specifically the AI coding patterns and failure cases.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/coding-conventions.md",
  "newString": "## Common Anti-Patterns to Avoid\n\nBased on analysis of AI-generated code, these anti-patterns frequently occur and should be explicitly avoided:\n\n| Anti-Pattern | AI Mistake | Correct Approach |\n|--------------|------------|------------------|\n| Input Class Location | Creates separate Input.java file | Inner class in UseCase interface |\n| Repository Pattern | Custom repository with findBy methods | Use framework GenericInMemoryRepository |\n| instanceof Chain | Multiple if-else with instanceof | Switch expression with pattern matching |\n| Value Object Validation | Uses Contract.requireNotNull() | Uses Objects.requireNonNull() |\n| Primitive Obsession | Uses String/Map for domain types | Value Objects with validation |\n| Nested Classes | Separate Input/Output files | Inner classes within UseCase |\n\n### AI-Specific Naming Conventions\n\nWhen AI generates code, ensure these naming patterns are followed:\n\n```typescript\n// ✅ CORRECT: Use Case Interface Naming\ninterface AddTaskUseCase extends Command<AddTaskInput, CqrsOutput> {}\n\n// ✅ CORRECT: Input as Inner Class\ninterface AddTaskUseCase {\n    class AddTaskInput implements Input {\n        public toDoListId: string;\n        public projectName: string;\n        public description: string;\n    }\n}\n\n// ✅ CORRECT: Service Implementation Naming\nclass AddTaskService implements AddTaskUseCase {\n    constructor(\n        private repository: ToDoListRepository,\n        private presenter: TaskPresenter\n    ) {}\n}\n\n// ✅ CORRECT: Value Object with toString() for Outbox Pattern\nrecord TaskId(String value) implements ValueObject {\n    @Override\n    public String toString() {\n        return value;\n    }\n}\n```\n\n### Bounded Context Rules\n\nFrom DDD analysis, enforce these bounded context rules:\n\n| Rule | Description |\n|------|-------------|\n| BC-001 | Each Aggregate has its own top-level package |\n| BC-002 | Value Objects defined once per Aggregate, imported by others |\n| BC-003 | Cross-BC communication via Domain Events only |\n| BC-004 | Repository only within BC, no cross-BC queries |\n\n1. **Primitive Obsession**: Using primitives instead of value objects\n2. **Feature Envy**: Method accesses data of another class excessively\n3. **God Class**: Class with too many responsibilities\n4. **Magic Numbers/Strings**: Hardcoded values without explanation\n5. **Long Methods**: Methods doing too many things\n6. **Deep Nesting**: Multiple levels of conditionals/loops\n7. **Public Fields**: Exposing internal state\n8. **Null Returns**: Returning null instead of Optional/empty collections\n9. **Mutable Collections**: Exposing internal collections\n10. **Unused Code**: Code that is never called",
  "oldString": "## Common Anti-Patterns to Avoid\n\n1. **Primitive Obsession**: Using primitives instead of value objects\n2. **Feature Envy**: Method accesses data of another class excessively\n3. **God Class**: Class with too many responsibilities\n4. **Magic Numbers/Strings**: Hardcoded values without explanation\n5. **Long Methods**: Methods doing too many things\n6. **Deep Nesting**: Multiple levels of conditionals/loops\n7. **Public Fields**: Exposing internal state\n8. **Null Returns**: Returning null instead of Optional/empty collections\n9. **Mutable Collections**: Exposing internal collections\n10. **Unused Code**: Code that is never called"
}
```
```

---

## Assistant

_Thinking:_

Good, the edit was successful. Now let me enhance the design-patterns.md file to include the Input/Output Port pattern and Command/Query Separation pattern that were detailed in the analysis files.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/design-patterns.md",
  "newString": "## Design Pattern Selection Guide\n\n| Problem | Recommended Patterns |\n|---------|---------------------|\n| Creating objects with complex construction | Builder, Factory Method, Abstract Factory |\n| Ensuring single instance | Singleton |\n| Adapting incompatible interfaces | Adapter |\n| Decoupling abstraction from implementation | Bridge |\n| Treating individual and composite objects uniformly | Composite |\n| Adding responsibilities dynamically | Decorator |\n| Providing simplified interface to complex system | Facade |\n| Sharing common state efficiently | Flyweight |\n| Controlling access to objects | Proxy |\n| Handling requests through a chain | Chain of Responsibility |\n| Encapsulating requests as objects | Command |\n| Traversing collections | Iterator |\n| Centralizing communication between objects | Mediator |\n| Notifying objects about state changes | Observer |\n| Changing behavior with state | State |\n| Selecting algorithm at runtime | Strategy |\n| Defining algorithm skeleton | Template Method |\n\n---\n\n## Clean Architecture Patterns\n\n### Input/Output Port Pattern\n\nThe Input/Output Port pattern is central to Clean Architecture, defining the interface between use cases and the outside world:\n\n```typescript\n// Input Port: Defines what the use case accepts\ninterface CreateProductUseCase extends Command<\n    CreateProductUseCase.CreateProductInput, \n    CqrsOutput<ProductDto>\n> {\n    class CreateProductInput implements Input {\n        public productId: string;\n        public name: string;\n        public userId: string;\n    }\n}\n\n// Output Port: Maps domain objects to presentation\ninterface ProductPresenter {\n    present(product: Product): ProductDto;\n}\n\n// Service Implementation: Orchestrates the use case\nclass CreateProductService implements CreateProductUseCase {\n    constructor(\n        private repository: Repository<Product, ProductId>,\n        private messageBus: MessageBus\n    ) {}\n\n    async execute(input: CreateProductInput): Promise<CqrsOutput<ProductDto>> {\n        const product = new Product(\n            ProductId.valueOf(input.productId),\n            input.name,\n            UserId.valueOf(input.userId)\n        );\n\n        repository.save(product);\n        messageBus.publish(product.getUncommittedEvents());\n\n        return CqrsOutput.of(ProductMapper.toDto(product));\n    }\n}\n```\n\n### Command/Query Separation (CQS)\n\nSeparate operations that change state from those that read state:\n\n```typescript\n// Command: Modifies state, returns CqrsOutput\ninterface CreateProductUseCase extends Command<Input, CqrsOutput> {}\n\n// Query: Read-only, returns custom Output\ninterface GetProductUseCase extends Query<Input, GetProductOutput> {\n    class GetProductOutput implements Output {\n        public exitCode: ExitCode;\n        public message: string;\n        public product: ProductDto;\n    }\n}\n```\n\n### Repository Pattern\n\nAbstracts data access behind a generic interface:\n\n```typescript\n// ✅ CORRECT: Generic Repository Usage\n// NO custom Repository interface needed\nRepository<Product, ProductId> repository;\n\n// Standard methods only\nrepository.findById(ProductId id);  // Optional<Product>\nrepository.save(Product aggregate); // void\nrepository.delete(Product aggregate); // void\n```\n\n### Bridge Pattern\n\nDecouples abstraction from implementation for multiple repository types:\n\n```typescript\n// Abstraction\ninterface ToDoListRepository {\n    findById(id: ToDoListId): Promise<ToDoList | null>;\n    save(aggregate: ToDoList): Promise<void>;\n}\n\n// Implementation 1: In-memory\nclass ToDoListInMemoryRepository implements ToDoListRepository {\n    private store: Map<string, ToDoList> = new Map();\n    // ... implementations\n}\n\n// Implementation 2: JPA/Spring Data\nclass ToDoListCrudRepository implements ToDoListRepository {\n    private crudRepository: SpringDataRepository;\n    // ... implementations\n}\n\n// Client uses abstraction, not concrete implementation\nclass UseCase {\n    constructor(private repository: ToDoListRepository) {}\n}\n```\n\n---\n\n## References and Further Reading",
  "oldString": "## Design Pattern Selection Guide\n\n| Problem | Recommended Patterns |\n|---------|---------------------|\n| Creating objects with complex construction | Builder, Factory Method, Abstract Factory |\n| Ensuring single instance | Singleton |\n| Adapting incompatible interfaces | Adapter |\n| Decoupling abstraction from implementation | Bridge |\n| Treating individual and composite objects uniformly | Composite |\n| Adding responsibilities dynamically | Decorator |\n| Providing simplified interface to complex system | Facade |\n| Sharing common state efficiently | Flyweight |\n| Controlling access to objects | Proxy |\n| Handling requests through a chain | Chain of Responsibility |\n| Encapsulating requests as objects | Command |\n| Traversing collections | Iterator |\n| Centralizing communication between objects | Mediator |\n| Notifying objects about state changes | Observer |\n| Changing behavior with state | State |\n| Selecting algorithm at runtime | Strategy |\n| Defining algorithm skeleton | Template Method |\n\n---\n\n## References and Further Reading"
}
```
```

---

## Assistant

_Thinking:_

Good, the edit was successful. Now let me update the index.md to add references to the enhanced documentation sections.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/index.md",
  "newString": "# Software Engineering Concepts and Methodologies\n\nA comprehensive documentation of modern software engineering concepts, methodologies, and practices relevant to AI-assisted development.\n\n## Documentation Index\n\n### Requirements & Analysis\n- [Requirements and Specification](requirements-and-specification.md) - Understanding software requirements and specification documents\n- [Specification Driven Development](specification-driven-development.md) - Development approach centered around formal specifications\n- [Problem Frames Approach](problem-frames-approach.md) - Michael Jackson's framework for requirements analysis\n\n### Domain-Driven Design\n- [Domain-Driven Design](domain-driven-design-megred.md) - Strategic and tactical DDD patterns\n- [Domain Event Storming](domain-event-storming.md) - Collaborative event modeling technique\n- [Domain Event Storytelling](domain-event-storytelling.md) - Narrative approach to domain events\n\n### Architecture & Code Quality\n- [Layered Architecture](layered-architecture.md) - Ports and Adapters / Clean Architecture principles\n- [Clean Code](clean-code.md) - Writing maintainable and expressive code (enhanced with AI coding anti-patterns)\n- [Vertical Slice Architecture](vertical-slice-architecture-megred.md) - Feature-based project organization\n\n### Specification & Testing\n- [Specification by Example](specification-by-example.md) - Requirements through concrete examples\n- [Design by Contract](design-by-contract.md) - Formal verification through contracts\n- [Executable Specifications](executable-specifications.md) - Living documentation as tests\n- [Living Documentation](living-documentation.md) - Self-updating project documentation\n\n### Testing Methodologies\n- [Test-Driven Development](test-driven-development.md) - TDD principles and practices\n- [Behavior-Driven Development](behavior-driven-development.md) - BDD with Gherkin and Cucumber\n\n### Advanced Patterns\n- [Event Sourcing](event-sourcing.md) - Pattern for storing state changes\n- [CQRS](cqrs.md) - Command Query Responsibility Segregation\n- [Design Patterns](design-patterns.md) - Gang of Four patterns catalog (enhanced with Clean Architecture patterns)\n- [Pattern Language](pattern-language.md) - Christopher Alexander's pattern language concept\n\n### Collaborative Planning\n- [Impact Mapping](impact-mapping.md) - Gojko Adzic's strategic planning technique\n- [Example Mapping](example-mapping.md) - BDD example discovery technique\n\n### Coding Standards\n- [Coding Conventions](coding-conventions.md) - Universal naming, organization, and style guidelines (enhanced with AI-specific conventions)\n\n### AI-Assisted Development\n- [AGENTS.md and CLI AI Agent Tools](agents-md-cli-ai-agent-tools.md) - The AGENTS.md standard and supporting tools\n- [Prompt Engineering for AI Agents](prompt-engineering-ai-agents.md) - Techniques for effective agent instructions\n- [AI Agent Limitations](ai-agent-limitations.md) - Why agents skip instructions and fundamental constraints\n- [On-Demand Rule Loading](on-demand-rule-loading.md) - Techniques to load rules only when needed",
  "oldString": "# Software Engineering Concepts and Methodologies\n\nA comprehensive documentation of modern software engineering concepts, methodologies, and practices relevant to AI-assisted development.\n\n## Documentation Index\n\n### Requirements & Analysis\n- [Requirements and Specification](requirements-and-specification.md) - Understanding software requirements and specification documents\n- [Specification Driven Development](specification-driven-development.md) - Development approach centered around formal specifications\n- [Problem Frames Approach](problem-frames-approach.md) - Michael Jackson's framework for requirements analysis\n\n### Domain-Driven Design\n- [Domain-Driven Design](domain-driven-design-megred.md) - Strategic and tactical DDD patterns\n- [Domain Event Storming](domain-event-storming.md) - Collaborative event modeling technique\n- [Domain Event Storytelling](domain-event-storytelling.md) - Narrative approach to domain events\n\n### Architecture & Code Quality\n- [Layered Architecture](layered-architecture.md) - Ports and Adapters / Clean Architecture principles\n- [Clean Code](clean-code.md) - Writing maintainable and expressive code\n- [Vertical Slice Architecture](vertical-slice-architecture-megred.md) - Feature-based project organization\n\n### Specification & Testing\n- [Specification by Example](specification-by-example.md) - Requirements through concrete examples\n- [Design by Contract](design-by-contract.md) - Formal verification through contracts\n- [Executable Specifications](executable-specifications.md) - Living documentation as tests\n- [Living Documentation](living-documentation.md) - Self-updating project documentation\n\n### Testing Methodologies\n- [Test-Driven Development](test-driven-development.md) - TDD principles and practices\n- [Behavior-Driven Development](behavior-driven-development.md) - BDD with Gherkin and Cucumber\n\n### Advanced Patterns\n- [Event Sourcing](event-sourcing.md) - Pattern for storing state changes\n- [CQRS](cqrs.md) - Command Query Responsibility Segregation\n- [Design Patterns](design-patterns.md) - Gang of Four patterns catalog\n- [Pattern Language](pattern-language.md) - Christopher Alexander's pattern language concept\n\n### Collaborative Planning\n- [Impact Mapping](impact-mapping.md) - Gojko Adzic's strategic planning technique\n- [Example Mapping](example-mapping.md) - BDD example discovery technique\n\n### Coding Standards\n- [Coding Conventions](coding-conventions.md) - Universal naming, organization, and style guidelines\n\n### AI-Assisted Development\n- [AGENTS.md and CLI AI Agent Tools](agents-md-cli-ai-agent-tools.md) - The AGENTS.md standard and supporting tools\n- [Prompt Engineering for AI Agents](prompt-engineering-ai-agents.md) - Techniques for effective agent instructions\n- [AI Agent Limitations](ai-agent-limitations.md) - Why agents skip instructions and fundamental constraints\n- [On-Demand Rule Loading](on-demand-rule-loading.md) - Techniques to load rules only when needed"
}
```
```

---

## Assistant

_Thinking:_

Good, the index has been updated. Now let me create a new document that captures the key insights from the refactor-to-ca-analysis.md, specifically the 14-step refactoring journey. This will be a valuable addition to the documentation.

```
Tool: write

**Input:**
```json
{
  "content": "# Clean Architecture Refactoring Journey\n\n## Overview\n\nThis document captures a proven 14-step refactoring journey from a monolithic application to Clean Architecture with Domain-Driven Design. Each step builds upon the previous, maintaining working code throughout the transformation.\n\n**Project**: Task List Application (ToDo List)  \n**Language**: Java 21  \n**Framework**: Spring Boot 2.7.18  \n**Testing**: JUnit 5\n\n---\n\n## Architecture Evolution Summary\n\n### Final Architecture Structure\n\n```\ntw.teddysoft.tasks/\n├── entity/                          # Domain Layer\n│   ├── ToDoList.java               # Aggregate Root\n│   ├── Project.java                # Entity\n│   ├── Task.java                   # Entity\n│   ├── ProjectName.java            # Value Object\n│   ├── TaskId.java                 # Value Object\n│   ├── ToDoListId.java             # Value Object\n│   ├── ReadOnlyProject.java        # Interface\n│   └── ReadOnlyTask.java           # Interface\n│\n├── usecase/                        # Application Layer\n│   ├── AddProjectUseCase/          # Command\n│   ├── AddTaskUseCase/             # Command\n│   ├── SetDoneUseCase/             # Command\n│   ├── DeleteTaskUseCase/          # Command\n│   ├── ShowUseCase/                # Query\n│   ├── HelpUseCase/                # Query\n│   ├── ErrorUseCase/               # Query\n│   ├── DeadlineUseCase/            # Query\n│   ├── TodayUseCase/               # Query\n│   ├── ViewTaskUseCase/            # Query\n│   ├── port/                       # Output Port (Mappers)\n│   └── TestUtil.java\n│\n├── adapter/\n│   ├── in/\n│   │   ├── controller/\n│   │   │   ├── console/           # Console Controllers\n│   │   │   │   ├── ConsoleControllerExecutor.java\n│   │   │   │   ├── ToDoListCommands.java\n│   │   │   │   ├── Request.java\n│   │   │   │   ├── Response.java\n│   │   │   │   ├── AddProjectConsoleController.java\n│   │   │   │   ├── AddTaskConsoleController.java\n│   │   │   │   └── ...\n│   │   │   └── web/               # Web Controllers (REST)\n│   │   │       ├── AddProjectController.java\n│   │   │       ├── AddTaskController.java\n│   │   │       ├── SetDoneController.java\n│   │   │       ├── ShowController.java\n│   │   │       └── HelpController.java\n│   └── out/\n│       ├── presenter/              # Presenters\n│       │   ├── ShowConsolePresenter.java\n│       │   ├── HelpConsolePresenter.java\n│       │   ├── ViewTaskConsolePresenter.java\n│       │   └── HelpWebPresenter.java\n│       └── repository/             # Repositories\n│           ├── ToDoListCrudRepository.java\n│           ├── ToDoListInMemoryRepository.java\n│           ├── ToDoListCrudRepositoryPeer.java\n│           └── ToDoListInMemoryRepositoryPeer.java\n│\n└── io/\n    ├── standard/\n    │   └── ToDoListApp.java        # Console Entry Point\n    └── springboot/\n        ├── ToDoListSpringBootApp.java\n        └── config/\n            ├── UseCaseInjection.java\n            ├── RepositoryInjection.java\n            └── ToDoListDataSourceConfiguration.java\n```\n\n---\n\n## Step-by-Step Refactoring Guide\n\n### Step 1: Initial Foundation Setup\n\n**Focus**: Test structure and error handling setup\n\n**Changes**:\n- Configure JDK 21, JUnit 5\n- Establish test structure for help and error commands\n- Define error handling messages for add project, add task, check commands\n- Set up dependencies (ezddd, ezcqrs, uContract, ezspec)\n\n**Pattern**: Greenfield test setup\n\n---\n\n### Step 2: Establish Clean Architecture Layers\n\n**Focus**: Layer creation and DDD foundation\n\n**Changes**:\n1. Create four-layer package structure:\n   - `entity/` - Domain model\n   - `usecase/` - Business logic\n   - `adapter/` - Controllers, Presenters, Repositories\n   - `io/` - Entry points\n\n2. Move Task.java to entity package\n\n3. Apply DDD Tactical Design:\n   - ToDoList → Aggregate Root\n   - Project, Task → Entities\n   - ProjectName, TaskId, ToDoListId → Value Objects\n\n4. Remove Primitive Obsession:\n   - Replace `Map<String, List<Task>>` with `Tasks` class\n   - Replace `Map<ProjectName, List<Task>>` with `Project` class\n   - Create `ProjectName` record\n\n5. Establish Ubiquitous Language:\n   - Rename `Tasks` to `ToDoList`\n   - Rename methods to meaningful domain terms\n\n**Patterns Applied**:\n- Layered Architecture\n- Value Object\n- Entity\n- Aggregate Root\n\n**Key Insight**: Aggregate boundaries and invariant enforcement are established early\n\n---\n\n### Step 3: Extract Classes from Monolith\n\n**Focus**: Find more classes in entity package by extracting command/query logic\n\n**Changes** (using \"Extract Class from Private Method\" pattern):\n1. Extract TaskList::show → Show class\n2. Extract TaskList::add → Add class\n3. Extract TaskList::setDone → SetDone class\n4. Extract TaskList::help → Help class\n5. Extract TaskList::error → Error class\n6. Extract TaskList::execute → Execute class\n\n7. Clean entity package:\n   - Move classes with external/IO references to usecase package\n\n**Patterns Applied**:\n- Extract Class\n- Single Responsibility Principle\n- Method Object\n\n**Key Insight**: Each extracted class represents a potential Use Case\n\n---\n\n### Step 4: Form Commands and Queries\n\n**Focus**: Implement Use Case pattern with Input/Output\n\n**Command Use Cases**:\n1. AddProjectUseCase:\n   - AddProjectInput\n   - AddProjectService\n   - ToDoListRepository\n   - ToDoListInMemoryRepository\n\n2. AddTaskUseCase:\n   - AddTaskInput\n   - AddTaskService\n   - Inline Add to Execute\n\n3. SetDoneUseCase:\n   - SetDoneInput\n   - SetDoneService\n\n**Query Use Cases**:\n1. ShowUseCase:\n   - ShowInput, ShowOutput\n   - ShowService, ShowPresenter\n   - ShowConsolePresenter\n   - ToDoListDto, ProjectDto, TaskDto\n   - ToDoListMapper, ProjectMapper, TaskMapper\n\n2. HelpUseCase:\n   - HelpInput, HelpOutput\n   - HelpService, HelpDto\n   - HelpPresenter, HelpConsolePresenter\n\n3. ErrorUseCase:\n   - ErrorInput, ErrorService\n\n**Patterns Applied**:\n- Command Query Separation\n- Input/Output Port Pattern\n- Service Layer\n- Data Mapper\n\n---\n\n### Step 5: Form Controllers\n\n**Focus**: Extract and inject use case dependencies\n\n**Changes**:\n1. Move Execute to adapter.controller package, rename to ToDoListConsoleController\n2. Inject ShowUseCase and ShowPresenter\n3. Inject AddProjectUseCase\n4. Inject AddTaskUseCase\n5. Inject SetDoneUseCase\n6. Inject HelpUseCase\n7. Inject ErrorUseCase\n8. Remove unused ToDoList from controller\n\n**Patterns Applied**:\n- Dependency Injection (Constructor Injection)\n- Controller Pattern\n- Single Responsibility\n\n**Key Insight**: Controllers become thin, delegating all business logic to use cases\n\n---\n\n### Step 6: Form Main Component\n\n**Focus**: Identify entry point and apply DI framework\n\n**Changes**:\n1. Move TaskList to io package, rename to ToDoListApp (Main Component)\n2. Use ToDoListApp::main and ApplicationTest as DI frameworks manually\n3. Add SpringBoot dependencies to pom.xml\n4. Create ToDoListSpringBootApp\n5. Create SpringBootApplicationTest\n\n**Patterns Applied**:\n- Application Entry Point\n- Dependency Injection Container\n- Main Component Pattern\n\n**Key Insight**: Main component orchestrates dependency wiring\n\n---\n\n### Step 7: Create Web Controllers\n\n**Focus**: Support REST API alongside console\n\n**Changes**:\n1. Create AddProjectController in adapter.controller.web\n2. Move ToDoListController to adapter.controller.console\n3. Create AddTaskController\n4. Create SetDoneController\n5. Create HelpController\n6. Create ShowController\n\n**Patterns Applied**:\n- Adapter Pattern\n- Strategy Pattern (multiple controller implementations)\n- Interface Segregation\n\n**Key Insight**: Same use cases can be accessed via different adapters (console/web)\n\n---\n\n### Step 8: Add Persistent Objects\n\n**Focus**: Introduce persistence layer\n\n**Changes**:\n1. Define ToDoListPoRepository interface\n2. Implement ToDoListPo, ProjectPo, TaskPo\n3. Write Mappers (ToDoListMapperTest, ProjectMapperTest, TaskMapperTest)\n4. Apply Bridge Pattern:\n   - Move ToDoListPoRepository to adapter.repository\n   - Rename to ToDoListRepositoryPeer\n   - Implement ToDoListInMemoryRepositoryPeer\n5. Revise DI to inject repository peer\n\n**Patterns Applied**:\n- Bridge Pattern\n- Data Mapper Pattern\n- Repository Pattern\n- Strategy Pattern (in-memory vs persistence)\n\n**Key Insight**: Bridge pattern decouples abstraction (repository) from implementation (peer)\n\n---\n\n### Step 9: Use Relational Database via JPA\n\n**Focus**: Switch from in-memory to persistent storage\n\n**Changes**:\n1. Add JPA annotations to ToDoListPo, ProjectPo, TaskPo\n2. Create ToDoListCrudRepositoryPeer (Spring Data JPA)\n3. Create ToDoListCrudRepository\n4. Create ToDoListDataSourceConfiguration\n5. Revise RepositoryInjection for SpringBoot\n\n**Patterns Applied**:\n- Spring Data JPA\n- Repository Pattern\n- Dependency Injection\n\n**Key Insight**: Same repository interface, different implementations (in-memory/JPA)\n\n---\n\n### Step 10-11: Feature Expansion\n\n**Focus**: Add new features using established patterns\n\n**Step 10 - Add New Feature Deadlines**:\n1. TDD DeadlineUseCase with DeadlineUseCaseTest\n2. Implement DeadlineUseCase, DeadlineInput, DeadlineService\n3. Revise Task to support deadline\n4. Create TaskPo deadline field\n5. Revise console controller for deadline command\n\n**Step 11 - Add New Feature Customisable IDs**:\n1. TDD AddTaskUseCase with custom ID support\n2. Revise AddTaskInput, AddTaskService\n3. Implement TaskId value object\n4. Revise console controller for task2 command\n\n**Patterns Applied**:\n- Test-Driven Development\n- Value Object\n- Open/Closed Principle (extend without modifying existing code)\n\n**Key Insight**: New features integrate seamlessly with existing architecture\n\n---\n\n### Step 12-13: Console Controller Refinement\n\n**Focus**: Improve console interface\n\n**Step 12 - Add New Feature Deletion**:\n1. TDD DeleteTaskUseCase\n2. Implement DeleteTaskUseCase, DeleteTaskInput, DeleteTaskService\n3. Revise console controller for delete command\n4. Refactor adapter package: add in/out packages\n5. Move controller to in, presenter and repository to out\n\n**Step 13 - Add New Feature View by Deadline**:\n1. TDD ViewTaskUseCase\n2. Implement ViewTaskUseCase, ViewTaskInput, ViewTaskOutput\n3. Create ViewTaskDto, ViewTaskService\n4. Revise console controller and tests\n\n**Patterns Applied**:\n- Input/Output Port Naming Convention (in/out)\n- RESTful resource naming\n- Use Case reusability\n\n---\n\n### Step 14: Extract Console Controllers\n\n**Focus**: Decouple console controllers using executor pattern\n\n**Changes**:\n1. Define Request, Response, ConsoleController interfaces\n2. Create ToDoListCommands enum (commands mapping)\n3. Implement ConsoleControllerExecutor\n4. Extract individual controllers:\n   - AddProjectConsoleController\n   - AddTaskConsoleController\n   - CheckConsoleController\n   - SetDoneConsoleController\n   - DeleteConsoleController\n   - ShowConsoleController\n   - HelpConsoleController\n   - DeadlineConsoleController\n   - TodayConsoleController\n   - ErrorConsoleController\n\n**Patterns Applied**:\n- Command Pattern\n- Strategy Pattern\n- Extensible Enum\n- Template Method\n\n**Key Insight**: Each command becomes a standalone controller, executor dispatches based on command type\n\n---\n\n## Design Pattern Usage Summary\n\n| Pattern | Steps Applied | Purpose |\n|---------|---------------|---------|\n| Layered Architecture | 2 | Clear separation of concerns |\n| Aggregate Root | 2 | Domain model encapsulation |\n| Value Object | 2 | Type safety, domain modeling |\n| Entity | 2 | Domain identity |\n| Extract Class | 3 | Single responsibility |\n| Command/Query Separation | 4 | Clear use case definition |\n| Input/Output Port | 4 | Use case interface |\n| Service Layer | 4 | Business logic orchestration |\n| Data Mapper | 4, 8 | Object-relational mapping |\n| Dependency Injection | 5, 6 | Loose coupling |\n| Adapter Pattern | 7 | Interface adaptation |\n| Bridge Pattern | 8 | Decouple abstraction from implementation |\n| Repository Pattern | 8, 9 | Data access abstraction |\n| Strategy Pattern | 8, 14 | Interchangeable algorithms |\n| Command Pattern | 14 | Encapsulate request as object |\n| Template Method | 7, 14 | Algorithm skeleton |\n\n---\n\n## Dependency Evolution\n\n### Step 1-4\n```\nToDoList (monolith) → direct dependencies\n```\n\n### Step 5-6\n```\nToDoListConsoleController → UseCase → Service → Repository → ToDoList (Aggregate)\n```\n\n### Step 7-9\n```\nController (in) → UseCase (usecase) → Repository (out) → PO (out) → DB\nPresenter (out)\n```\n\n### Step 14\n```\nConsoleControllerExecutor → Request/Response → ConsoleController (in) → UseCase\n                                                              ↓\n                                                  Presenter/Repository (out)\n```\n\n---\n\n## Testing Strategy Evolution\n\n### Step 1\n- Basic unit tests for commands\n- Error message validation\n\n### Step 4\n- Use Case Tests (TDD approach)\n- Input/Output boundary testing\n- Service layer testing\n\n### Step 8\n- Mapper Tests (domain ↔ PO)\n- Repository Tests (with TestContainers or in-memory)\n\n### Step 12-13\n- Feature tests following TDD\n- Integration tests for console/web\n\n---\n\n## Key Architectural Principles Demonstrated\n\n1. **Dependency Inversion**: High-level modules (use cases) don't depend on low-level modules (frameworks)\n\n2. **Single Responsibility**: Each class has one reason to change\n\n3. **Open/Closed**: Open for extension, closed for modification\n\n4. **Interface Segregation**: Small, focused interfaces\n\n5. **Aggregate Boundary**: Aggregate root controls access to entities\n\n6. **Ubiquitous Language**: Code expresses domain concepts clearly\n\n7. **Testability**: Every layer can be tested in isolation\n\n---\n\n## Lessons Learned\n\n1. **Start with tests**: Each refactoring step is verified by existing tests\n\n2. **Extract by responsibility**: When code does multiple things, extract until single responsibility\n\n3. **Bridge for abstractions**: Use bridge pattern when you need multiple implementations\n\n4. **Controllers are adapters**: They're entry points, not business logic\n\n5. **Use cases are stable**: Once use cases are defined, they're rarely changed\n\n6. **Value objects add safety**: Replace primitives with domain-specific types\n\n7. **Inject dependencies**: Makes testing and future changes easier\n\n---\n\n## References\n\n- Original repository: `refactor-to-ca` (14 git branches: main, step1-step14)\n- Related documentation: Clean Architecture, Domain-Driven Design, Test-Driven Development\n",
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/clean-architecture-refactoring-journey.md"
}
```
```

---

## Assistant

_Thinking:_

Good, I've created a new document for the Clean Architecture refactoring journey. Now let me update the index.md to include this new document.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/index.md",
  "newString": "### Architecture & Code Quality\n- [Layered Architecture](layered-architecture.md) - Ports and Adapters / Clean Architecture principles\n- [Clean Architecture Refactoring Journey](clean-architecture-refactoring-journey.md) - 14-step proven refactoring journey from monolith to Clean Architecture\n- [Clean Code](clean-code.md) - Writing maintainable and expressive code (enhanced with AI coding anti-patterns)\n- [Vertical Slice Architecture](vertical-slice-architecture-megred.md) - Feature-based project organization",
  "oldString": "### Architecture & Code Quality\n- [Layered Architecture](layered-architecture.md) - Ports and Adapters / Clean Architecture principles\n- [Clean Code](clean-code.md) - Writing maintainable and expressive code (enhanced with AI coding anti-patterns)\n- [Vertical Slice Architecture](vertical-slice-architecture-megred.md) - Feature-based project organization"
}
```
```

---

## Assistant

_Thinking:_

Good. Now let me also create a document that captures the AI coding pattern language and specification-driven development approach from the ai-coding-exercise-analysis.md file.

```
Tool: write

**Input:**
```json
{
  "content": "# AI Coding Pattern Language\n\n## Overview\n\nA pattern language for AI-assisted software development that enables AI agents to understand, generate, and validate code patterns consistently. This document provides a formal specification format and catalog of patterns used in AI-assisted development.\n\n---\n\n## Pattern Definition Language\n\n### Pattern Specification Format\n\nAll patterns are defined using a JSON-based specification format:\n\n```json\n{\n  \"useCase\": \"CreateProduct\",\n  \"behavior\": \"Create a new product\",\n  \"input\": [\n    { \"name\": \"productId\", \"type\": \"String\", \"note\": \"唯一識別碼\" },\n    { \"name\": \"name\", \"type\": \"String\", \"note\": \"顯示名稱\" }\n  ],\n  \"aggregate\": \"Product\",\n  \"domainEvent\": \"ProductEvents.ProductCreated\",\n  \"repository\": \"ProductRepository\"\n}\n```\n\n### Pattern Catalog\n\n| Pattern Category | Patterns Defined | Documentation Location |\n|------------------|------------------|------------------------|\n| Use Case | Command, Query, Reactor | `.ai/prompts/*-sub-agent-prompt.md` |\n| Aggregate | Event Sourcing, State-based | `.ai/tech-stacks/*/coding-standards/aggregate-standards.md` |\n| Repository | InMemory, Outbox, EventStore | `.ai/tech-stacks/*/coding-standards/repository-standards.md` |\n| Controller | REST API, Adapter | `.ai/prompts/controller-code-generation-prompt.md` |\n| Testing | ezSpec, Unit, Integration | `.ai/prompts/test-generation-prompt.md` |\n\n---\n\n## Pattern Anti-Patterns\n\nCommon AI coding mistakes are documented as anti-patterns with corrections:\n\n### Anti-Pattern 1: Nested Input/Output Classes\n\n```typescript\n// ❌ AI creates: Separate Input.java, Output.java files\n// ✅ CORRECT: Inner classes within UseCase interface\ninterface CreateProductUseCase {\n    class CreateProductInput implements Input {\n        public productId: string;\n        public name: string;\n        public userId: string;\n    }\n    \n    class CreateProductOutput implements Output {\n        public productId: string;\n        public status: string;\n    }\n}\n```\n\n### Anti-Pattern 2: instanceof Chain\n\n```typescript\n// ❌ AI writes: Multiple if-else checks with instanceof\nif (event instanceof X) {\n    // handle X\n} else if (event instanceof Y) {\n    // handle Y\n}\n\n// ✅ CORRECT: Switch expression with pattern matching\nswitch (event) {\n    case ProductEvents.ProductCreated e -> {\n        this.id = e.productId();\n        this.name = e.name();\n    }\n    case ProductEvents.ProductRenamed e -> {\n        this.name = e.newName();\n    }\n}\n```\n\n### Anti-Pattern 3: Automatic Repository Implementation\n\n```typescript\n// ❌ AI generates: Custom repository interface\ninterface ProductRepository extends Repository<Product, ProductId> {\n    List<Product> findBySprintId(SprintId id);  // WRONG: Custom query\n}\n\n// ✅ CORRECT: Use framework GenericInMemoryRepository\n// No custom Repository interface needed\nRepository<Product, ProductId> repository;\n```\n\n### Anti-Pattern 4: Contract Validation in Value Objects\n\n```typescript\n// ❌ AI uses: Contract.requireNotNull() in Value Object\nrecord ProductId(String value) implements ValueObject {\n    public ProductId {\n        requireNotNull(\"value\", value);  // WRONG\n    }\n}\n\n// ✅ CORRECT: Use Objects.requireNonNull()\nrecord ProductId(String value) implements ValueObject {\n    public ProductId {\n        Objects.requireNonNull(value, \"Product ID cannot be null\");\n    }\n}\n```\n\n---\n\n## Specification-Driven Development\n\n### Specification Structure\n\nSpecifications are organized by aggregate in a structured directory:\n\n```\n.dev/specs/\n├── [aggregate]/\n│   ├── entity/\n│   │   └── [aggregate]-spec.md          # Domain model specification\n│   └── usecase/\n│       ├── [action]-[aggregate].json    # Use case specification\n│       └── projection/                  # Query specifications\n```\n\n### Specification Schema\n\n```json\n{\n  \"$schema\": \"./.ai/schemas/use-case-spec-schema.json\",\n  \"useCase\": \"CreateProduct\",\n  \"behavior\": \"Create a new product\",\n  \"input\": [...],\n  \"aggregate\": \"Product\",\n  \"aggregateId\": \"ProductId\",\n  \"method\": \"Product constructor\",\n  \"domainEvent\": \"ProductEvents.ProductCreated\",\n  \"output\": \"CqrsOutput with productId\",\n  \"domainModelNotes\": [\n    \"Product 是 Aggregate Root\",\n    \"Product 使用建構函數建立新實例\"\n  ],\n  \"constructorPreconditions\": [...],\n  \"constructorPostconditions\": [...],\n  \"aggregates\": [...],\n  \"domainEvents\": [...],\n  \"entities\": [...],\n  \"valueObjects\": [...],\n  \"enums\": [...]\n}\n```\n\n### Naming Conventions\n\n| Element | Convention | Example |\n|---------|------------|---------|\n| Use Case | `[action]-[aggregate].json` | `create-product.json` |\n| Entity Spec | `[aggregate]-spec.md` | `product-spec.md` |\n| Controller Spec | `[action]-[aggregate]-controller.json` | `get-product-controller.json` |\n| Projection | `[dto-name]-projection.json` | `product-dto-projection.json` |\n\n---\n\n## Sub-Agent System Architecture\n\n### Agent Specialization Model\n\n| Agent Type | Responsibility | Prompt File |\n|------------|----------------|-------------|\n| Command | Generate Command Use Cases | `command-sub-agent-prompt.md` |\n| Query | Generate Query Use Cases | `query-sub-agent-prompt.md` |\n| Aggregate | Generate DDD Aggregates | `aggregate-sub-agent-prompt.md` |\n| Reactor | Generate Event Handlers | `reactor-sub-agent-prompt.md` |\n| Controller | Generate REST Controllers | `controller-code-generation-prompt.md` |\n| Outbox | Generate Outbox Pattern | `outbox-sub-agent-prompt.md` |\n| Test | Generate Tests | `test-generation-prompt.md` |\n| Code Review | Review Generated Code | `code-review-prompt.md` |\n\n### Context Management\n\nSub-agents receive complete context including:\n\n```python\ncontext = {\n    \"spec\": read_file(spec_path),\n    \"coding_standards\": read_file(\".ai/tech-stacks/*/coding-standards/*.md\"),\n    \"examples\": read_file(\".ai/tech-stacks/*/examples/*.java\"),\n    \"common_rules\": read_file(\".ai/prompts/shared/common-rules.md\"),\n    \"framework_api\": read_file(\".ai/guides/FRAMEWORK-API-INTEGRATION-GUIDE.md\"),\n    \"project_config\": read_file(\".dev/project-config.json\")\n}\n```\n\n---\n\n## Code Review Framework\n\n### Mandatory Code Review Process\n\n```mermaid\nflowchart TD\n    A[Code Review Request] --> B[Read CODE-REVIEW-INDEX.md]\n    B --> C[Identify File Type]\n    C --> D[Read Corresponding Checklist]\n    D --> E[Execute Review Checklist]\n    E --> F[Generate Review Table]\n    F --> G[Calculate Score]\n    G --> H[Generate Report]\n```\n\n### Review Checklist Structure\n\n| Level | Focus Area | Checklist Items |\n|-------|------------|-----------------|\n| 1 | Architecture | Package location, Layer compliance, Clean Architecture |\n| 2 | Coding Standards | Input/Output patterns, Annotations, Repository usage |\n| 3 | Business Logic | Contract validation, Error handling, Domain rules |\n\n### Review Output Format\n\n```markdown\n## Code Review Report\n\n| Check Item | Result | Location | Issue Description |\n|------------|--------|----------|-------------------|\n| Package Location | ✅ | - | Correct |\n| Input as Inner Class | ❌ | Line 45 | Input should be UseCase inner class |\n| Repository Pattern | ✅ | - | Correct |\n\n### Summary\n- **Critical Issues**: 0\n- **Must Fix Issues**: 1\n- **Should Fix Issues**: 2\n- **Score**: 4/5 ⭐⭐⭐⭐\n```\n\n---\n\n## Testing Strategy\n\n### Testing Pyramid\n\n```\n        ┌─────────────┐\n        │   E2E Tests │    ← Minimum (Critical user flows only)\n        ├─────────────┤\n       │ Integration  │   ← Moderate (API, Database, Events)\n       │    Tests     │\n       ├─────────────┤\n      │   Unit Tests  │   ← Maximum (Domain logic, Use cases)\n      │  (ezSpec BDD) │\n      └─────────────┘\n```\n\n### ezSpec BDD Framework\n\n```java\n@EzFeature\npublic class CreateProductUseCaseTest {\n    static Feature feature = Feature.New(\"Create Product Use Case\");\n    \n    @EzScenario\n    public void should_create_product_with_valid_input() {\n        feature.newScenario(\"Successfully create a product with valid input\")\n            .Given(\"valid product creation input\", env -> {\n                var input = CreateProductInput.create();\n                input.productId = \"prod-001\";\n                input.name = \"Test Product\";\n                input.userId = \"user-001\";\n                env.put(\"input\", input);\n            })\n            .When(\"the use case is executed\", env -> {\n                var input = env.get(\"input\", CreateProductInput.class);\n                var output = useCase.execute(input);\n                env.put(\"output\", output);\n            })\n            .Then(\"the product should be created successfully\", env -> {\n                var output = env.get(\"output\", CqrsOutput.class);\n                assertThat(output.isSuccessful()).isTrue();\n                assertThat(publishedEvents).hasSize(1);\n                assertThat(publishedEvents.get(0))\n                    .isInstanceOf(ProductEvents.ProductCreated.class);\n            })\n            .Execute();\n    }\n}\n```\n\n---\n\n## Architecture Decision Records (ADRs)\n\n### ADR Format\n\n```markdown\n# ADR-005: AI Task Execution SOP\n\n## Status\nAccepted\n\n## Context\nAI agents need a standardized workflow for task execution\n\n## Decision\nImplement 5-step SOP:\n1. Create TodoWrite tracking\n2. Execute pipeline.steps\n3. Execute postChecks\n4. Update results\n5. Update status\n\n## Consequences\n- ✅ Consistent task execution\n- ✅ Traceable AI behavior\n- ✅ Automated reporting\n```\n\n### Common ADRs\n\n| ADR ID | Title | Impact |\n|--------|-------|--------|\n| ADR-001 | Use Case Package Structure | High |\n| ADR-005 | AI Task Execution SOP | High |\n| ADR-010 | No @Component for Services | High |\n| ADR-019 | OutboxMapper as Inner Class | High |\n| ADR-021 | Profile-Based Testing | High |\n| ADR-031 | Reactor Interface | High |\n| ADR-043 | Audit Fields in Metadata | Medium |\n\n---\n\n## Failure Case Analysis\n\n### Documented Failure Cases\n\n#### Case 1: Value Object Validation Error\n\n```typescript\n// ❌ AI Code:\nrecord ProductId(String value) implements ValueObject {\n    public ProductId {\n        requireNotNull(\"value\", value);  // WRONG: Uses Contract\n    }\n}\n\n// ✅ CORRECT:\nrecord ProductId(String value) implements ValueObject {\n    public ProductId {\n        Objects.requireNonNull(value, \"value cannot be null\");\n    }\n}\n```\n\n#### Case 2: Repository Pattern Violation\n\n```typescript\n// ❌ AI Code:\ninterface ProductRepository extends Repository<Product, ProductId> {\n    List<Product> findBySprintId(SprintId id);  // WRONG: Custom query\n}\n\n// ✅ CORRECT:\n// Use framework GenericInMemoryRepository\n// Complex queries via Projection\n```\n\n#### Case 3: Input Class Location\n\n```typescript\n// ❌ AI Code:\n// Separate Input.java file\npublic class CreateProductInput implements Input { ... }\n\n// ✅ CORRECT:\n// Inner class in UseCase interface\npublic interface CreateProductUseCase {\n    class CreateProductInput implements Input { ... }\n}\n```\n\n### Learning Integration\n\nAll failure cases are:\n1. Documented in `.dev/lessons/FAILURE-CASES.md`\n2. Integrated into Sub-agent prompts\n3. Added to Code Review checklists\n4. Used for automated detection\n\n---\n\n## References\n\n- AI Coding Exercise Repository: https://gitlab.com/TeddyChen/ai-coding-exercise.git\n- Related documentation: Clean Architecture, DDD, Test-Driven Development\n",
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/ai-coding-patterns.md"
}
```
```

---

## Assistant

_Thinking:_

Good. Now let me update the index.md to include this new document.

```
Tool: edit

**Input:**
```json
{
  "filePath": "/Users/erictam/Desktop/projects/ai-learning/ai-assisted-development/doc/index.md",
  "newString": "### AI-Assisted Development\n- [AI Coding Patterns](ai-coding-patterns.md) - Pattern language for AI-assisted development with anti-patterns and specification-driven development\n- [AGENTS.md and CLI AI Agent Tools](agents-md-cli-ai-agent-tools.md) - The AGENTS.md standard and supporting tools\n- [Prompt Engineering for AI Agents](prompt-engineering-ai-agents.md) - Techniques for effective agent instructions\n- [AI Agent Limitations](ai-agent-limitations.md) - Why agents skip instructions and fundamental constraints\n- [On-Demand Rule Loading](on-demand-rule-loading.md) - Techniques to load rules only when needed",
  "oldString": "### AI-Assisted Development\n- [AGENTS.md and CLI AI Agent Tools](agents-md-cli-ai-agent-tools.md) - The AGENTS.md standard and supporting tools\n- [Prompt Engineering for AI Agents](prompt-engineering-ai-agents.md) - Techniques for effective agent instructions\n- [AI Agent Limitations](ai-agent-limitations.md) - Why agents skip instructions and fundamental constraints\n- [On-Demand Rule Loading](on-demand-rule-loading.md) - Techniques to load rules only when needed"
}
```
```

---

## Assistant

_Thinking:_

Good. Now I have enhanced the documentation with knowledge from both analysis files. Let me summarize what I've done:

1. Enhanced `clean-code.md` with:
   - Common AI Coding Anti-Patterns section
   - AI Code Review Checklist

2. Enhanced `coding-conventions.md` with:
   - AI-specific anti-patterns and correct approaches
   - AI-specific naming conventions
   - Bounded Context Rules from DDD

3. Enhanced `design-patterns.md` with:
   - Input/Output Port Pattern
   - Command/Query Separation (CQS)
   - Repository Pattern
   - Bridge Pattern

4. Created `clean-architecture-refactoring-journey.md`:
   - Complete 14-step refactoring journey from monolith to Clean Architecture
   - Detailed step-by-step analysis
   - Design pattern usage summary
   - Dependency evolution
   - Testing strategy evolution

5. Created `ai-coding-patterns.md`:
   - Pattern Definition Language
   - Pattern Anti-Patterns
   - Specification-Driven Development
   - Sub-Agent System Architecture
   - Code Review Framework
   - Testing Strategy
   - ADR Format
   - Failure Case Analysis

6. Updated `index.md` to include references to all enhanced and new documents

The documentation is now enhanced with practical knowledge from real-world refactoring journeys and AI-assisted development patterns.

I've enhanced the documentation in `doc/` with knowledge from both analysis files. Here's a summary of the enhancements:

## Enhanced Documents

### 1. `clean-code.md`
- Added **Common AI Coding Anti-Patterns** section documenting frequent AI mistakes (nested Input/Output classes, instanceof chains, automatic repository implementations)
- Added **AI Code Review Checklist** for reviewing AI-generated code

### 2. `coding-conventions.md`
- Added **AI-Specific Anti-Patterns** table with correct approaches
- Added **AI-Specific Naming Conventions** for use cases, services, value objects
- Added **Bounded Context Rules** (BC-001 to BC-004)

### 3. `design-patterns.md`
- Added **Input/Output Port Pattern** - central to Clean Architecture
- Added **Command/Query Separation (CQS)** pattern
- Added **Repository Pattern** with generic interface approach
- Added **Bridge Pattern** for decoupling abstraction from implementation

### 4. Created `clean-architecture-refactoring-journey.md`
- Complete **14-step refactoring journey** from monolith to Clean Architecture
- Detailed step-by-step analysis with patterns applied at each step
- **Dependency evolution** showing how dependencies flow across layers
- **Testing strategy evolution** from basic tests to TDD
- **Design pattern usage summary** with step mappings
- **Key lessons learned** from the refactoring process

### 5. Created `ai-coding-patterns.md`
- **Pattern Definition Language** with JSON specification format
- **Pattern Anti-Patterns** with correct approaches
- **Specification-Driven Development** structure and schema
- **Sub-Agent System Architecture** with agent specialization model
- **Code Review Framework** with checklist structure
- **Testing Pyramid** with ezSpec BDD framework examples
- **ADR Format** with common ADRs
- **Failure Case Analysis** with documented mistakes

### 6. Updated `index.md`
- Added references to all enhanced and new documentation files

---

