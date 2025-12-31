# Design Patterns

## Core Concept

Design patterns are reusable solutions to commonly occurring software design problems. They represent best practices evolved over time by experienced software developers and provide a common vocabulary for discussing software architecture and design decisions.

The concept of design patterns was popularized by the 1994 book "Design Patterns: Elements of Reusable Object-Oriented Software" by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides, collectively known as the "Gang of Four" (GoF). This seminal work catalogued 23 design patterns that have since become fundamental tools in software engineering.

Design patterns are not specific algorithms or complete solutions but rather templates for how to solve a problem in a way that is proven to work, maintainable, and flexible. They capture the wisdom of experienced developers and help less experienced developers avoid common pitfalls and make better design decisions.

Understanding design patterns is essential for several reasons: they provide a shared vocabulary that improves communication between developers, they help in creating more maintainable and flexible code, and they represent battle-tested solutions to common problems that have been refined over decades of collective experience.

---

## Design Pattern Categories

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DESIGN PATTERN CATEGORIES                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    CREATIONAL PATTERNS                           │    │
│  │                                                                 │    │
│  │  Focus: Object creation mechanisms                              │    │
│  │                                                                 │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐ │    │
│  │  │Factory  │  │Abstract │  │Builder  │  │Prototype│  │Singleton│ │    │
│  │  │Method   │  │Factory  │  │         │  │         │  │        │ │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘ │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                   STRUCTURAL PATTERNS                            │    │
│  │                                                                 │    │
│  │  Focus: Composition of classes and objects                       │    │
│  │                                                                 │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐ │    │
│  │  │Adapter  │  │Bridge   │  │Composite│  │Decorator│  │Facade │ │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘ │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐                 │    │
│  │  │Flyweight│  │Proxy    │  │                 │                 │    │
│  │  └─────────┘  └─────────┘  └─────────────────┘                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                   BEHAVIORAL PATTERNS                            │    │
│  │                                                                 │    │
│  │  Focus: Communication between objects                            │    │
│  │                                                                 │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────┐ │    │
│  │  │Chain of │  │Command  │  │Interpreter││Iterator │  │Mediator│ │    │
│  │  │Responsib.│  │         │  │          │  │        │  │       │ │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └───────┘ │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────────┐   │    │
│  │  │Memento  │  │Observer │  │State    │  │Strategy         │   │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────────────┘   │    │
│  │  ┌─────────────────────┐                                         │    │
│  │  │Template Method      │                                         │    │
│  │  └─────────────────────┘                                         │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Creational Patterns

### 1. Factory Method

```typescript
// Factory Method: Create objects without specifying exact class
interface Product {
  operation(): string;
}

class ConcreteProductA implements Product {
  operation(): string {
    return 'Product A operation';
  }
}

class ConcreteProductB implements Product {
  operation(): string {
    return 'Product B operation';
  }
}

abstract class Creator {
  abstract factoryMethod(): Product;

  someOperation(): string {
    const product = this.factoryMethod();
    return `Creator: ${product.operation()}`;
  }
}

class ConcreteCreatorA extends Creator {
  factoryMethod(): Product {
    return new ConcreteProductA();
  }
}

class ConcreteCreatorB extends Creator {
  factoryMethod(): Product {
    return new ConcreteProductB();
  }
}

// Usage
const creatorA = new ConcreteCreatorA();
console.log(creatorA.someOperation());

const creatorB = new ConcreteCreatorB();
console.log(creatorB.someOperation());
```

### 2. Abstract Factory

```typescript
// Abstract Factory: Create families of related objects
interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

interface Button {
  render(): void;
  onClick(): void;
}

interface Checkbox {
  render(): void;
  toggle(): void;
}

class WinFactory implements GUIFactory {
  createButton(): Button {
    return new WinButton();
  }
  createCheckbox(): Checkbox {
    return new WinCheckbox();
  }
}

class MacFactory implements GUIFactory {
  createButton(): Button {
    return new MacButton();
  }
  createCheckbox(): Checkbox {
    return new MacCheckbox();
  }
}

class WinButton implements Button {
  render(): void {
    console.log('Rendering Windows button');
  }
  onClick(): void {
    console.log('Windows button clicked');
  }
}

class MacButton implements Button {
  render(): void {
    console.log('Rendering macOS button');
  }
  onClick(): void {
    console.log('macOS button clicked');
  }
}

class WinCheckbox implements Checkbox {
  render(): void {
    console.log('Rendering Windows checkbox');
  }
  toggle(): void {
    console.log('Windows checkbox toggled');
  }
}

class MacCheckbox implements Checkbox {
  render(): void {
    console.log('Rendering macOS checkbox');
  }
  toggle(): void {
    console.log('macOS checkbox toggled');
  }
}

// Usage
function createUI(factory: GUIFactory) {
  const button = factory.createButton();
  const checkbox = factory.createCheckbox();
  button.render();
  checkbox.render();
}

createUI(new WinFactory());
createUI(new MacFactory());
```

### 3. Builder

```typescript
// Builder: Construct complex objects step by step
class House {
  private foundation: string = '';
  private structure: string = '';
  private roof: string = '';
  private hasGarage: boolean = false;
  private hasPool: boolean = false;
  private hasGarden: boolean = false;

  setFoundation(foundation: string): this {
    this.foundation = foundation;
    return this;
  }

  setStructure(structure: string): this {
    this.structure = structure;
    return this;
  }

  setRoof(roof: string): this {
    this.roof = roof;
    return this;
  }

  setGarage(hasGarage: boolean): this {
    this.hasGarage = hasGarage;
    return this;
  }

  setPool(hasPool: boolean): this {
    this.hasPool = hasPool;
    return this;
  }

  setGarden(hasGarden: boolean): this {
    this.hasGarden = hasGarden;
    return this;
  }

  describe(): string {
    return `House with ${this.foundation} foundation, ${this.structure} structure, ${this.roof} roof, ` +
      `${this.hasGarage ? 'with garage' : 'no garage'}, ` +
      `${this.hasPool ? 'with pool' : 'no pool'}, ` +
      `${this.hasGarden ? 'with garden' : 'no garden'}`;
  }
}

class HouseBuilder {
  private house: House;

  constructor() {
    this.house = new House();
  }

  foundation(type: string): HouseBuilder {
    this.house.setFoundation(type);
    return this;
  }

  structure(type: string): HouseBuilder {
    this.house.setStructure(type);
    return this;
  }

  roof(type: string): HouseBuilder {
    this.house.setRoof(type);
    return this;
  }

  garage(hasGarage: boolean): HouseBuilder {
    this.house.setGarage(hasGarage);
    return this;
  }

  pool(hasPool: boolean): HouseBuilder {
    this.house.setPool(hasPool);
    return this;
  }

  garden(hasGarden: boolean): HouseBuilder {
    this.house.setGarden(hasGarden);
    return this;
  }

  build(): House {
    return this.house;
  }
}

// Usage
const house = new HouseBuilder()
  .foundation('Concrete')
  .structure('Brick')
  .roof('Tile')
  .garage(true)
  .pool(false)
  .garden(true)
  .build();

console.log(house.describe());
```

### 4. Prototype

```typescript
// Prototype: Create objects by cloning existing ones
interface Prototype {
  clone(): Prototype;
  getInfo(): string;
}

class User implements Prototype {
  constructor(
    private name: string,
    private age: number,
    private permissions: string[]
  ) {}

  clone(): User {
    return new User(this.name, this.age, [...this.permissions]);
  }

  getInfo(): string {
    return `User: ${this.name}, Age: ${this.age}, Permissions: ${this.permissions.join(', ')}`;
  }

  addPermission(permission: string): void {
    this.permissions.push(permission);
  }
}

class UserRegistry {
  private prototypes: Map<string, Prototype> = new Map();

  addPrototype(name: string, prototype: Prototype): void {
    this.prototypes.set(name, prototype);
  }

  getPrototype(name: string): Prototype | undefined {
    return this.prototypes.get(name)?.clone();
  }
}

// Usage
const adminTemplate = new User('Admin Template', 30, ['read', 'write']);
const guestTemplate = new User('Guest Template', 25, ['read']);

const registry = new UserRegistry();
registry.addPrototype('admin', adminTemplate);
registry.addPrototype('guest', guestTemplate);

const newAdmin = registry.getPrototype('admin');
if (newAdmin) {
  newAdmin.addPermission('delete');
  console.log(newAdmin.getInfo());
}
```

### 5. Singleton

```typescript
// Singleton: Ensure only one instance exists
class DatabaseConnection {
  private static instance: DatabaseConnection | null = null;
  private connection: string = '';

  private constructor() {
    this.connection = 'Connected to database';
  }

  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }

  query(sql: string): string {
    return `Executing: ${sql}`;
  }

  getConnectionInfo(): string {
    return this.connection;
  }
}

class ConfigManager {
  private static instance: ConfigManager;
  private config: Map<string, string> = new Map();

  private constructor() {
    this.config.set('app.name', 'MyApp');
    this.config.set('app.version', '1.0.0');
  }

  static getInstance(): ConfigManager {
    if (!ConfigManager.instance) {
      ConfigManager.instance = new ConfigManager();
    }
    return ConfigManager.instance;
  }

  get(key: string): string | undefined {
    return this.config.get(key);
  }

  set(key: string, value: string): void {
    this.config.set(key, value);
  }
}

// Usage
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true

console.log(db1.query('SELECT * FROM users'));
console.log(ConfigManager.getInstance().get('app.name'));
```

---

## Structural Patterns

### 1. Adapter

```typescript
// Adapter: Make incompatible interfaces compatible
interface ModernPaymentProcessor {
  processPayment(amount: number): Promise<{ success: boolean; transactionId: string }>;
}

class LegacyPaymentSystem {
  legacyProcess(amount: number, currency: string): string {
    return `Legacy processing: ${amount} ${currency}`;
  }
}

class PaymentAdapter implements ModernPaymentProcessor {
  constructor(private legacySystem: LegacyPaymentSystem) {}

  async processPayment(amount: number): Promise<{ success: boolean; transactionId: string }> {
    const result = this.legacySystem.legacyProcess(amount, 'USD');
    return {
      success: true,
      transactionId: `TXN-${Date.now()}`,
    };
  }
}

// Third-party library with different interface
class ExternalPaymentService {
  charge(cardToken: string, amountInCents: number): { id: string; status: string } {
    return {
      id: `EXT-${Date.now()}`,
      status: 'charged',
    };
  }
}

class ExternalPaymentAdapter implements ModernPaymentProcessor {
  constructor(private externalService: ExternalPaymentService) {}

  async processPayment(amount: number): Promise<{ success: boolean; transactionId: string }> {
    const result = this.externalService.charge('token', Math.round(amount * 100));
    return {
      success: result.status === 'charged',
      transactionId: result.id,
    };
  }
}

// Usage
const legacySystem = new LegacyPaymentSystem();
const adapter = new PaymentAdapter(legacySystem);
await adapter.processPayment(99.99);

const externalService = new ExternalPaymentService();
const externalAdapter = new ExternalPaymentAdapter(externalService);
await externalAdapter.processPayment(149.99);
```

### 2. Bridge

```typescript
// Bridge: Decouple abstraction from implementation
interface Device {
  turnOn(): void;
  turnOff(): void;
  setVolume(percent: number): void;
}

class TV implements Device {
  private volume: number = 0;
  private isOn: boolean = false;

  turnOn(): void {
    this.isOn = true;
    console.log('TV is on');
  }

  turnOff(): void {
    this.isOn = false;
    console.log('TV is off');
  }

  setVolume(percent: number): void {
    this.volume = percent;
    console.log(`TV volume set to ${percent}%`);
  }
}

class Radio implements Device {
  private volume: number = 0;
  private isOn: boolean = false;

  turnOn(): void {
    this.isOn = true;
    console.log('Radio is on');
  }

  turnOff(): void {
    this.isOn = false;
    console.log('Radio is off');
  }

  setVolume(percent: number): void {
    this.volume = percent;
    console.log(`Radio volume set to ${percent}%`);
  }
}

abstract class RemoteControl {
  protected device: Device;

  constructor(device: Device) {
    this.device = device;
  }

  abstract power(): void;
  abstract volumeUp(): void;
  abstract volumeDown(): void;
}

class BasicRemote extends RemoteControl {
  power(): void {
    this.device.turnOn();
  }

  volumeUp(): void {
    this.device.setVolume(100);
  }

  volumeDown(): void {
    this.device.setVolume(0);
  }
}

class AdvancedRemote extends RemoteControl {
  power(): void {
    if (this.device instanceof TV) {
      this.device.turnOn();
    } else {
      this.device.turnOff();
    }
  }

  volumeUp(): void {
    this.device.setVolume(80);
  }

  volumeDown(): void {
    this.device.setVolume(20);
  }

  mute(): void {
    this.device.setVolume(0);
  }
}

// Usage
const tv = new TV();
const basicRemote = new BasicRemote(tv);
basicRemote.power();

const radio = new Radio();
const advancedRemote = new AdvancedRemote(radio);
advancedRemote.power();
advancedRemote.mute();
```

### 3. Composite

```typescript
// Composite: Compose objects into tree structures
interface FileSystemComponent {
  getName(): string;
  getSize(): number;
  list(depth?: number): string;
}

class File implements FileSystemComponent {
  constructor(
    private name: string,
    private size: number
  ) {}

  getName(): string {
    return this.name;
  }

  getSize(): number {
    return this.size;
  }

  list(depth: number = 0): string {
    return `${'  '.repeat(depth)}📄 ${this.name} (${this.size} bytes)`;
  }
}

class Directory implements FileSystemComponent {
  private children: FileSystemComponent[] = [];

  constructor(private name: string) {}

  add(component: FileSystemComponent): void {
    this.children.push(component);
  }

  remove(component: FileSystemComponent): void {
    const index = this.children.indexOf(component);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }

  getName(): string {
    return this.name;
  }

  getSize(): number {
    return this.children.reduce((total, child) => total + child.getSize(), 0);
  }

  list(depth: number = 0): string {
    const indent = '  '.repeat(depth);
    const lines = [`${indent}📁 ${this.name}/`];
    for (const child of this.children) {
      lines.push(child.list(depth + 1));
    }
    return lines.join('\n');
  }
}

// Usage
const root = new Directory('root');
const docs = new Directory('documents');
const pictures = new Directory('pictures');

docs.add(new File('resume.pdf', 1024));
docs.add(new File('budget.xlsx', 2048));
pictures.add(new File('vacation.jpg', 4096));
pictures.add(new File('profile.png', 1536));

root.add(docs);
root.add(pictures);

console.log(root.list());
console.log(`\nTotal size: ${root.getSize()} bytes`);
```

### 4. Decorator

```typescript
// Decorator: Add behavior dynamically
interface Coffee {
  getDescription(): string;
  getCost(): number;
}

class Espresso implements Coffee {
  getDescription(): string {
    return 'Espresso';
  }

  getCost(): number {
    return 3.00;
  }
}

abstract class CoffeeDecorator implements Coffee {
  protected coffee: Coffee;

  constructor(coffee: Coffee) {
    this.coffee = coffee;
  }

  abstract getDescription(): string;
  abstract getCost(): number;
}

class MilkDecorator extends CoffeeDecorator {
  getDescription(): string {
    return this.coffee.getDescription() + ', Milk';
  }

  getCost(): number {
    return this.coffee.getCost() + 0.50;
  }
}

class SugarDecorator extends CoffeeDecorator {
  getDescription(): string {
    return this.coffee.getDescription() + ', Sugar';
  }

  getCost(): number {
    return this.coffee.getCost() + 0.25;
  }
}

class WhipDecorator extends CoffeeDecorator {
  getDescription(): string {
    return this.coffee.getDescription() + ', Whip';
  }

  getCost(): number {
    return this.coffee.getCost() + 0.75;
  }
}

class CaramelDecorator extends CoffeeDecorator {
  getDescription(): string {
    return this.coffee.getDescription() + ', Caramel';
  }

  getCost(): number {
    return this.coffee.getCost() + 0.60;
  }
}

// Usage
let coffee: Coffee = new Espresso();
console.log(`${coffee.getDescription()}: $${coffee.getCost().toFixed(2)}`);

coffee = new MilkDecorator(coffee);
console.log(`${coffee.getDescription()}: $${coffee.getCost().toFixed(2)}`);

coffee = new WhipDecorator(coffee);
console.log(`${coffee.getDescription()}: $${coffee.getCost().toFixed(2)}`);
```

### 5. Facade

```typescript
// Facade: Provide simplified interface to complex subsystem
class CPU {
  startProcess(): void {
    console.log('CPU: Starting process...');
  }

  executeInstruction(): void {
    console.log('CPU: Executing instruction...');
  }
}

class Memory {
  allocate(size: number): void {
    console.log(`Memory: Allocating ${size} bytes...`);
  }

  load(address: number, data: string): void {
    console.log(`Memory: Loading ${data} at address ${address}`);
  }
}

class HardDrive {
  readSector(sector: number): string {
    console.log(`HardDrive: Reading sector ${sector}`);
    return '0101010';
  }
}

class ComputerFacade {
  private cpu = new CPU();
  private memory = new Memory();
  private hardDrive = new HardDrive();

  startComputer(): void {
    console.log('=== Computer Startup ===');
    this.cpu.startProcess();
    this.memory.allocate(1024);
    const data = this.hardDrive.readSector(100);
    this.memory.load(0x1000, data);
    this.cpu.executeInstruction();
    console.log('=== Computer Ready ===\n');
  }
}

// Usage
const computer = new ComputerFacade();
computer.startComputer();
```

### 6. Flyweight

```typescript
// Flyweight: Share common state to support large numbers of objects
interface TreeType {
  render(x: number, y: number, color: string): void;
}

class PineTree implements TreeType {
  render(x: number, y: number, color: string): void {
    console.log(`Rendering PINE tree at (${x}, ${y}) with color ${color}`);
  }
}

class OakTree implements TreeType {
  render(x: number, y: number, color: string): void {
    console.log(`Rendering OAK tree at (${x}, ${y}) with color ${color}`);
  }
}

class TreeFactory {
  private static treeTypes: Map<string, TreeType> = new Map();

  static getTreeType(type: string): TreeType {
    if (!this.treeTypes.has(type)) {
      switch (type) {
        case 'pine':
          this.treeTypes.set(type, new PineTree());
          break;
        case 'oak':
          this.treeTypes.set(type, new OakTree());
          break;
      }
    }
    return this.treeTypes.get(type)!;
  }
}

class Tree {
  constructor(
    private x: number,
    private y: number,
    private type: string,
    private color: string
  ) {}

  render(): void {
    const treeType = TreeFactory.getTreeType(this.type);
    treeType.render(this.x, this.y, this.color);
  }
}

class Forest {
  private trees: Tree[] = [];

  plantTree(x: number, y: number, type: string, color: string): void {
    this.trees.push(new Tree(x, y, type, color));
  }

  renderAll(): void {
    console.log('=== Rendering Forest ===');
    this.trees.forEach((tree) => tree.render());
  }
}

// Usage
const forest = new Forest();
forest.plantTree(10, 20, 'pine', 'darkgreen');
forest.plantTree(15, 25, 'oak', 'lightgreen');
forest.plantTree(12, 22, 'pine', 'green');
// ... plant thousands of trees with only 2 TreeType instances
forest.renderAll();
```

### 7. Proxy

```typescript
// Proxy: Provide a surrogate for another object
interface Database {
  query(sql: string): Promise<any[]>;
}

class RealDatabase implements Database {
  async query(sql: string): Promise<any[]> {
    console.log(`Executing query: ${sql}`);
    await new Promise((resolve) => setTimeout(resolve, 1000));
    return [{ id: 1, name: 'John' }];
  }
}

class CachedDatabase implements Database {
  private cache: Map<string, any[]> = new Map();
  private database: RealDatabase;

  constructor(database: RealDatabase) {
    this.database = database;
  }

  async query(sql: string): Promise<any[]> {
    if (this.cache.has(sql)) {
      console.log('Returning cached result');
      return this.cache.get(sql)!;
    }

    const result = await this.database.query(sql);
    this.cache.set(sql, result);
    return result;
  }
}

class LoggingDatabase implements Database {
  private database: Database;

  constructor(database: Database) {
    this.database = database;
  }

  async query(sql: string): Promise<any[]> {
    const start = Date.now();
    const result = await this.database.query(sql);
    const duration = Date.now() - start;
    console.log(`Query executed in ${duration}ms, returned ${result.length} rows`);
    return result;
  }
}

// Usage
const realDb = new RealDatabase();
const cachedDb = new CachedDatabase(realDb);
const loggedDb = new LoggingDatabase(cachedDb);

await loggedDb.query('SELECT * FROM users');
await loggedDb.query('SELECT * FROM users'); // Uses cache
```

---

## Behavioral Patterns

### 1. Chain of Responsibility

```typescript
// Chain of Responsibility: Pass request along a chain of handlers
abstract class Handler {
  protected next: Handler | null = null;

  setNext(handler: Handler): Handler {
    this.next = handler;
    return handler;
  }

  abstract handle(request: string): string;
}

class AuthenticationHandler extends Handler {
  handle(request: string): string {
    if (request.includes('token')) {
      return `AuthenticationHandler: Authenticated ${request}`;
    }
    if (this.next) {
      return this.next.handle(request);
    }
    return 'Request rejected: No authentication';
  }
}

class ValidationHandler extends Handler {
  handle(request: string): string {
    if (request.includes('valid')) {
      return `ValidationHandler: Validated ${request}`;
    }
    if (this.next) {
      return this.next.handle(request);
    }
    return 'Request rejected: Validation failed';
  }
}

class LoggingHandler extends Handler {
  handle(request: string): string {
    if (request.includes('log')) {
      return `LoggingHandler: Logged ${request}`;
    }
    if (this.next) {
      return this.next.handle(request);
    }
    return 'Request rejected: Logging failed';
  }
}

// Usage
const auth = new AuthenticationHandler();
const validation = new ValidationHandler();
const logging = new LoggingHandler();

auth.setNext(validation).setNext(logging);

console.log(auth.handle('request with token'));
console.log(auth.handle('request valid'));
console.log(auth.handle('request log'));
```

### 2. Command

```typescript
// Command: Encapsulate request as an object
interface Command {
  execute(): void;
  undo(): void;
}

class Light {
  private isOn: boolean = false;

  on(): void {
    this.isOn = true;
    console.log('Light is ON');
  }

  off(): void {
    this.isOn = false;
    console.log('Light is OFF');
  }

  getState(): boolean {
    return this.isOn;
  }
}

class LightOnCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.on();
  }

  undo(): void {
    this.light.off();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.off();
  }

  undo(): void {
    this.light.on();
  }
}

class RemoteControl {
  private commands: Map<number, Command> = new Map();
  private history: Command[] = [];

  setCommand(slot: number, command: Command): void {
    this.commands.set(slot, command);
  }

  pressButton(slot: number): void {
    const command = this.commands.get(slot);
    if (command) {
      command.execute();
      this.history.push(command);
    }
  }

  pressUndo(): void {
    const command = this.history.pop();
    if (command) {
      command.undo();
    }
  }
}

// Usage
const light = new Light();
const lightOn = new LightOnCommand(light);
const lightOff = new LightOffCommand(light);

const remote = new RemoteControl();
remote.setCommand(1, lightOn);
remote.setCommand(2, lightOff);

remote.pressButton(1);
remote.pressButton(2);
remote.pressUndo();
```

### 3. Iterator

```typescript
// Iterator: Traverse collections without exposing underlying structure
interface Iterator<T> {
  hasNext(): boolean;
  next(): T;
}

interface Aggregator<T> {
  createIterator(): Iterator<T>;
}

class Node<T> {
  constructor(public value: T, public left: Node<T> | null = null, public right: Node<T> | null = null) {}
}

class BinaryTreeIterator<T> implements Iterator<T> {
  private stack: Node<T>[] = [];
  private current: Node<T> | null;

  constructor(root: Node<T> | null) {
    this.current = root;
  }

  hasNext(): boolean {
    return this.current !== null || this.stack.length > 0;
  }

  next(): T {
    while (this.current) {
      this.stack.push(this.current);
      this.current = this.current.left;
    }
    this.current = this.stack.pop()!;
    const value = this.current.value;
    this.current = this.current.right;
    return value;
  }
}

class BinaryTree<T> implements Aggregator<T> {
  constructor(public root: Node<T> | null) {}

  createIterator(): Iterator<T> {
    return new BinaryTreeIterator(this.root);
  }
}

// Usage
const root = new Node(1);
root.left = new Node(2);
root.right = new Node(3);
root.left.left = new Node(4);
root.left.right = new Node(5);

const tree = new BinaryTree(root);
const iterator = tree.createIterator();

console.log('In-order traversal:');
while (iterator.hasNext()) {
  console.log(iterator.next());
}
```

### 4. Mediator

```typescript
// Mediator: Define object that encapsulates interaction
interface ChatMediator {
  sendMessage(msg: string, user: User): void;
  addUser(user: User): void;
}

class ChatRoom implements ChatMediator {
  private users: User[] = [];

  addUser(user: User): void {
    this.users.push(user);
    user.setMediator(this);
  }

  sendMessage(msg: string, sender: User): void {
    for (const user of this.users) {
      if (user !== sender) {
        user.receive(msg);
      }
    }
  }
}

abstract class User {
  protected mediator: ChatMediator | null = null;
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }

  setMediator(mediator: ChatMediator): void {
    this.mediator = mediator;
  }

  abstract send(msg: string): void;
  abstract receive(msg: string): void;
}

class ConcreteUser extends User {
  constructor(name: string) {
    super(name);
  }

  send(msg: string): void {
    console.log(`${this.name} sends: ${msg}`);
    this.mediator?.sendMessage(msg, this);
  }

  receive(msg: string): void {
    console.log(`${this.name} receives: ${msg}`);
  }
}

// Usage
const chatRoom = new ChatRoom();

const user1 = new ConcreteUser('Alice');
const user2 = new ConcreteUser('Bob');
const user3 = new ConcreteUser('Charlie');

chatRoom.addUser(user1);
chatRoom.addUser(user2);
chatRoom.addUser(user3);

user1.send('Hello everyone!');
user2.send('Hi Alice!');
```

### 5. Observer

```typescript
// Observer: Notify multiple objects about state changes
interface Observer {
  update(subject: Subject): void;
}

interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

class WeatherStation implements Subject {
  private observers: Observer[] = [];
  private temperature: number = 0;
  private humidity: number = 0;

  attach(observer: Observer): void {
    this.observers.push(observer);
  }

  detach(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }

  notify(): void {
    for (const observer of this.observers) {
      observer.update(this);
    }
  }

  setMeasurements(temp: number, humidity: number): void {
    this.temperature = temp;
    this.humidity = humidity;
    this.notify();
  }

  getTemperature(): number {
    return this.temperature;
  }

  getHumidity(): number {
    return this.humidity;
  }
}

class PhoneDisplay implements Observer {
  update(subject: Subject): void {
    if (subject instanceof WeatherStation) {
      console.log(`Phone Display: Temp=${subject.getTemperature()}°C, Humidity=${subject.getHumidity()}%`);
    }
  }
}

class WebDisplay implements Observer {
  update(subject: Subject): void {
    if (subject instanceof WeatherStation) {
      console.log(`Web Display: Updated weather - ${subject.getTemperature()}°C`);
    }
  }
}

// Usage
const weatherStation = new WeatherStation();
const phone = new PhoneDisplay();
const web = new WebDisplay();

weatherStation.attach(phone);
weatherStation.attach(web);

weatherStation.setMeasurements(25, 60);
weatherStation.setMeasurements(28, 55);
```

### 6. State

```typescript
// State: Alter behavior based on internal state
interface State {
  handle(context: Context): void;
}

class Context {
  private state: State;

  constructor(state: State) {
    this.state = state;
  }

  setState(state: State): void {
    this.state = state;
  }

  request(): void {
    this.state.handle(this);
  }
}

class PendingState implements State {
  handle(context: Context): void {
    console.log('Request is pending...');
    context.setState(new ProcessingState());
  }
}

class ProcessingState implements State {
  handle(context: Context): void {
    console.log('Request is processing...');
    context.setState(new CompletedState());
  }
}

class CompletedState implements State {
  handle(context: Context): void {
    console.log('Request is already completed!');
  }
}

class RejectedState implements State {
  handle(context: Context): void {
    console.log('Request has been rejected');
  }
}

// Usage
const context = new Context(new PendingState());
context.request();
context.request();
context.request();
```

### 7. Strategy

```typescript
// Strategy: Define family of algorithms and make them interchangeable
interface PaymentStrategy {
  pay(amount: number): boolean;
}

class CreditCardStrategy implements PaymentStrategy {
  constructor(
    private cardNumber: string,
    private cvv: string,
    private expiry: string
  ) {}

  pay(amount: number): boolean {
    console.log(`Paying $${amount} with credit card ${this.cardNumber}`);
    return true;
  }
}

class PayPalStrategy implements PaymentStrategy {
  constructor(private email: string) {}

  pay(amount: number): boolean {
    console.log(`Paying $${amount} with PayPal account ${this.email}`);
    return true;
  }
}

class CryptoStrategy implements PaymentStrategy {
  constructor(private walletAddress: string) {}

  pay(amount: number): boolean {
    console.log(`Paying $${amount} with crypto wallet ${this.walletAddress}`);
    return true;
  }
}

class ShoppingCart {
  private items: { name: string; price: number }[] = [];
  private paymentStrategy: PaymentStrategy | null = null;

  addItem(name: string, price: number): void {
    this.items.push({ name, price });
  }

  getTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }

  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.paymentStrategy = strategy;
  }

  checkout(): boolean {
    if (!this.paymentStrategy) {
      console.log('No payment strategy set');
      return false;
    }
    return this.paymentStrategy.pay(this.getTotal());
  }
}

// Usage
const cart = new ShoppingCart();
cart.addItem('Laptop', 999);
cart.addItem('Mouse', 29);

cart.setPaymentStrategy(new CreditCardStrategy('4111', '123', '12/25'));
cart.checkout();

cart.setPaymentStrategy(new PayPalStrategy('user@example.com'));
cart.checkout();
```

### 8. Template Method

```typescript
// Template Method: Define skeleton of algorithm in a method
abstract class Game {
  protected abstract initialize(): void;
  protected abstract startPlay(): void;
  protected abstract endPlay(): void;

  play(): void {
    this.initialize();
    this.startPlay();
    this.endPlay();
  }
}

class Cricket extends Game {
  protected initialize(): void {
    console.log('Cricket game initialized');
  }

  protected startPlay(): void {
    console.log('Cricket game started');
  }

  protected endPlay(): void {
    console.log('Cricket game ended');
  }
}

class Football extends Game {
  protected initialize(): void {
    console.log('Football game initialized');
  }

  protected startPlay(): void {
    console.log('Football game started');
  }

  protected endPlay(): void {
    console.log('Football game ended');
  }
}

// Usage
const cricket = new Cricket();
cricket.play();

const football = new Football();
football.play();
```

---

## Design Pattern Selection Guide

| Problem | Recommended Patterns |
|---------|---------------------|
| Creating objects with complex construction | Builder, Factory Method, Abstract Factory |
| Ensuring single instance | Singleton |
| Adapting incompatible interfaces | Adapter |
| Decoupling abstraction from implementation | Bridge |
| Treating individual and composite objects uniformly | Composite |
| Adding responsibilities dynamically | Decorator |
| Providing simplified interface to complex system | Facade |
| Sharing common state efficiently | Flyweight |
| Controlling access to objects | Proxy |
| Handling requests through a chain | Chain of Responsibility |
| Encapsulating requests as objects | Command |
| Traversing collections | Iterator |
| Centralizing communication between objects | Mediator |
| Notifying objects about state changes | Observer |
| Changing behavior with state | State |
| Selecting algorithm at runtime | Strategy |
| Defining algorithm skeleton | Template Method |

---

## Clean Architecture Patterns

### Input/Output Port Pattern

The Input/Output Port pattern is central to Clean Architecture, defining the interface between use cases and the outside world:

```typescript
// Input Port: Defines what the use case accepts
interface CreateProductUseCase extends Command<
    CreateProductUseCase.CreateProductInput, 
    CqrsOutput<ProductDto>
> {
    class CreateProductInput implements Input {
        public productId: string;
        public name: string;
        public userId: string;
    }
}

// Output Port: Maps domain objects to presentation
interface ProductPresenter {
    present(product: Product): ProductDto;
}

// Service Implementation: Orchestrates the use case
class CreateProductService implements CreateProductUseCase {
    constructor(
        private repository: Repository<Product, ProductId>,
        private messageBus: MessageBus
    ) {}

    async execute(input: CreateProductInput): Promise<CqrsOutput<ProductDto>> {
        const product = new Product(
            ProductId.valueOf(input.productId),
            input.name,
            UserId.valueOf(input.userId)
        );

        repository.save(product);
        messageBus.publish(product.getUncommittedEvents());

        return CqrsOutput.of(ProductMapper.toDto(product));
    }
}
```

### Command/Query Separation (CQS)

Separate operations that change state from those that read state:

```typescript
// Command: Modifies state, returns CqrsOutput
interface CreateProductUseCase extends Command<Input, CqrsOutput> {}

// Query: Read-only, returns custom Output
interface GetProductUseCase extends Query<Input, GetProductOutput> {
    class GetProductOutput implements Output {
        public exitCode: ExitCode;
        public message: string;
        public product: ProductDto;
    }
}
```

### Repository Pattern

Abstracts data access behind a generic interface:

```typescript
// ✅ CORRECT: Generic Repository Usage
// NO custom Repository interface needed
Repository<Product, ProductId> repository;

// Standard methods only
repository.findById(ProductId id);  // Optional<Product>
repository.save(Product aggregate); // void
repository.delete(Product aggregate); // void
```

### Bridge Pattern

Decouples abstraction from implementation for multiple repository types:

```typescript
// Abstraction
interface ToDoListRepository {
    findById(id: ToDoListId): Promise<ToDoList | null>;
    save(aggregate: ToDoList): Promise<void>;
}

// Implementation 1: In-memory
class ToDoListInMemoryRepository implements ToDoListRepository {
    private store: Map<string, ToDoList> = new Map();
    // ... implementations
}

// Implementation 2: JPA/Spring Data
class ToDoListCrudRepository implements ToDoListRepository {
    private crudRepository: SpringDataRepository;
    // ... implementations
}

// Client uses abstraction, not concrete implementation
class UseCase {
    constructor(private repository: ToDoListRepository) {}
}
```

---

## References and Further Reading

1. Gamma, Erich, et al. "Design Patterns: Elements of Reusable Object-Oriented Software." Addison-Wesley, 1994.
2. Freeman, Eric, and Elisabeth Robson. "Head First Design Patterns." O'Reilly, 2004.
3. Refactoring Guru. "Design Patterns." https://refactoring.guru/design-patterns
4. "AntiPatterns: Refactoring Software, Architectures, and Projects in Crisis." Brown et al., Wiley, 1998.

---

## Additional Architectural Patterns

### Data Transfer Object (DTO)

```typescript
// DTO: Plain objects for data transfer between layers
interface ProductDto {
  id: string;
  name: string;
  price: number;
  category: string;
}

// ✅ CORRECT: Simple POJO with public fields
class ProductDtoImpl implements ProductDto {
  constructor(
    public id: string,
    public name: string,
    public price: number,
    public category: string
  ) {}
}

// ❌ INCORRECT: Adding behavior to DTOs
class BadProductDto {
  constructor(
    public id: string,
    public name: string,
    public price: number
  ) {}

  // DTOs should NOT have behavior - this is wrong!
  calculateDiscount(): number {
    return this.price * 0.1;
  }
}
```

### Persistent Object (PO)

```typescript
// PO: Database entity representation, separate from domain
interface ProductPo {
  id: string;
  name: string;
  price_cents: number;
  category_id: string;
  created_at: Date;
  updated_at: Date;
}

// PO is implementation-specific, may use framework annotations
class JpaProductPo implements ProductPo {
  constructor(
    @Id public id: string,
    @Column public name: string,
    @Column(name = "price_cents") public priceCents: number,
    @ManyToOne @JoinColumn(name = "category_id") public categoryId: string,
    @Column public createdAt: Date,
    @Column public updatedAt: Date
  ) {}
}
```

### Mapper Pattern

```typescript
// Mapper: Convert between Domain, DTO, and PO representations
class ProductMapper {
  // Domain → DTO
  static toDto(product: Product): ProductDto {
    return {
      id: product.id.value,
      name: product.name.value,
      price: product.price.amount,
      category: product.category.value
    };
  }

  // PO → Domain
  static toDomain(po: ProductPo): Product {
    return Product.reconstruct(
      ProductId.fromString(po.id),
      ProductName.of(po.name),
      Money.fromCents(po.priceCents),
      CategoryId.fromString(po.categoryId)
    );
  }

  // Domain → PO
  static toPo(product: Product): ProductPo {
    return {
      id: product.id.value,
      name: product.name.value,
      price_cents: product.price.toCents(),
      category_id: product.category.value,
      created_at: product.createdAt,
      updated_at: new Date()
    };
  }

  // Collection transformations
  static toDtoList(products: Product[]): ProductDto[] {
    return products.map(p => this.toDto(p));
  }

  static toDomainList(pos: ProductPo[]): Product[] {
    return pos.map(po => this.toDomain(po));
  }
}
```

### Null Object Pattern

```typescript
// Null Object: Eliminate null checks with a meaningful no-op
interface Input {
  getString(key: string): string | null;
  getNumber(key: string): number | null;
}

// Null Input provides sensible defaults
class NullInput implements Input {
  getString(key: string): string {
    return '';
  }

  getNumber(key: string): number {
    return 0;
  }

  getBoolean(key: string): boolean {
    return false;
  }

  hasValue(key: string): boolean {
    return false;
  }
}

// Usage - no null checks needed
class UserService {
  constructor(private input: Input) {}

  createUser(): void {
    const name = this.input.getString('name');
    const age = this.input.getNumber('age');

    // No null checks needed - NullInput returns empty values
    if (name && age > 0) {
      // Process user
    }
  }
}
```

### Optional Pattern

```typescript
// Optional: Handle potentially missing values explicitly
interface UserRepository {
  findById(id: UserId): Promise<Optional<User>>;
  findByEmail(email: string): Promise<Optional<User>>;
}

class UserService {
  constructor(private userRepository: UserRepository) {}

  async getUser(id: UserId): Promise<UserDto | null> {
    const user = await this.userRepository.findById(id);

    // Transform optional to result
    return user
      .map(u => UserMapper.toDto(u))
      .orElse(null);
  }

  async getUserByEmail(email: string): Promise<UserDto> {
    const user = await this.userRepository.findByEmail(email);

    // Throw if not found - clear intent
    return user
      .map(u => UserMapper.toDto(u))
      .orElseThrow(() => new NotFoundError(`User with email ${email} not found`));
  }

  async findActiveUsers(): Promise<UserDto[]> {
    const users = await this.userRepository.findAll();

    // Chain operations
    return users
      .filter(u => u.isActive())
      .map(u => UserMapper.toDto(u))
      .toList();
  }
}
```

### Builder Pattern (Fluent API)

```typescript
// Builder: Construct complex objects step by step with method chaining
interface CqrsOutputBuilder<T> {
  succeed(): CqrsOutput<T>;
  succeedWith(data: T): CqrsOutput<T>;
  fail(message: string): CqrsOutput<T>;
  withError(error: Error): CqrsOutput<T>;
}

class CqrsOutput<T> {
  private constructor(
    private readonly _success: boolean,
    private readonly _data: T | null,
    private readonly _message: string | null,
    private readonly _error: Error | null
  ) {}

  get success(): boolean {
    return this._success;
  }

  get data(): T | null {
    return this._data;
  }

  get message(): string | null {
    return this._message;
  }

  get error(): Error | null {
    return this._error;
  }

  static create<T>(): CqrsOutputBuilder<T> {
    return new CqrsOutputBuilderImpl<T>();
  }

  static of<T>(data: T): CqrsOutput<T> {
    return new CqrsOutput(true, data, null, null);
  }

  static success<T>(message: string): CqrsOutput<T> {
    return new CqrsOutput(true, null, message, null);
  }

  static failure<T>(error: Error): CqrsOutput<T> {
    return new CqrsOutput(false, null, error.message, error);
  }
}

class CqrsOutputBuilderImpl<T> implements CqrsOutputBuilder<T> {
  private _success = false;
  private _data: T | null = null;
  private _message: string | null = null;
  private _error: Error | null = null;

  succeed(): CqrsOutput<T> {
    this._success = true;
    return new CqrsOutput(this._success, this._data, this._message, this._error);
  }

  succeedWith(data: T): CqrsOutput<T> {
    this._success = true;
    this._data = data;
    return new CqrsOutput(this._success, this._data, this._message, this._error);
  }

  fail(message: string): CqrsOutput<T> {
    this._success = false;
    this._message = message;
    return new CqrsOutput(this._success, this._data, this._message, this._error);
  }

  withError(error: Error): CqrsOutput<T> {
    this._success = false;
    this._error = error;
    this._message = error.message;
    return new CqrsOutput(this._success, this._data, this._message, this._error);
  }
}

// Usage - fluent builder pattern
const result = CqrsOutput.create<UserDto>()
  .succeedWith(userDto)
  .data;

const errorResult = CqrsOutput.create<Task>()
  .fail('Task not found')
  .message;

const successResult = CqrsOutput.create<Order>()
  .withError(new ValidationError('Invalid order'))
  .success;
```

### Factory Method Pattern (Domain)

```typescript
// Factory Method: Create objects with validation and business logic
class ProductName {
  private constructor(private readonly value: string) {}

  // Static factory method as standard
  static of(value: string): ProductName {
    // Validation in factory
    if (!value || value.trim().length === 0) {
      throw new Error('Product name cannot be empty');
    }
    if (value.length > 100) {
      throw new Error('Product name cannot exceed 100 characters');
    }
    return new ProductName(value.trim());
  }

  // Alternative factory for special cases
  static fromExisting(value: string): ProductName {
    // Bypass validation for reconstructed objects
    return new ProductName(value);
  }

  get value(): string {
    return this.value;
  }
}

// Value Object with record (TypeScript)
class ProductId {
  constructor(private readonly value: string) {}

  static generate(): ProductId {
    return new ProductId(UUID.randomUUID().toString());
  }

  static fromString(value: string): ProductId {
    if (!value) {
      throw new Error('Product ID cannot be empty');
    }
    return new ProductId(value);
  }

  get value(): string {
    return this._value;
  }
}
```

### Read-Only Interface Pattern

```typescript
// Read-Only Interface: Prevent unauthorized modifications
interface ReadOnlyProject {
  readonly id: ProjectId;
  readonly name: ProjectName;
  readonly status: ProjectStatus;
  readonly createdAt: Date;
  getMembers(): ReadonlyArray<ProjectMember>;
}

class Project implements ReadOnlyProject {
  // Public readonly properties
  readonly id: ProjectId;
  readonly name: ProjectName;
  readonly createdAt: Date;

  // Private mutable properties
  private _status: ProjectStatus;
  private _members: ProjectMember[] = [];

  constructor(id: ProjectId, name: ProjectName) {
    this.id = id;
    this.name = name;
    this._status = ProjectStatus.ACTIVE;
    this.createdAt = new Date();
  }

  // Read-only access to mutable state
  get status(): ProjectStatus {
    return this._status;
  }

  getMembers(): ReadonlyArray<ProjectMember> {
    return [...this._members];
  }

  // Modifying methods - only for internal use
  addMember(member: ProjectMember): void {
    this._members.push(member);
  }

  // Return read-only view from domain methods
  getSnapshot(): ReadOnlyProject {
    return {
      id: this.id,
      name: this.name,
      status: this._status,
      createdAt: this.createdAt,
      getMembers: () => this.getMembers()
    };
  }
}
```

### Strategy Pattern (Repository Implementations)

```typescript
// Strategy: Multiple implementations of same interface
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

// Strategy 1: In-memory for testing
class InMemoryUserRepository implements UserRepository {
  private store: Map<string, User> = new Map();

  async findById(id: string): Promise<User | null> {
    return this.store.get(id) || null;
  }

  async save(user: User): Promise<void> {
    this.store.set(user.id.value, user);
  }

  async delete(id: string): Promise<void> {
    this.store.delete(id);
  }
}

// Strategy 2: JPA/Spring Data
class JpaUserRepository implements UserRepository {
  private crudRepository: SpringDataUserRepository;

  constructor(crudRepository: SpringDataUserRepository) {
    this.crudRepository = crudRepository;
  }

  async findById(id: string): Promise<User | null> {
    const entity = await this.crudRepository.findById(id);
    return entity ? UserMapper.toDomain(entity) : null;
  }

  async save(user: User): Promise<void> {
    const entity = UserMapper.toPo(user);
    await this.crudRepository.save(entity);
  }

  async delete(id: string): Promise<void> {
    await this.crudRepository.deleteById(id);
  }
}

// Client uses abstraction - interchangeable at runtime
class UserService {
  constructor(private repository: UserRepository) {}

  async getUser(id: string): Promise<User | null> {
    return this.repository.findById(id);
  }
}
```

### Command Pattern (Console Controllers)

```typescript
// Command: Encapsulate requests as objects
interface ConsoleCommand {
  execute(args: string[]): Promise<void>;
  getName(): string;
  getDescription(): string;
}

abstract class AbstractCommand implements ConsoleCommand {
  abstract getName(): string;
  abstract getDescription(): string;
  abstract execute(args: string[]): Promise<void>;

  protected validateArgs(args: string[], required: number): void {
    if (args.length < required) {
      throw new Error(`Usage: ${this.getName()} <${Array(required).fill('arg').join('> <')}>`);
    }
  }
}

class AddProjectCommand extends AbstractCommand {
  constructor(
    private addProjectUseCase: AddProjectUseCase,
    private presenter: AddProjectPresenter
  ) {
    super();
  }

  getName(): string {
    return 'add-project';
  }

  getDescription(): string {
    return 'Add a new project';
  }

  async execute(args: string[]): Promise<void> {
    this.validateArgs(args, 2);

    const input = new AddProjectInput();
    input.name = args[0];
    input.description = args[1];

    const output = await this.addProjectUseCase.execute(input);
    this.presenter.present(output);
  }
}

// Executor pattern for command dispatching
class CommandExecutor {
  private commands: Map<string, ConsoleCommand> = new Map();

  register(command: ConsoleCommand): void {
    this.commands.set(command.getName(), command);
  }

  async execute(commandName: string, args: string[]): Promise<void> {
    const command = this.commands.get(commandName);
    if (!command) {
      throw new Error(`Unknown command: ${commandName}`);
    }
    await command.execute(args);
  }

  getHelp(): string {
    const commands = Array.from(this.commands.values())
      .map(c => `  ${c.getName().padEnd(15)} - ${c.getDescription()}`)
      .join('\n');
    return `Available commands:\n${commands}`;
  }
}
```

### Template Method Pattern (Use Cases)

```typescript
// Template Method: Define execution contract in base class
interface UseCase<I extends Input, O extends Output> {
  execute(input: I): Promise<O>;
}

// Base template with common validation
abstract class AbstractUseCase<I extends Input, O extends Output> implements UseCase<I, O> {
  abstract execute(input: I): Promise<O>;

  protected validateInput(input: I): void {
    // Common validation logic
    if (!input) {
      throw new Error('Input cannot be null');
    }
  }
}

// Concrete implementation provides specific behavior
class CreateOrderUseCase extends AbstractUseCase<CreateOrderInput, CqrsOutput<OrderDto>> {
  constructor(
    private orderRepository: Repository<Order, OrderId>,
    private inventoryService: InventoryService,
    private eventPublisher: EventPublisher
  ) {
    super();
  }

  async execute(input: CreateOrderInput): Promise<CqrsOutput<OrderDto>> {
    // Common validation from template
    this.validateInput(input);

    try {
      // Specific business logic
      await this.inventoryService.checkAvailability(input.items);

      const order = Order.create(input);

      this.orderRepository.save(order);

      for (const event of order.getUncommittedEvents()) {
        this.eventPublisher.publish(event);
      }

      return CqrsOutput.success(OrderMapper.toDto(order));
    } catch (error) {
      return CqrsOutput.failure(error as Error);
    }
  }
}
```

---

## Pattern Selection Matrix

| Need | GoF Patterns | Architectural Patterns | DDD Patterns |
|------|--------------|----------------------|--------------|
| Object creation | Factory, Builder, Prototype, Singleton | Factory Method, Abstract Factory | Factory, Value Object |
| Structure | Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy | Ports & Adapters, Layered Architecture | Aggregate, Bounded Context |
| Behavior | Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Chain of Responsibility | CQRS, Event Sourcing, Use Case | Domain Event, Specification, Service |

## Pattern Relationships

```
                    ┌─────────────────┐
                    │  ARCHITECTURAL  │
                    │     PATTERNS    │
                    │                 │
                    │  Clean Arch    │
                    │  Layered Arch  │
                    │  Ports/Adapters│
                    └────────┬────────┘
                             │
             ┌───────────────┼───────────────┐
             │               │               │
             ▼               ▼               ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │    GoF      │ │    DDD      │ │   DATA TX   │
    │   PATTERNS  │ │   PATTERNS  │ │   PATTERNS  │
    │             │ │             │ │             │
    │ Creational │ │  Aggregate  │ │    DTO      │
    │ Structural │ │  Entity     │ │    PO       │
    │ Behavioral │ │  Value Obj  │ │  Mapper     │
    └─────────────┘ └─────────────┘ └─────────────┘
```
