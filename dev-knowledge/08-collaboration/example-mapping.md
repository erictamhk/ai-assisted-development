# Example Mapping

## Core Concept

Example Mapping is a collaborative technique for discovering and documenting examples that illustrate system behavior. Created by Seb Rose, Example Mapping helps teams have structured conversations about requirements by using concrete examples to explore and clarify domain rules.

The technique addresses the common problem of vague or ambiguous requirements that lead to misunderstandings between business stakeholders and development teams. Instead of abstractly discussing "what the system should do," Example Mapping uses specific, tangible examples to ground the conversation in reality.

Example Mapping is particularly valuable in the context of BDD (Behavior-Driven Development) and specification by example. It provides a lightweight framework for teams to collaboratively explore and document the examples that will become executable specifications.

The core insight of Example Mapping is that examples are more effective than abstract descriptions for communicating complex business rules. By working through concrete scenarios, teams can quickly identify gaps, edge cases, and misunderstandings before they become expensive problems in development.

---

## Example Mapping Structure

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    EXAMPLE MAPPING COMPONENTS                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                        CARD TYPES                               │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │    RULE     │  │   EXAMPLE   │  │   QUESTION  │  │    ISSUE    │    │
│  │             │  │             │  │             │  │             │    │
│  │  Business  │  │  Concrete   │  │  Unanswered │  │  Blockers   │    │
│  │  Rule or   │  │  instance   │  │  questions  │  │  or risks   │    │
│  │  Constraint│  │  of a rule  │  │  that need  │  │  that need  │    │
│  │             │  │             │  │   answers   │  │   answers   │    │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
│      (Blue)          (Green)          (Yellow)          (Red)          │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                      SESSION STRUCTURE                           │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────┐      │
│  │  ┌─────────────────────────────────────────────────────────┐  │      │
│  │  │  RULE CARD: Business rule being explored                │  │      │
│  │  │                                                          │  │      │
│  │  │  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐               │  │      │
│  │  │  │ Ex 1  │ │ Ex 2  │ │ Ex 3  │ │ Ex 4  │ ...          │  │      │
│  │  │  └───────┘ └───────┘ └───────┘ └───────┘               │  │      │
│  │  │                                                          │  │      │
│  │  │  ┌───────┐ ┌───────┐                                   │  │      │
│  │  │  │ Q 1   │ │ Q 2   │ ...  Questions needing answers    │  │      │
│  │  │  └───────┘ └───────┘                                   │  │      │
│  │  │                                                          │  │      │
│  │  │  ┌───────┐                                             │  │      │
│  │  │  │ I 1   │ ...  Issues requiring resolution            │  │      │
│  │  │  └───────┘                                             │  │      │
│  │  └─────────────────────────────────────────────────────────┘  │      │
│  └───────────────────────────────────────────────────────────────┘      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Rule Cards (Blue)
Rule cards represent business rules or constraints that the system must enforce. Each rule card should be a single, atomic business rule that can be illustrated with multiple examples.

### Example Cards (Green)
Example cards represent concrete instances that demonstrate how the rule should work. Each example should be a specific scenario that can be used as the basis for an executable specification.

### Question Cards (Yellow)
Question cards represent topics that need further investigation. These might be clarifications needed from business stakeholders or technical questions for the development team.

### Issue Cards (Red)
Issue cards represent blockers or risks that prevent the team from proceeding. These might be dependencies, policy decisions, or architectural constraints that need resolution.

---

## Example Mapping Session Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   EXAMPLE MAPPING WORKFLOW                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│  │   PICK A    │───►│  DISCUSS    │───►│   CREATE    │                 │
│  │   STORY     │    │  THE RULE   │    │   RULE      │                 │
│   │             │    │             │    │   CARD      │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│         │                  │                  │                         │
│         ▼                  ▼                  ▼                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│   │  Read the   │    │  What does  │    │  Write a    │                 │
│   │  card and   │    │  this mean? │    │  concise    │                 │
│   │  clarify    │    │  What are   │    │  rule card  │                 │
│   │  scope      │    │  edge cases?│    │  on blue    │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│                                                                     │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│   │  ADD GOOD   │───►│  ADD        │───►│  ADDRESS    │                 │
│   │  EXAMPLES   │    │  QUESTIONS  │    │  ISSUES     │                 │
│   │             │    │  & ISSUES   │    │             │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│         │                  │                  │                         │
│         ▼                  ▼                  ▼                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│   │  Create     │    │  Write down │    │  Schedule   │                 │
│   │  concrete   │    │  unclear    │    │  follow-up  │                 │
│   │  examples   │    │  topics     │    │  discussions│                 │
│   │  on green   │    │  on yellow  │    │  on red     │                 │
│   └─────────────┘    └─────────────┘    └─────────────┘                 │
│                                                                     │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│   │  DISCUSS    │───►│   MARK      │───►│   REPEAT    │                 │
│   │  EDGE       │    │   SCOPE     │    │             │                 │
│   │  CASES      │    │             │    │             │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│         │                  │                  │                         │
│         ▼                  ▼                  ▼                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│   │  Explore    │    │  ✓ All      │    │  Pick next  │                 │
│   │  boundary   │    │  examples   │    │  story and  │                 │
│   │  conditions │    │  covered    │    │  start      │                 │
│   │  and what   │    │  ✓ No       │    │  again      │                 │
│   │  ifs        │    │  questions  │    │             │                 │
│   │             │    │  ✓ No       │    │             │                 │
│   │             │    │  issues     │    │             │                 │
│  └─────────────┘    └─────────────┘    └─────────────┘                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Step 1: Pick a Story
Select a user story or feature slice to explore. Read it aloud and clarify any ambiguences about scope.

### Step 2: Discuss the Rule
Have a focused discussion about what the story means. Identify the underlying business rule or rules.

### Step 3: Create a Rule Card
Write a concise rule card on blue paper or sticky note. This represents the business constraint being explored.

### Step 4: Add Examples
Create concrete examples on green cards that demonstrate the rule in various scenarios. Start with happy path, then add edge cases.

### Step 5: Add Questions and Issues
Write down any questions that need answers on yellow cards, and any issues or blockers on red cards.

### Step 6: Discuss Edge Cases
Explore boundary conditions and "what if" scenarios to ensure completeness.

### Step 7: Mark Scope
When all examples are covered, no questions remain, and no issues block progress, mark the story as ready for development.

---

## Example Mapping Example: E-Commerce Discounts

```
┌─────────────────────────────────────────────────────────────────────────┐
│              EXAMPLE MAPPING: DISCOUNT RULES                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  RULE CARD (Blue)                                               │    │
│  │                                                                 │    │
│  │  Customers with loyalty tier "Gold" or "Platinum" receive       │    │
│  │  a 15% discount on all purchases.                               │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│         │                                                           │
│         │  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐        │
│         └──│ Ex 1  │ │ Ex 2  │ │ Ex 3  │ │ Ex 4  │ │ Ex 5  │        │
│            └───────┘ └───────┘ └───────┘ └───────┘ └───────┘        │
│                                                                         │
│  EXAMPLES (Green):                                                      │
│  ─────────────────────────────────────────────────────────────────────  │
│  │ Ex 1: Gold customer, $100 order → $15 discount, $85 total        │  │
│  │ Ex 2: Platinum customer, $50 order → $7.50 discount, $42.50 total│  │
│  │ Ex 3: Silver customer, $100 order → No discount, $100 total      │  │
│  │ Ex 4: No tier customer, $100 order → No discount, $100 total     │  │
│  │ Ex 5: Gold customer with $10 item → $1.50 discount, $8.50 total  │  │
│                                                                         │
│  QUESTIONS (Yellow):                                                    │
│  ─────────────────────────────────────────────────────────────────────  │
│  │ Q1: Does the discount apply to shipping costs?                   │  │
│  │ Q2: Can the discount be combined with promotional coupons?       │  │
│  │ Q3: Is the discount applied before or after tax?                 │  │
│                                                                         │
│  ISSUES (Red):                                                          │
│  ─────────────────────────────────────────────────────────────────────  │
│  │ I1: Need confirmation from Finance on tax calculation policy     │  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  RULE CARD (Blue)                                                       │
│                                                                         │
│  Discounts are capped at $50 per order.                                 │
│                                                                         │
│  ┌───────┐ ┌───────┐ ┌───────┐                                         │
│  │ Ex 1  │ │ Ex 2  │ │ Ex 3  │                                         │
│  └───────┘ └───────┘ └───────┘                                         │
│                                                                         │
│  EXAMPLES:                                                              │
│  - Gold customer with $500 order → $50 discount (cap applied)          │
│  - Platinum customer with $200 order → $30 discount                    │
│  - Customer with $300 order + $20 coupon → $50 max discount            │
│                                                                         │
│  QUESTIONS:                                                             │
│  - Q: Does the cap apply before or after coupon combination?           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Example Mapping: Order Shipping

```
┌─────────────────────────────────────────────────────────────────────────┐
│              EXAMPLE MAPPING: FREE SHIPPING RULES                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  RULE CARD:                                                             │
│  ────────────────────────────────────────────────────────────────────   │
│  Orders over $75 qualify for free standard shipping.                    │
│                                                                         │
│  EXAMPLES:                                                              │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │ ✓ Order total $75.01 → Free shipping                           │     │
│  │ ✓ Order total exactly $75.00 → Free shipping                   │     │
│  │ ✓ Order total $74.99 → Standard shipping ($5.99)               │     │
│  │ ✓ Order total $75.00 with $10 coupon → Shipping cost $5.99     │     │
│  │ ✓ Gold member order total $60 → Free shipping (tier benefit)   │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                         │
│  QUESTIONS:                                                             │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │ ? Does the $75 threshold include or exclude tax?               │     │
│  │ ? Is shipping calculated before or after promotions?           │     │
│  │ ? What's the cutoff time for same-day shipping?                │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                         │
│  ISSUES:                                                                │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │ ! Need to confirm shipping cost model with logistics team      │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Example Mapping in TypeScript

```typescript
// example-mapping.types.ts

interface RuleCard {
  id: string;
  title: string;
  description: string;
  storyId: string;
  examples: ExampleCard[];
  questions: QuestionCard[];
  issues: IssueCard[];
}

interface ExampleCard {
  id: string;
  ruleId: string;
  description: string;
  given: string[];
  when: string[];
  then: string[];
  tags: string[];
  status: 'agreed' | 'pending' | 'disagreed';
}

interface QuestionCard {
  id: string;
  ruleId: string;
  question: string;
  askedBy: string;
  status: 'open' | 'answered' | 'deferred';
  answer?: string;
}

interface IssueCard {
  id: string;
  ruleId: string;
  description: string;
  severity: 'blocker' | 'major' | 'minor';
  owner?: string;
  status: 'open' | 'in-progress' | 'resolved';
}

interface Session {
  id: string;
  date: Date;
  participants: string[];
  stories: string[];
  rules: RuleCard[];
}

class ExampleMappingSession {
  private session: Session;
  private rules: Map<string, RuleCard>;

  constructor(participants: string[], storyIds: string[]) {
    this.session = {
      id: `session-${Date.now()}`,
      date: new Date(),
      participants,
      stories: storyIds,
      rules: [],
    };
    this.rules = new Map();
  }

  addRule(storyId: string, title: string, description: string): RuleCard {
    const rule: RuleCard = {
      id: `rule-${Date.now()}`,
      title,
      description,
      storyId,
      examples: [],
      questions: [],
      issues: [],
    };
    this.rules.set(rule.id, rule);
    return rule;
  }

  addExample(
    ruleId: string,
    description: string,
    given: string[],
    when: string[],
    then: string[],
    tags: string[] = []
  ): ExampleCard {
    const rule = this.rules.get(ruleId);
    if (!rule) throw new Error('Rule not found');

    const example: ExampleCard = {
      id: `example-${Date.now()}`,
      ruleId,
      description,
      given,
      when,
      then,
      tags,
      status: 'agreed',
    };
    rule.examples.push(example);
    return example;
  }

  addQuestion(
    ruleId: string,
    question: string,
    askedBy: string
  ): QuestionCard {
    const rule = this.rules.get(ruleId);
    if (!rule) throw new Error('Rule not found');

    const q: QuestionCard = {
      id: `question-${Date.now()}`,
      ruleId,
      question,
      askedBy,
      status: 'open',
    };
    rule.questions.push(q);
    return q;
  }

  addIssue(
    ruleId: string,
    description: string,
    severity: 'blocker' | 'major' | 'minor'
  ): IssueCard {
    const rule = this.rules.get(ruleId);
    if (!rule) throw new Error('Rule not found');

    const issue: IssueCard = {
      id: `issue-${Date.now()}`,
      ruleId,
      description,
      severity,
      status: 'open',
    };
    rule.issues.push(issue);
    return issue;
  }

  generateGherkin(): string {
    let gherkin = '';

    for (const rule of this.rules.values()) {
      gherkin += `Feature: ${rule.title}\n\n`;
      gherkin += `  ${rule.description}\n\n`;

      for (const example of rule.examples) {
        gherkin += `  Scenario: ${example.description}\n`;
        example.given.forEach((g) => (gherkin += `    Given ${g}\n`));
        example.when.forEach((w) => (gherkin += `    When ${w}\n`));
        example.then.forEach((t) => (gherkin += `    Then ${t}\n`));
        gherkin += '\n';
      }
    }

    return gherkin;
  }

  getSessionSummary(): object {
    const rules = Array.from(this.rules.values());
    return {
      sessionId: this.session.id,
      date: this.session.date,
      participants: this.session.participants,
      stories: this.session.stories,
      summary: {
        totalRules: rules.length,
        totalExamples: rules.reduce((acc, r) => acc + r.examples.length, 0),
        totalQuestions: rules.reduce((acc, r) => acc + r.questions.length, 0),
        totalIssues: rules.reduce((acc, r) => acc + r.issues.length, 0),
        openIssues: rules.reduce(
          (acc, r) => acc + r.issues.filter((i) => i.status === 'open').length,
          0
        ),
      },
    };
  }
}

// Usage Example
const session = new ExampleMappingSession(
  ['Product Owner', 'Tech Lead', 'QA Lead', 'Developer'],
  ['US-123', 'US-124', 'US-125']
);

const discountRule = session.addRule(
  'US-123',
  'Loyalty Discount',
  'Gold and Platinum customers receive 15% discount on all purchases'
);

session.addExample(
  discountRule.id,
  'Gold customer gets 15% discount',
  ['customer is logged in with Gold tier'],
  ['customer adds $100 item to cart'],
  ['discount of $15 is applied', 'final total is $85']
);

session.addExample(
  discountRule.id,
  'Silver customer does not get discount',
  ['customer is logged in with Silver tier'],
  ['customer adds $100 item to cart'],
  ['no discount is applied', 'final total is $100']
);

session.addQuestion(
  discountRule.id,
  'Does the discount apply to shipping costs?',
  'Tech Lead'
);

session.addIssue(
  discountRule.id,
  'Need confirmation from Finance on tax calculation',
  'blocker'
);

console.log(session.generateGherkin());
// Outputs Gherkin feature file content
```

---

## Example Mapping Best Practices

### 1. Timebox Each Story
Limit the time spent on each story (typically 5-10 minutes). If you can't reach agreement quickly, mark questions and issues and move on.

### 2. Use Physical Materials
Color-coded cards (blue for rules, green for examples, yellow for questions, red for issues) make the mapping process visual and accessible.

### 3. Involve the Right People
Include at least one business stakeholder who can make decisions, a developer who will implement, and a tester who can identify edge cases.

### 4. One Rule Per Card
Keep each rule card focused on a single business rule. Complex rules should be split into multiple simpler rules.

### 5. Start with Happy Path
Begin with a clear example of the expected behavior before exploring edge cases and exceptions.

### 6. Explicitly Capture Questions
Don't skip over ambiguities. Write them down as question cards to be addressed later.

### 7. Use the Examples for Gherkin
The examples created during Example Mapping can be directly converted into Gherkin scenarios for BDD frameworks.

### 8. Mark Scope Clearly
When a rule and its examples are complete, mark the scope clearly so everyone knows which stories are ready for development.

### 9. Keep Sessions Short
Example Mapping sessions should be 60-90 minutes maximum. Take breaks to maintain focus and energy.

### 10. Review and Refine
After the session, review the generated examples and rules to ensure accuracy before proceeding to implementation.

---

## Example Mapping Anti-Patterns

### 1. Infinite Examples
Adding examples endlessly without finishing. Set a limit and trust that the pattern is understood.

### 2. Missing Business Stakeholders
Running sessions without business decision-makers leads to questions that can't be answered and delays.

### 3. Too Many Rules at Once
Trying to map complex features with many rules at once overwhelms participants. Start with smaller slices.

### 4. Skipping Edge Cases
Focusing only on happy path examples misses important boundary conditions that cause defects.

### 5. Ignoring Questions
Letting question cards pile up without answers leads to implementation based on assumptions.

### 6. Technical Discussions
Example Mapping is about behavior, not implementation. Technical debates should be deferred to separate sessions.

### 7. No Follow-Up
Not scheduling time to address questions and issues leads to blocked work and rework.

### 8. Single Facilitator
Having one person control the session limits participation. Rotate facilitation to keep energy high.

### 9. Skipping the Rule Card
Jumping straight to examples without first articulating the underlying rule leads to inconsistent examples.

### 10. Not Timeboxing
Letting discussions run too long reduces the number of stories covered and tires participants.

---

## Example Mapping in AI-Assisted Development

When working with AI assistants for Example Mapping, follow these practices:

### 1. Generate Initial Examples
Provide the AI with a rule description and ask it to generate potential examples, including edge cases you might have missed.

### 2. Structure the Session
Ask the AI to help structure an Example Mapping session by suggesting rules to explore and examples to consider.

### 3. Convert to Gherkin
After completing Example Mapping, use the AI to automatically convert examples into Gherkin scenarios.

### 4. Identify Gaps
Ask the AI to review your examples and identify missing scenarios or edge cases.

### 5. Facilitate Remotely
Use AI to help run virtual Example Mapping sessions by suggesting prompts and guiding the discussion.

### 6. Document the Session
Use AI to generate structured documentation from session notes and card contents.

### 7. Generate Test Cases
Transform examples directly into executable test cases for various testing frameworks.

### 8. Consistency Check
Ask AI to ensure consistent terminology and structure across all rules and examples.

---

## Example Mapping Tools

| Tool | Type | Best For |
|------|------|----------|
| **Miro** | Digital Whiteboard | Remote team collaboration |
| **Mural** | Visual Workspace | Async Example Mapping |
| **Trello** | Kanban Board | Tracking rules and examples |
| **Miro Example Mapping Template** | Template | Structured sessions |
| **Physical Stickies** | Physical | Co-located team sessions |
| **Notion** | Documentation | Session documentation |
| **Lucidchart** | Diagramming | Visual representation |

---

## Example Mapping in BDD Context

Example Mapping is a core practice in Behavior-Driven Development (BDD) that helps teams translate user stories into executable specifications. The connection between Example Mapping and BDD is direct: examples discovered during Example Mapping sessions become the Gherkin scenarios that drive BDD tests.

### Mapping Examples to Gherkin

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    EXAMPLE TO GHERKIN MAPPING                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐               │
│  │    RULE     │────►│    GIVEN    │────►│  Background │               │
│  │   CARD      │     │             │     │   SECTION   │               │
│  └─────────────┘     └─────────────┘     └─────────────┘               │
│                                                                         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐               │
│  │   EXAMPLE   │────►│ WHEN / THEN │────►│ Scenario    │               │
│  │   CARD      │     │             │     │   Outline   │               │
│  └─────────────┘     └─────────────┘     └─────────────┘               │
│                                                                         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐               │
│  │   DATA      │────►│    Examples │────►│   Data      │               │
│  │   TABLE     │     │   TABLE     │     │   Table     │               │
│  └─────────────┘     └─────────────┘     └─────────────┘               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Example Mapping with BDD Framework

```typescript
// After Example Mapping, convert examples to BDD tests
describe('User Registration Feature', () => {
  
  describe('Successful Registration', () => {
    it('should register user with valid email and password', async () => {
      // Example from Example Mapping session
      const result = await registerUser({
        email: 'newuser@example.com',
        password: 'SecurePass123!',
        name: 'John Doe'
      });

      expect(result.success).toBe(true);
      expect(result.user.email).toBe('newuser@example.com');
    });
  });

  describe('Registration Validation', () => {
    // Examples demonstrating validation rules
    it.each([
      { email: '', password: 'Valid123!', error: 'Email required' },
      { email: 'invalid', password: 'Valid123!', error: 'Invalid email' },
      { email: 'valid@example.com', password: 'short', error: 'Password too short' },
    ])('should reject $error', async ({ email, password, error }) => {
      const result = await registerUser({ email, password, name: 'Test' });
      expect(result.success).toBe(false);
      expect(result.error).toContain(error);
    });
  });
});
```

---

## Collaboration Best Practices

### 1. Include All Perspectives

Example Mapping sessions should include:
- **Product Owners/Business Analysts** - Clarify requirements and priorities
- **Developers** - Identify technical constraints and implementation details
- **Testers** - Uncover edge cases and validation scenarios
- **Domain Experts** - Validate business rules and terminology

### 2. Timebox Sessions

Keep Example Mapping sessions focused:
- **Duration**: 25-30 minutes per story
- **Rules per session**: 3-5 rules maximum
- **Examples per rule**: 3-5 positive examples, 2-3 negative examples

### 3. Document Outcomes

Always document the results of Example Mapping:
- **Acceptance Criteria**: Derived directly from examples
- **Known Edge Cases**: Explicitly documented
- **Open Questions**: Tracked for follow-up
- **Implementation Notes**: Technical insights captured

---

## References and Further Reading

1. Rose, Seb. "Introducing Example Mapping." Cucumber Blog, 2015.
2. Adzic, Gojko. "Specification by Example: How Successful Teams Deliver the Right Software." Manning, 2011.
3. Wynne, Matt, and Aslak Hellesøy. "The Cucumber Book: Behaviour-Driven Development for Testers and Developers." Pragmatic Programmers, 2012.
4. "Example Mapping Session Guide." Cucumber Ltd.
5. "BDD and Example Mapping in Practice." Agile Testing Days Conference.
6. ref/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md - BDD methodology
7. ref/engineering/specification_by_example/SPECIFICATION_BY_EXAMPLE.md - Specification by Example
8. doc/behavior-driven-development.md - BDD comprehensive guide
