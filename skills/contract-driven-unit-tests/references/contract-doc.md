# Writing the contract down

Read this in Phase B, when a promise or a non-promise needs to be recorded somewhere a
consumer will actually see it.

## Pick the channel by where the consumer looks

A contract written where nobody reads it does no work. The right place is the nearest
channel the consumer sees *at the moment they use the unit*:

| Unit | Channel |
|---|---|
| Exported function, hook, or utility | TSDoc directly above the export — it surfaces in the IDE hover |
| Module or feature folder | header comment in `index.ts`, or a short README beside it |
| A single field with surprising behaviour | comment on that field in the type, not on the whole type |
| Package entry point | the published README, since consumers may never open the source |
| HTTP endpoint | the `description` in the API schema (OpenAPI, GraphQL SDL) |
| Tool exposed to an AI agent | the tool description and parameter schema — the agent sees nothing else |
| Database function, table, or column | a comment stored in the database, plus the constraint that enforces it |

Nobody opens `docs/` to look up an internal helper, so a doc file is the wrong home for a
unit-level contract. Keep it adjacent to the code.

## Prefer the type system over prose

Before writing a sentence, check whether the type can carry it. A contract expressed in a
type cannot drift out of date, and violating it fails the build instead of surprising
somebody at runtime.

```ts
// "do not mutate the result"
function getCases(): readonly Case[]

// "the order is not guaranteed" — hand back a container that has no order
function getTagIds(): ReadonlySet<string>

// "the id is opaque; do not parse it or infer anything from its shape"
type CaseId = string & { readonly __brand: unique symbol }

// "new enum values will appear" — force the caller to handle the unknown case
function label(p: Priority): string {
  switch (p) {
    case 'low': return 'Low'
    // ...
    default: return 'Unknown'
  }
}
```

Prose is for what types cannot express: rounding behaviour, performance, error message
stability, and behaviour at the edges.

## The shape of the line

Two short lists. The second one is the half that usually gets skipped, and it is the half
that makes refactoring possible later — silence about a behaviour reads as a promise, so an
explicit "not guaranteed" is what keeps a consumer from building on an accident.

```ts
/**
 * Reduces a batch of outcomes into the summary shown in headers and reports.
 *
 * Guarantees:
 *  - every counter is present, including zeros;
 *  - total equals the sum of the counters;
 *  - successRate is rounded to one decimal, or null when the batch is empty;
 *  - entries with an unknown status are ignored rather than throwing.
 *
 * Not guaranteed:
 *  - the rounding direction exactly at .5 — do not build logic on it;
 *  - object identity between calls — this may become memoised;
 *  - computational complexity.
 */
export function summarize(entries: readonly Entry[]): Readonly<Summary>
```

Keep both lists to what was actually decided in A3. A guarantee invented while writing
the comment is a guarantee nobody agreed to.

## When the unit is a database function

The comment belongs in the database itself, which is where anyone inspecting the function
finds it — put it in the migration that creates or changes it.

```sql
COMMENT ON FUNCTION get_statistics(bigint) IS
  'Guarantees: returns exactly one row; counters are >= 0; the counters sum to total.
   Not guaranteed: ordering of rows inside the nested array;
   performance at large result volumes.';
```

For data rules, prefer the constraint over the comment — `CHECK`, `UNIQUE`, `NOT NULL`, and
foreign keys are invariants the database refuses to let anyone violate, whichever code path
writes the row. A comment describes; a constraint enforces.

## When the consumer is an AI agent

An agent reads the tool description and the parameter schema and nothing else, so anything
omitted there genuinely does not exist for it. Non-promises matter more here than anywhere:
an agent will build a workflow on a behaviour it happened to observe twice, with no instinct
that the behaviour might be incidental.

Keep it to one clause inside the existing description rather than a separate paragraph —
tool descriptions are read in full on every call, so length has a running cost.

Check whether enum values or schema fragments are mirrored elsewhere in the codebase before
changing one; a partial change to a duplicated definition fails at runtime rather than at
build time.

## Common non-promises worth stating

| Behaviour | Line |
|---|---|
| Order | "The order of returned items is not guaranteed. Sort on your side if you need a stable one." |
| Identifiers | "`id` is an opaque string. Do not parse it or assume it is sequential." |
| Error text | "`message` is written for humans and may be reworded. Branch on `code`." |
| Pagination cursor | "`next_cursor` is opaque; its format is an implementation detail." |
| Forward compatibility | "New fields may appear in the response and new values in enums. Ignore what you do not recognise instead of failing." |
| Timing | "Response time is not part of the contract." |
| Shared references | "The returned object may be shared; do not mutate it." |
| Rounding | "Intermediate rounding is an implementation detail; only the final total is guaranteed." |

## Stronger than a comment: make the assumption impossible

Documentation is weak because it is not read. When a non-promise keeps getting depended on
anyway, break it deliberately so nothing can lean on it — the way Go randomises map
iteration order at runtime, precisely so no program can come to rely on an order nobody
promised.

In practice: shuffle an unordered result in dev and test builds, add noise inside an opaque
cursor, vary a limit in the test environment. This turns a line of prose into something the
test suite enforces.

Reserve it for cases where the accidental dependency has actually bitten someone —
otherwise it is complexity bought for a hypothetical.

## Keeping it honest

A stale non-promise is worse than none: it makes a real guarantee look optional and
eventually gets ignored wholesale. When a behaviour graduates from accident to promise —
because a consumer now legitimately relies on it — move the line from one list to the
other in the same change that creates the dependency, not later.
