# Testing F# Elmish React Components with Fable: A Practical Approach

*How to test F# React components when AI assistance and traditional approaches fall short*

## The Challenge

Testing React components in F# with Fable and Elmish presents unique challenges. When I first attempted this with AI assistance, even advanced language models struggled to provide a working solution. Claude Code with Opus initially had trouble understanding the nuances of Fable's type system constraints. The initial breakthrough came from ChatGPT 5.0, which provided the key insight about avoiding type tests on ReactElement. This was then refined through continued prompting and iterating between Claude Code with Opus and GitHub Copilot with Claude Sonnet 4. The main issues that needed to be overcome were:

1. **Type system constraints**: F# and Fable's type system doesn't allow runtime type tests on erased types like `ReactElement`
2. **JavaScript interop complexity**: Extracting text from React elements requires careful handling of the JavaScript object model
3. **Limited documentation**: Most React testing examples are for JavaScript/TypeScript, not F#

## The Solution: Custom React Test Helpers

After the initial breakthrough from ChatGPT 5.0 about avoiding type tests, and several iterations between Claude Code with Opus and GitHub Copilot with Claude Sonnet 4, we developed a working approach using custom test helpers that properly extract text from React components rendered with Fable.

### Package Dependencies

First, add these dependencies to your test project's `package.json`:

```json
{
  "devDependencies": {
    "mocha": "^10.2.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  }
}
```

Note: React is needed as a dev dependency because the Fable.React components compile to JavaScript that expects React to be available.

### The ReactTestHelpers Module

Here's the complete helper module that enables testing React components in F#:

```fsharp
module TestUtils.ReactTestHelpers

open Fable.React
open Fable.Core
open Fable.Core.JsInterop

// Simple helper to extract text content from a ReactElement
// This is a basic implementation that extracts text from the element tree
let rec extractText (element: ReactElement) : string =
    // In JavaScript, we can directly access the properties
    let obj = element :> obj

    if isNull obj then
        ""
    else
        // Get children from props
        let children = obj?props?children

        if isNull children then
            ""
        else
            // Process children based on their type
            let childrenType = jsTypeof children

            if childrenType = "string" then
                // Direct string child
                unbox<string> children
            elif childrenType = "object" then
                // Could be array or single element
                if JS.Constructors.Array.isArray (children) then
                    // Array of children
                    let arr = unbox<obj array> children

                    arr
                    |> Array.map (fun child ->
                        if isNull child then
                            ""
                        else
                            let childType = jsTypeof child

                            if childType = "string" then
                                unbox<string> child
                            elif childType = "object" then
                                // Recursively extract from child element
                                try
                                    extractText (unbox child)
                                with _ ->
                                    ""
                            else
                                "")
                    |> String.concat " "
                else
                    // Single child element
                    try
                        extractText (unbox children)
                    with _ ->
                        ""
            else
                ""

// Helper to check if an element contains specific text
let containsText (text: string) (element: ReactElement) : bool =
    let content = extractText element
    content.Contains(text)

// Helper to check if element has a specific class name
let hasClassName (className: string) (element: ReactElement) : bool =
    let obj = element :> obj
    let cls = obj?props?className

    if isNull cls then
        false
    else
        let clsStr = string cls
        clsStr.Contains(className)

// Helper to count elements with specific class
let rec countElementsWithClass (className: string) (element: ReactElement) : int =
    let obj = element :> obj
    let selfCount = if hasClassName className element then 1 else 0

    let childCount =
        let children = obj?props?children

        if isNull children then
            0
        elif JS.Constructors.Array.isArray (children) then
            let arr = unbox<obj array> children

            arr
            |> Array.sumBy (fun child ->
                if isNull child then
                    0
                else
                    try
                        countElementsWithClass className (unbox child)
                    with _ ->
                        0)
        else
            try
                countElementsWithClass className (unbox children)
            with _ ->
                0

    selfCount + childCount

// Helper to find an element by its text content
let rec findElementByText (text: string) (element: ReactElement) : ReactElement option =
    if extractText element = text then
        Some element
    else
        let obj = element :> obj
        let children = obj?props?children

        if isNull children then
            None
        elif JS.Constructors.Array.isArray (children) then
            let arr = unbox<obj array> children

            arr
            |> Array.tryPick (fun child ->
                if isNull child then
                    None
                else
                    try
                        findElementByText text (unbox child)
                    with _ ->
                        None)
        else
            try
                findElementByText text (unbox children)
            with _ ->
                None

// Helper to check if element has a specific prop value
let hasProp (propName: string) (expectedValue: obj) (element: ReactElement) : bool =
    let obj = element :> obj
    obj?props?(propName) = expectedValue
```

## Key Implementation Details

### 1. Avoiding Type Test Errors

The critical insight was using `jsTypeof` instead of F# pattern matching with `:?` operator, which causes compilation errors with Fable:

```fsharp
// ❌ This causes "Cannot type test" errors
match box children with
| :? string as s -> s
| :? ReactElement as el -> extractText el

// ✅ This works correctly
let childrenType = jsTypeof children
if childrenType = "string" then
    unbox<string> children
elif childrenType = "object" then
    // handle object case
```

### 2. Handling JavaScript Arrays

React children can be a single element or an array. We handle this using JavaScript's `Array.isArray`:

```fsharp
if JS.Constructors.Array.isArray (children) then
    let arr = unbox<obj array> children
    // process array
else
    // process single element
```

### 3. Safe Recursion

Since we're traversing potentially deeply nested React elements, we wrap recursive calls in try-catch blocks to handle any unexpected structures gracefully.

## Example Usage

Here's a simple example of testing a todo list component:

```fsharp
module TodoList.Tests

open Fable.Mocha
open Fable.React
open Fable.React.Props
open TestUtils.ReactTestHelpers

// Simple todo list component
let todoListView (items: string list) =
    div [ ClassName "todo-list" ] [
        h2 [] [ str "My Todo List" ]
        ul [] [
            for item in items do
                li [ ClassName "todo-item" ] [ str item ]
        ]
        div [ ClassName "todo-count" ] [
            str (sprintf "%d items" (List.length items))
        ]
    ]

// Tests
let todoListTests =
    testList "TodoList View Tests" [
        
        test "renders todo items correctly" {
            let todos = ["Buy milk"; "Write blog post"; "Learn F#"]
            let element = todoListView todos
            
            // Check that all items are rendered
            Expect.isTrue (containsText "Buy milk" element) "Should show first item"
            Expect.isTrue (containsText "Write blog post" element) "Should show second item"
            Expect.isTrue (containsText "Learn F#" element) "Should show third item"
        }
        
        test "shows correct item count" {
            let todos = ["Task 1"; "Task 2"]
            let element = todoListView todos
            
            Expect.isTrue (containsText "2 items" element) "Should show item count"
        }
        
        test "renders with correct CSS classes" {
            let todos = ["Test"]
            let element = todoListView todos
            
            Expect.isTrue (hasClassName "todo-list" element) "Should have todo-list class"
            Expect.equal (countElementsWithClass "todo-item" element) 1 "Should have one todo-item"
        }
        
        test "handles empty list" {
            let element = todoListView []
            
            Expect.isTrue (containsText "0 items" element) "Should show zero items"
            Expect.equal (countElementsWithClass "todo-item" element) 0 "Should have no todo items"
        }
    ]
```

## Running the Tests

With Fable.Mocha, run your tests using:

```bash
npm test
```

Where your `package.json` test script is:
```json
{
  "scripts": {
    "test": "dotnet fable . -o dist && mocha dist/Tests.js"
  }
}
```

## Lessons Learned

1. **AI Collaboration Required**: Current AI models (as of 2025) struggle individually with the specific intersection of F#, Fable, and React testing. ChatGPT 5.0 provided the crucial insight about avoiding type tests, while Claude Code with Opus and GitHub Copilot with Claude Sonnet 4 helped refine the implementation. Initially, all models suggested using pattern matching with type tests (`:?` operator) which Fable doesn't support for `ReactElement` types.

2. **JavaScript Interop is Key**: The solution requires thinking in terms of JavaScript's runtime behavior rather than F#'s compile-time type system.

3. **Simple is Better**: Rather than trying to use complex React testing libraries, a simple text extraction approach covers most testing needs for Elmish applications.

4. **Type Annotations Matter**: Using `unbox` with explicit type parameters helps avoid type inference issues.

## Conclusion

Testing Elmish React components with Fable requires a different approach than traditional React testing. By understanding how Fable compiles F# to JavaScript and working with the grain of both type systems, we can create effective test helpers that enable comprehensive UI testing.

This approach has been successfully used to test complex UI components including progress bars, modal dialogs, and interactive forms, proving that F# React applications can have the same level of test coverage as their JavaScript counterparts.

## Resources

- [Fable Documentation](https://fable.io/)
- [Elmish Documentation](https://elmish.github.io/elmish/)
- [Fable.Mocha](https://github.com/Zaid-Ajaj/Fable.Mocha)

---

*Note: This solution was developed through collaborative AI assistance. ChatGPT 5.0 provided the initial breakthrough insight about avoiding type tests on ReactElement, which was then refined through iterative debugging with continued prompts between Claude Code (Opus) and GitHub Copilot (with Claude Sonnet 4). The final working implementation emerged from this multi-AI collaboration, where each assistant contributed different insights that eventually led to understanding the specific constraints of how Fable compiles F# to JavaScript.*