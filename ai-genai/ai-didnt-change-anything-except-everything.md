# AI Didn't Change Anything (Except Everything)

## The Paradox

I've been following discussions about AI in software development, and I keep noticing something: none of the issues people raise are new.

Time pressure. Cognitive load. Code quality. Performance concerns. Economic reality. The tension between shipping fast and shipping right.

These existed long before AI. They exist now. They'll exist after.

What changed for me isn't accountability. It's feasibility—and time compression so dramatic it feels like a different job.

## The Problems That Never Left

The real problems in software development aren't about AI. They're about complexity, constraints, and tradeoffs we've been navigating for decades.

**Domain logic scattered across system layers.** Business rules split between UI validation, API handlers, scheduled jobs, application code, and stored procedures. To understand what the system actually does, you trace through five layers. To test it, you need integration tests for everything. This predates AI by decades.

**Monoliths that are hard to maintain and deploy.** Fragile systems where a change in one area breaks something unrelated. Deployments that require coordination across teams. Code that nobody wants to touch because the blast radius is unknown.

**The iron triangle: schedule, quality, scope.** Pick two. This constraint—articulated by Steve McConnell and others—hasn't changed. Every project negotiates between shipping fast, shipping well, and shipping everything. AI doesn't resolve that tension.

**Teams that are understaffed for the work expected.** Not enough people, too many priorities, constant context switching. The math never worked, but we shipped anyway.

No system is perfect, including anything I've developed. AI didn't create these problems. But here's what's interesting: AI changes one variable in the equation.

## What Actually Changed

Here's what's different: time collapsed.

I've helped others solve problems in 10 minutes that they'd spent days on. Multiple times. Not because I'm smarter, but because I can explore solution spaces faster.

I'm architecting and implementing production systems at a pace I couldn't achieve before. Not by skipping steps, but by compressing the mechanical work. The thinking still takes the same time. The typing doesn't.

Days became hours. Hours became minutes. The ceiling moved—what used to require a team, one person can now prototype. What used to take weeks of boilerplate, now takes hours of focused design work.

But here's what didn't change: **I'm still responsible for the systems I create.**

AI needs guidance. AI needs understanding. AI needs someone who knows what "correct" looks like—not just what "runs" looks like.

Code that runs isn't the same as code that works well. That distinction existed before AI, and it's even more critical now.

## The Accountability Paradox

I keep reading stories about developers who shipped AI-generated code that didn't work, code they didn't test, code they didn't understand. And my reaction is always the same:

*Why wouldn't they have done this anyway?*

Developers who ship untested code with AI would ship untested code without AI. Developers who don't understand what they're deploying didn't suddenly start when AI arrived.

AI didn't lower the bar. It rewards those who were already doing the work well.

The developers getting value from AI are the ones who were already doing the work: understanding requirements, validating implementations, owning their systems. AI accelerates their process. It doesn't replace their judgment.

## What I've Found Actually Works

I'm not going to pretend I have all the answers. But I know what works for me because I've done it—repeatedly, across different projects and domains.

**Planning-Driven Development.** The quality of your planning directly determines the quality of your output. I spend hours on planning documents before writing code. It sounds slow. It's the fastest approach I've found.

**Example-Driven Development.** Working examples that demonstrate the patterns you want. AI learns from examples better than from abstract instructions. So do humans.

**Discussing tradeoffs explicitly.** Architectural decisions, framework choices, pros and cons of every approach. AI is a surprisingly good collaborator for exploring decision spaces—but only if you engage it that way.

**Treating AI as a collaborator, not a replacement.** AI works *for* me. It doesn't work *instead* of me. That distinction changes everything about how you interact with these tools.

I've written extensively about these approaches elsewhere. The short version: the fundamentals of good software development—planning, testing, understanding your domain, owning your systems—matter more now, not less.

## Should I Even Post This?

I almost didn't write this. There's so much noise about AI. So many misconceptions. So many people talking past each other.

But maybe that's exactly why it's worth saying:

**Nothing fundamental changed about what makes software good or bad.** The same practices that worked before—planning, testing, understanding, ownership—still work. They just work faster now for those who invest the time to properly leverage GenAI agents.

**What changed is what's possible.** The scope of what one person can accomplish expanded dramatically. The speed of iteration accelerated. The barrier to exploring ideas dropped.

**But the responsibility stayed exactly where it was.** With the developer. With you. With me.

AI won't do your work for you. If you learn to make it work *for* you—not *instead* of you—the results are genuinely astounding.

But that requires doing the work. Same as it ever was.

---

*For the specific methodologies I've found effective:*
- *[Planning-Driven Development](https://www.nathanfox.net/p/planning-driven-development) - Why planning documents are the biggest productivity multiplier*
- *[Example-Driven Development](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code) - Using working examples to guide AI*
- *[Taming GenAI Agents with TDD](https://www.nathanfox.net/p/taming-genai-agents-like-claude-code) - Tests as guardrails for AI-assisted development*
