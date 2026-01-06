# Cross-Team Collaboration

> **Last reviewed**: 2026-01-06


## Week Nine: The Dependency Web

Two teams block each other on a shared API, and the launch date is slipping. Sarah asks you to create clearer contracts and coordination, while Marcus points out that cross-team work succeeds only when expectations are explicit. This week focuses on API contracts, shared libraries, RFCs, and dependency management that reduce friction and surprises.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **API contracts** | A treaty - clear rules both sides follow |
| **Shared libraries** | Public utilities - maintained for everyone |
| **Guilds** | A town hall - align across teams |
| **Dependencies** | A supply chain - manage risk and delays |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Lead Module 08 (Documentation Culture) - Comfort documenting decisions and standards.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Define API contracts and expectations across teams
- [ ] Establish governance for shared libraries
- [ ] Run lightweight cross-team forums and RFCs
- [ ] Manage dependencies with clear ownership
- [ ] Reduce coupling with fallbacks and versioning
- [ ] Communicate changes without surprises

---

## Time Estimate

- **Reading**: 60-80 minutes
- **Exercises**: 3-4 hours
- **Mastery**: Practice cross-team collaboration over 8-12 weeks

---

## Chapter 1: Define Clear API Contracts

You write the contract before you promise anything to another team.

Sarah wants contracts that specify behavior, not just endpoints. Marcus suggests you write the guarantees down so there is no debate later.

Sarah says, "Contracts prevent guesswork."

```markdown
## API Contract Template

Endpoint: GET /api/users/{id}
Owner: User Team
Consumers: Profile Team, Orders Team

Request:
GET /api/users/123
Authorization: Bearer <token>

Response (200):
{
  "id": "123",
  "name": "Alice Smith",
  "email": "alice@example.com",
  "createdAt": "2024-01-15T10:00:00Z"
}

Response (404):
{
  "error": "USER_NOT_FOUND",
  "message": "User with id 123 not found"
}

Guarantees:
- Response time: < 200ms p95
- Availability: 99.9%
- Rate limit: 100 req/min

Breaking Changes:
- 30-day notice
- Old version supported for 90 days
- Versioned via Accept header
```

## Chapter 2: Test the Contract

You turn promises into tests so they cannot drift.

Marcus recommends consumer-driven tests so producers and consumers stay aligned.

Marcus says, "If the contract isn't tested, it's not real."

```typescript
// Consumer-driven contract test
describe('User API Contract', () => {
  it('returns user with expected shape', async () => {
    const user = await api.getUser('123');

    expect(user).toMatchObject({
      id: expect.any(String),
      name: expect.any(String),
      email: expect.stringMatching(/@/),
      createdAt: expect.any(String),
    });
  });
});
```

## Chapter 3: Govern Shared Libraries

You define ownership so shared code does not become abandoned code.

Sarah wants shared libraries only when they reduce overall complexity. Marcus warns that shared code without ownership becomes a liability.

Sarah says, "Shared libraries need shared ownership."

```markdown
## Shared Library Decision Guide

Share when:
- 3 or more teams need the same capability
- Behavior must be consistent
- A team is willing to maintain it

Duplicate when:
- It is simple and unlikely to change
- Teams need different behavior
- Coupling cost is higher than duplication cost
```

```markdown
## Governance Rules

Owner responsibilities:
- Maintain and update the library
- Review contributions
- Communicate changes

Consumer responsibilities:
- Report bugs and issues
- Test updates before adopting
- Pin versions appropriately

Breaking changes:
- RFC required
- Migration path included
- 90-day deprecation window
```

## Chapter 4: Create Cross-Team Forums

You create a lightweight forum so decisions do not stall in DMs.

Sarah asks you to create lightweight ways for teams to coordinate. Marcus suggests a guild and a simple RFC process.

Marcus says, "Forums keep teams aligned without endless meetings."

```markdown
## Frontend Guild Charter

Purpose: Share knowledge and align practices across teams.
Members: One representative per team.
Cadence: Monthly, 60 minutes.
Agenda: Team updates, deep dive, open discussion.
Action items: Tracked in shared doc.
```

```markdown
## RFC Lifecycle

1. Draft the proposal
2. One-week review window
3. Discussion if needed
4. Decision (accept, revise, reject)
5. Track implementation
```

## Chapter 5: Manage Dependencies

You map dependencies early so launches do not get blocked.

Marcus encourages you to treat dependencies like risks and plan around them.

Sarah says, "Dependencies are risks unless you plan them."

```markdown
## Dependency Request Template

From: Frontend Team
To: User Service Team
Type: New API endpoint

Need:
GET /api/users/{id}/preferences
Response: { theme: "dark", notifications: true }

Timeline:
- Need by Sprint 15
- Ideal by Sprint 14

Impact if delayed:
- Feature launch delayed
- Workaround: store in local storage
```

```markdown
## Reduce Dependency Risk

- Add fallbacks for soft dependencies
- Cache data where possible
- Prefer async integration over synchronous calls
- Isolate interfaces to reduce change blast radius
```

---

## Common Mistakes

1. **Undefined ownership** - Nobody knows who owns the contract or library.
2. **Breaking changes without notice** - Trust erodes quickly.
3. **Shared library sprawl** - Too many packages create hidden coupling.
4. **Synchronous dependency chains** - Failures cascade across teams.

## Practice Exercises

1. Document an API contract for a cross-team dependency
2. Propose a shared library and define its governance
3. Create a guild charter for your area of expertise

### Solutions

<details>
<summary>Exercise 1: API Contract</summary>

```markdown
Endpoint: GET /api/orders/{id}
Owner: Orders Team
Guarantees: 200ms p95, 99.9% availability
Breaking changes: 30-day notice, 90-day deprecation
```

**Key points:**
- Include performance and availability targets.
- Define breaking change policy.
- Name owners and consumers.

</details>

<details>
<summary>Exercise 2: Shared Library Governance</summary>

```markdown
Library: @company/ui
Owner: Design Systems Team
Rules: RFC for major changes, 90-day deprecations, monthly release notes
```

**Key points:**
- Ownership is explicit.
- Consumers know how changes happen.
- Release notes reduce surprise.

</details>

<details>
<summary>Exercise 3: Guild Charter</summary>

```markdown
Frontend Guild
Purpose: Align tooling and patterns across teams
Cadence: Monthly
Agenda: Updates, deep dive, open Q&A
Action items: Tracked and reviewed next meeting
```

**Key points:**
- Lightweight structure beats heavy process.
- Regular cadence builds trust.
- Action items keep momentum.

</details>

---

## What You Learned

This module covered:

- **API contracts**: Clear expectations between teams
- **Shared libraries**: Governance prevents chaos
- **Cross-team forums**: Guilds and RFCs align decisions
- **Dependency management**: Reduce risk and surprises
- **Communication**: Keep changes visible and predictable

**Key takeaway**: Cross-team success depends on explicit agreements and steady communication.

---

## Real-World Application

This week at work, you might use these concepts to:

- Write a contract for a shared API
- Start a monthly guild meeting
- Create a governance doc for a shared library
- Document a dependency timeline for a launch
- Add fallbacks for a fragile service

---

## Further Reading

- [Team Topologies](https://teamtopologies.com/)
- [Building Microservices](https://www.oreilly.com/library/view/building-microservices-2nd/9781492034018/)

---

**Navigation**: [← Previous Module](./08-documentation-culture.md) | [Next Module →](./10-effective-one-on-ones.md)
