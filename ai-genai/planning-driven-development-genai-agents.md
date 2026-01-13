# Planning-Driven Development: The Single Biggest Productivity Multiplier with GenAI Agents

## From Prompts to Plans: A Paradigm Shift

When I first started using GenAI coding agents, I did what everyone does: I wrote prompts to construct code. "Create a domain model for user management." "Build a REST endpoint for user authentication." "Add a repository class with Dapper."

It worked. Sort of. The code came out, but then came the endless cycle: editing, refactoring, re-prompting, fixing edge cases, realigning with requirements. Every correction required another conversation. Every misunderstanding meant more rework.

Then I discovered something that fundamentally changed my productivity: **planning documents are infinitely easier to edit than code**.

In a recent project, I spent roughly 6 hours generating a 10-page planning document and a 48-page implementation plan with my GenAI agent. Yes, that's a lot of reading. But the development that followed? It went so smoothly that the output was 90-95% of what was expected on the first pass. And this wasn't a one-off—I'm seeing this level of success repeatedly across different projects and problem domains.

This post expands on my previous article about [Planning-First Development](https://www.nathanfox.net/p/planning-first-development-claude-code), diving deeper into the specific patterns and document structures that make this approach so effective. I call it "Planning-Driven" here because the plan isn't just something you do first—it's what drives the entire development process, guiding every session and decision along the way.

## Why Planning Documents Beat Prompt Engineering

### The Economics of Editing

Consider the cost of changing direction at different stages:

| Stage | Cost to Change Direction |
|-------|-------------------------|
| Planning Document | Minutes (edit text) |
| Prompt Refinement | Hours (regenerate, review, test) |
| Code Editing | Hours to Days (refactor, test, debug) |
| Post-Deployment | Days to Weeks (hotfix, rollback, user impact) |

When you're working with a GenAI agent, every code change requires:
1. Understanding existing context
2. Formulating the right prompt
3. Reviewing generated code
4. Testing for regressions
5. Integrating with existing code

When you're refining a planning document, you:
1. Edit text
2. Done

### Plans as Shared Context

A well-structured planning document becomes the shared understanding between you and the GenAI agent. When you start each development session with "Follow the plan in IMPLEMENTATION_PLAN.md, we're working on Phase 3," the agent has:
- Complete project context
- Architectural decisions already made
- Clear deliverables for the current phase
- Testing strategy defined
- Integration points documented

No need to re-explain. No context drift. No misunderstandings about what "done" looks like.

## The Two-Document Pattern

Through experimentation across multiple projects, I've settled on a two-document approach:

### Document 1: The Planning Document (Business-Facing)

This is the high-level document you can share with stakeholders, business users, and team members. It answers:
- **What** are we building?
- **Why** are we building it?
- **What** are the requirements?
- **What** does success look like?

This document is typically 5-15 pages and uses language that non-technical stakeholders can understand.

### Document 2: The Implementation Plan (Developer-Facing)

This is the detailed technical roadmap. It answers:
- **How** will we build it?
- **What** is the architecture?
- **What** are the phases and tasks?
- **What** are the file structures?
- **What** code patterns will we use?

This document can be substantial—40-60 pages for complex projects—but it's worth every line.

## Anatomy of an Effective Planning Document

Based on patterns I've observed across dozens of planning documents, here's the structure that works:

### 1. Executive Overview

```markdown
# Project Name Planning Document

## Overview
Brief description of what we're building and why.

## Background
Current state, problems being solved, reason for change.

## Goals
- Primary goal 1
- Primary goal 2
- Success criteria
```

The overview should be understandable by anyone in the organization. No jargon, no implementation details.

### 2. Requirements Section

```markdown
## Requirements

### Core Requirements
1. **Requirement Name**: Description of what must be accomplished
   - Acceptance criteria
   - Edge cases to consider

### Business Rules
1. **Rule Name**: Business logic that must be enforced
```

Be specific. Vague requirements lead to vague implementations. If a business user can misinterpret a requirement, they will—and so will your GenAI agent.

### 3. Proposed Architecture (High-Level)

**Example Components Table:**

| Component | Description |
|-----------|-------------|
| API Layer | Handles HTTP requests, validation |
| Domain Layer | Business logic, rules enforcement |
| Data Layer | Database operations, caching |

**Example Key Decisions Table:**

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Database | SQL Server | Enterprise support, team expertise |
| Framework | ASP.NET Core | C# ecosystem, performance, tooling |

Decision tables are crucial. They capture not just *what* you chose, but *why*. This prevents re-litigating decisions later and gives the GenAI agent context for making consistent choices.

### 4. Implementation Phases

```markdown
## Implementation Phases

### Phase 1: Foundation
- Set up project structure
- Define domain model (entities, enums, value objects)
- Implement domain services and business rules
- Write unit tests for domain logic

### Phase 2: Infrastructure
- Create database schema and migrations
- Implement repositories with Dapper
- Write repository integration tests

### Phase 3: API Layer
- Create API endpoints
- Add request validation
- Integration testing

### Phase 4: Polish
- Error handling
- Logging and observability
- Documentation
- Performance optimization
```

Phases provide natural checkpoints for review and create clear scopes for development sessions with the GenAI agent.

## Anatomy of an Implementation Plan

The implementation plan is where the magic happens. This is the document you and your GenAI agent work from directly.

### 1. Document Metadata

Track status so you always know where you left off:

| Field | Value |
|-------|-------|
| Created | 2024-01-15 |
| Status | In Progress |
| Current Phase | Phase 2 |

### 2. Domain Model First

My implementation plans always start with the domain model. This aligns with my [Domain-First Development](https://www.nathanfox.net/p/domain-first-development-building) approach—build your domain types and business logic before thinking about databases, APIs, or UI.

```markdown
## Domain Model

### Core Entities

Define your domain types with their properties and relationships:
```

```csharp
public record User
{
    public Guid Id { get; init; }
    public string Email { get; init; } = string.Empty;
    public string Name { get; init; } = string.Empty;
    public UserRole Role { get; init; }
    public DateTime CreatedAt { get; init; }
    public DateTime? LastLogin { get; init; }
}

public enum UserRole
{
    Admin,
    Member,
    Viewer
}
```

```markdown
### Domain Rules

Document the business rules that govern your domain:

1. **Email Uniqueness**: No two users can share the same email address
2. **Role Transitions**: Only Admins can promote users to Admin role
3. **Soft Delete**: Users are never hard-deleted; they are deactivated
```

Starting with the domain model ensures you and the GenAI agent have a shared understanding of the business concepts before any infrastructure code is written.

### 3. Technical Decisions

**Example Technical Decisions Table:**

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Data Access | Repository Pattern | Testability, abstraction |
| API Style | RESTful + OpenAPI | Industry standard, tooling |
| Error Handling | Result types | Explicit, composable |
| Testing | TDD with xUnit | Fast feedback, documentation |

### 4. Project Structure

```
MyProject/
├── src/
│   ├── MyProject.Domain/           # Pure domain model (no dependencies)
│   │   ├── Entities/
│   │   │   └── User.cs
│   │   ├── Services/
│   │   │   └── UserService.cs
│   │   └── Validation/
│   │       └── UserValidator.cs
│   ├── MyProject.Api/              # HTTP endpoints
│   │   ├── Controllers/
│   │   │   └── UsersController.cs
│   │   └── Program.cs
│   └── MyProject.Infrastructure/   # Data access, external services
│       ├── Repositories/
│       │   └── UserRepository.cs
│       └── External/
│           └── EmailService.cs
├── tests/
│   ├── MyProject.Domain.Tests/
│   └── MyProject.Api.Tests/
└── docs/
    ├── PLANNING.md
    └── IMPLEMENTATION_PLAN.md
```

ASCII directory trees are surprisingly effective for GenAI agents. They understand the structure immediately and maintain consistency when creating new files.

### 5. Domain Logic Examples

Include actual code in your implementation plan. The GenAI agent will use these as templates and maintain consistency throughout the codebase.

```csharp
// Domain service pattern - pure business logic
public class UserService
{
    public Result<User> CreateUser(CreateUserCommand command)
    {
        var validationResult = ValidateCreateCommand(command);
        if (!validationResult.IsSuccess)
            return Result<User>.Failure(validationResult.Errors);

        var user = new User
        {
            Id = Guid.NewGuid(),
            Email = command.Email.ToLowerInvariant(),
            Name = command.Name,
            Role = UserRole.Member,
            CreatedAt = DateTime.UtcNow
        };

        return Result<User>.Success(user);
    }

    public bool CanPromoteToAdmin(User currentUser, User targetUser)
    {
        return currentUser.Role == UserRole.Admin
            && targetUser.Role != UserRole.Admin;
    }
}
```

### 6. API Contracts

Document your API endpoints with request/response examples:

**POST /api/users** - Create a new user

Request:
```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "role": "member"
}
```

Response (201):
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "name": "John Doe",
  "role": "member",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

Errors:
- 400: Validation error
- 409: Email already exists

### 7. Implementation Checklist

This is the heart of the implementation plan—a detailed checklist organized by phase:

```markdown
## Implementation Checklist

### Phase 1: Foundation

**Project Setup:**
- [x] Create solution and project structure
- [x] Configure EditorConfig and code analysis
- [x] Set up xUnit test projects
- [x] Create GitHub Actions CI pipeline

**Domain Layer:**
- [x] Define User entity and UserRole enum
- [x] Implement UserService with business logic
- [x] Create UserValidator with FluentValidation
- [x] Write unit tests for domain logic (100% coverage)

### Phase 2: Infrastructure

**Database:**
- [x] Create SQL migration scripts
- [x] Implement UserRepository with Dapper
- [x] Write repository integration tests

### Phase 3: API Layer

**Endpoints:**
- [x] POST /api/users (create)
- [ ] GET /api/users/{id} (read)
- [ ] PUT /api/users/{id} (update)
- [ ] DELETE /api/users/{id} (delete)
- [ ] GET /api/users (list with pagination)

**Authentication:**
- [ ] JWT token generation
- [ ] Token validation middleware
- [ ] Role-based authorization
```

Update the checklist as you work. It provides:
- Clear progress visibility
- Natural stopping points
- Context for resuming work
- Documentation of what's complete

### 8. Code Patterns

Document the patterns you'll use throughout the codebase:

```csharp
// Repository pattern with Dapper
public class UserRepository : IUserRepository
{
    private readonly IDbConnectionFactory _connectionFactory;

    public UserRepository(IDbConnectionFactory connectionFactory)
    {
        _connectionFactory = connectionFactory;
    }

    public async Task<User?> GetByIdAsync(Guid id)
    {
        using var connection = _connectionFactory.CreateConnection();
        return await connection.QuerySingleOrDefaultAsync<User>(
            "SELECT Id, Email, Name, Role, CreatedAt, LastLogin FROM Users WHERE Id = @Id",
            new { Id = id });
    }

    public async Task<User> CreateAsync(User user)
    {
        using var connection = _connectionFactory.CreateConnection();
        await connection.ExecuteAsync(
            @"INSERT INTO Users (Id, Email, Name, Role, CreatedAt)
              VALUES (@Id, @Email, @Name, @Role, @CreatedAt)",
            user);
        return user;
    }
}
```

```csharp
// Result type for explicit error handling
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public string[] Errors { get; }

    private Result(bool isSuccess, T? value, string[] errors)
    {
        IsSuccess = isSuccess;
        Value = value;
        Errors = errors;
    }

    public static Result<T> Success(T value) =>
        new(true, value, Array.Empty<string>());

    public static Result<T> Failure(params string[] errors) =>
        new(false, default, errors);
}
```

## The Development Workflow

Here's how a typical development session looks with planning-driven development:

### 1. Session Start

```
You: "Let's continue with the implementation plan. We're on Phase 2.
Read IMPLEMENTATION_PLAN.md and let's work on the GET /api/users/{id} endpoint."

Agent: [Reads plan, understands context, sees patterns]
"I see we're implementing the user retrieval endpoint. Based on the plan,
I'll follow the repository pattern established in Phase 1. Let me start
with the tests following TDD..."
```

### 2. Implementation

The agent works through the checklist items, following established patterns from the plan. Because the patterns are documented, there's consistency. Because the checklist exists, there's clarity on what "done" means.

### 3. Session End

```
You: "Good progress. Update the checklist in the implementation plan
to mark what we completed."

Agent: [Updates checklist, marks items complete]
"Updated. We completed 3 of 5 endpoint implementations in Phase 2.
Next session we can tackle the remaining endpoints."
```

### 4. Resume Later

The next day, with a fresh context window:

```
You: "Continue with IMPLEMENTATION_PLAN.md, Phase 2."

Agent: [Reads plan, sees exactly where you left off]
"I see we have PUT and DELETE endpoints remaining. Let me continue
with the update endpoint..."
```

No re-explanation needed. The plan carries all context.

## Refining Plans with GenAI

One of the most powerful aspects of this approach is using the GenAI agent to *help create and refine the plans themselves*.

### Initial Planning Session

```
You: "I need to build a user management API for our new application.
Let me describe the requirements, and then help me create a planning document..."

[Describe business requirements, constraints, existing systems]

Agent: [Asks clarifying questions, proposes structure]
"Based on what you've described, here are some questions:
1. What authentication method should we use (JWT, OAuth)?
2. What are the user roles and permissions?
3. Are there existing patterns from other services we should follow?

Let me start drafting a planning document..."
```

### Iterative Refinement

```
You: "The database section needs more detail. We need to handle
status transitions with audit trails."

Agent: [Expands section with specific table designs, constraints, indexes]
```

```
You: "The testing strategy should emphasize TDD. Reference my post
about taming GenAI agents with TDD."

Agent: [Updates testing section to align with TDD methodology]
```

### Review and Finalize

Once the plan feels complete:

```
You: "Review the entire implementation plan. Identify any gaps,
inconsistencies, or missing details."

Agent: "I notice:
1. Phase 3 mentions notifications but there's no notification service design
2. The API contract doesn't specify pagination for list endpoints
3. Error codes are defined but error message format isn't standardized

Want me to address these?"
```

## Common Planning Document Patterns

After analyzing planning documents across many projects, these patterns consistently appear in effective plans:

### Decision Rationale Tables

| Decision | Options Considered | Choice | Rationale |
|----------|-------------------|--------|-----------|
| Data Access | Entity Framework, Dapper | Dapper | Simpler, explicit SQL, better GenAI compatibility |
| Auth | JWT, Session | JWT | Stateless, works with microservices |
| Queue | RabbitMQ, Kafka, Azure Service Bus | Azure Service Bus | Team expertise, cloud-native |

### Status Tracking

| Phase | Status | Notes |
|-------|--------|-------|
| Phase 1: Foundation | Complete | All tests passing |
| Phase 2: Core API | In Progress | 3/5 endpoints done |
| Phase 3: Notifications | Not Started | Blocked on email service access |

### Risk Register

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Third-party API changes | High | Medium | Version-pin dependencies, integration tests |
| Database migration failures | High | Low | Test migrations in staging, backup before prod |

### Open Questions

1. **Email template approval**: Need marketing sign-off on notification templates
   - Status: Pending
   - Owner: Sarah

2. **Rate limiting strategy**: How aggressive should we be?
   - Options: 100/min, 1000/min, no limit initially
   - Leaning: 100/min to start, adjust based on usage

## Scaling to Large Projects

For substantial projects, consider:

### Hierarchical Planning

```
docs/
├── PLANNING.md                    # High-level (stakeholder-facing)
├── IMPLEMENTATION_PLAN.md         # Overall technical plan
├── phases/
│   ├── PHASE_1_FOUNDATION.md      # Detailed phase plan
│   ├── PHASE_2_CORE_API.md
│   └── PHASE_3_NOTIFICATIONS.md
└── designs/
    ├── DATABASE_DESIGN.md         # Deep-dive documents
    ├── API_SPECIFICATION.md
    └── SECURITY_MODEL.md
```

### Version Control for Plans

Treat planning documents like code:
- Commit changes with meaningful messages
- Review significant plan changes
- Tag releases when plans are "approved"

### Team Synchronization

When working with teams:
- Plans become the source of truth
- Code reviews reference the plan
- Deviations from plan require discussion
- Plan updates follow change control

## Practical Tips

### Start Small

Don't try to plan everything upfront. Begin with:
1. An initial overview
2. A list of phases
3. A simple checklist

Expand as you learn more about the problem.

### Keep It Living

The plan should evolve. When you discover something during implementation:
1. Update the plan
2. Document why you changed course
3. Adjust remaining phases if needed

### Reference Liberally

In your plan, reference:
- Other planning documents (exploring specific decisions in more depth)
- External documentation
- Your own blog posts or team wikis
- Related codebases

The more context, the better.

### Include Code Examples

Don't be afraid to put code in your planning documents. Actual code examples:
- Set the style for the project
- Provide copy-paste starting points
- Reduce ambiguity about patterns

### Timebox Planning

I typically spend:
- 2-4 hours on initial planning document
- 4-8 hours on detailed implementation plan
- Ongoing updates as we implement

The upfront investment pays off exponentially.

## The Results

Since adopting planning-driven development:

- **First-pass accuracy** consistently reaches 90-95%
- **Context switching** became trivial (just reference the plan)
- **Stakeholder communication** improved (share the planning document)
- **Decision archaeology** simplified (rationale is documented)

The planning document becomes an asset that outlives the implementation itself, serving as documentation, training material, and historical record.

## Conclusion

The shift from prompt-engineering code to planning-first development is the single biggest productivity multiplier I've found when working with GenAI agents. Instead of fighting with code corrections, you refine documents. Instead of re-explaining context, you reference plans. Instead of debugging misunderstandings, you clarify upfront.

Yes, a 48-page implementation plan sounds like a lot of reading. But compare that to days of debugging misaligned implementations, or weeks of refactoring code that solved the wrong problem.

**Planning documents are cheap to write, cheap to edit, and expensive not to have.**

The next time you start a new feature or project with a GenAI agent, resist the urge to start prompting for code. Instead, say: "Let's create a planning document first." Then watch as your development sessions become focused, efficient, and productive.

---

*For the foundation of this approach, see [Planning-First Development: How Markdown Documents Drive Structured AI-Assisted Development](https://www.nathanfox.net/p/planning-first-development-claude-code).*

*For why I always start with domain models, see [Domain-First Development: Building Robust Applications from the Core Out](https://www.nathanfox.net/p/domain-first-development-building).*

*For combining planning with test-driven development, see [Taming GenAI Agents Like Claude Code with Test-Driven Development](https://www.nathanfox.net/p/taming-genai-agents-like-claude-code).*
