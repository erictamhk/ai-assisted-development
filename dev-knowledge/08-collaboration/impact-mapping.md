# Impact Mapping

## Core Concept

Impact Mapping is a strategic planning technique that connects business goals to project scope through a hierarchical tree structure. Created by Gojko Adzic, Impact Mapping helps teams answer the fundamental question of "why are we building this?" by tracing the causal chain from organizational goals to specific deliverables.

The technique addresses a common problem in software projects: starting with a list of features or user stories without clearly understanding the underlying business objectives. This often leads to feature bloat, wasted effort, and projects that fail to deliver meaningful value even when delivered on time and within budget.

Impact Mapping provides a visual framework for exploring and documenting the connections between business goals, actors, impacts, and deliverables. By making these connections explicit, teams can make better-informed decisions about scope, prioritize more effectively, and avoid building features that don't contribute to organizational goals.

The key insight of Impact Mapping is that every feature or user story should trace back to a specific business goal, and every business goal should be supported by specific impacts from specific actors. If you cannot trace a feature through this chain, it should be questioned or eliminated.

---

## Impact Mapping Structure

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      IMPACT MAPPING TREE                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│                         ┌─────────────────┐                              │
│                         │      GOAL       │                              │
│                         │                 │                              │
│                         │  Why are we     │                              │
│                         │  doing this?    │                              │
│                         └────────┬────────┘                              │
│                                  │                                       │
│                                  ▼                                       │
│                         ┌─────────────────┐                              │
│                         │      ACTOR      │                              │
│                         │                 │                              │
│                         │   Who can       │                              │
│                         │   make it       │                              │
│                         │   happen?       │                              │
│                         └────────┬────────┘                              │
│                                  │                                       │
│                     ┌────────────┼────────────┐                          │
│                     ▼            ▼            ▼                          │
│              ┌───────────┐ ┌───────────┐ ┌───────────┐                  │
│              │  IMPACT   │ │  IMPACT   │ │  IMPACT   │                  │
│              │           │ │           │ │           │                  │
│              │  How can  │ │  How can  │ │  How can  │                  │
│              │  they help│ │  they help│ │  they help│                  │
│              │  achieve  │ │  achieve  │ │  achieve  │                  │
│              │  the goal?│ │  the goal?│ │  the goal?│                  │
│              └─────┬─────┘ └─────┬─────┘ └─────┬─────┘                  │
│                    │             │             │                         │
│                    ▼             ▼             ▼                         │
│              ┌───────────┐ ┌───────────┐ ┌───────────┐                  │
│              │ DELIVER-  │ │ DELIVER-  │ │ DELIVER-  │                  │
│              │   ABLE    │ │   ABLE    │ │   ABLE    │                  │
│              │           │ │           │ │           │                  │
│              │ What do   │ │ What do   │ │ What do   │                  │
│              │ we need   │ │ we need   │ │ we need   │                  │
│              │ to build? │ │ to build? │ │ to build? │                  │
│              └───────────┘ └───────────┘ └───────────┘                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Goal (Why)
The starting point of an impact map is a specific, measurable business goal. This should be something the organization wants to achieve, such as increasing revenue, reducing costs, improving customer satisfaction, or entering a new market.

### Actor (Who)
Actors are the individuals or groups who can help achieve the goal. Actors can be internal (employees, management) or external (customers, partners, competitors). Understanding who can influence the goal is crucial for identifying potential impacts.

### Impact (How)
Impacts describe how actors can help achieve the goal. These can be behavioral changes, new capabilities, or different ways of working. Impacts should be specific and measurable.

### Deliverable (What)
Deliverables are the concrete outputs needed to enable the desired impacts. These translate directly into features, user stories, or project tasks.

---

## Impact Mapping Process

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   IMPACT MAPPING WORKFLOW                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    WORKSHOP PHASES                              │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                │
│   │   DEFINE    │───►│  IDENTIFY   │───►│   EXPLORE   │                │
│   │   GOALS     │    │   ACTORS    │    │   IMPACTS   │                │
│   │             │    │             │    │             │                │
│   │  Start with │    │  Who can    │    │  How can    │                │
│   │  business   │    │  influence  │    │  they help? │                │
│   │  objectives │    │  outcomes?  │    │             │                │
│   └─────────────┘    └─────────────┘    └─────────────┘                │
│         │                  │                  │                         │
│         ▼                  ▼                  ▼                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                │
│   │    SMART    │    │  PERSONAS   │    │  SCENARIOS  │                │
│   │  GOALS      │    │  & ROLES    │    │  & STORIES  │                │
│   └─────────────┘    └─────────────┘    └─────────────┘                │
│                                                                     │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                │
│   │   MAP TO    │───►│  PRIORITIZE │───►│  ITERATE    │                │
│   │ DELIVERABLES│    │  & SCOPE    │    │  & VALIDATE │                │
│   │             │    │             │    │             │                │
│   │  What do    │    │  What has   │    │  Review &   │                │
│   │  we build?  │    │  most value?│    │  adjust     │                │
│   └─────────────┘    └─────────────┘    └─────────────┘                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Phase 1: Define Goals
Start by articulating the business goals that the project aims to achieve. Goals should be SMART: Specific, Measurable, Achievable, Relevant, and Time-bound. Avoid vague goals like "improve user experience" and instead aim for specific outcomes like "increase customer retention rate by 15% in Q4."

### Phase 2: Identify Actors
Identify all the actors who can influence the achievement of the goals. Consider internal actors (executives, employees, teams), external actors (customers, partners, competitors), and system actors (automated processes, external systems).

### Phase 3: Explore Impacts
For each actor, explore the specific impacts that would help achieve the goal. Think about how the actor's behavior or capabilities would need to change to contribute to the goal.

### Phase 4: Map to Deliverables
Translate the desired impacts into concrete deliverables. These deliverables should directly enable the impacts and should be traceable back to specific goals through the impact map.

### Phase 5: Prioritize and Scope
Use the impact map to prioritize deliverables based on their contribution to business goals. Eliminate or defer deliverables that don't clearly trace back to specific impacts and goals.

### Phase 6: Iterate and Validate
Review and validate the impact map regularly. As you learn more about the project, update the map to reflect new understanding and adjust scope accordingly.

---

## Impact Mapping Example: E-Commerce Platform

```markdown
# Impact Map: E-Commerce Platform Redesign

## Goal
**Increase online revenue by 25% within 12 months**

### Success Metrics
- Monthly revenue growth: +25% YoY
- Customer lifetime value: +20%
- Conversion rate: from 2.1% to 3.5%
- Average order value: from $45 to $55

## Actor: Customers
### Impacts
1. **Find products faster** - Reduce search time by 40%
   - Deliverable: Improved search with filters and suggestions
   - Deliverable: Enhanced product categorization

2. **Complete purchases easily** - Increase checkout completion rate by 30%
   - Deliverable: Streamlined one-page checkout
   - Deliverable: Multiple payment options (Apple Pay, Google Pay)
   - Deliverable: Guest checkout option

3. **Trust the platform** - Improve trust indicators
   - Deliverable: Display security badges and reviews
   - Deliverable: Transparent shipping costs and returns policy

4. **Receive personalized recommendations** - Increase average order value by 15%
   - Deliverable: AI-powered product recommendations
   - Deliverable: "Frequently bought together" bundles

## Actor: Marketing Team
### Impacts
1. **Launch campaigns quickly** - Reduce campaign setup time by 50%
   - Deliverable: Campaign management dashboard
   - Deliverable: A/B testing framework

2. **Target effectively** - Improve campaign ROI by 20%
   - Deliverable: Customer segmentation tool
   - Deliverable: Behavioral analytics dashboard

3. **Measure performance in real-time** - Access to live metrics
   - Deliverable: Real-time analytics reporting
   - Deliverable: Custom report builder

## Actor: Customer Support Team
### Impacts
1. **Resolve issues faster** - Reduce average handling time by 30%
   - Deliverable: Integrated CRM with order history
   - Deliverable: Chatbot for common queries

2. **Access customer data easily** - Single view of customer
   - Deliverable: Unified customer profile
   - Deliverable: Order tracking integration

## Actor: Warehouse Team
### Impacts
1. **Process orders efficiently** - Increase daily processing capacity by 40%
   - Deliverable: Mobile order picking app
   - Deliverable: Barcode scanning integration

2. **Manage inventory accurately** - Reduce stock discrepancies by 50%
   - Deliverable: Real-time inventory updates
   - Deliverable: Low-stock alert system

## Actor: Competitors
### Impacts
1. **Differentiate from competitors** - Capture market share
   - Deliverable: Unique loyalty program
   - Deliverable: Subscription service option

## Prioritized Deliverables (Q1-Q2)
1. Improved search with filters
2. Streamlined one-page checkout
3. AI-powered recommendations
4. Campaign management dashboard
5. Mobile order picking app

## Deferrred Items (Q3-Q4)
1. Voice commerce integration
2. AR product visualization
3. Marketplace for third-party sellers
```

---

## Impact Mapping in Practice

```typescript
// impact-mapping.types.ts

interface BusinessGoal {
  id: string;
  title: string;
  description: string;
  metrics: {
    name: string;
    current: number;
    target: number;
    unit: string;
  }[];
  timeline: {
    start: Date;
    end: Date;
  };
  priority: 'critical' | 'high' | 'medium' | 'low';
}

interface Actor {
  id: string;
  name: string;
  type: 'internal' | 'external' | 'system';
  description: string;
  goals?: string[];
  characteristics?: string[];
}

interface Impact {
  id: string;
  actorId: string;
  description: string;
  measurement: string;
  expectedChange: string;
  priority: 'critical' | 'high' | 'medium' | 'low';
  dependencies?: string[];
}

interface Deliverable {
  id: string;
  impactId: string;
  title: string;
  description: string;
  type: 'feature' | 'story' | 'task' | 'epic';
  priority: 'critical' | 'high' | 'medium' | 'low';
  estimatedSize: 'small' | 'medium' | 'large' | 'xlarge';
  status: 'pending' | 'in-progress' | 'completed' | 'blocked';
  acceptanceCriteria: string[];
}

class ImpactMap {
  private goal: BusinessGoal;
  private actors: Map<string, Actor>;
  private impacts: Map<string, Impact>;
  private deliverables: Map<string, Deliverable>;

  constructor(goal: BusinessGoal) {
    this.goal = goal;
    this.actors = new Map();
    this.impacts = new Map();
    this.deliverables = new Map();
  }

  addActor(actor: Actor): void {
    this.actors.set(actor.id, actor);
  }

  addImpact(impact: Impact): void {
    this.impacts.set(impact.id, impact);
  }

  addDeliverable(deliverable: Deliverable): void {
    this.deliverables.set(deliverable.id, deliverable);
  }

  getDeliverablesByGoal(): Deliverable[] {
    const goalImpacts = Array.from(this.impacts.values()).filter(
      (impact) => this.actors.get(impact.actorId)?.goals?.includes(this.goal.id)
    );
    const impactIds = new Set(goalImpacts.map((i) => i.id));
    return Array.from(this.deliverables.values()).filter((d) =>
      impactIds.has(d.impactId)
    );
  }

  getImpactTree(actorId?: string): object {
    const actorsToInclude = actorId
      ? [this.actors.get(actorId)].filter(Boolean) as Actor[]
      : Array.from(this.actors.values());

    return {
      goal: this.goal,
      actors: actorsToInclude.map((actor) => ({
        ...actor,
        impacts: Array.from(this.impacts.values())
          .filter((impact) => impact.actorId === actor.id)
          .map((impact) => ({
            ...impact,
            deliverables: Array.from(this.deliverables.values())
              .filter((deliverable) => deliverable.impactId === impact.id)
              .map((d) => ({
                id: d.id,
                title: d.title,
                priority: d.priority,
                status: d.status,
              })),
          })),
      })),
    };
  }

  getMetricsProgress(): object {
    return this.goal.metrics.map((metric) => ({
      name: metric.name,
      current: metric.current,
      target: metric.target,
      progress: ((metric.current - metric.current) / (metric.target - metric.current)) * 100,
    }));
  }
}

// Usage Example
const revenueGoal: BusinessGoal = {
  id: 'G-001',
  title: 'Increase Online Revenue',
  description: 'Increase online revenue by 25% within 12 months',
  metrics: [
    { name: 'Monthly Revenue', current: 1000000, target: 1250000, unit: '$' },
    { name: 'Conversion Rate', current: 2.1, target: 3.5, unit: '%' },
  ],
  timeline: { start: new Date('2024-01-01'), end: new Date('2024-12-31') },
  priority: 'critical',
};

const impactMap = new ImpactMap(revenueGoal);

impactMap.addActor({
  id: 'A-001',
  name: 'Customers',
  type: 'external',
  description: 'End customers who purchase products',
  goals: ['G-001'],
});

impactMap.addImpact({
  id: 'I-001',
  actorId: 'A-001',
  description: 'Find products faster',
  measurement: 'Search time reduction',
  expectedChange: '40% reduction in search time',
  priority: 'high',
});

impactMap.addDeliverable({
  id: 'D-001',
  impactId: 'I-001',
  title: 'Improved Search with Filters',
  description: 'Search with category filters and auto-suggestions',
  type: 'feature',
  priority: 'high',
  estimatedSize: 'large',
  status: 'in-progress',
  acceptanceCriteria: [
    'Search returns results within 200ms',
    'Filters reduce result set by at least 50%',
    'Auto-suggest shows relevant products',
  ],
});
```

---

## Impact Mapping with Mind Maps

```markdown
# Impact Map: Mobile Banking App Redesign

## Center: GOAL
📱 **Increase mobile banking adoption by 40%**

## Branch 1: ACTOR → Customers

### Impacts
- **[IMP]** Complete banking tasks without visiting branch
  - **DEL:** Mobile check deposit feature
  - **DEL:** QR code payments
  - **DEL:** In-app appointment booking

- **[IMP]** Trust the mobile app with finances
  - **DEL:** Biometric authentication
  - **DEL:** Security center dashboard
  - **DEL:** Real-time fraud alerts

- **[IMP]** Get personalized financial insights
  - **DEL:** Spending analytics dashboard
  - **DEL:** Budget planning tools
  - **DEL:** Investment recommendations

## Branch 2: ACTOR → Customer Support

### Impacts
- **[IMP]** Handle inquiries without database queries
  - **DEL:** Integrated customer profile view
  - **DEL:** Conversation history sync

- **[IMP:** Proactively reach out to struggling customers
  - **DEL:** Risk indicator dashboard
  - **DEL:** Automated outreach triggers

## Branch 3: ACTOR → Marketing Team

### Impacts
- **[IMP]** Drive app downloads through campaigns
  - **DEL:** Referral program integration
  - **DEL:** In-app promotional banner system

- **[IMP]** Re-engage dormant users
  - **DEL:** Push notification framework
  - **DEL:** Personalized offer engine

## Branch 4: ACTOR → Regulatory Team

### Impacts
- **[IMP]** Ensure compliance with new regulations
  - **DEL:** Enhanced audit logging
  - **DEL:** Consent management system

## Branch 5: ACTOR → Competitors

### Impacts
- **[IMP]** Match or exceed competitor features
  - **DEL:** Peer-to-peer payments
  - **DEL:** Digital wallet integration
```

---

## Impact Mapping Best Practices

### 1. Start with Business Goals
Begin every impact mapping session by clearly articulating the business goals. Without a clear goal, the impact map becomes unfocused and loses its strategic value.

### 2. Use Specific, Measurable Goals
Vague goals lead to vague impact maps. Use metrics and specific targets to make goals measurable and the map actionable.

### 3. Involve Diverse Stakeholders
Impact mapping workshops should include representatives from business, development, testing, and operations to ensure a complete perspective.

### 4. Think in Impacts, Not Features
Resist the temptation to jump straight to features. Focus on what impacts you want to achieve, and let features emerge as enablers.

### 5. Challenge Every Deliverable
For each deliverable, ask: "If we build this, what impact will it have? If we can't trace an impact back to a goal, question the deliverable."

### 6. Use the Map for Prioritization
The impact map should drive prioritization. High-priority goals should receive high-priority impacts and deliverables.

### 7. Keep the Map Alive
Update the impact map regularly as you learn more. Remove items that are no longer relevant and add new branches as understanding evolves.

### 8. Visualize Extensively
Use physical or digital whiteboards to make impact maps visible and accessible. The visual nature helps communicate connections to stakeholders.

### 9. Limit Scope
Don't try to map everything at once. Focus on the most important goals first and expand the map incrementally.

### 10. Connect to Delivery
Link deliverables in the impact map directly to your backlog items, user stories, or tasks to maintain traceability throughout development.

---

## Impact Mapping Anti-Patterns

### 1. Feature-First Mapping
Starting with a list of features and trying to back-fill goals and impacts defeats the purpose of impact mapping. Always start with goals.

### 2. Vague Goals
Goals like "improve user experience" or "increase engagement" are too vague to be actionable. Use specific metrics and targets.

### 3. Missing Actors
Forgetting to include key actors leads to incomplete impact maps. Consider all internal and external actors who can influence the goal.

### 4. Technical Deliverables
Listing technical tasks (like "refactor database" or "upgrade framework") without clear impacts makes the map less useful for business discussions.

### 5. No Prioritization
Without prioritization, impact maps become wish lists. Always include priority indicators to guide scope decisions.

### 6. One-Time Exercise
Creating an impact map and never updating it leads to stale documentation. Treat the map as a living document.

### 7. Too Many Goals
Trying to map too many goals at once dilutes focus. Limit initial mapping to 2-3 critical goals.

### 8. Skipping Impacts
Jumping directly from actors to deliverables without defining impacts misses the key insight of impact mapping.

### 9. No Success Metrics
Goals without measurable metrics make it impossible to validate success or adjust course.

### 10. Exclusive Ownership
Having one person create the impact map alone reduces buy-in and misses perspectives. Use collaborative workshops.

---

## Impact Mapping in AI-Assisted Development

When working with AI assistants for impact mapping, follow these practices:

### 1. Generate Initial Drafts
Provide the AI with business context and ask it to help draft an initial impact map that you can then refine with stakeholders.

### 2. Identify Missing Actors
Ask the AI to suggest potential actors you might have overlooked, based on the business domain and goals.

### 3. Brainstorm Impacts
Use AI to brainstorm possible impacts for each actor, helping expand thinking beyond obvious options.

### 4. Trace Deliverables
Ask the AI to review your backlog and trace how each item connects through the impact map to business goals.

### 5. Prioritization Assistance
Use AI to help prioritize deliverables based on their impact on goals and estimated effort.

### 6. Gap Analysis
Ask AI to identify gaps in your impact map, such as goals without clear actors or actors without clear impacts.

### 7. Scenario Exploration
Use AI to explore "what-if" scenarios by extending the impact map with new goals or actors.

### 8. Documentation Generation
Generate human-readable impact map documentation from structured data for stakeholder review.

---

## Impact Mapping Tools

| Tool | Type | Best For |
|------|------|----------|
| **XMind** | Mind Mapping | Visual brainstorming |
| **MindMeister** | Mind Mapping | Collaborative editing |
| **Miro** | Whiteboard | Workshop collaboration |
| **Mural** | Visual Workspace | Remote team workshops |
| **Lucidchart** | Diagramming | Professional diagrams |
| **draw.io** | Diagramming | Freeform diagrams |
| **Notion** | Documentation | Living documentation |

---

## Impact Mapping and Strategic Planning

Impact Mapping is a strategic planning technique that bridges the gap between business objectives and software deliverables. It helps teams answer the fundamental question: "Why are we building this?" by creating a traceable connection from organizational goals to specific features.

### Strategic Value Chain

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   IMPACT MAPPING VALUE CHAIN                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│    ┌─────────────┐     ┌─────────────┐     ┌─────────────┐              │
│    │   BUSINESS  │────►│    PROJECT  │────►│   PRODUCT   │              │
│    │   STRATEGY  │     │   SCOPE     │     │   BACKLOG   │              │
│    └─────────────┘     └─────────────┘     └─────────────┘              │
│          │                   │                   │                      │
│          │                   │                   │                      │
│          ▼                   ▼                   ▼                      │
│    ┌─────────────┐     ┌─────────────┐     ┌─────────────┐              │
│    │   Mission   │     │  Features   │     │   User      │              │
│    │   Vision    │     │  & Stories  │     │   Stories   │              │
│    └─────────────┘     └─────────────┘     └─────────────┘              │
│                                                                         │
│    "Where do we    "What should we     "What exactly                    │
│     want to be?"    build?"             to build?"                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Impact Mapping vs. Traditional Planning

| Aspect | Traditional Planning | Impact Mapping |
|--------|---------------------|----------------|
| Starting Point | List of features | Business goals |
| Focus | What to build | Why we're building it |
| Traceability | Weak or none | Strong goal-to-deliverable trace |
| Stakeholder Alignment | Often unclear | Explicit through impact chain |
| Scope Control | Feature creep common | Goal-focused prioritization |
| Validation | Delayed until delivery | Continuous through impact chain |

### Impact Mapping in Agile Context

Impact Mapping complements Agile methodologies by providing strategic alignment:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 IMPACT MAPPING IN AGILE CYCLE                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                │
│  │    RELEASE  │────►│   SPRINT    │────►│  INCREMENT  │                │
│  │  PLANNING   │     │   PLANNING  │     │  REVIEW     │                │
│  └─────────────┘     └─────────────┘     └─────────────┘                │
│        │                   │                   │                        │
│        │                   │                   │                        │
│        ▼                   ▼                   ▼                        │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                │
│  │   Impact    │     │  Sprint     │     │  Validate   │                │
│  │   Map       │     │  Backlog    │     │  Impacts    │                │
│  └─────────────┘     └─────────────┘     └─────────────┘                │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Continuous Alignment Check: Does each increment contribute     │    │
│  │  to business goals through the impact chain?                    │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Collaboration Best Practices

### 1. Workshop-Based Approach

Impact Mapping is most effective as a collaborative workshop activity:

- **Duration**: 2-4 hours for initial impact map
- **Participants**: Product owner, key stakeholders, technical lead
- **Facilitation**: Skilled facilitator to guide exploration
- **Output**: Shared understanding and documented impact map

### 2. Start with Goals, Not Features

The most common mistake is starting with a feature list. Impact Mapping requires starting with business goals:

```
❌ WRONG: "We need a shopping cart"
         ↓
         "Why?"
         ↓
         "Users need to buy things"
         ↓
         "That's not a goal, that's a feature"

✅ CORRECT: "We want to increase revenue by 20%"
           ↓
           "Who can help achieve this?"
           ↓
           "Existing customers making larger purchases"
           ↓
           "How can they help?"
           ↓
           "Add items to cart before checkout"
           ↓
           "What do we need to build?"
           ↓
           "Shopping cart functionality"
```

### 3. Validate Impact Assumptions

Every impact should be validated:

- **Question**: "How will this actor change their behavior?"
- **Evidence**: What data supports this assumption?
- **Risk**: What if we're wrong about this impact?
- **Experiment**: How can we test this impact hypothesis?

### 4. Keep the Map Alive

Impact maps are living documents:

- **Update**: As goals change, update the impact map
- **Review**: Regularly validate impacts against outcomes
- **Prune**: Remove impacts that no longer align with goals
- **Extend**: Add new goals, actors, and impacts as needed

---

## References and Further Reading

1. Adzic, Gojko. "Impact Mapping: Delivering Business Value." 2012.
2. Adzic, Gojko. "Specification by Example: How Successful Teams Deliver the Right Software." Manning, 2011.
3. "Impact Mapping Workshop Guide." Gojko Adzic.
4. "Strategic Planning with Impact Mapping." Agile Alliance.
5. "From Goals to Delivered Features: Impact Mapping in Practice." NDC Conferences.
6. ref/engineering/behavior_driven/BEHAVIOR_DRIVEN_DEVELOPMENT.md - BDD methodology
7. doc/requirements-and-specification.md - Requirements and specification guide
8. doc/behavior-driven-development.md - BDD comprehensive guide
