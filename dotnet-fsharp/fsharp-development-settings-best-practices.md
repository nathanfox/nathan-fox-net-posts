# F# Settings for Claude Code: Configuring CLAUDE.md for Better F# Code Generation


## TL;DR

For best F# code generation with Claude Code, add a CLAUDE.md file to your project with clear style, architecture, and testing guidelines. Use discriminated unions, record-based services, proper indentation, parameterized queries, and domain-focused naming. See the comprehensive CLAUDE.md template at the end of this post for copy-paste guidance.

---

When using Claude Code (claude.ai/code) with F# projects, providing explicit guidance through a CLAUDE.md file can significantly improve the quality of generated code. Claude Code, like many AI coding assistants, may have certain formatting tendencies or code generation patterns that don't align with F# best practices or your team's preferences. This article presents a comprehensive set of F# settings and guidelines that can be placed in your project's CLAUDE.md file to help Claude Code generate more idiomatic and consistent F# code.

**Note:** These settings address current Claude Code behavior and may need adjustment as the AI models evolve. Feel free to customize these guidelines to match your team's specific coding standards and preferences.

## Project Configuration Essentials

## Code Style Best Practices

### 1. Prefer Discriminated Unions Over Enums

Discriminated unions provide better type safety and more functional style:

```fsharp
// ✅ Preferred - Discriminated Union
type DifficultyClass = 
    | Trivial | Easy | Moderate | Hard | VeryHard | Legendary
    
let getDifficultyValue = function
    | Trivial -> 5 | Easy -> 8 | Moderate -> 12
    | Hard -> 15 | VeryHard -> 18 | Legendary -> 25

// ❌ Avoid - Enum  
type DifficultyClass = Trivial = 5 | Easy = 8 | Moderate = 12
```

### 2. Service Architecture: Record-Based Pattern

Never use classes with constructors for services in F#. Instead, use record types with function fields:

```fsharp
// ✅ Preferred: Record-based services  
type DataProcessingService = {
    validateInput: string -> Context -> Async<Result<ValidatedData, string>>
    processData: ValidatedData -> IProcessor -> Async<Result<ProcessingResult, string>>
    generateReport: ProcessingResult -> Context -> Async<Result<Report, string>>
}

// Init function to create the service
let createDataProcessingService (provider: IDataProvider) (processor: IProcessor) : DataProcessingService =
    {
        validateInput = fun input context -> async { ... }
        processData = fun data proc -> async { ... }
        generateReport = fun results context -> async { ... }
    }

// ❌ Avoid: Class-based services
type DataProcessingService(provider: IDataProvider, processor: IProcessor) =
    member _.validateInput input context = ...
```

### 3. Match Expression Indentation

Proper indentation prevents compiler errors:

```fsharp
// ✅ Correct indentation
let resultType =
    match command.Type with
    | ProcessCommand _ -> "Process" 
    | ValidateCommand _ -> "Validate"

// ❌ Causes compiler error
let resultType = match command.Type with
    | ProcessCommand _ -> "Process"
    | ValidateCommand _ -> "Validate"
```

### 4. F# List Syntax for Complex Expressions

When creating lists with complex expressions, use proper indentation:

```fsharp
// ✅ Correct
let items = 
    [ match value with
      | Case1 -> "result1" 
      | Case2 -> "result2"
      
      if condition then
          "extra"
      else
          ""
    ]
    |> List.filter (fun s -> not (String.IsNullOrEmpty s))

// ❌ Incorrect (causes compiler errors)
let items = [
    match value with
    | Case1 -> "result1"
    | Case2 -> "result2"
]
```

## Testing Configuration

### Expecto Test Framework Settings

When using Expecto for testing, remember these critical settings:

```fsharp
// ALWAYS use dotnet run instead of dotnet test for Expecto
// Example commands:
// cd tests/MyProject.Tests && dotnet run
// cd tests/MyProject.Tests && dotnet run -- --filter-test-case "specific test"
// cd tests/MyProject.Tests && dotnet run -- --filter "Test Category"
```

### Async Testing Best Practices

```fsharp
// ✅ Preferred: Proper async testing
testCaseAsync "Can fetch data" <| async {
    let! result = fetchDataAsync()
    Expect.equal result.Status "success" "Should succeed"
}

// ❌ Avoid: Blocking async operations
testCase "Can fetch data" <| fun _ ->
    let result = fetchDataAsync() |> Async.RunSynchronously
    Expect.equal result.Status "success" "Should succeed"
```

## API Error Handling Pattern

All API calls should return `Async<Result<T, string>>` for consistent error handling:

```fsharp
// API Contract
type IDataAPI = {
    createSession: UserId -> Async<Result<SessionId, string>>
    getSessionState: SessionId -> Async<Result<SessionState, string>>
}

// Server Implementation
let createSession userId = async {
    try
        // implementation
        return Ok sessionId
    with
    | ex -> return Error ex.Message
}

// Client Usage
match! api.createSession userId with
| Ok sessionId -> 
    // handle success
| Error err -> 
    // handle error
```

## Code Organization Principles

### Types in Namespaces, Functions in Modules

```fsharp
namespace MyProject.Domain

// Types defined at namespace level
type User = {
    Id: UserId
    Name: string
    Status: int
}

type ApplicationState = 
    | Active of User list
    | Completed of result: User

module UserLogic =
    // Functions in modules
    let createUser name = 
        { Id = UserId.create(); Name = name; Status = 1 }
    
    let updateStatus user =
        { user with Status = user.Status + 1 }
```

## Security Best Practices

### Parameterized Queries (Critical for Database Operations)

```fsharp
// ✅ ALWAYS: Use parameterized queries
let queryText = "SELECT * FROM c WHERE c.userId = @userId AND c.documentType = @docType"
let queryDefinition = 
    QueryDefinition(queryText)
        .WithParameter("@userId", string userId)
        .WithParameter("@docType", "User")

// ❌ NEVER: String interpolation (SQL injection risk)
let query = $"SELECT * FROM c WHERE c.userId = '{userId}'"
```

## Function Size Guidelines

Keep functions concise and focused:

- **10-15 lines**: Ideal for most functions
- **20-25 lines**: Still acceptable, but consider refactoring
- **30+ lines**: Should be refactored unless there's a specific reason
- **Rule of thumb**: If you need to scroll to see the whole function, it's probably too long

## Interpolated String Handling

Handle F# interpolated string compiler warnings properly:

```fsharp
// For complex interpolations with conditionals
let status = if result.Success then "Success" else "Failure"
$"Skill check: {status} (rolled {result.RollValue}, gained {result.ExperienceGained} XP)"

// For triple-quote strings when needed
$"""Step {result.StoryStep}: {if result.Success then "✓" else "✗"}"""

// Escape percentage signs in format specifiers
$"Success Rate: {float successfulSteps / float totalSteps * 100.0:F1}%%"
```

## Naming Conventions

Avoid phase-based or temporal naming in code:

```fsharp
// ❌ Avoid: Phase-based names
Phase2CoreSystemsTests.fs
type Phase2Command = ...
let phase2Implementation () = ...

// ✅ Preferred: Domain-focused names
DataValidationTests.fs
ProcessingSystemTests.fs
type ValidationCommand = ...
type ProcessingCommand = ...
let processValidationAction () = ...
```

## Development Workflow

### Test-Driven Development Approach

1. Generate code and types with stubbed return values
2. Create unit tests that check for expected values
3. Implement functions progressively
4. Run tests to verify implementation
5. When modifying existing functions, ensure tests exist first

### Branch Protection and Git Workflow

- Never push directly to protected branches (main, develop)
- Create feature branches for new work
- Use pull requests for code review
- Use the `gh` CLI tool for GitHub operations

## Performance Considerations

### Async and Parallel Processing

```fsharp
// Parallel processing pattern
let! resultsArray = 
    batch.Commands
    |> List.map processor.validateData
    |> Async.Parallel

let results = Array.toList resultsArray
```

## Logging Configuration

### Structured Logging with Serilog

```fsharp
// Always use structured parameters
Log.Information("Processing user input for session {SessionId}", sessionId)

// Not string interpolation
// ❌ Bad: Log.Information($"Processing user input for session {sessionId}")

// Include context in errors
Log.Error(ex, "Failed to process command {CommandId} for session {SessionId}", cmdId, sessionId)
```

## Conclusion

These F# development settings and best practices have been refined through building complex, production-ready F# applications. By following these guidelines, you'll create more maintainable, performant, and idiomatic F# code. Remember that consistency is key - pick conventions that work for your team and stick to them throughout your project.

The settings and patterns presented here aren't just theoretical - they come from real-world experience building F# applications that need to scale, be maintainable, and work reliably in production environments. Whether you're building web APIs with Giraffe, SPAs with Elmish, or complex domain models, these practices will help you write better F# code.

## CLAUDE.md Template for F# Projects

Below is a comprehensive template you can paste into your project's CLAUDE.md file to provide guidance when working with Claude Code on F# projects. This template helps address common formatting and code generation issues, ensuring Claude Code produces F# code that follows best practices and your preferred conventions:

<pre><code># CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## F# Code Style Preferences

### F# List Syntax for Complex Expressions
When creating lists with complex expressions (like match statements or conditionals), use proper indentation to avoid syntax errors:

**❌ Incorrect (causes compiler errors):**
```fsharp
let items = [
    match value with
    | Case1 -> "result1"
    | Case2 -> "result2"
    
    if condition then "extra"
    else ""
]
```

**✅ Correct:**
```fsharp
let items = 
    [ match value with
      | Case1 -> "result1" 
      | Case2 -> "result2"
      
      if condition then
          "extra"
      else
          ""
    ]
    |> List.filter (fun s -> not (String.IsNullOrEmpty s))
```

### Code Organization Principles
- **Types in namespaces**: Domain types, data structures, and shared contracts should be defined at the namespace level for maximum accessibility
- **Functions in modules**: Implementation logic, utility functions, and business operations should be organized in modules  
- **Exception**: When a type is tightly coupled to specific module functionality and not needed elsewhere, it can be defined within the module

This separation provides:
- Clear boundaries between data definitions and operations
- Better discoverability of types across the codebase  
- Cleaner module interfaces focused on behavior rather than data structure

### Discriminated Unions Over Enums
- **Prefer discriminated unions** over enums for better F# idiomatic code and type safety
- **Use mapping functions** to convert discriminated union values to primitives when needed
- **Example**: 
  ```fsharp
  // ✅ Preferred - Discriminated Union
  type StatusLevel = 
      | Draft | InReview | Approved | Published | Archived | Deprecated
      
  let getStatusPriority = function
      | Draft -> 1 | InReview -> 2 | Approved -> 3
      | Published -> 4 | Archived -> 5 | Deprecated -> 6
  ```
  ```fsharp
  // ❌ Avoid - Enum  
  type StatusLevel = Draft = 1 | InReview = 2 | Approved = 3
  ```
- **Benefits**: Better pattern matching, no incomplete cases warnings, more functional style, easier testing

### F# Match Expression Indentation
- **Pattern**: When using match expressions in let bindings, proper indentation prevents compiler errors
- **Issue**: `let value = match expr with | Case1 -> ...` causes "offside" errors
- **Solution**: Put match on new line with consistent indentation
- **Examples**:
  ```fsharp
  // ❌ Causes compiler error
  let resultType = match command.Type with
      | ProcessCommand _ -> "Process"
      | ValidateCommand _ -> "Validate"
  
  // ✅ Correct indentation
  let resultType =
      match command.Type with
      | ProcessCommand _ -> "Process" 
      | ValidateCommand _ -> "Validate"
  ```
- **Rule**: Match keyword should align with assignment (=) indentation level, with cases further indented

### Service Architecture: Record-Based Pattern
- **NEVER use classes with constructors and members for services** - this is not idiomatic F#
- **ALWAYS use record types with function fields** for service abstractions and implementations
- **Use init functions** to create service instances with proper dependency injection and defaults

#### ❌ Avoid: Class-based services
```fsharp
// Don't do this - not functional style
type DataProcessingService(provider: IDataProvider, processor: IProcessor) =
    member _.validateInput input context = ...
    member _.processData data processor = ...
```

#### ✅ Preferred: Record-based services  
```fsharp
// Service contract as record with function fields
type DataProcessingService = {
    validateInput: string -> Context -> Async&lt;Result&lt;ValidatedData, string&gt;&gt;
    processData: ValidatedData -> IProcessor -> Async&lt;Result&lt;ProcessingResult, string&gt;&gt;
    generateReport: ProcessingResult -> Context -> Async&lt;Result&lt;Report, string&gt;&gt;
    updateContext: Context -> ProcessingResult -> Context
    getAvailableOperations: Context -> UserProfile -> Async&lt;Operation list&gt;
}

// Init function to create the service
let createDataProcessingService (provider: IDataProvider) (processor: IProcessor) : DataProcessingService =
    let validateInputImpl input context = async {
        // implementation
    }
    
    let processDataImpl data proc = async {
        // implementation  
    }
    
    // Return record with function implementations
    {
        validateInput = validateInputImpl
        processData = processDataImpl
        generateReport = generateReportImpl
        updateContext = updateContextImpl
        getAvailableOperations = getAvailableOperationsImpl
    }
```

#### Benefits of Record-Based Services:
- **Pure functional approach** - no mutable state, no side effects in constructors
- **Easier testing** - can easily mock individual functions or create test doubles
- **Better composition** - services can be combined and extended naturally
- **Dependency injection** - dependencies passed to init functions, not stored in mutable fields
- **Immutable by default** - service records are immutable, implementations can be pure functions
- **F# idiomatic** - follows functional programming principles consistently

### API Error Handling
- **ALL API calls must return `Async&lt;Result&lt;T, string&gt;&gt;`** where T is the success type
- **Server implementations** must wrap all operations in try/catch blocks and return proper Result types
- **Client-side calls** must handle Result types with pattern matching: `match! api.method() with | Ok result -&gt; ... | Error err -&gt; ...`
- **Never use `failwith` in API implementations** - always return `Error "description"` instead
- This ensures consistent, predictable error handling across the entire application
- Example API contract: `createSession: UserId -> Async&lt;Result&lt;SessionId, string&gt;&gt;`
- Example server implementation: `try ... return Ok result with | ex -> return Error ex.Message`
- Example client usage: `match! api.createSession userId with | Ok sessionId -> return sessionId | Error err -> return failwith err`

### Function Size Guidelines
- **10-15 lines**: Ideal for most functions - strive for this range
- **20-25 lines**: Still acceptable, but consider breaking down into smaller functions
- **30+ lines**: Should be refactored into smaller functions unless there's a specific reason
- **Exceptions**: Sequential operations that are tightly coupled, domain-specific algorithms that lose meaning when split, or extensive pattern matching blocks
- **Rule of thumb**: If you need to scroll to see the whole function, it's probably too long
- **Priority**: Readability and single responsibility over strict line count - focus on functions that do one thing well

## Testing Configuration

### Expecto Test Framework
- **ALWAYS use `dotnet run` instead of `dotnet test`** for running Expecto tests
- This project uses Expecto testing framework with executable test projects
- `dotnet test` doesn't work properly with Expecto's filtering and output

### Expecto Test Filtering
- Running tests by filter for Expecto:
  - `--filter`: Filters the list of tests by a hierarchy that's slash (/) separated
  - `--filter-test-list`: Filters the list of test lists by a given substring
  - `--filter-test-case`: Filters the list of test cases by a given substring

### Async Testing Best Practices
- **ALWAYS use `testCaseAsync` for async operations** instead of `testCase` with `Async.RunSynchronously`
- **NEVER use `Async.RunSynchronously` in tests** - this blocks threads and can cause deadlocks
- Example:
  ```fsharp
  // ❌ Avoid: Blocking async operations
  testCase "Can fetch data" &lt;| fun _ ->
      let result = fetchDataAsync() |> Async.RunSynchronously
      Expect.equal result.Status "success" "Should succeed"
  
  // ✅ Preferred: Proper async testing
  testCaseAsync "Can fetch data" &lt;| async {
      let! result = fetchDataAsync()
      Expect.equal result.Status "success" "Should succeed"
  }
  ```

## F# Interpolated Strings Compiler Warnings

### Interpolated String Handling Strategy
- Avoid direct interpolation in single-quote or verbatim strings
- Use explicit `let` bindings for complex interpolation scenarios
- Prefer triple-quote strings for complex interpolated expressions
- **Example Fix for Compiler Warning 3373**:
  ```fsharp
  // ❌ Problematic: Direct complex interpolation
  $"Processing: {if result.Success then "Success" else "Failure"} (value {result.Value}, time {result.ElapsedMs} ms)"

  // ✅ Corrected: Using let binding for status
  let status = if result.Success then "Success" else "Failure"
  $"Processing: {status} (value {result.Value}, time {result.ElapsedMs} ms)"
  ```
- Apply this pattern consistently across result formatting in pattern matching contexts
- Ensures clear, compiler-friendly string interpolation
- Improves readability by extracting complex conditions into named variables

### Specific Compiler Warning 3373 Resolution
- **Problem**: Invalid interpolated string with single quote or verbatim string literals
- **Solution**: Use triple-quote strings for complex interpolated expressions
- **Specific Example**:
  - Problematic: `$"Step {result.Step}: {if result.Success then "✓" else "✗"}"`
  - Corrected: `$"""Step {result.Step}: {if result.Success then "✓" else "✗"}"""`
- This fixes the compiler error by using a triple-quote string literal
- Ensures clean string interpolation for result formatting

### Compiler Warning 3376 - Missing Format Specifier
- **Problem**: Percentage sign (%) in interpolated string requires escaping when used with format specifiers
- **Example Error**: `$"Success Rate: {float successful / float total * 100.0:F1}%"`
  - Error: "Invalid interpolated string. Missing format specifier F# Compiler3376"
- **Solution**: Double the percentage sign to escape it: `%%`
- **Corrected Example**: `$"Success Rate: {float successful / float total * 100.0:F1}%%"`
- This ensures the format specifier `:F1` is properly recognized and the `%` character is displayed correctly

## Async and Parallel Processing Patterns

### Resolving Async Parallel Mapping Challenges
- When working with `Async.Parallel` and needing to convert results, use the following pattern:
  ```fsharp
  let! resultsArray = 
      batch.Commands
      |> List.map processor.validateData
      |> Async.Parallel
  
  let results = Array.toList resultsArray
  ```
- This resolves compiler errors related to missing `.map()` method on `Async`
- Ensures correct parallel processing of commands with result conversion
- Maintains type safety and functional composition of async workflows

## Security Best Practices

### Parameterized Queries (REQUIRED)
- **CRITICAL**: ALL database queries MUST use parameterized queries to prevent SQL injection
- **NEVER use string interpolation** in query strings - this creates security vulnerabilities
- **ALWAYS use parameterized query methods** for dynamic values

#### ❌ NEVER Do This (Security Vulnerability):
```fsharp
// DON'T: String interpolation creates SQL injection risk
let query = $"SELECT * FROM c WHERE c.userId = '{userId}' AND c.documentType = 'User'"
let queryDefinition = QueryDefinition(query)
```

#### ✅ ALWAYS Do This (Secure):
```fsharp
// DO: Use parameterized queries for security
let queryText = "SELECT * FROM c WHERE c.userId = @userId AND c.documentType = @docType"
let queryDefinition = 
    QueryDefinition(queryText)
        .WithParameter("@userId", string userId)
        .WithParameter("@docType", "User")
```

### Discriminated Union Query Syntax
- **Issue**: F# discriminated unions serialize to JSON with lowercase field names and reserved keywords
- **Solution**: Use square bracket notation for reserved keywords and correct case with parameterized queries
- **Examples**:
  ```fsharp
  // ✅ Correct: Secure parameterized query with proper JSON field access
  let queryText = "SELECT * FROM c WHERE c.data.status['case'] = @statusType AND c.data.status['fields'][0] = @statusId"
  let queryDefinition = 
      QueryDefinition(queryText)
          .WithParameter("@statusType", "Active")
          .WithParameter("@statusId", statusId)
  ```
- **Key Points**:
  - Use square bracket notation for reserved keywords: `['case']`, `['fields']`
  - Always parameterize dynamic values: `@statusId` instead of string interpolation
  - This matches the actual JSON serialization format used by System.Text.Json

## Naming Conventions

### Avoid Phase-Based Naming in Code
- **NEVER use "Phase1", "Phase2", etc. in file names, type names, or function names** - these lose meaning over time
- **NEVER use phase references in code comments** - comments should explain what code does, not when it was implemented
- **Use descriptive, domain-focused names** that explain what the code actually does
- **Phase references are acceptable only in documentation and commit messages** where they provide implementation context

#### ❌ Avoid: Phase-based names and comments
```fsharp
// Don't name files, types, or use phase comments
Phase2CoreSystemsTests.fs
type Phase2Command = ...
let phase2Implementation () = ...

// Phase 3: Enhanced features (bad comment)
type EnhancedFeature = ...
```

#### ✅ Preferred: Domain-focused names and comments
```fsharp
// Name based on what the code actually does
DataValidationTests.fs
ProcessingSystemTests.fs
NotificationSystemTests.fs
type ValidationCommand = ...
type ProcessingCommand = ...
let processValidationAction () = ...

// User Authentication and Authorization System (good comment)
type AuthenticationPattern = ...

// Event Processing Pipeline (good comment)  
type EventPipeline = ...
```

#### Benefits of Domain-Focused Naming:
- **Self-documenting** - code explains its purpose immediately
- **Maintainable** - easy to find relevant code when fixing bugs or adding features
- **Future-proof** - names remain meaningful as the codebase evolves
- **Clear separation** - each system has distinct, recognizable boundaries
- **Professional** - code reads like a domain model, not a project timeline

## Development Workflow

### Test-Driven Development Approach
1. Generate code and types with stubbed return values
2. Create unit tests that check for expected values
3. Implement functions progressively
4. Run tests to verify implementation
5. When modifying existing functions, ensure tests exist first

### Branch Protection and Git Workflow
- Never push directly to protected branches (main, develop)
- Create feature branches for new work
- Use pull requests for code review
- Use the `gh` CLI tool for GitHub operations

## Logging Configuration

### Structured Logging with Serilog
```fsharp
// Always use structured parameters
Log.Information("Processing input for session {SessionId}", sessionId)

// Not string interpolation
// ❌ Bad: Log.Information($"Processing input for session {sessionId}")

// Include context in errors
Log.Error(ex, "Failed to process {CommandId} for session {SessionId}", cmdId, sessionId)
```

## Performance Considerations

### Async and Parallel Processing
```fsharp
// Parallel processing pattern
let! resultsArray = 
    batch.Commands
    |> List.map processor.validateData
    |> Async.Parallel

let results = Array.toList resultsArray
```
</code></pre>