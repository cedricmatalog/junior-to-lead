# Cross-Team Collaboration

Work effectively across team boundaries.

## Learning Objectives

By the end of this module, you will:
- Establish clear API contracts
- Create and maintain shared libraries
- Share knowledge across teams
- Manage dependencies gracefully

## API Contracts

### Defining Contracts

```markdown
## API Contract Template

### Endpoint: GET /api/users/{id}

**Owner**: User Team
**Consumers**: Profile Team, Orders Team

**Request**
\`\`\`
GET /api/users/123
Authorization: Bearer <token>
\`\`\`

**Response (200)**
\`\`\`json
{
  "id": "123",
  "name": "Alice Smith",
  "email": "alice@example.com",
  "createdAt": "2024-01-15T10:00:00Z"
}
\`\`\`

**Response (404)**
\`\`\`json
{
  "error": "USER_NOT_FOUND",
  "message": "User with id 123 not found"
}
\`\`\`

**Guarantees**
- Response time: < 200ms p95
- Availability: 99.9%
- Rate limit: 100 req/min

**Breaking Changes Policy**
- 30-day notice for breaking changes
- Old version supported for 90 days
- Versioned via Accept header
```

### Contract Testing

```typescript
// Consumer-driven contract test
describe('User API Contract', () => {
  it('returns user with expected shape', async () => {
    const user = await api.getUser('123');

    // Test contract, not specific values
    expect(user).toMatchObject({
      id: expect.any(String),
      name: expect.any(String),
      email: expect.stringMatching(/@/),
      createdAt: expect.any(String),
    });
  });

  it('returns 404 for unknown user', async () => {
    await expect(api.getUser('unknown')).rejects.toMatchObject({
      status: 404,
      error: 'USER_NOT_FOUND',
    });
  });
});
```

## Shared Libraries

### When to Create Shared Libraries

```markdown
## Decision: Shared vs Duplicate

**Share when:**
- 3+ teams need the same thing
- Behavior must be consistent
- One team willing to maintain

**Duplicate when:**
- Simple, unlikely to change
- Teams need different behavior
- Coupling cost > duplication cost
```

### Shared Library Structure

```
packages/
├── ui/                    # Shared UI components
│   ├── src/
│   ├── package.json
│   └── README.md
├── utils/                 # Shared utilities
│   ├── src/
│   ├── package.json
│   └── README.md
└── api-client/           # Shared API client
    ├── src/
    ├── package.json
    └── README.md
```

### Library Maintenance

```markdown
## Shared Library Ownership

### Owner Responsibilities
- Maintain and update the library
- Review and merge contributions
- Communicate changes
- Keep documentation current

### Consumer Responsibilities
- Report bugs and issues
- Propose features via RFC
- Test updates before adopting
- Pin versions appropriately

### Governance
- Major changes require RFC
- Breaking changes need migration path
- Deprecations: 90-day notice
- Regular dependency updates
```

## Knowledge Sharing

### Cross-Team Communication

```markdown
## Communication Channels

### Synchronous
- **Guild meetings**: Monthly, topic-focused
- **Office hours**: Weekly, drop-in Q&A
- **Pair programming**: Ad-hoc, cross-team

### Asynchronous
- **Slack channels**: #frontend, #architecture
- **RFCs**: Major decisions
- **Tech blog**: Internal knowledge sharing
- **Demo days**: Monthly showcase
```

### Frontend Guild

```markdown
## Frontend Guild Charter

### Purpose
Share knowledge, align practices, and solve common problems
across frontend teams.

### Members
Representatives from each team with frontend code.

### Meetings
- **When**: First Thursday, 3 PM
- **Duration**: 1 hour
- **Format**: Rotating presenters

### Topics
- New technologies and patterns
- Cross-cutting concerns
- Shared tooling discussions
- Problem-solving sessions

### Agenda Template
1. Updates from each team (15 min)
2. Deep dive topic (30 min)
3. Open discussion (15 min)

### Action Items
- Tracked in shared doc
- Assigned to individuals
- Reviewed at next meeting
```

### Technical RFCs

```markdown
## RFC Process

### When to Write an RFC
- Changes affecting multiple teams
- New shared infrastructure
- Major architectural decisions
- Breaking changes to shared code

### RFC Template
1. **Summary**: One paragraph
2. **Motivation**: Why is this needed?
3. **Proposal**: What are you proposing?
4. **Alternatives**: What else was considered?
5. **Impact**: Who is affected?
6. **Migration**: How do we get there?

### RFC Lifecycle
1. Draft: Author writes proposal
2. Review: 1-week comment period
3. Discussion: Sync meeting if needed
4. Decision: Accept, reject, or revise
5. Implementation: Track progress
```

## Dependency Management

### Dependency Types

```markdown
## Types of Dependencies

### Hard Dependencies
- Your code won't work without it
- Example: Auth service for login
- Mitigation: Graceful degradation, timeouts

### Soft Dependencies
- Features degraded without it
- Example: Recommendation service
- Mitigation: Fallback content

### Development Dependencies
- Needed for building/testing
- Example: Shared types package
- Mitigation: Version pinning
```

### Managing Team Dependencies

```markdown
## Dependency Request Template

**From**: Frontend Team
**To**: User Service Team
**Type**: New API endpoint

### What We Need
\`\`\`
GET /api/users/{id}/preferences
Response: { theme: "dark", notifications: true }
\`\`\`

### Why We Need It
User settings page needs to display and edit preferences.

### Timeline
- **Need by**: Sprint 15
- **Ideal**: Sprint 14 (buffer)

### Impact if Delayed
- Feature launch delayed
- Workaround: Store in localStorage (temp)

### Acceptance Criteria
- Endpoint available in staging
- Response format as specified
- Documentation updated
```

### Reducing Dependencies

```markdown
## Strategies for Reducing Dependencies

### Async Integration
Instead of synchronous API calls, use events.
- Your code doesn't wait for response
- Failure doesn't cascade
- Easier to scale

### Caching
Cache responses where appropriate.
- Reduces load on dependencies
- Improves resilience
- Watch for staleness

### Fallbacks
Have a plan B when dependencies fail.
- Graceful degradation
- Cached data
- Default values

### Interface Segregation
Don't depend on more than you need.
- Smaller contracts
- Fewer reasons to change
- Easier to mock
```

## Practice Exercises

1. Document an API contract for a cross-team dependency
2. Propose a shared library and define its governance
3. Create a guild charter for your area of expertise

## Further Reading

- [Team Topologies](https://teamtopologies.com/)
- [Building Microservices](https://www.oreilly.com/library/view/building-microservices-2nd/9781492034018/)
