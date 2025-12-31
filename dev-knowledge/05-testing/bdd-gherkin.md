# Behavior-Driven Development (BDD)

## Core Concept

Behavior-Driven Development (BDD) is an Agile software development methodology that extends Test-Driven Development by emphasizing collaboration between technical and non-technical stakeholders. BDD focuses on defining the behavior of a system from the user's perspective using natural language constructs that can be understood by both developers and business users.

BDD was created by Dan North in the early 2000s as a response to the challenges he encountered while teaching TDD. The key insight was that tests written in technical code (like JUnit) were inaccessible to business stakeholders, leading to miscommunication and misalignment between what was built and what was actually needed.

The core innovation of BDD is the use of a domain-specific language (DSL) called Gherkin, which allows you to write specifications in natural language that can be executed as tests. This creates a bridge between business requirements and technical implementation, ensuring that everyone shares a common understanding of the system's behavior.

BDD promotes the principle of "Given-When-Then" as a structured format for expressing examples of system behavior. This structure helps teams think through scenarios, identify edge cases, and create executable specifications that serve as both documentation and tests.

---

## The BDD Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    BDD DEVELOPMENT WORKFLOW                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────┐  │
│  │   DISCOVER  │───►│   FORMALIZE │───►│  AUTOMATE   │───►│IMPLEMENT│  │
│  │             │    │             │    │             │    │         │  │
│  │  Collaborate│    │  Write      │    │  Generate   │    │  Write  │  │
│  │  on examples│    │  scenarios  │    │  test code  │    │  code   │  │
│  └─────────────┘    └─────────────┘    └─────────────┘    └─────────┘  │
│         │                  │                  │                  │      │
│         ▼                  ▼                  ▼                  ▼      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────┐  │
│  │   Example   │    │   Gherkin   │    │   Step      │    │  Pass   │  │
│  │   Mapping   │    │   Scenarios │    │ Definitions │    │  Tests  │  │
│  └─────────────┘    └─────────────┘    └─────────────┘    └─────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Step 1: Discovery
Engage with stakeholders to understand the desired behavior of the system. Use collaborative techniques like Example Mapping to identify scenarios and examples that illustrate the expected behavior.

### Step 2: Formalize
Convert the discovered examples into formal Gherkin scenarios using the Given-When-Then structure. These scenarios serve as both specifications and executable tests.

### Step 3: Automate
Use a BDD framework like Cucumber, SpecFlow, or Behave to generate step definitions from the Gherkin scenarios. These step definitions will be implemented to make the scenarios pass.

### Step 4: Implement
Write the production code that implements the behavior described in the scenarios. Run the scenarios to verify that the implementation meets the specified behavior.

---

## Gherkin Syntax Reference

Gherkin is a domain-specific language that allows you to write human-readable specifications. It uses a simple, line-oriented syntax that can be understood by non-technical stakeholders.

```
Feature: Search functionality
  As a user
  I want to search for products
  So that I can find items I want to purchase

  Background:
    Given the user is on the homepage
    And the search box is visible

  Scenario: Successful product search
    When the user enters "laptop" in the search box
    And clicks the search button
    Then results containing "laptop" should be displayed
    And at least one result should be shown

  Scenario: Search with no results
    When the user enters "xyz123nonexistent" in the search box
    And clicks the search button
    Then a "No results found" message should be displayed
    And the results count should be 0

  Scenario Outline: Search with various products
    When the user enters "<product_name>" in the search box
    And clicks the search button
    Then results should be displayed
    And the results should contain "<product_name>"

    Examples:
      | product_name |
      | smartphone   |
      | headphones   |
      | smartwatch   |
```

### Gherkin Keywords

| Keyword | Purpose | Description |
|---------|---------|-------------|
| `Feature` | Grouping | Groups related scenarios into a single feature |
| `Background` | Precondition | Defines steps that run before each scenario |
| `Scenario` | Example | A specific example of behavior |
| `Scenario Outline` | Template | A template for multiple similar scenarios |
| `Examples` | Data table | Provides data for Scenario Outline |
| `Given` | Precondition | Sets up the initial state |
| `When` | Action | Describes the action being performed |
| `Then` | Assertion | Describes the expected outcome |
| `And` | Conjunction | Continues the previous step |
| `But` | Conjunction | Continues the previous step with contrast |
| `*` | Wildcard | Can be used as a generic step keyword |

---

## BDD with Cucumber and TypeScript

```typescript
// features/search.feature
Feature: Product Search
  As a customer
  I want to search for products
  So that I can find items I want to purchase

  Background:
    Given the user is on the homepage
    And the search box is visible

  Scenario: Successful product search
    When the user enters "laptop" in the search box
    And clicks the search button
    Then results containing "laptop" should be displayed
    And at least one result should be shown

  Scenario: Search with no results
    When the user enters "xyz123nonexistent" in the search box
    And clicks the search button
    Then a "No results found" message should be displayed

  Scenario: Search preserves original case
    When the user enters "HEADPHONES" in the search box
    And clicks the search button
    Then results should be displayed
    And the search query should be preserved as "HEADPHONES"

// features/step-definitions/search.steps.ts
import { Given, When, Then, And } from '@cucumber/cucumber';
import { expect } from 'chai';
import { SearchPage } from '../../pages/search-page';

let searchPage: SearchPage;

Given('the user is on the homepage', async function () {
  searchPage = new SearchPage(this.app);
  await searchPage.navigateToHomepage();
});

And('the search box is visible', async function () {
  const isVisible = await searchPage.isSearchBoxVisible();
  expect(isVisible).to.be.true;
});

When('the user enters {string} in the search box', async function (query: string) {
  await searchPage.enterSearchQuery(query);
});

And('clicks the search button', async function () {
  await searchPage.clickSearchButton();
});

Then('results containing {string} should be displayed', async function (expectedTerm: string) {
  const results = await searchPage.getSearchResults();
  const hasMatchingResults = results.some(result => 
    result.toLowerCase().includes(expectedTerm.toLowerCase())
  );
  expect(hasMatchingResults).to.be.true;
});

And('at least one result should be shown', async function () {
  const results = await searchPage.getSearchResults();
  expect(results.length).to.be.greaterThan(0);
});

Then('a {string} message should be displayed', async function (expectedMessage: string) {
  const message = await searchPage.getNoResultsMessage();
  expect(message).to.equal(expectedMessage);
});

And('the search query should be preserved as {string}', async function (expectedQuery: string) {
  const currentQuery = await searchPage.getCurrentSearchQuery();
  expect(currentQuery).to.equal(expectedQuery);
});

// features/support/hooks.ts
import { Before, After, ITestCaseHookParameter } from '@cucumber/cucumber';
import { chromium } from 'playwright';

Before(async function (scenario: ITestCaseHookParameter) {
  this.browser = await chromium.launch({ headless: true });
  this.context = await this.browser.newContext();
  this.app = await this.context.newPage();
  console.log(`Starting scenario: ${scenario.pickle.name}`);
});

After(async function (scenario: ITestCaseHookParameter) {
  if (this.app) {
    await this.app.close();
  }
  if (this.context) {
    await this.context.close();
  }
  if (this.browser) {
    await this.browser.close();
  }
  console.log(`Finished scenario: ${scenario.pickle.name}`);
});

// features/support/world.ts
import { IWorldOptions, World, setWorldConstructor } from '@cucumber/cucumber';
import { Browser, Page, BrowserContext } from 'playwright';

export class CustomWorld extends World {
  constructor(options: IWorldOptions) {
    super(options);
    this.browser = undefined as unknown as Browser;
    this.context = undefined as unknown as BrowserContext;
    this.app = undefined as unknown as Page;
  }
}

setWorldConstructor(CustomWorld);

// cucumber.js configuration
// {
//   "default": {
//     "paths": ["features/**/*.feature"],
//     "require": ["features/step-definitions/**/*.ts", "features/support/**/*.ts"],
//     "format": ["progress-bar", "summary"],
//     "worldParameters": {}
//   }
// }
```

---

## BDD with React and Testing Library

```typescript
// features/todo.feature
Feature: Todo List Management
  As a todo list user
  I want to manage my tasks
  So that I can keep track of what I need to do

  Background:
    Given I am on the todo list page

  Scenario: Adding a new todo item
    When I enter "Buy groceries" in the input field
    And I click the add button
    Then "Buy groceries" should appear in the todo list

  Scenario: Completing a todo item
    Given I have added "Buy groceries" to my todo list
    When I click the checkbox next to "Buy groceries"
    Then "Buy groceries" should be marked as completed

  Scenario: Deleting a todo item
    Given I have added "Buy groceries" to my todo list
    When I click the delete button next to "Buy groceries"
    Then "Buy groceries" should not appear in the todo list

  Scenario: Filtering completed todos
    Given I have added the following items:
      | item            |
      | Buy groceries   |
      | Walk the dog    |
    And I have completed "Buy groceries"
    When I click the "Completed" filter
    Then I should see only "Buy groceries" in the list

// features/step-definitions/todo.steps.ts
import { Given, When, Then, And } from '@cucumber/cucumber';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { expect } from 'chai';
import React from 'react';
import { TodoApp } from '../../../src/todo/todo-app';

let container: HTMLElement;

Given('I am on the todo list page', () => {
  const { container: renderedContainer } = render(<TodoApp />);
  container = renderedContainer;
});

When('I enter {string} in the input field', (taskName: string) => {
  const input = screen.getByPlaceholderText('What needs to be done?');
  fireEvent.change(input, { target: { value: taskName } });
});

And('I click the add button', () => {
  const addButton = screen.getByText('Add');
  fireEvent.click(addButton);
});

Then('{string} should appear in the todo list', async (taskName: string) => {
  await waitFor(() => {
    expect(screen.getByText(taskName)).toBeInTheDocument();
  });
});

Given('I have added {string} to my todo list', (taskName: string) => {
  const input = screen.getByPlaceholderText('What needs to be done?');
  fireEvent.change(input, { target: { value: taskName } });
  const addButton = screen.getByText('Add');
  fireEvent.click(addButton);
  expect(screen.getByText(taskName)).toBeInTheDocument();
});

When('I click the checkbox next to {string}', (taskName: string) => {
  const task = screen.getByText(taskName);
  const checkbox = task.closest('.todo-item')?.querySelector('input[type="checkbox"]');
  fireEvent.click(checkbox!);
});

Then('{string} should be marked as completed', (taskName: string) => {
  const task = screen.getByText(taskName);
  expect(task.closest('.todo-item')).toHaveClass('completed');
});

When('I click the delete button next to {string}', (taskName: string) => {
  const task = screen.getByText(taskName);
  const deleteButton = task.closest('.todo-item')?.querySelector('.delete-btn');
  fireEvent.click(deleteButton!);
});

Then('{string} should not appear in the todo list', (taskName: string) => {
  expect(screen.queryByText(taskName)).not.toBeInTheDocument();
});

Given('I have added the following items:', async (dataTable: any) => {
  const items = dataTable.hashes();
  for (const row of items) {
    const input = screen.getByPlaceholderText('What needs to be done?');
    fireEvent.change(input, { target: { value: row.item } });
    const addButton = screen.getByText('Add');
    fireEvent.click(addButton);
  }
});

And('I have completed {string}', (taskName: string) => {
  const task = screen.getByText(taskName);
  const checkbox = task.closest('.todo-item')?.querySelector('input[type="checkbox"]');
  fireEvent.click(checkbox!);
});

When('I click the {string} filter', (filterName: string) => {
  const filterButton = screen.getByText(filterName);
  fireEvent.click(filterButton);
});

Then('I should see only {string} in the list', (taskName: string) => {
  const visibleItems = screen.getAllByRole('listitem');
  expect(visibleItems.length).to.equal(1);
  expect(visibleItems[0]).toHaveTextContent(taskName);
});

// src/todo/todo-app.tsx
import React, { useState } from 'react';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

export const TodoApp: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [inputValue, setInputValue] = useState('');
  const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('all');

  const addTodo = (text: string) => {
    const newTodo: Todo = {
      id: Date.now().toString(),
      text,
      completed: false,
    };
    setTodos([...todos, newTodo]);
  };

  const toggleTodo = (id: string) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const deleteTodo = (id: string) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  const filteredTodos = todos.filter((todo) => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <div className="todo-app">
      <h1>Todo List</h1>
      <div className="input-section">
        <input
          type="text"
          placeholder="What needs to be done?"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
        />
        <button onClick={() => addTodo(inputValue)}>Add</button>
      </div>
      <ul className="todo-list">
        {filteredTodos.map((todo) => (
          <li key={todo.id} className={`todo-item ${todo.completed ? 'completed' : ''}`}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span>{todo.text}</span>
            <button className="delete-btn" onClick={() => deleteTodo(todo.id)}>
              Delete
            </button>
          </li>
        ))}
      </ul>
      <div className="filters">
        <button onClick={() => setFilter('all')}>All</button>
        <button onClick={() => setFilter('active')}>Active</button>
        <button onClick={() => setFilter('completed')}>Completed</button>
      </div>
    </div>
  );
};
```

---

## BDD with API Testing

```typescript
// features/api/booking.feature
Feature: Hotel Booking API
  As a hotel booking system
  I want to provide booking functionality
  So that customers can make reservations

  Background:
    Given the booking API is available
    And I have a valid API key

  Scenario: Create a new booking
    When I send a POST request to /bookings with:
      | field         | value                    |
      | customerName  | John Doe                 |
      | roomNumber    | 101                      |
      | checkInDate   | 2024-03-15               |
      | checkOutDate  | 2024-03-20               |
    Then the response status should be 201
    And the response should contain a bookingId
    And the booking should be confirmed

  Scenario: Retrieve booking details
    Given a booking exists with id "BK-12345"
    When I send a GET request to /bookings/BK-12345
    Then the response status should be 200
    And the response should contain the booking details

  Scenario: Update booking
    Given a booking exists with id "BK-12345"
    When I send a PUT request to /bookings/BK-12345 with:
      | field         | value                    |
      | roomNumber    | 102                      |
    Then the response status should be 200
    And the booking should have room number 102

  Scenario: Cancel booking
    Given a booking exists with id "BK-12345"
    When I send a DELETE request to /bookings/BK-12345
    Then the response status should be 200
    And the booking should be marked as cancelled

  Scenario: List all bookings
    Given multiple bookings exist
    When I send a GET request to /bookings
    Then the response status should be 200
    And the response should contain a list of all bookings

// features/step-definitions/api.steps.ts
import { Given, When, Then, And } from '@cucumber/cucumber';
import { expect } from 'chai';
import axios, { AxiosInstance } from 'axios';

let apiClient: AxiosInstance;
let lastResponse: any;
let bookingId: string;

Given('the booking API is available', async function () {
  apiClient = axios.create({
    baseURL: 'https://api.hotelbooking.com/v1',
    timeout: 10000,
    headers: {
      'Content-Type': 'application/json',
    },
  });
});

And('I have a valid API key', async function () {
  apiClient.defaults.headers.common['Authorization'] = `Bearer ${this.apiKey}`;
});

When('I send a POST request to {string} with:', async function (
  endpoint: string,
  dataTable: any
) {
  const data: Record<string, any> = {};
  dataTable.hashes().forEach((row: any) => {
    data[row.field] = row.value;
  });
  lastResponse = await apiClient.post(endpoint, data);
});

And('the response should contain a bookingId', function () {
  expect(lastResponse.data).to.have.property('bookingId');
  bookingId = lastResponse.data.bookingId;
});

And('the booking should be confirmed', function () {
  expect(lastResponse.data.status).to.equal('confirmed');
});

Given('a booking exists with id {string}', async function (id: string) {
  bookingId = id;
  const response = await apiClient.get(`/bookings/${id}`);
  expect(response.status).to.equal(200);
  bookingId = response.data.bookingId;
});

When('I send a GET request to {string}', async function (endpoint: string) {
  lastResponse = await apiClient.get(endpoint);
});

Then('the response status should be {int}', function (status: number) {
  expect(lastResponse.status).to.equal(status);
});

And('the response should contain the booking details', function () {
  expect(lastResponse.data).to.have.property('bookingId');
  expect(lastResponse.data).to.have.property('customerName');
});

When('I send a PUT request to {string} with:', async function (
  endpoint: string,
  dataTable: any
) {
  const data: Record<string, any> = {};
  dataTable.hashes().forEach((row: any) => {
    data[row.field] = row.value;
  });
  lastResponse = await apiClient.put(endpoint, data);
});

And('the booking should have room number {string}', function (roomNumber: string) {
  expect(lastResponse.data.roomNumber).to.equal(roomNumber);
});

When('I send a DELETE request to {string}', async function (endpoint: string) {
  lastResponse = await apiClient.delete(endpoint);
});

And('the booking should be marked as cancelled', function () {
  expect(lastResponse.data.status).to.equal('cancelled');
});

Given('multiple bookings exist', async function () {
  // Create multiple bookings for test data
  for (let i = 0; i < 3; i++) {
    await apiClient.post('/bookings', {
      customerName: `Customer ${i}`,
      roomNumber: `10${i}`,
      checkInDate: '2024-03-15',
      checkOutDate: '2024-03-20',
    });
  }
});

When('I send a GET request to {string}', async function (endpoint: string) {
  lastResponse = await apiClient.get(endpoint);
});

And('the response should contain a list of all bookings', function () {
  expect(lastResponse.data).to.be.an('array');
  expect(lastResponse.data.length).to.be.greaterThan(0);
});

// cucumber.js
// {
//   "default": {
//     "paths": ["features/**/*.feature"],
//     "require": ["features/step-definitions/**/*.ts"],
//     "format": ["progress-bar", "html:reports/cucumber-report.html"],
//     "worldParameters": {
//       "apiKey": "test-api-key-12345"
//     }
//   }
// }
```

---

## BDD Best Practices

### 1. Write Scenarios from the User's Perspective
Focus on what the user wants to achieve, not on technical implementation details. Use user-focused language and avoid exposing internal mechanics in scenarios.

### 2. Keep Scenarios Short and Focused
Each scenario should test one specific behavior. If you find yourself with many steps, consider splitting the scenario into multiple smaller scenarios.

### 3. Use Concrete Examples
Avoid abstract descriptions. Use specific, realistic examples that demonstrate the expected behavior. This makes the scenarios more meaningful and easier to understand.

### 4. Avoid Technical Language in Scenarios
Business stakeholders should be able to read and understand the scenarios without technical knowledge. Keep the language business-oriented.

### 5. Use Background Wisely
The Background section is useful for setting up common preconditions, but avoid overusing it. If a precondition is specific to a scenario, keep it in the Scenario.

### 6. Use Scenario Outlines for Data-Driven Tests
When you have multiple similar scenarios with different data, use Scenario Outline with Examples tables instead of repeating similar scenarios.

### 7. Maintain the Single Source of Truth
BDD scenarios should be the authoritative specification. All development should trace back to scenarios, and scenarios should be kept up to date.

### 8. Involve All Stakeholders
BDD requires collaboration between developers, testers, business analysts, and product owners. Ensure all perspectives are included in the discovery process.

### 9. Treat Scenarios as Living Documentation
Scenarios should be maintained and updated as the system evolves. They serve as both tests and documentation, so keep them accurate.

### 10. Don't Test Implementation in Scenarios
Focus on observable behavior, not on how something is implemented. This allows for refactoring without breaking scenarios.

---

## BDD Anti-Patterns

### 1. Technical Scenarios
Scenarios that describe implementation details instead of user behavior make BDD less effective for communication with business stakeholders.

### 2. Scenarios as Test Scripts
Writing scenarios that are simply recorded steps without business value defeats the purpose of BDD as a specification tool.

### 3. Missing the "Then" Part
Scenarios must have assertions (Then steps) that verify the expected outcome. Without them, scenarios don't validate behavior.

### 4. Overly Complex Step Definitions
If step definitions are complex, it indicates that either the scenario is too detailed or the implementation needs better abstractions.

### 5. Ignoring Failing Scenarios
Failing scenarios should be treated as bugs in the specification or implementation. Ignoring them leads to technical debt.

### 6. Gherkin as Just Another Test Framework
Using Gherkin without the collaborative discovery process misses the main value of BDD. BDD is about communication, not just automation.

### 7. Scenario Proliferation
Having too many scenarios for simple functionality indicates that scenarios may be testing implementation details rather than behavior.

### 8. No Business Involvement
If business stakeholders don't participate in writing or reviewing scenarios, BDD loses its effectiveness as a communication tool.

### 9. Hard-Coded Data in Scenarios
Using specific values that don't represent realistic scenarios makes tests less meaningful. Use representative example data.

### 10. Scenarios Without Acceptance Criteria
Vague scenarios that don't clearly define what constitutes success lead to misunderstanding and rework.

---

## BDD in AI-Assisted Development

When working with AI assistants for BDD, follow these practices:

### 1. Use AI for Scenario Generation
Provide the AI with user stories or requirements and ask it to generate Gherkin scenarios. Review and refine the scenarios for accuracy.

### 2. AI as a BDD Coach
Ask the AI to review your Gherkin scenarios for clarity, completeness, and adherence to BDD best practices.

### 3. Automate Step Definition Generation
After writing scenarios, ask the AI to generate the initial step definition stubs that you can then implement.

### 4. Combine with Example Mapping
Use AI to help facilitate Example Mapping sessions by generating examples and scenarios from natural language requirements.

### 5. AI for Edge Case Discovery
Ask the AI to identify edge cases and unusual scenarios that you might have missed in your BDD scenarios.

### 6. Translate Domain Knowledge
Use AI to help translate domain-specific language and business rules into clear, executable Gherkin scenarios.

### 7. Maintain Consistency
Use AI to ensure consistent terminology and structure across all scenarios in a feature file.

### 8. Documentation Generation
Ask AI to generate human-readable documentation from Gherkin scenarios for stakeholders who prefer plain English.

---

## BDD Framework Comparison

| Framework | Language | Key Features | Best For |
|-----------|----------|--------------|----------|
| **Cucumber** | Multi-language | Large community, extensive integrations | Cross-team collaboration |
| **SpecFlow** | C#/.NET | Visual Studio integration, SpecFlow+ | .NET ecosystems |
| **Behave** | Python | Simple setup, Pythonic | Python teams |
| **Jest BDD** | JavaScript | Integrated with Jest, great for React | JavaScript/TypeScript |
| **Gauge** | Multi-language | Multi-modal, Markdown-based | Rapid test development |
| **Concordion** | Java | HTML-based specifications | Java teams wanting rich reports |

---

## Living Documentation

### Concept
BDD scenarios serve as both tests and documentation. This "living documentation" stays in sync with the codebase and provides up-to-date specifications.

### Benefits

| Benefit | Description |
|---------|-------------|
| Single source of truth | Scenarios define expected behavior |
| Always current | Tests prove documentation is accurate |
| Stakeholder accessible | Non-technical users can read scenarios |
| Automated regression | Scenarios catch regressions automatically |

### Maintaining Living Documentation

```gherkin
Feature: Order Processing
  As a customer
  I want to place orders
  So that I can purchase products

  # This scenario is both a test AND documentation
  Scenario: Customer places a valid order
    Given the product "Laptop" is in stock
    When I place an order for 1 "Laptop"
    Then the order status should be "confirmed"
    And I should receive an order confirmation email

  # Document business rules explicitly
  Rule: Orders cannot exceed available stock
    Scenario: Order exceeds available stock
      Given "Laptop" has 5 units in stock
      When I place an order for 10 "Laptop" units
      Then the order should be rejected
      And I should see an "insufficient stock" message
```

---

## Example Mapping with BDD

### Process
Example Mapping is a collaborative technique for discovering examples that become BDD scenarios.

```
┌─────────────────────────────────────────────────────────────┐
│                    EXAMPLE MAPPING SESSION                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   RULE      │  │  EXAMPLE    │  │  QUESTION   │          │
│  │             │  │             │  │             │          │
│  │ Business    │  │ Concrete    │  │ Needs       │          │
│  │ rule or     │  │ instance    │  │ clarification│         │
│  │ constraint  │  │ of the rule │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│       │                │                 │                   │
│       ▼                ▼                 ▼                   │
│  ┌─────────────────────────────────────────────┐            │
│  │            USER STORY                        │            │
│  │  "As a customer, I want to track orders     │            │
│  │   so that I can know when they'll arrive"   │            │
│  └─────────────────────────────────────────────┘            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Example Mapping Output to BDD

| Example Map Element | BDD Equivalent |
|---------------------|----------------|
| Rule | Feature/Background |
| Example | Scenario |
| Counter-example | Negative Scenario |
| Question | Clarified in Background or Rules |

---

## AI-Assisted BDD Best Practices

### 1. AI for Scenario Generation

Provide the AI with user stories and ask for Gherkin scenarios:

```
User Story:
As an online shopper
I want to see estimated delivery dates
So that I can plan my purchases

AI generates:
Feature: Delivery Date Estimation
  As an online shopper
  I want to see estimated delivery dates
  So that I can plan my purchases

  Scenario: Standard delivery estimation
    Given I am viewing a product available for shipping
    When I check the product page
    Then I should see an estimated delivery date
    And the date should be 3-5 business days from today

  Scenario: Express delivery option
    Given I am viewing a product with express shipping
    When I select express shipping
    Then I should see an estimated delivery date
    And the date should be 1-2 business days from today
```

### 2. AI for Edge Case Discovery

Ask AI to identify missing scenarios:

```
Current scenarios:
1. Valid order placement
2. Order with insufficient stock
3. Order with invalid payment

AI identifies missing:
4. Order during promotional period
5. Order with partially available items
6. Order with shipping address change
7. Order cancellation before processing
```

### 3. AI for Step Definition Generation

After writing scenarios, AI generates initial step definitions:

```typescript
// AI-generated step definitions (stub)
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from 'chai';
import { ShoppingCart } from '../../src/cart/shopping-cart';

let cart: ShoppingCart;

Given('I have an empty shopping cart', () => {
    cart = new ShoppingCart();
});

When('I add {string} to the cart', async function (productName: string) {
    // TODO: Implement step definition
    // HINT: Use ProductCatalog to find product by name
});

Then('the cart should contain {int} item(s)', async function (itemCount: number) {
    // TODO: Implement step definition
    // HINT: Check cart.getItems().length
});
```

### 4. AI for BDD Code Review

Use AI to review Gherkin scenarios for quality:

| Check | AI Review Focus |
|-------|-----------------|
| Clarity | Are scenarios understandable by non-technical stakeholders? |
| Completeness | Are all acceptance criteria covered? |
| Independence | Can scenarios run in any order? |
| Traceability | Do scenarios map to user stories? |

---

## References and Further Reading

1. North, Dan. "Introducing BDD." Dan North Blog, 2006.
2. Adzic, Gojko. "Specification by Example: How Successful Teams Deliver the Right Software." Manning, 2011.
3. Wynne, Matt, and Aslak Hellesøy. "The Cucumber Book: Behaviour-Driven Development for Testers and Developers." Pragmatic Programmers, 2012.
4. "Gherkin Syntax Reference." Cucumber Documentation.
5. "Behaviour-Driven Development." Agile Alliance.
6. Smart, John Ferguson. "BDD in Action: Behavior-driven development for the whole software lifecycle." Manning, 2014.
7. AI Coding Exercise Repository - BDD and Specification Patterns (internal reference)

---

## BDD Checklist

### Before Writing Scenarios

- [ ] User story has clear acceptance criteria
- [ ] Stakeholders have provided examples
- [ ] Edge cases have been identified
- [ ] Technical approach is understood

### While Writing Scenarios

- [ ] Scenarios use Given-When-Then structure
- [ ] Language is business-focused, not technical
- [ ] Each scenario tests one behavior
- [ ] Scenario names are descriptive
- [ ] Examples use realistic data

### After Writing Scenarios

- [ ] Stakeholders have reviewed scenarios
- [ ] Scenarios are linked to requirements
- [ ] Step definitions can be implemented
- [ ] Automated tests will cover scenarios
