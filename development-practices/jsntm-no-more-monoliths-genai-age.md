# JSNTM: Just Say No to Monoliths in the GenAI Age

![JSNTM - Just Say No to Monoliths](../images/jsntm.png)

## The Pledge

I'm making a commitment: **I will no longer create monolithic applications or services, no matter how small the project.**

This isn't aspirational thinking or architectural idealism. It's a practical decision enabled by GenAI coding agents. The calculus that once made monoliths the "pragmatic choice" has fundamentally changed.

## The Old Excuse

For years, the conversation went like this:

"This should really be its own service."

"Yes, but setting up a new service takes 3-5 days. Let's just add it to the existing one."

We knew it would stay there. One bounded context bled into another. Services that started focused accumulated unrelated responsibilities. We knew better, but the overhead of doing it right exceeded the immediate cost of doing it wrong.

The result? Systems with 25 services that should have been 40 or more. Monoliths wearing microservice clothing. Technical debt disguised as pragmatism.

## What Creating a New Service Actually Required

Setting up a new microservice traditionally meant:

1. **Repository Setup** - Create repo, configure branch policies, add team access
2. **Project Scaffolding** - Solution files, folder structure, configuration
3. **Boilerplate Code** - Startup code, dependency injection, authentication
4. **Infrastructure** - Health checks, logging, metrics, configuration management
5. **Containerization** - Dockerfile, docker-compose for local development
6. **Orchestration** - Kubernetes manifests, service definitions, ingress rules
7. **CI/CD Pipeline** - Build, test, and deployment automation
8. **Database Setup** - Schema design, migrations, connection configuration
9. **Documentation** - README, API documentation, architecture notes

**Three to five days of tedious work before writing a single line of business logic.**

No wonder we cut corners.

## The GenAI Transformation

With tools like Claude Code, that 3-5 day overhead collapses to 2-4 hours—including review and refinement.

GenAI excels at exactly the work we avoided:
- Project scaffolding and structure
- Boilerplate startup code
- Dockerfile generation
- Kubernetes manifests
- CI/CD pipeline templates
- Database schema scaffolding
- API contract definitions
- Test scaffolding

The repetitive, pattern-based work that made service creation burdensome is precisely what AI handles best. You describe the service boundaries, the data it owns, the contracts it exposes—and the scaffolding materializes.

Better yet, if you already have microservices, they become templates for new ones. Point the AI at an existing service and say "create a new service like this one, but for X." This is [Example-Driven Development](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code)—your existing codebase becomes the specification. The patterns, conventions, and infrastructure decisions you've already made get replicated automatically.

This isn't about AI writing your business logic. It's about AI eliminating the friction that pushed you toward architectural compromise.

## The New Standard

When service creation is cheap, the architecture question changes.

**Before GenAI:**
"Is this different enough to justify the overhead of a new service?"

**After GenAI:**
"Is this a distinct bounded context with its own data ownership?"

If yes, it gets its own service. Full stop.

The decision criteria should be architectural merit, not implementation burden. GenAI makes that possible.

## Decision Framework

Create a new service when:
- It represents a distinct bounded context
- It has clear data ownership (its own database or data store)
- It could reasonably scale independently
- It has a different deployment cadence
- A different team could own it
- It's a significant new feature area

Keep functionality in an existing service when:
- It's a new endpoint within an existing domain
- It shares most data with existing functionality
- It's a small utility function
- It's tightly coupled to existing business logic

When uncertain, ask: "If another team were building this, what would they need from us?"

If the answer is "just the database," it belongs in the existing service.
If the answer is "a well-defined API contract," it's a candidate for extraction.

## The Micro-Frontend Parallel

This principle extends beyond backend services to the user interface.

With micro-frontends (MFEs), each microservice can own its UI components. The same GenAI-enabled workflow that makes backend service creation trivial applies to frontend modules.

Consider a shell application that composes independently deployable frontend modules:
- Each bounded context owns its UI
- Teams can deploy UI changes independently
- Technology choices can vary by module
- The frontend mirrors the backend architecture

I'll be exploring this pattern in depth in a future post, including a reference implementation demonstrating how to build a Svelte-based micro-frontend shell with dynamically loaded modules.

## What This Enables

When you stop compromising on architecture:

**Independent Deployability** - Each service deploys on its own schedule. A bug fix in one area doesn't require coordinating with unrelated changes.

**Clear Ownership** - No more debates about which service owns which functionality. Boundaries are explicit.

**Independent Scaling** - Scale the services that need it without over-provisioning everything else.

**Fault Isolation** - Problems stay contained. A failure in one bounded context doesn't cascade.

**Technology Flexibility** - Different services can use different technologies where appropriate.

**Smaller Codebases** - GenAI tools work better with focused codebases. Smaller context windows, more accurate suggestions.

## The Counter-Argument

"But microservices add operational complexity!"

True. But this complexity exists whether you acknowledge it or not. A monolith with mixed concerns has the same logical complexity—it's just hidden behind a single deployment unit. When something breaks, you're still debugging across conceptual boundaries.

Explicit boundaries don't create complexity. They make existing complexity visible and manageable.

And with modern orchestration platforms and GenAI-assisted operations, the operational overhead of running multiple services has never been lower.

## The Commitment

This is my line in the sand:

**Every new bounded context gets its own service.**

The tools exist to make this practical. The patterns are established. The only remaining barrier is discipline.

GenAI hasn't just changed how we write code. It's changed which architectural compromises are acceptable. The old excuses don't apply anymore.

No more "just add it to the existing service for now."
No more monoliths, even small ones.

The JSNTM pledge: Just Say No to Monoliths.

---

*This post is part of a series on leveraging GenAI tools for better software architecture. For related reading on GenAI-assisted development practices, see [Achieving 4x+ Productivity Gains with GenAI Coding Agents](https://www.nathanfox.net/p/achieving-4x-productivity-gains-claude-code) and [Planning-First Development with Claude Code](https://www.nathanfox.net/p/planning-first-development-claude-code).*

---

*This post was written with Claude Code. I described the concept, provided context from my own architecture decisions, and Claude helped draft and structure the content. I reviewed and edited the result. The logo was generated with ChatGPT. This is how I work now. You can see the revision history in my [blog posts repo](https://github.com/nathanfox/nathan-fox-net-posts).*
