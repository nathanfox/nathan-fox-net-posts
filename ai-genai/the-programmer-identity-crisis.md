# The Programmer Identity Crisis: What Do We Call Ourselves Now?

## A Note on How This Was Written

I wrote this blog post with Claude Code. I described what I wanted to explore, we discussed the structure together, and it generated the draft. I'm editing as we go. Even writing—documentation, blog posts, prose—is no longer "writing" in the classic sense. Which rather proves the point.

## The Title That No Longer Fits

For decades, the answer was simple. You wrote code. You were a programmer, a developer, a software engineer. The title matched the activity. But something has shifted.

With GenAI coding agents like Claude Code, Cursor, and GitHub Copilot, the day-to-day reality of building software looks fundamentally different. You might spend more time describing what you want than typing the implementation. You review and refine more than you write from scratch. You orchestrate and direct rather than line-by-line construct.

So what are we now?

## The Framings We're Trying On

The industry hasn't settled on new terminology, but several framings are emerging:

**Code Orchestrator** - You're conducting a symphony. Multiple AI agents, different specialized tools, various codebases. Your job is directing their focus, combining their outputs, ensuring coherence. Less about writing every line, more about coordinating the ensemble.

**Intent Translator** - The core skill becomes articulating *what* you want with enough precision that AI can execute it. You're bridging human goals to machine-executable specifications. Vague requirements produce vague code. Precise intent produces working software.

**Code Curator** - Like an editor working with writers. You review, refine, accept, reject, and shape AI-generated code. Quality control and taste become paramount. The AI proposes; you dispose.

**Systems Architect** - You focus on high-level design, constraints, integration patterns. The "what" and "why" over the "how." Implementation details get delegated while you hold the vision.

**Verification Engineer** - Your value is knowing what *correct* looks like. You validate outputs, catch edge cases the AI misses, ensure the generated code actually does what it claims. The AI writes; you verify.

None of these fully capture it. Perhaps because the role is genuinely multifaceted now.

## The New Skills That Matter

Whatever we call ourselves, certain capabilities have become more valuable:

**Specification Precision** - The better you describe what you want, the better the output. This isn't just "prompting"—it's requirements engineering compressed into conversation. You need to think clearly about edge cases, constraints, and success criteria *before* the code exists.

**Rapid Verification** - Can you quickly determine if generated code is correct? This requires reading code you didn't write, understanding patterns you didn't choose, and spotting subtle bugs in unfamiliar implementations. Review skills matter more than ever.

**System Thinking** - When you can generate components quickly, integration becomes the bottleneck. Understanding how pieces fit together, managing dependencies, seeing the whole system—these architectural skills compound.

**Tool Fluency** - Knowing which AI tool to use when, how to structure prompts for different agents, when to intervene versus let the AI continue. There's craft in working with these tools effectively.

**Domain Knowledge** - AI can generate code, but it can't know your business domain, your users, your constraints. Deep context remains human territory. The person who understands *why* we're building something guides *what* gets built.

## What Remains Unchanged

Despite the transformation, some fundamentals persist:

**Taste and Judgment** - Knowing good code from bad. Recognizing when a solution is elegant versus merely functional. Understanding when to optimize and when "good enough" actually is. AI can generate many solutions; choosing the right one requires judgment.

**Understanding Tradeoffs** - Every technical decision involves tradeoffs. AI can explain options, but weighing them against your specific context—team capabilities, timeline, maintenance burden, performance requirements—remains human work.

**Debugging Complex Systems** - When something breaks in production, you still need to understand the system deeply enough to diagnose it. AI can help, but the intuition about where to look, what to try, how systems fail—that's experience.

**Communication** - Explaining technical decisions to stakeholders, mentoring others, writing documentation that humans will read. The soft skills haven't softened.

**Ownership** - When the system fails at 3 AM, there's still a human on call. Accountability doesn't delegate to AI.

## The Leverage Shift

Here's what's genuinely new: the leverage curve has changed.

The old "10x engineer" meme suggested some developers were an order of magnitude more productive. It was exaggerated, but pointed at real variance in individual output.

Now? That leverage is accessible differently. A developer who deeply understands their domain, can specify precisely, and knows how to work with AI tools can produce output that previously required teams. Not because they're typing faster, but because they're directing effectively.

This cuts both ways:
- One person with AI can prototype what used to require a team
- But that person needs broader skills—you can't just be deep in one area
- The floor rises (everyone gets AI assistance) while the ceiling gets higher (mastery compounds with AI leverage)

The question isn't whether you can write code. It's whether you can direct its creation, verify its correctness, and integrate it into systems that work.

## So What Do We Call Ourselves?

Maybe the answer isn't a new title. Maybe it's accepting that "software engineer" or "developer" now encompasses a different set of activities than it did five years ago. The title persists; the job transforms.

Or maybe new terminology will emerge organically as the field matures. "Webmaster" gave way to more specific roles. Perhaps "programmer" will too.

For now, I'm comfortable with the ambiguity. The work is evolving faster than the vocabulary. What matters isn't what we call ourselves, but whether we're adapting to do the work effectively.

The identity crisis is real. But it might also be a feature, not a bug—a sign that we're in genuine transition rather than incremental change.

What we call ourselves matters less than what we're capable of doing. And on that front, the possibilities have never been larger.

## Where Do We Go From Here?

The good news: we're not starting from scratch. New practices are emerging that help us work effectively in this new paradigm. If you're navigating this transition, here are concrete approaches worth learning:

### Learn to Plan Before You Code

With AI agents, the quality of your planning directly determines the quality of your output. [Planning-First Development](https://www.nathanfox.net/p/planning-first-development-claude-code) explores how structured markdown planning documents become the foundation for focused AI-assisted development. The plan becomes the conversation starter, the context anchor, and the progress tracker.

### Use Tests as Guardrails

AI agents are powerful but need constraints. [Taming GenAI Agents with TDD](https://www.nathanfox.net/p/taming-genai-agents-like-claude-code) shows how test-driven development transforms AI from a wild code generator into a disciplined development partner. Write tests first, let AI implement to pass them, verify the results. The red-green-refactor cycle works even better with AI.

### Build From the Domain Out

When AI can generate boilerplate instantly, your focus shifts to what matters: the domain model. [Domain-First Development](https://www.nathanfox.net/p/domain-first-development) advocates building pure domain logic first, independent of infrastructure. AI excels at the repetitive persistence and API layers; you focus on the business rules that require human understanding.

### Capture Working Patterns

Once you solve a hard problem, don't lose that knowledge. [Example-Driven Development](https://www.nathanfox.net/p/example-driven-development-ai-agents) shows how to create working example repositories that AI agents can reference. Your past solutions become training data for future development—both for AI and for yourself.

### Create Safety Nets

Working with AI means rapid iteration, which means things will sometimes go wrong. [Using Git's Staging Area as Save Points](https://www.nathanfox.net/p/git-staging-ai-savepoints) provides a lightweight mechanism for checkpointing your work before letting AI make changes. Small technique, big confidence boost.

### Understand the Productivity Potential

This isn't incremental improvement. [Achieving 4x+ Productivity Gains](https://www.nathanfox.net/p/achieving-4x-productivity-genai-coding-agents) lays out what's actually achievable with the right tooling and mindset. The numbers are real, but they require investing in these new skills.

---

The programmer identity crisis is real. But crises often precede growth. The practices above aren't just coping mechanisms—they're the emerging craft of a new discipline. We're figuring it out as we go, and that's been true of this industry since the beginning.

What we call ourselves matters less than what we're capable of doing. And on that front, the possibilities have never been larger.
