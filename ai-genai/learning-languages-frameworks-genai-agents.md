# Comparison-Driven Learning: Using GenAI Agents to Learn New Languages and Frameworks

## The Fastest Path from Zero to Productive

Over the past year, I've built production applications in four languages and frameworks I had never used before: Go, React Native, Flutter/Dart, and Svelte. Not toy projects or tutorials—real applications deployed to app stores and production environments.

How? By leveraging GenAI agents not just as code generators, but as personalized tutors that teach through comparison to what I already know.

## My Background: .NET and Vue.js Experience

I've spent decades in the .NET ecosystem—C#, F#, ASP.NET—and have solid experience with Vue.js for frontend development. When I decided to build [PulseURL](https://github.com/orgs/pulseurl/repositories) (a high-performance traffic monitoring service), [DevHours](https://www.nathanfox.net/p/devhours-a-time-tracking-app) (a time tracking app), [HayTracker](https://www.nathanfox.net/p/haytracker-planning-driven-development) (hay inventory management), and an [MFE shell example](https://www.nathanfox.net/p/jsntm-micro-frontends-svelte), I chose technologies completely outside my comfort zone:

- **Go** for PulseURL's backend services
- **React Native** for DevHours mobile app
- **Flutter/Dart** for HayTracker and PropaneTracker
- **Svelte** for the micro-frontend shell

Each required learning a new language, new runtime, new ecosystem, and new idioms. The traditional path—tutorials, documentation, Stack Overflow—would have taken months. Instead, I was productive within days.

## The Comparison-Driven Learning Method

The key insight is this: **GenAI agents can teach you a new language by constantly comparing it to what you already know**. This isn't just faster—it's deeper. You understand not just *what* to do, but *why* the language does it that way and *how* it relates to concepts you've already internalized.

### Step 1: Start with Planning-Driven Development

Before writing any code, I follow my [Planning-Driven Development](https://www.nathanfox.net/p/planning-driven-development) approach. The first conversation with the GenAI agent isn't about code—it's about choices:

```
Me: "I need to build a high-throughput traffic monitoring service with gRPC ingestion.
I'm most familiar with .NET/C#, but I want to use this as an opportunity to learn
a new language. What are my options and the pros/cons of each?"
```

The agent responded with four options—Go, Node.js, Rust, and Python—each with detailed pros and cons considering:
- Runtime characteristics
- Concurrency models
- Ecosystem maturity
- Learning curve from my .NET background
- Deployment considerations

I asked follow-up questions about Go specifically, and we concluded it would be an excellent learning experience while being particularly well-suited to the problem: goroutines for high concurrency, excellent gRPC support, and a straightforward deployment story.

This isn't about getting a "right" answer—it's about understanding the tradeoffs before committing, while being intentional about learning.

### Step 2: Establish Industry Standards

Once I choose a framework, the next question is crucial:

```
Me: "I've decided on Go for this project. What are the industry-standard
patterns and libraries for:
- Project structure
- HTTP routing
- Database access
- Configuration management
- Testing
- Error handling"
```

The response gives me the lay of the land. For Go, I learned about:
- The standard `cmd/` and `internal/` directory structure
- Popular routers like Chi and Gin
- The convention of `_test.go` files
- The Go error handling philosophy

This prevents me from accidentally adopting non-idiomatic patterns that would confuse experienced Go developers reviewing my code later.

### Step 3: Teach Through Comparison

Here's where the real learning happens. I tell the GenAI agent what I know:

```
Me: "I'm very familiar with C# and F#. As we work on this Go project,
please explain Go concepts by comparing them to their .NET equivalents.
Show me both the Go way and how I would have done it in C#."
```

For mobile/frontend work, I'd anchor to Vue.js instead:

```
Me: "I have experience with Vue.js. As we build this React Native app,
compare React patterns to Vue equivalents—components, state management,
lifecycle hooks, reactivity."
```

From this point forward, every explanation comes with a comparison:

**Go's goroutines vs. C#'s Tasks:**

```go
// Go: goroutine
go func() {
    result := fetchData()
    channel <- result
}()
```

```csharp
// C#: Task
Task.Run(async () => {
    var result = await FetchDataAsync();
    // result available here
});
```

The agent explains: "Goroutines are lighter than Tasks—Go can run millions of them. But unlike async/await, you communicate through channels rather than awaiting results directly."

This comparison-based teaching creates mental bridges. I'm not learning from scratch; I'm extending what I already know.

### Step 4: Deep Dive on Syntax and Semantics

When I encounter unfamiliar constructs, I ask:

```
Me: "What are goroutines exactly? How do they compare to Task.Run and
async/await in C#? What about F#'s async workflows?"
```

The response covers:
- What goroutines are (lightweight threads managed by Go runtime)
- How channels differ from awaiting results
- The select statement for multiplexing
- When to use buffered vs. unbuffered channels
- Common pitfalls coming from an async/await background

For Dart, I asked similar questions:

```
Me: "What does 'final' mean in Dart? How is it different from 'const'?
Compare to TypeScript's const and readonly."
```

```
Me: "What is a Future in Dart? How does it compare to Promises in JavaScript?"
```

Each answer deepened my understanding by anchoring new concepts to familiar ones.

### Step 5: Libraries and Ecosystem Navigation

Every language has its ecosystem. Rather than googling "best Go gRPC library," I ask:

```
Me: "I need to build a gRPC service in Go. I've never used gRPC before.
What are the standard libraries and patterns? How does it compare to
building REST APIs, which I'm familiar with?"
```

The agent explains Go's official gRPC library, protoc code generation, the request/response model, and how it differs from REST—all while teaching me both Go and gRPC simultaneously.

For Svelte, I asked about UI component libraries:

```
Me: "What UI component libraries are available for Svelte? I'm familiar with
Vuetify in Vue.js. What are the options and their pros/cons?"
```

After evaluating the options, we decided on shadcn-svelte. Then I learned that shadcn copies components into your codebase rather than installing as dependencies—a fundamentally different approach from Vuetify that makes sense once explained.

### Step 6: Read Every Line of Generated Code

This is critical: **never blindly accept generated code**. I read every file, every function, every line. When something is unclear:

```
Me: "In this Go code, what does 'defer' do? I see it before the
db.Close() call. What's the equivalent in C#?"
```

Answer: "defer schedules a function to run when the surrounding function returns—like a finally block or IDisposable.Dispose(), but more flexible. In C# you'd use 'using' or try/finally."

This questioning habit ensures I actually learn the language rather than just generating code I don't understand.

### Step 7: Discover Paradigm Differences

Sometimes, a new framework challenges your assumptions. When working with React Native, I noticed the code was constantly creating copies of objects rather than mutating them:

```
Me: "I notice React code keeps creating new object copies instead of mutating.
This looks like functional programming—immutability, pure functions. How does
this relate to patterns I know from F#?"
```

The agent explained React's functional philosophy:
- Components as pure functions of props and state
- Hooks as composable state machines
- Immutable state updates (spread operators, new arrays) vs. Vue's reactive mutations
- How this mirrors F#'s immutability-by-default approach

This wasn't just syntax translation—it was understanding a different paradigm. Vue's reactivity "just works" with direct assignment; React requires you to create new state objects. Once I recognized the F# connection, React's patterns clicked.

## Example Questions Across Projects

Here are examples of the kinds of comparison questions that accelerate learning in each stack:

### PulseURL (Go)

Coming from C#, Go felt minimalist. Examples of what to ask:

- "Go doesn't have exceptions. How do I handle errors compared to try/catch?"
- "What's the equivalent of dependency injection in Go?"
- "How do interfaces work in Go vs. C#? I see no 'implements' keyword."

Go's implicit interface satisfaction is worth understanding early—types implement interfaces automatically if they have the right methods. Coming from C#'s explicit `class Foo : IFoo`, this feels strange until you understand the design philosophy.

### DevHours (React Native)

React Native still uses TypeScript, so the language was familiar—but the React paradigm was new. Coming from Vue.js, examples of what to ask:

- "How does React's useState compare to Vue's ref() and reactive()?"
- "What's the equivalent of Vue's computed properties in React?"
- "How do React's useEffect hooks compare to Vue's watch and lifecycle hooks?"
- "Vue uses v-model for two-way binding. How does React handle form inputs?"

React's explicit state management (useState, useReducer) vs. Vue's reactivity system is the biggest mental shift. Understanding how React's unidirectional data flow differs from Vue's more magical reactivity is key.

### HayTracker (Flutter/Dart)

Flutter/Dart was completely new, but comparing to JavaScript/TypeScript patterns from Vue.js still helps. Examples of what to ask:

- "How does Flutter's StatefulWidget compare to a Vue component with reactive state?"
- "Dart has 'final' and 'const'. How do these map to TypeScript's const and readonly?"
- "What's a Future<T> and how does async/await work in Dart vs. JavaScript Promises?"
- "How does Flutter's build() method compare to Vue's template rendering?"

Flutter's "everything is a widget" philosophy and the widget tree takes adjustment. But Dart itself feels comfortable—the syntax has familiar async/await patterns from both JavaScript and C#.

### MFE Shell (Svelte)

Svelte's reactivity model actually feels closer to Vue than React. Examples of what to ask:

- "How does Svelte 5's $state and $derived runes compare to Vue's ref() and computed()?"
- "Svelte compiles away the framework. How is this different from Vue's virtual DOM?"
- "How do Svelte stores compare to Vue's Pinia or Vuex?"
- "How do I use shadcn-svelte? In Vue I'd npm install Vuetify."

Interestingly, Svelte's approach to reactivity feels more natural coming from Vue than React did. The runes (`$state`, `$derived`) map conceptually to Vue's `ref()` and `computed()`.

## The Compound Effect

After building four projects in four new stacks, something interesting happened: learning accelerated. Each new language became easier because:

1. **Pattern recognition improves**: "Oh, this is like goroutines in Go / Futures in Dart"
2. **Paradigm awareness grows**: Functional, OOP, and hybrid approaches become familiar
3. **Ecosystem navigation becomes intuitive**: Package managers, testing frameworks, and build tools follow patterns
4. **Questions become more precise**: I know what to ask about

## Am I an Expert? No. Do I Need to Be? Also No.

I'll be honest: I had never done mobile development before these projects. I'm not a Flutter/Dart expert. I don't have years of production debugging experience, deep knowledge of the widget lifecycle internals, or muscle memory for every Flutter package. I'm not a React Native expert either.

But I understand:
- Each language's philosophy and idioms
- How to read and understand Dart and React Native code
- Where to look when I encounter problems
- How to write idiomatic code rather than forcing familiar patterns where they don't belong

And crucially: I have two apps in the iOS App Store and Google Play Store built with Flutter/Dart—[HayTracker](https://www.nathanfox.net/p/haytracker-planning-driven-development) and [PropaneTracker](https://www.nathanfox.net/p/propanetracker-from-planning-doc-to-working-app-90-minutes).

As I continue working with each language, depth builds naturally. Each bug I fix, each feature I add deepens understanding. The GenAI-accelerated learning got me to productive—and for these projects, productive is exactly what I needed.

## The Method Summarized

1. **Start with planning**: Ask about language/framework choice with pros and cons
2. **Establish standards**: Learn industry-standard patterns before writing code
3. **Declare your background**: Tell the GenAI what you know so it can compare
4. **Learn through comparison**: Every new concept tied to a familiar one
5. **Ask about everything**: Syntax, semantics, runtime, libraries, idioms
6. **Read every line**: Don't accept code you don't understand
7. **Question paradigm differences**: Understand *why* the language works this way

## Conclusion

GenAI agents have fundamentally changed how I approach learning new technologies. Instead of months of tutorials and documentation, I can become productive in days by leveraging comparison-based learning.

The key is treating the GenAI agent not as a code generator, but as a tutor who knows both what you're learning and what you already know. Every explanation anchored to familiar concepts. Every new pattern compared to old ones. Every question answered with context.

The result: four production applications in four new stacks, built while actually understanding the code I was shipping.

---

*For the foundation of this approach, see [Planning-Driven Development: The Single Biggest Productivity Multiplier with GenAI Agents](https://www.nathanfox.net/p/planning-driven-development).*

*For examples of what's possible with this approach, see [DevHours](https://www.nathanfox.net/p/devhours-a-time-tracking-app), [HayTracker](https://www.nathanfox.net/p/haytracker-planning-driven-development), and [PropaneTracker](https://www.nathanfox.net/p/propanetracker-from-planning-doc-to-working-app-90-minutes).*
