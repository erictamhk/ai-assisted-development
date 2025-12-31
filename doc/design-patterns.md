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

## References and Further Reading

1. Gamma, Erich, et al. "Design Patterns: Elements of Reusable Object-Oriented Software." Addison-Wesley, 1994.
2. Freeman, Eric, and Elisabeth Robson. "Head First Design Patterns." O'Reilly, 2004.
3. Refactoring Guru. "Design Patterns." https://refactoring.guru/design-patterns
4. "AntiPatterns: Refactoring Software, Architectures, and Projects in Crisis." Brown et al., Wiley, 1998.
