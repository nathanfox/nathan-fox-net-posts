# F# Active Patterns: Type-Safe Event Processing

## The Problem: Flexible Events with Strong Typing

While developing a game with multiple game modes, I faced an architectural challenge: how to handle mode-specific events in a type-safe way without creating rigid coupling between modes. Each game mode (Sealed Evil, Faction War, Time Loop, etc.) needed to process different events with different data, but I wanted to maintain F#'s strong type safety.

The initial suggestion from Claude Code Opus was to use strings as part of discriminated unions:

```fsharp
// Initial problematic approach
type GameEvent =
    | SealDiscovered of string  // "SealDiscovered_3"
    | BossPhase of string       // "BossPhase_2"
    | Custom of string          // Too generic!
```

This approach had several issues:
- Error-prone string parsing
- No compile-time validation
- Loss of type information
- Difficult to test and maintain

## The Solution: Active Patterns

Claude Code helped me discover that F# active patterns provide the perfect solution. They allow flexible string-based events while extracting strongly-typed values with validation.

## Understanding Active Patterns

Active patterns in F# are a powerful feature that allows you to define custom patterns for use in pattern matching. They act as parsers that can extract and transform data during the matching process.

### Basic Active Pattern Example

```fsharp
// Define an active pattern to extract seal discovery events
let (|SealDiscovered|_|) (input: string) =
    // Use regex to match pattern
    let pattern = @"SealDiscovered_(\d+)"
    let m = System.Text.RegularExpressions.Regex.Match(input, pattern)
    if m.Success then
        match System.Int32.TryParse(m.Groups.[1].Value) with
        | true, n when n >= 1 && n <= 12 -> Some n
        | _ -> None
    else
        None

// Usage in pattern matching
let processEvent (event: string) =
    match event with
    | SealDiscovered sealNum ->
        // sealNum is an int, validated to be 1-12
        printfn "Found seal number %d" sealNum
    | _ ->
        printfn "Unknown event"

// Examples
processEvent "SealDiscovered_3"   // Output: Found seal number 3
processEvent "SealDiscovered_99"  // Output: Unknown event (validation failed)
processEvent "RandomString"       // Output: Unknown event
```

## Real-World Implementation: Game Mode Events

Here's how I implemented active patterns for my game's event system:

### Flexible Event System with Type Safety

```fsharp
// Core events common to all game modes
type CoreMechanicalConsequence =
    | ItemDiscovered of itemType: string * itemId: string * quantity: int

// Core event type that supports both common and mode-specific events
type MechanicalConsequence =
    | Core of CoreMechanicalConsequence
    | ModeSpecific of eventType: string * data: Map<string, string>

// Sealed Evil mode events with active patterns
module SealedEvilEvents =
    open System
    
    /// Extract seal discovery events with validation
    let (|SealDiscovered|_|) = function
        | ModeSpecific("SealDiscovered", data) ->
            data.TryFind "sealNumber"
            |> Option.bind (fun s ->
                match Int32.TryParse(s) with
                | true, n when n >= 1 && n <= 12 -> Some n
                | _ -> None)
        | _ -> None
    
    /// Extract boss phase events
    let (|BossPhaseStarted|_|) = function
        | ModeSpecific("BossPhase", data) ->
            data.TryFind "phase"
            |> Option.bind (fun s ->
                match Int32.TryParse(s) with
                | true, p when p >= 1 && p <= 3 -> Some p
                | _ -> None)
        | _ -> None
    
    /// Extract seal activation with multiple parameters
    let (|SealActivated|_|) = function
        | ModeSpecific("SealActivated", data) ->
            match data.TryFind "sealNumber", data.TryFind "power" with
            | Some numStr, Some power ->
                match Int32.TryParse(numStr) with
                | true, n when n >= 1 && n <= 12 -> 
                    Some (n, power)  // Returns tuple (int * string)
                | _ -> None
            | _ -> None
        | _ -> None
```

### Using Active Patterns in Game Logic

```fsharp
let processGameEvent (gameState: GameState) (event: MechanicalConsequence) =
    match event with
    | SealDiscovered sealNum ->
        // sealNum is already validated as int between 1-12
        if hasDiscoveredSeal gameState sealNum then
            Error $"Seal {sealNum} already discovered"
        else
            Ok (discoverSeal gameState sealNum)
    
    | BossPhaseStarted phase ->
        // phase is validated as int between 1-3
        Ok (updateBossPhase gameState phase)
    
    | SealActivated (sealNum, power) ->
        // sealNum is int, power is string
        Ok (applySealPower gameState sealNum power)
    
    | Core coreEvent ->
        // Handle events common to all game modes
        processCoreEvent gameState coreEvent
    
    | _ ->
        // Unknown mode-specific event - safe to ignore
        Ok gameState
```

## Benefits of This Approach

### 1. Type Safety Without Rigidity

The active pattern approach provides the best of both worlds:

```fsharp
// Without active patterns - verbose and error-prone
match consequence with
| ModeSpecific("SealDiscovered", data) ->
    match data.TryFind "sealNumber" with
    | Some s -> 
        match Int32.TryParse(s) with
        | true, n when n >= 1 && n <= 12 -> 
            // Finally we can use n
            processSeal n
        | _ -> // handle error
    | None -> // handle error

// With active patterns - clean and safe
match consequence with
| SealDiscovered n -> 
    // n is already an int, validated 1-12
    processSeal n
```

### 2. Mode Independence

Each game mode can define its own active patterns without affecting others:

```fsharp
// Faction War mode events
module FactionWarEvents =
    let (|TerritoryConquered|_|) = function
        | ModeSpecific("TerritoryConquered", data) ->
            match data.TryFind "locationId", data.TryFind "faction" with
            | Some locId, Some faction -> 
                Some (LocationId locId, FactionId faction)
            | _ -> None
        | _ -> None
    
    let (|AllianceFormed|_|) = function
        | ModeSpecific("AllianceFormed", data) ->
            match data.TryFind "faction1", data.TryFind "faction2" with
            | Some f1, Some f2 -> 
                Some (FactionId f1, FactionId f2)
            | _ -> None
        | _ -> None

// Time Loop mode events
module TimeLoopEvents =
    let (|LoopReset|_|) = function
        | ModeSpecific("LoopReset", data) ->
            data.TryFind "iteration"
            |> Option.bind (fun s ->
                match Int32.TryParse(s) with
                | true, n -> Some n
                | _ -> None)
        | _ -> None
    
    let (|MemoryRetained|_|) = function
        | ModeSpecific("MemoryRetained", data) ->
            data.TryFind "memoryId"
        | _ -> None
```

### 3. Testability

Active patterns are easy to test in isolation:

```fsharp
open Expecto

let activePatternTests =
    testList "Active Pattern Tests" [
        test "SealDiscovered extracts valid seal numbers" {
            let event = 
                ModeSpecific("SealDiscovered", Map.ofList ["sealNumber", "5"])
            match event with
            | SealDiscovered n ->
                Expect.equal n 5 "Should extract seal number 5"
            | _ ->
                failtest "Pattern should match"
        }
        
        test "SealDiscovered rejects invalid seal numbers" {
            let event = 
                ModeSpecific("SealDiscovered", Map.ofList ["sealNumber", "99"])
            match event with
            | SealDiscovered _ ->
                failtest "Should not match invalid seal number"
            | _ ->
                () // Expected - pattern didn't match
        }
        
        test "SealDiscovered rejects non-numeric values" {
            let event = 
                ModeSpecific("SealDiscovered", Map.ofList ["sealNumber", "abc"])
            match event with
            | SealDiscovered _ ->
                failtest "Should not match non-numeric value"
            | _ ->
                () // Expected
        }
    ]
```

### 4. Composability

Active patterns can be composed and reused:

```fsharp
// Reusable pattern for extracting integers with validation
let (|ValidInt|_|) min max str =
    match System.Int32.TryParse(str) with
    | true, n when n >= min && n <= max -> Some n
    | _ -> None

// Use the reusable pattern in specific contexts
let (|SealNumber|_|) = function
    | ValidInt 1 12 n -> Some n
    | _ -> None

let (|BossPhase|_|) = function
    | ValidInt 1 3 n -> Some n
    | _ -> None

let (|PlayerLevel|_|) = function
    | ValidInt 1 100 n -> Some n
    | _ -> None
```

## Integration with Game Architecture

Active patterns fit perfectly into a functional game architecture using record of functions:

```fsharp
type IGameMode = {
    ModeId: GameModeId
    Name: string
    
    // Extract events from AI-generated strings
    ExtractMechanicalConsequences: string list -> MechanicalConsequence list
    
    // Process events using active patterns
    ProcessMechanicalConsequence: GameState -> MechanicalConsequence -> Result<GameState, string>
    
    // Validate events before processing
    ValidateConsequence: GameState -> MechanicalConsequence -> Result<unit, string>
}

let sealedEvilMode = {
    ModeId = SealedEvil
    Name = "The Sealed Evil"
    
    ExtractMechanicalConsequences = fun consequences ->
        consequences
        |> List.choose (fun str ->
            // Convert strings to typed events
            match str with
            | Regex @"SealDiscovered_(\d+)" [num] ->
                Some (ModeSpecific("SealDiscovered", 
                                  Map.ofList ["sealNumber", num]))
            | Regex @"BossPhase_(\d+)" [phase] ->
                Some (ModeSpecific("BossPhase", 
                                  Map.ofList ["phase", phase]))
            | _ -> None
        )
    
    ProcessMechanicalConsequence = fun gameState event ->
        // Use active patterns for clean processing
        match event with
        | SealDiscovered n -> Ok (discoverSeal gameState n)
        | BossPhaseStarted p -> Ok (startBossPhase gameState p)
        | SealActivated (n, power) -> Ok (activateSeal gameState n power)
        | _ -> Ok gameState
    
    ValidateConsequence = fun gameState event ->
        match event with
        | SealDiscovered n ->
            if hasDiscoveredSeal gameState n then
                Error $"Seal {n} already discovered"
            else Ok ()
        | BossPhaseStarted p ->
            if not (hasAllSeals gameState) then
                Error "Cannot start boss phase without all seals"
            else Ok ()
        | _ -> Ok ()
}
```

## Lessons Learned

1. **Active patterns are perfect for parsing and validation**: They combine extraction, transformation, and validation in a single, composable unit.

2. **They preserve type safety while allowing flexibility**: You can work with strings when needed (for serialization, AI integration) but immediately convert to typed values.

3. **Mode-specific patterns prevent coupling**: Each game mode can define its own patterns without affecting others, maintaining clean separation of concerns.

4. **Testing becomes straightforward**: Each pattern can be tested independently with clear inputs and expected outputs.

5. **The code reads like the domain**: Pattern matching with active patterns creates code that closely mirrors the game's logic and rules.

## When to Use Active Patterns

Active patterns are ideal when you need to:
- Parse and validate external input (user input, AI responses, file formats)
- Extract typed values from unstructured or semi-structured data
- Create domain-specific pattern matching
- Maintain backwards compatibility while evolving data structures
- Bridge between flexible external formats and strongly-typed internal models

## Conclusion

Active patterns proved to be the perfect solution for my game's event system. They allowed me to maintain the flexibility needed for AI-generated content and extensible game modes while preserving F#'s excellent type safety. The result is code that's both robust and readable, with clear separation between different game modes and their specific mechanics.

This experience reinforced an important lesson: when faced with a choice between type safety and flexibility, F# often provides elegant features like active patterns that give you both. Working with Claude Code to explore this solution showed me how AI assistants can help discover language features that perfectly match your architectural needs.