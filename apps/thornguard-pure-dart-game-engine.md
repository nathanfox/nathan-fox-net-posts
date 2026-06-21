# Thornguard: My First Mobile Game, and the Pure-Dart Engine Underneath

<a href="https://apps.apple.com/us/app/thornguard/id6760564452" target="_blank" rel="noopener"><img src="https://www.codepasture.com/app-store-badge.svg" alt="Download on the App Store" height="40" style="vertical-align: middle;" /></a>&nbsp;&nbsp;<a href="https://play.google.com/store/apps/details?id=com.codepasture.thornguard" target="_blank" rel="noopener"><img src="https://www.codepasture.com/google-play-badge.png" alt="Get it on Google Play" height="60" style="vertical-align: middle;" /></a>

---

## A Humble First Game

Thornguard is my first mobile game. I want to set expectations honestly up front: it's deliberately retro, it's not trying to be the next big thing, and I'm making no claims that it reinvents the genre. It's a small tower defense roguelike with simple vector art and a clear, readable interface.

What it turned out to be, though, is far more complicated than I expected going in—and a lot more fun to play. The tagline on the [Code Pasture](https://www.codepasture.com/thornguard) page is "more depth than pixels," and that's the honest pitch. It looks simple. It plays deep. I've genuinely enjoyed both building it and playing it.

This post is less a sales pitch and more a look at how it's built. The most interesting decision—the one that made everything else manageable—was keeping the game engine completely separate from the UI.

## What the Game Is

You build elemental towers (fire, frost, lightning, thorn) plus plain arrow and cannon towers, draft abilities as you go, collect relics, and fight your way through waves of enemies and bosses. Before each run you pick a prophecy—a themed challenge track that shapes what you'll face. Each run is different.

It's local-first: no servers, no accounts, no ads, no data collection. All the game content ships with the app, so it works completely offline. The full feature list lives on the [Thornguard page](https://www.codepasture.com/thornguard); I'll keep the gameplay description short here, because the part I actually want to talk about is the code.

## The Engine Is Just Dart

The core architectural decision in Thornguard is that the game engine is pure Dart with zero Flutter dependencies. Nothing under `lib/domain/` imports `package:flutter`. The engine doesn't know a UI exists.

It exposes a tiny surface area—essentially two functions:

```dart
// Advance the simulation by one tick.
TickResult tick(GameState state, Random rng);

// Apply a player action (build a tower, draft an ability, start a wave...).
TickResult processAction(GameState state, PlayerAction action, Random rng);
```

Both take an immutable `GameState` and return a `TickResult`, which is just the new state plus a list of events that happened:

```dart
@freezed
abstract class TickResult with _$TickResult {
  const factory TickResult({
    required GameState state,
    @Default([]) List<GameEvent> events,
  }) = _TickResult;
}
```

That's the whole mental model. State goes in, a new state and a list of events come out. `PlayerAction` is a sealed union of every move the player can make, and `GameEvent` is a sealed union of everything that can happen in a tick (an enemy died, a tower was hit, an ability triggered). The UI's only jobs are to render the current state and to produce actions.

Inside, `tick()` reads as a clean pipeline—spawn enemies, move them, resolve tower attacks, run boss abilities, resolve enemy attacks, tick status effects, and so on—each step taking a state and returning a new one. Because every step is a plain function over immutable data, the whole thing is easy to follow and easy to reason about.

The only bridge between this pure core and Flutter is a single Riverpod notifier (`lib/state/game_notifier.dart`) that holds the current state, calls `tick()` on a timer during a wave, and forwards player actions through `processAction()`. That one file is the entire seam between the game and the framework. This is the same [domain-first instinct](https://www.nathanfox.net/p/domain-first-development-building) I've written about before, applied to a game: get the logic right in plain Dart, then let the framework be a thin shell around it.

## Testing Nearly Fell Out for Free

When your engine is pure functions over immutable state, testing stops being a chore. Every test is the same shape: build a state, apply a tick or an action, assert on the result. No widget pumping, no mocking the framework, no fighting with async lifecycles.

```dart
test('tick increments the tick counter', () {
  final state = inWaveState(
    towers: [testTower(typeId: 'arrow', slot: 0)],
    enemies: [testEnemy(id: 'e', hp: 100, targetSlot: 0, progress: 0.5)],
  );
  final result = tick(state, Random(1));
  expect(result.state.tick, state.tick + 1);
});
```

That one is trivial on purpose—it just shows the shape. But the same shape scales to genuinely complex mechanics. Here's a real test for the fire tower's Eruption. A fire tower builds heat as it attacks, and once it reaches the top tier the player can trigger an Eruption on its slot: it deals area damage to every enemy in *that* slot (and only that slot), resets the tower's heat, and emits an event the UI can react to.

```dart
test('damages all enemies in slot, resets heat, emits event', () {
  final state = combatTestState(
    towers: [
      testTower(typeId: 'fire', slot: 0, heatStacks: heatTier3),
      testTower(id: 't2', typeId: 'fire', slot: 1, heatStacks: heatTier3),
    ],
    enemies: [
      testEnemy(id: 'e1', hp: 100, targetSlot: 0),
      testEnemy(id: 'e2', hp: 100, targetSlot: 0),
      testEnemy(id: 'e3', hp: 100, targetSlot: 1), // other slot: untouched
    ],
    wave: 2,
  );
  final (newState, events) = executeEruption(state, 0);

  // Heat reset on slot 0 tower only.
  expect(newState.towers[0].heatStacks, 0);
  expect(newState.towers[1].heatStacks, heatTier3);

  // Enemies in slot 0 damaged, enemy in slot 1 unchanged.
  expect(newState.enemies[0].currentHp, lessThan(100));
  expect(newState.enemies[1].currentHp, lessThan(100));
  expect(newState.enemies[2].currentHp, 100);

  // AbilityActivated event emitted.
  expect(
    events.whereType<AbilityActivated>().any(
          (e) => e.abilityTypeId == 'eruption' && e.targetSlot == 0,
        ),
    isTrue,
  );
});
```

Same recipe—build a state, run the logic, assert on the new state and the events—but now it's pinning down a real interaction: who gets hit, who doesn't, what gets reset, and what the rest of the game gets told about it. There's no rendering involved and nothing to mock, so a test like this is quick to write and fast to run.

Seed the random number generator and the engine is fully deterministic, so even combat outcomes are testable. There's no flakiness to manage because there's nothing nondeterministic to manage.

The result is a test suite of **2,003 tests across 118 files**. I'll be honest about why that number is so high: it's not because I'm unusually disciplined, it's because the mechanics are numerous and they interact, so the edge cases multiply. But that's exactly the point—the architecture made it cheap to write a test for each of those edge cases instead of expensive, so I actually did.

## Simulating for Balance

Here's the payoff that surprised me most. Because the engine runs without a UI, I could write bot players and have them play the game thousands of times, headlessly, in seconds.

The simulation tools (`tool/run_simulation.dart` and a boss-specific variant) run a roster of bot archetypes—greedy, rush, turtle, hoarder, conservative, element specialists for fire/frost/lightning, and more—each playing many full games. They report how far each archetype gets: average, median, and max wave reached, abilities used, boss survival rates, stall diagnostics, and a CSV export when I want to slice the data in a spreadsheet.

```
dart tool/run_simulation.dart 200 --inscriptions
```

This is only possible because the engine doesn't need to be rendered to run. A game that's tangled into its widgets can't do this. A game that's a pure function can play itself ten thousand times before you finish your coffee.

## Balance Is Hard

The simulations are genuinely useful, but I don't want to oversell them. They catch the gross outliers—an archetype that steamrolls to wave 30 every time, or one that dies on wave 3—and those are exactly the signals you want for spotting something badly miscalibrated.

What they don't do is tell you the game is *fun*, or surface the subtle stuff.

And there is a lot of subtle stuff. Six tower types with branching upgrade trees, twenty-plus enemies with their own traits, a dozen status effects, dozens of abilities and relics, prophecy tracks, and trial mutators that change the rules mid-run. The interactions between all of these explode combinatorially. Fire towers build heat that triggers an eruption; the burning that leaves behind slows enemies; that slow changes how they bunch up against your walls; a reflection relic then bounces damage back in a way you never specifically designed.

Balancing that is genuinely difficult, and I won't pretend I've nailed it. I'm sure there are still balance opportunities hiding in there—combos that are too strong, paths that are too weak, prophecies that are easier or harder than I intended. The simulations and a lot of playing have gotten it to a place I'm happy with, but "happy with" is not "solved." If anything, building this gave me a deep respect for how hard game balance actually is.

## Surprisingly Large for a Few Months

I built most of Thornguard over a few months of evenings and weekends, and it ended up much bigger than I'd have guessed. Excluding generated code, it's roughly **79,000 lines of Dart**: about 45,000 in the app itself (the pure engine is around 17,000 of that, the rest UI), plus another 32,000 lines of tests.

I didn't set out to write a small game's worth of test code. It accumulated naturally because the architecture made each test cheap, and because—like the rest of my recent work—I built this with [Claude Code](https://www.nathanfox.net/p/100000-lines-in-7-months-how-claude-code) alongside me. A clean engine boundary plus an agent that's happy to write the four-hundredth edge-case test is what makes a codebase this size tractable as a solo side project. The separation of concerns wasn't just good hygiene; it's what kept the whole thing from collapsing under its own weight.

## Try It

That's the honest story: a small retro tower defense game with a pure-Dart engine underneath, a pile of tests that came cheap, and a balance problem I'm still chipping away at. I had a lot of fun making it, and I have a lot of fun playing it.

If you like classic tower defense and don't mind a game that looks simpler than it plays, give it a shot. And if you find a combo that feels broken—too strong or too weak—I'd genuinely love to hear about it.

<a href="https://apps.apple.com/us/app/thornguard/id6760564452" target="_blank" rel="noopener"><img src="https://www.codepasture.com/app-store-badge.svg" alt="Download on the App Store" height="40" style="vertical-align: middle;" /></a>&nbsp;&nbsp;<a href="https://play.google.com/store/apps/details?id=com.codepasture.thornguard" target="_blank" rel="noopener"><img src="https://www.codepasture.com/google-play-badge.png" alt="Get it on Google Play" height="60" style="vertical-align: middle;" /></a>
