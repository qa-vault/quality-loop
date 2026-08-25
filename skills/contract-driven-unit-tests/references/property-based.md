# Property-based tests

Read this when step A4 produced an invariant, or when a unit's input space
is far too large to cover with hand-picked examples.

## What it is

An ordinary test states an **example**: given this input, expect this output.
A property-based test states a **rule**: for *any* input, this condition holds. The library
generates hundreds of inputs — empty, huge, negative, unicode, zero, boundary values — and
checks the rule against every one.

```ts
// example-based: three cases somebody thought of
expect(reverse([1, 2, 3])).toEqual([3, 2, 1])
expect(reverse([])).toEqual([])

// property-based: the rule, with inputs generated for you
fc.assert(fc.property(fc.array(fc.integer()), (arr) => {
  expect(reverse(reverse(arr))).toEqual(arr)
}))
```

A property-based test is the executable form of an **invariant**, in the same way an
example-based test is the executable form of a **contract** for one case. They cover
different halves and do not compete.

## Why it earns its keep: shrinking

When a property fails, the library does not hand back the 47-element monster that broke it.
It shrinks the counterexample until it stops failing, then reports the minimum:

```
Property failed after 23 tests. Counterexample: [0, -1]
```

That is usually enough to explain the bug without a debugger, which is the main practical
payoff over fuzzing by hand.

## Finding the property

This is the step people give up on. Properties almost always come from one of these
shapes — try them in order:

| Shape | Rule | Example |
|---|---|---|
| Round-trip | decoding what you encoded returns the original | `parse(serialize(x))` equals `x` |
| Invariant | a condition true in every state | parts sum to the total; a counter never goes negative |
| Idempotence | doing it twice changes nothing | `normalize(normalize(x))` equals `normalize(x)` |
| Order independence | the sequence of operations does not matter | `sort(shuffle(x))` equals `sort(x)` |
| Oracle | agrees with an obviously-correct but slow version | optimised lookup vs naive scan |
| Metamorphic | change the input this way, the output must change that way | add an item → the total never decreases |
| Never throws | any input yields either a valid result or a declared error | the cheapest property; nearly free fuzzing |

The last row is the right starting point when nothing else comes to mind. "No input makes
this throw something undeclared" catches a surprising amount on its own.

## When it fits

- **Parsers, serialisers, formatters, codecs** — round-trip is free here and catches the
  nasty edges (unicode, nesting, empty).
- **Calculations** — prices, discounts, tax, rounding, dates and timezones. Huge input
  space, simple rules: "a discount never makes the total negative".
- **Data structures and algorithms** — sorting, searching, dedup, caches.
- **State machines (model-based testing)** — generate random *sequences of operations* and
  check the invariants after each. This finds combinations no manual case would list, such
  as create → archive → restore → delete.
- **Idempotence and retries** — "calling twice with the same key does not create a second
  record" is critical for anything queue- or payment-shaped.
- **Validation and sanitising** — "no generated string survives the sanitiser with
  executable content intact".

## When it does not fit

- **Small input space.** Three plans × two regions is a six-row table of examples, not a
  property. A generator here adds noise and hides the intent.
- **Business rules that are a list of cases.** "Pro plan in Germany is 19% VAT" is a
  lookup, not a rule. Trying to abstract it produces a property that restates the table.
- **When the property would restate the implementation.** Then the test is a tautology: the
  same mistaken assumption lands in both places and the test passes on broken code. If you
  cannot state the rule without re-deriving the algorithm, write examples instead.
- **UI and e2e.** Too slow, too non-deterministic. Generating input *data* for a form is
  fine; generating the scenario is not.
- **Slow or external dependencies.** 200 runs at 300 ms is a minute for one test.

## Discipline that keeps it from feeling flaky

Two habits, both cheap:

- **Report the seed** on failure so a run is reproducible. Without it, a property failure
  looks like a flake and gets re-run instead of investigated.
- **Turn every counterexample into a permanent example-based test.** The property keeps
  hunting for new failures; the example guards the one already found. Skipping this is how
  the same bug comes back.

## Tooling

In JavaScript and TypeScript the standard choice is **fast-check**, which integrates with
vitest and jest directly — `fc.assert(fc.property(...))` inside an ordinary `it`.

It is part of this skill's instrument package: if it is not already a dependency, propose
the install — small, dev-only — at the instruments step and get agreement before adding
it. The proposal is the mandatory part; the dependency stays the engineer's decision.

If the install is declined or blocked, an invariant can still be checked the plain
way — a loop over a
hand-built list of awkward inputs inside a normal test. No generation, no shrinking, and
far less thorough, but it captures the rule in executable form and can be upgraded in place
later.

## The limit that carries over from SKILL.md

A property may not assert what the contract does not promise. If the order of results is
not guaranteed, state the property over the *set* of results, never the sequence —
otherwise the generator will simply find the refactor-sensitive failure faster than a human
would.
