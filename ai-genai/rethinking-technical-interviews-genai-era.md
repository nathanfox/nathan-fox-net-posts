# Rethinking Technical Interviews in the GenAI Era

## The Problem We're Facing

Something strange happened during our recent hiring round. We received a flood of applications—more than usual—and noticed a peculiar pattern. Many resumes hit all the right keywords, demonstrated broad technical knowledge, and read impressively well. Too well.

Then came the interviews.

In three separate interviews, we uncovered candidates clearly using GenAI assistance without disclosure. The tell? They knew what Reservoir Sampling was.

Let me explain. I have an old interview question: select a random line from a text file for a "quote of the day" feature. Simple enough. Then comes the constraint—you can only read through the file once, and you don't know the total line count ahead of time.

This question isn't about getting the right answer. In my entire career, exactly one person solved it correctly: a Ph.D. who had written a book on Lambda Calculus. The question exists to watch candidates *think*. How do they approach an unfamiliar problem? Do they ask clarifying questions? Can they reason through edge cases? Do they recognize when they're stuck and pivot?

But suddenly, multiple candidates were confidently explaining Reservoir Sampling—the textbook-perfect algorithm that solves this exact problem. Not fumbling toward it. Not discovering it through reasoning. Just... knowing it. With the kind of precision that comes from asking an AI moments before answering—which, in these cases, they were doing in real time.

This isn't about judging people for using AI. It's about a fundamental mismatch: **our interview processes were designed to test knowledge that GenAI now commoditizes**.

## The Knowledge Testing Paradox

Traditional technical interviews test things like:
- Can you explain how a hash map works?
- What's the time complexity of this algorithm?
- How does garbage collection work in language X?
- Write a function to reverse a linked list

Here's the uncomfortable truth: **GenAI can answer all of these better than most humans**. Not just adequately—better. With more precision, more edge cases covered, and more nuance than even experienced developers typically provide off the cuff.

So what are we actually testing? The ability to memorize information that's instantly accessible to anyone with a GenAI tool?

This feels similar to testing someone's arithmetic skills when they'll always have a calculator on the job. Yes, understanding the fundamentals matters. But is rote recall the skill that will determine their effectiveness in a modern development environment?

The same question applies to pair coding sessions. We sit candidates down, share a screen, and watch them write code in real-time. It's supposed to reveal how they think, how they collaborate, how they approach problems. But in practice? They'll never code this way on the job. They'll have an AI agent in their IDE, suggesting completions, generating functions, catching errors before they happen.

Are we testing a skill they'll actually use, or are we testing their ability to perform without the tools they'll have every single day?

## A Different Approach: The Live AI-Assisted Build

What if instead of testing what candidates know, we tested **how effectively they work with GenAI**?

Here's a format I'm considering:

### The Setup
- 90 minutes, live session with screen sharing
- Candidate uses their preferred GenAI tools (Claude, ChatGPT, Copilot, etc.)
- They're given requirements for a small but complete application
- The goal: get something working that meets the requirements

### What We'd Observe

**Planning Behavior**
- Do they immediately start prompting for code?
- Do they first discuss requirements and architecture with the AI?
- Do they create any form of plan before implementation?
- How do they handle ambiguous requirements?

**Prompting Effectiveness**
- How specific are their prompts?
- Do they provide context effectively?
- How do they handle AI mistakes or misunderstandings?
- Do they iterate effectively when output isn't right?

**Technical Judgment**
- Can they evaluate if AI-generated code is good?
- Do they catch bugs or security issues?
- Do they understand the code well enough to modify it?
- Can they explain what the code does and why?

**Problem-Solving Approach**
- How do they break down the problem?
- Do they test as they go or all at the end?
- How do they handle unexpected issues?
- What's their debugging process when AI suggestions don't work?

## The Challenges

This approach isn't without significant problems:

### The Fairness Question
Different candidates have different levels of GenAI experience. Someone who's been using Claude Code daily for six months will naturally outperform someone using ChatGPT for the first time. Is that fair?

Then again, is it fair to test algorithm implementation when some candidates grind LeetCode for months while others don't?

### The Evaluation Criteria
What actually makes someone "good" at AI-assisted development? We'd need to define rubrics for:
- Planning quality
- Prompting effectiveness
- Code comprehension
- Technical judgment
- Problem decomposition

These are harder to evaluate than "did the algorithm pass the test cases."

### The Time Factor
Can someone actually build something meaningful in 90 minutes, even with AI assistance? The scope would need to be carefully calibrated—complex enough to reveal problem-solving skills, simple enough to be achievable.

### The Reproducibility Problem
Unlike algorithm questions with clear correct/incorrect answers, evaluating AI-assisted development is inherently subjective. Two evaluators might assess the same session differently.

## What This Interview Might Reveal

Despite the challenges, I think this format could uncover things traditional interviews miss:

**Adaptability**: How quickly do they adjust when their first approach doesn't work?

**Communication**: Can they effectively communicate with both AI and humans about technical concepts?

**Quality Sense**: Do they accept the first thing the AI generates, or do they critically evaluate output?

**Systematic Thinking**: Do they approach problems methodically or chaotically?

**Learning Speed**: How quickly do they pick up on what works and what doesn't with AI assistance?

These feel like the skills that will actually matter for the next decade of software development.

## The Uncomfortable Questions

I keep coming back to some fundamental tensions:

**Are we hiring for today or tomorrow?** Someone who's excellent at traditional coding but mediocre at AI collaboration might be less valuable than the reverse in 2-3 years.

**What is "real" skill now?** If AI handles most code generation, is deep algorithmic knowledge a prerequisite or a nice-to-have? Is "prompt engineering" a real skill or a temporary artifact?

**How do we value human judgment?** Perhaps the most important skill is knowing when to trust AI output and when to be skeptical. How do you test that in 90 minutes?

**What about fundamentals?** There's an argument that you need to understand code deeply to evaluate AI-generated code effectively. Should we still test fundamentals separately?

## Where I'm Landing (For Now)

I don't have answers, but I have a direction I want to explore:

**A two-part interview:**

1. **Traditional Technical Discussion (60 min)**: Not algorithm coding, but a conversation about past projects, technical decisions, tradeoffs they've navigated. This reveals depth of experience that AI can't fake in real-time conversation.

2. **Live AI-Assisted Build (60-90 min)**: The format described above. See how they actually work, not just what they know.

The combination might give us:
- Evidence of genuine experience (hard to fake in real-time discussion)
- Insight into how they'll actually work day-to-day
- Assessment of both foundational knowledge and modern tooling skills

## An Open Question

I'm genuinely uncertain about this. The traditional interview is broken for the AI era, but we don't yet know what the replacement looks like.

What I do know:
- Testing pure knowledge recall is increasingly meaningless
- How someone works with AI tools is increasingly relevant
- The best developers will use AI as a multiplier, not a replacement for thinking
- The worst AI-assisted code comes from people who don't understand what they're generating

The interview process needs to evolve. I'm not sure this is the right evolution, but it feels closer to testing what actually matters.

