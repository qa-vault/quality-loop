---
name: contract-driven-unit-tests
description: >-
  Work out what a unit actually promises, then write or judge its tests against that —
  asserting the contract, refusing to assert accidents, and catching assertions that pass no
  matter what the code does. Use this skill whenever writing, adding, or reviewing unit tests
  in JavaScript or TypeScript — vitest, jest, node:test — including "write tests for this
  function", "cover this hook with tests", "add a spec for this", "does this need a test?",
  "review these tests", "are these tests any good?", "why did this test break?". Also use it
  whenever a unit test failed after a refactor, when deciding what to mock, and before
  asserting on any pure function, reducer, selector, formatter, calculator, parser,
  validator, or custom hook — even when the user never says the word "contract". It governs
  what may be asserted and what must be left alone, and it deliberately overrides existing
  test conventions rather than matching them. Not for end-to-end or browser-driven tests.
---

# Contract-Driven Unit Tests

## Why this exists

Writing a unit test is easy. Deciding what it should assert is the hard part, and it is
where unit suites go wrong — in both directions.

Assert too much and every accident of the implementation becomes a promise: any legal
refactor turns the suite red, the team learns that red means "someone touched the code",
and the suite stops carrying information. Assert the wrong thing and a test can be green
forever while checking nothing at all — the most expensive kind, because it is invisible
and inspires confidence it has not earned.

So the job is not "write a test". It is: work out what this unit promises, then hold the
tests to exactly that.

## How this works

The work splits into two phases, and the seam between them matters:

```
PHASE A — investigate the subject
    consumers → observable behaviours → promise/accident → invariants → verdict
    Output: a model of the contract. No tests involved yet.

PHASE B — act on the model
    authoring: write tests that cover the model
    review:    compare existing tests against the model, in both directions
```

Keeping Phase A free of tests is not tidiness. In review, reading the tests first makes you
read the module *through* them: the question silently changes from "what should be asserted
here?" to "is this existing assertion defensible?" — and you only ever look where a test
already points. Gaps become invisible by construction. Build the model first and gaps show
up by subtraction.

## Existing conventions do not get a vote

A codebase's current tests are evidence of what a team did. They are not evidence of what is
correct, and this skill does not defer to them.

Suites drift toward implementation-coupling by default, because each individual
over-assertion looks harmless when it is written — a whole-object `toEqual` is quicker than
naming three fields, a snapshot is quicker than deciding what matters. The cost only shows
up later, spread across every future refactor, by which point nobody attributes it to the
assertion that caused it. That is exactly why a defective convention can be pervasive and
long-lived in a healthy team: nothing in the day-to-day feedback loop punishes it.

Matching such a precedent propagates the defect at the moment someone is finally paying
attention to it. Do not treat "this is how tests are written here" as permission.

| Cosmetic — match the repository | Substantive — this skill decides |
|---|---|
| Where test files live and how they are named | Whether the unit needs a test at all |
| Import style (explicit imports vs configured globals) | What is asserted and what is left alone |
| Fixture and factory placement, naming, formatting — not their mutability | Whole-object equality, snapshots; shared mutable fixtures |
| `describe` / `it` nesting shape | What is mocked, and at which seam |
| Which assertion library helpers are in use | Asserting order, error text, call counts, private state |

Cosmetic conventions carry no correctness content, so conforming costs nothing and keeps a
diff readable. Substantive ones decide whether the suite is an asset or a liability, and
there precedent has no standing. Where you diverge, say so in one sentence rather than
diverging silently.

## Delegating Phase A to a fresh agent

Propose this whenever subagents are available. Two independent reasons, either one enough:

- **Bias** — the module was written or edited in this session. Delegate even if it is the
  only module.
- **Volume** — several independent modules are in scope. Delegate one per module.

The bias reason is the important one and it follows directly from the skill's premise: a
contract is what is visible from outside without reading the source. An agent that just
wrote the module holds the implementation *and the intent*, so at the classification step it
does not classify — it remembers. Everything it meant to do looks like a promise; everything
that fell out of the code is invisible to it precisely because it feels obvious. A fresh
agent sees what a consumer sees.

Delegate **Phase A only**. Writing tests from a context that knows the implementation is far
less dangerous, because the model already constrains what may be asserted.

Five rules make the delegation worth its cost:

1. **Carry decisions, not implementation.** Pass "we chose `null` rather than `0` because
   *no data* is not *0%*" — that is knowledge no fresh agent can recover from code, and
   without it the classification step stalls or guesses. Do not pass how the code works;
   that is the bias being escaped.
2. **Bound the investigation.** The delegated prompt carries Phase A's scope rule
   verbatim: the unit's source, its consumers' call sites, collaborators' public
   signatures — never collaborators' implementations.
3. **Require falsifiable evidence in the report.** A concrete counterexample, a call site
   with a line number, a command that reproduces the finding — not "this test looks weak".
4. **Require an investigation manifest.** The report lists what was read — files with
   line ranges — and every consumer found. The dispatcher greps for consumers itself and
   compares: a call site missing from the manifest means the investigation was
   incomplete, caught before the gap can propagate into the model.
5. **Verify before acting.** The dispatching session checks the evidence rather than
   relaying the conclusion. A finding that cannot be checked from outside the subagent is
   not yet a finding.

The report is the only thing that survives the subagent — anything not written into it is
lost by construction. The rules above make loss visible rather than impossible: the fixed
table schema (A5) leaves an empty cell where a free-form report would compress a nuance
away invisibly, the manifest exposes what was never looked at, and evidence anchors every
claim back to the source for re-checking.

One loss mode remains: a behaviour the subagent never *noticed* is simply absent from the
table, and no check on the report's contents can see an absence. The counter is a
**completeness cross-check**, and in the bias scenario the dispatching session is
uniquely equipped to run it: the context that wrote the module is a poor classifier but
an excellent completeness checker, because it remembers what it built. After the report
arrives, diff the A2 list against the source — and against your own knowledge of the
module when you have it. The asymmetry is strict: the dispatcher may **add** missed
behaviours to the table; it may not reclassify the subagent's entries without evidence,
because reclassifying from memory is the bias the delegation exists to escape.

If subagents are unavailable, build the model from the source and its consumers **before**
consulting your own recollection of the implementation, and label the result as classified
without isolation, so its weight is known.

## The core distinction

- **Contract** — what a consumer may rely on without reading the source: inputs accepted,
  outputs returned, errors thrown, side effects performed, guarantees offered.
- **Invariant** — a condition that holds in every state, after every operation.
- **Accident** — everything else the implementation happens to do.

The critical fact: **you cannot read the contract off the implementation.** The code exhibits
promises and accidents in exactly the same way; nothing in it marks which is which. Only a
decision separates them, and that decision is made by a person, not discovered by reading.

Second fact, which is why the decision must be deliberate: **whatever you assert, you
promise.** An assertion is a public commitment written in code — the next person reads a
failing test as "this behaviour was required". Silence, meanwhile, reads as permission.

## Phase A — investigate the subject

**The scope rule.** Phase A reads exactly three things: the unit's source, its consumers'
call sites, and the *public signatures* of the unit's collaborators. Collaborator
implementations are out of bounds. Capable models drift past the module boundary while
"understanding the system", and what they learn there leaks straight into assertions: the
test of this unit quietly starts pinning an adjacent module's logic, and a legitimate
change to that module turns this unit's suite red. If a collaborator's signature is not
enough to classify a behaviour, that is a question to raise — A3's default — not a licence
to read deeper.

### A1. Find the consumers

Grep the actual call sites; do not assume. For each, note what it relies on.

Two things count as consumers, and the second is easy to forget:

- **code that calls the unit** — another module, a component, a route handler, a package
  export;
- **tests at other layers** that assert the behaviour — an e2e or integration spec pinning a
  rule is a human having deliberately written down "this is required", which is *stronger*
  evidence of a decision than a call site, not weaker.

Also check whether the behaviour is already pinned by a type, a schema, or a database
constraint. Anything found there is settled before you start.

### A2. List every observable behaviour

Everything a consumer could notice, unfiltered and including the boring entries: return
shape, values at the edges, thrown errors, mutation of arguments, ordering, identity of
returned objects, calls crossing a system boundary — which requests, and how many
times — and what happens on unknown input.

Be exhaustive here and judgemental in A3, not the other way round — behaviours filtered out
at this stage never get classified, and unclassified behaviour is what tests accidentally
freeze.

### A3. Classify each one: promise or accident

The crux. Apply in order:

1. Already expressed in a type, a schema, or a database constraint → **promise**. It was
   committed to elsewhere; a test only makes it executable.
2. A consumer **depends** on it → **promise** in practice, whether or not it was intended.
3. It is a decision somebody would defend in review — `null` vs `0` for "no data", ignoring
   unknown enum values rather than throwing → **promise**, and worth stating explicitly
   because a future reader cannot re-derive it.
4. It falls out of a data structure or library detail — object key order, reference identity,
   iteration order, collation, allocation, memoisation → **accident**.
5. It is the number, order, or existence of internal operations → **accident**.

**Scope of heuristic 2.** "Depends" means the consumer *branches on it, counts it, compares
it, or asserts it* — not merely that the value flows through to a screen. A table that
renders whatever list it is handed depends on the list's membership and on the order the
sort parameters asked for; it does not thereby promise how the underlying comparator orders
accented characters. Without this scoping, heuristic 2 promotes everything downstream of any
rendered output to a promise and swallows heuristic 4 whole.

**Pass-through values.** A detail the unit merely transports from a collaborator —
returned unchanged, not branched on, not transformed — is the collaborator's decision
showing through, not this unit's promise: heuristic 4, not heuristic 2. The unit may
promise *that* the field is present and carried; the field's internal shape belongs to the
collaborator's own contract and its own tests. This is the trap behind tests that break
when an adjacent module legitimately changes.

**When none of these settle it, do not guess.** Leave the behaviour unasserted and raise the
question: "does X promise Y, or is that incidental?"

The asymmetry justifies that default: an unasserted accident costs nothing today and can be
asserted later; an asserted accident costs a false failure on every future refactor. Guessing
here is the most expensive mistake available in this workflow.

### A4. Name the invariants

Conditions true in every state: parts summing to a total, a counter never going negative, a
value staying in range, an operation being idempotent, output being a permutation of input.

Invariants are worth separating because they are testable across *all* inputs rather than a
few examples — see `references/property-based.md`.

### A5. Verdict — does this unit warrant tests?

Now that consumers and decided rules are known, this is answerable. Yes when at least one
holds:

- exported across a module boundary with a consumer outside its own file;
- non-obvious behaviour at the edges — empty input, rounding, unknown enum value, `null` vs
  `0`, timezone, unicode, very large input;
- it encodes a rule somebody decided, rather than plumbing that moves data along.

No when it is a private helper with one call site beside it, a thin pass-through, or
something the type system already guarantees. The coverage instinct produces piles of tests
on internal helpers, and each becomes a tripwire that fires on legal refactors; a missing
test on a trivial helper costs approximately nothing.

**Output of Phase A**: the classified behaviour table, the invariants, and this verdict.
The table has fixed columns — *behaviour | classification | heuristic applied | evidence
(file:line) | open question* — because a free-form report can compress a nuance away
invisibly, while an empty cell in a required column is visible. Keep it compact and
checkable — a table and a short list, never an essay. Show it, so the classification can
be corrected; A3 is the part a human should review.

## Standing up the instruments

Phase B leans on two dev-only dependencies, and the moment to check for them is now —
after the verdict says tests are warranted, before the first test is written:

- **Stryker** (`@stryker-mutator/core` plus the runner plugin matching the suite) — runs
  the mutation step that closes every engagement.
- **fast-check** — turns A4's invariants into property-based tests.

If either is missing, propose the install now — the package, that it is dev-only, and
the mandatory step it unlocks — and wait for agreement before touching `package.json`.
A dependency is always the engineer's call. What is not optional is the proposal
itself: the failure mode this step exists to prevent is silence, where the tool is
absent so the mutation run or the property tests quietly never happen and the summary
never mentions them. A declined install is a legitimate outcome and goes into the
summary as the reason a step did not run. What warrants stopping entirely is a package
with no test runner at all: standing up a harness from nothing changes how the project
builds, and that too is the engineer's call, as the orientation section says.

## Phase B (authoring) — write the tests

- **Promise** → an assertion, and a type if it can carry it. A promise expressed in the type
  system cannot drift from the code, so prefer that half.
- **Invariant** → a database constraint when it is a data rule; a property-based test
  (fast-check) when it is a computation rule. Not an optional extra: an invariant sampled
  by a few examples is a rule left mostly unproven, so every invariant A4 named either
  becomes a property or falls under the reference's "when it does not fit" catalogue —
  named in the summary, never silently dropped.
- **Non-promise** → one line of documentation on the unit itself, so the next consumer is
  warned before leaning on it. Templates: `references/contract-doc.md`.

**Choosing example inputs.** For each promise, partition its input space into the
equivalence classes the contract distinguishes and take one example per class, plus the
boundaries where classes meet: the empty list and the single element, the value at the
limit and the first one past it, the last accepted input and the first rejected one.
Classes and boundaries of the *promised* behaviour only — this is not a combinatorial
sweep of every parameter. When the space is too large for examples, see
`references/property-based.md`.

Every assertion must pass both questions below before it is finished.

## Phase B (review) — judge the existing tests

**Pass 1 — model against tests, in both directions.**

- *Promise with no test* → a gap. Rank it: genuinely worth adding, or over-testing.
- *Assertion with no promise behind it* → over-assertion. It freezes an accident.
- *Test whose subject is not in the model at all* → either the model is incomplete, or the
  test exists for a reason nobody recorded. Ask.

Doing this by subtraction is the point of building the model first. Walking the test file
instead finds only the first kind of defect.

**Pass 2 — each assertion against the two questions.**

Read the tests now as evidence too: a test at any layer that pins a behaviour is a recorded
decision, so it may *update the model* — but only after Pass 1, when it can no longer quietly
substitute itself for the question of what should be asserted.

Report per assertion: what it claims, contract or implementation, and whether it
discriminates.

## The two questions every assertion must pass

**Question 1 — does it survive a rewrite?** Would this assertion still hold after a complete
rewrite of the internals that keeps consumer-visible behaviour intact? If not, it is testing
implementation. The table below is the catalogue of ways this goes wrong.

**Question 2 — does it discriminate?** Would this assertion **fail** against an
implementation from which the promise had been removed? If it passes either way, it is
decorative: green forever, informative never.

Question 2 is the one people never ask, and a test can be flawless by Question 1 while
failing Question 2 completely. Where it bites hardest:

- **Tiebreakers** — if the fixture is already in the tiebroken order, a stable sort produces
  the expected output even with the tiebreak deleted. The tie must be *reversed* in the input
  for the assertion to mean anything. Expectations do not change; only the fixture does.
- **Defaults** — a fixture that supplies the default value cannot show the default was
  applied.
- **Fallbacks and guards** — input that never reaches the guard leaves it unproven.
- **Fixtures whose shape or order can substitute for the rule under test** — the general
  form of all three.

When unsure, do not reason about it: copy the function into a scratch file, delete the
promise, and run the assertion against it. This takes a minute and settles the question.

## Mutation testing — Question 2, automated

The scratch-file check above has an automated form: a mutation tool — Stryker, for
JavaScript and TypeScript — applies hundreds of such deletions and inversions to the
module and reports which ones no test noticed. It is the mandatory verification step
closing every engagement, authoring or review, after Phase B — not an option to weigh.
Three rules govern the run:

- **Coverage first.** A mutation score over uncovered code is noise — mutants in lines no
  test executes survive trivially. The Phase B suite exists before the mutator runs.
- **Whole module, every run.** The unit of work in this skill is one logical module, and
  the mutator covers all of it — never a sampled subset of "important" lines. At module
  scope a full run is affordable, and the resource cost of mutating and of fixing what it
  reveals is not a factor to weigh against test quality. Suite-wide runs are a separate
  undertaking, not part of this loop.
- **Triage every survivor; never chase the score.** A surviving mutant is a behaviour
  change no test noticed — an unclassified behaviour. Classify it with A3:
  - the mutated behaviour was a **promise** → a real gap; add the test;
  - an **accident** → the mutant *should* survive; killing it would freeze the accident,
    the exact failure this skill exists to prevent;
  - an **equivalent mutant** (no observable behaviour change at all) → fine; note it and
    move on.

100% mutation score is not the goal, and chasing it manufactures over-assertions. The
goal is zero *untriaged* survivors. A missing tool is no exemption — the instruments
section's proposal covers it. The only summaries allowed to report the step as not run
are one naming a technical blocker outside the suite's control, and one recording that
the engineer declined the install.

## Assertion rules

| Pattern to replace | Why it hurts | Write instead |
|---|---|---|
| `expect(result).toEqual({ ...whole object })` | Freezes every field, default, and shape at once — accidents included. Adding one field breaks it. | Assert the fields the contract names: `toMatchObject`, or one assertion per promise |
| Asserting array order with no promised sort — **but read the exception below first** | Any query-plan, index, or map change flips it | Compare as a set: sort first, or `arrayContaining` + `toHaveLength` |
| `expect(collaborator.method).toHaveBeenCalledTimes(n)` inside the system | Call counts are implementation; refactoring is exactly what changes them | Assert the observable result. Count calls only at system boundaries, where the call *is* the side effect |
| Asserting exact error message text | Messages are written for humans and get reworded | Assert the error type or `code` |
| Whole-object or DOM snapshots | Same problem as `toEqual`, and auto-accepted updates hide the drift | Targeted assertions on the promised parts |
| Reaching into private state (`obj._items`) | Not a contract at any level | Assert through the public surface |
| Timing or performance in a unit test | Not a promise, and flaky by nature | Leave it out |

**The ordering exception.** When producing an order *is the unit's job* — a sort, a tree
builder, a ranker, a scheduler — the order is the promise and asserting it is correct.
Mechanically applying "compare as a set" there destroys the only assertions that test the
unit's actual purpose. Check what the unit exists to do before flagging an ordering
assertion.

The exception has a boundary, though: only the **chosen** dimensions are promised — the sort
key, the direction, the tiebreak somebody decided on. What the underlying comparator does
beneath those, such as `localeCompare`'s handling of case, accents, or non-Latin scripts, is
a library and locale detail nobody decided. Asserting the first is contract; asserting the
second is heuristic 4 territory and belongs in a "not guaranteed" line instead.

## Name tests as the promise they encode

`returns null when there are no results` — not `summarize works` or `test case 2`.

The suite becomes a readable contract a newcomer can use instead of the source; and a
behaviour whose test name cannot be phrased as a promise is usually an accident that slipped
through A3. Trouble naming the test is a signal to go back.

The name binds in the other direction too: **the assertions must be sufficient to prove
the name's full claim.** A test named `records the payment successfully` whose only
assertion checks that an ID came back proves less than it announces — an ID can exist
while the write failed. Read the name as a specification of the assertions: whatever the
name claims and the assertions do not prove is unearned confidence. Either strengthen the
assertions or narrow the name.

**One test, one promise.** A test proves exactly one promise; how many assertions that
takes is how many it gets — several assertions jointly proving one behaviour are fine,
and "one assertion per promise" in the rules table bounds what a single assertion may
claim, not how many a test may hold. Two promises sharing a test is a signal to split
the test, never to trim the assertions.

Lay the test body out as arrange–act–assert, in that order, with one blank line between
the sections and no other blank lines inside the test. This is cosmetic, but it is the
one cosmetic rule this skill fixes rather than delegating to the repository: the shape
makes a failing test legible at a glance — what was set up, what ran, what was expected.

Nested `describe` blocks should group a sub-condition so the full path still reads as a
sentence. Let the contract decide how many tests a unit gets — never a quota, in either
direction.

## Mocking

Mock the system boundary; run everything inside it for real. The boundary is the network, the
database, the filesystem, the clock, randomness, and third-party SDKs.

Mocks of internal collaborators encode the current call graph — precisely what refactoring
rearranges — so they turn every internal change into a test change. Boundary mocks encode a
real contract with the outside world, which is why asserting *those* calls is legitimate: the
call itself is the promised side effect. The clock and randomness count as boundaries;
freezing them is determinism, not coupling.

**Before choosing a double, ask what this test must prove.** If the promise is the
outgoing call itself — a command crossing the boundary, where the call *is* the side
effect — assert the call. If the promise is what the unit does with data coming back — a
query — the double exists only to supply that data, and asserting how it was called
couples the test to implementation.

**For the network, intercept at the protocol level** — MSW or an equivalent — rather
than mocking the HTTP client's module. A mock of `axios` fails Question 1: rewrite the
unit to use `fetch` and no consumer can tell the difference, yet the test breaks,
because it pinned *how* the request is made instead of *what* crosses the wire.
Protocol-level interception survives that rewrite, and it fails loudly on requests no
handler expected (`onUnhandledRequest: 'error'`) — a request the test did not anticipate
is a finding, never something to let pass silently. The interceptor is a dev-only
dependency and follows the same proposal rule as the instruments section: propose,
agree, then install.

Note that failing on unhandled requests does not catch a *duplicated* request — the
second call to a handled endpoint is still handled. When A3 classified "one request per
resource" as a promise, assert the handler's call count explicitly: that assertion is
what catches the duplicate before anyone sees it in DevTools, and without it a
duplicate is a behaviour change no test notices — a Question 2 failure.

To intercept a single export from an adjacent module, use a partial mock that keeps the real
implementation for everything else — replacing a whole module hides real behaviour and turns
the test into a test of the mock:

```ts
vi.mock("./submit", async (importOriginal) => {
  const real = await importOriginal<typeof import("./submit")>();
  return { ...real, submitCase: (...args: unknown[]) => submitCase(...args) };
});
```

## Test independence and fixtures

Every test must pass alone, in any order, and in parallel — runners parallelise by
default, and a suite that only passes serially has hidden coupling that will surface as
flake. No test may read state another test wrote: no module-level mutable variables
shared across cases, no `beforeAll` state that cases mutate, no reliance on a previous
test's side effects.

The common carrier of hidden coupling is the fixture. A fixture is a function returning a
fresh deep copy — never an exported object:

```ts
// never — a shared object: one test's mutation leaks into every test that
// touches the fixture after it, non-deterministically under parallel runs
export const paymentOkResponse = { ... }

// always — a factory returning a fresh deep copy per call
export const paymentOkResponse = () => structuredClone({ ... })
```

Keep the `structuredClone` even when the literal is built inside the factory: nested
references to shared constants slip into a literal unnoticed, and the clone is cheap
insurance. Fixture *placement and naming* are cosmetic and follow the repository; fixture
*mutability* is substantive and follows this rule.

## When a unit test fails after a refactor

Treat the failure as evidence about the test before it is evidence about the code:

- **Implementation only, behaviour identical** → the test asserted an accident. Narrow the
  assertion. Do not restore the old internals to please the test.
- **The contract changed on purpose** → update the test and the documented contract line
  together, and check the consumers found in A1.
- **The contract changed by accident** → that is the bug the test exists for. Fix the code.

The move to avoid is loosening the assertion until it passes without deciding which of the
three happened — that leaves a test still present but no longer asserting anything, which is
a Question 2 failure created on purpose.

## When the work uncovers a product defect

Holding code to its promises finds broken promises. A defect surfacing mid-engagement is
this skill working, not the work going wrong, and it arrives by several routes:

- a test written for a genuine promise fails — the unit does not honour its own contract;
- review finds an existing test asserting behaviour that contradicts the contract — a bug
  pinned green and documented as if it were required;
- Phase A turns up a consumer relying on behaviour the unit observably does not deliver;
- the pre-change suite run is already red for a reason that is not environmental.

The protocol is the same on every route:

1. **Do not silently fix the product code.** The engagement is tests; a product change is
   a scope change that belongs to whoever owns the code. The one exception is a bug your
   own change introduced this session — that one is yours, fix it on the spot.
2. **Report it explicitly**, in consumer terms rather than internals: what was promised,
   what actually happens, the evidence — failing test, call site, reproduction — and
   which consumers are affected.
3. **Offer to file it** in whatever the project uses to track work. Which tracker and by
   what mechanism is a runtime decision, not this skill's to prescribe.
4. **Do not bend the test to green.** Asserting the buggy behaviour freezes the bug as
   contract. Keep the test as written and mark it with the runner's expected-failure
   facility (`it.fails` in vitest, `it.failing` in jest) so the defect stays visible
   without breaking the suite — or hold the test out pending the decision; offer that
   choice alongside the report.

## Orienting in an unfamiliar repository

Keep this strictly to the cosmetic column: the runner and how to invoke it (`package.json`
scripts, `vitest.config.*`, `jest.config.*`), the test file pattern and whether tests are
colocated, and one nearby test file read for import style, fixture placement, and formatting.

Run the suite once before changing anything, so you know which failures are yours. If a unit
needs a test in a package with no runner at all, stop and ask — standing up a test harness is
the engineer's decision, not a side effect of writing a test.

Read that nearby file for idiom, not for permission.

## Close with a summary

End every engagement — authoring or review — with a compact summary by category, built
for fast assimilation: one line per category, counts and locations, omit empty
categories, never a wall of prose.

| Category | What to state |
|---|---|
| Tests created | file, count, and the promises they cover |
| Tests changed | what was strengthened or narrowed, and why |
| Assertions removed | which accidents they froze |
| Mutation run | score; survivors triaged as gaps filled / accidents left alive / equivalent — or the blocker that stopped it |
| Defects found | each: the promise broken, the evidence, and the filing offer made |
| Contract lines written | where: TSDoc, type, schema, database comment |
| Open questions | undecidable classifications awaiting a human call |
| Precedent diverged from | the convention not followed, in one sentence |

The Phase A table shows what the unit promises; this table shows what was done about it.

## Before you finish

Both modes:

- [ ] Phase A output exists and is visible: classified behaviours, invariants, verdict
- [ ] If Phase A was delegated: manifest checked against your own consumer grep, missed
      behaviours added to the table, no evidence-free reclassification of the subagent's entries
- [ ] Nothing classified by guesswork — undecidable items raised as questions, not asserted
- [ ] Every assertion passes Question 1 (survives a rewrite)
- [ ] Every assertion passes Question 2 (fails when the promise is removed)
- [ ] Every test's assertions are sufficient to prove its name's full claim
- [ ] Mocks only at system boundaries
- [ ] Network interception is protocol-level, or the summary names the reason it is not
      (declined install or technical blocker) — never a silent fallback to mocking the
      HTTP client's module
- [ ] Tests pass in any order — no shared state; fixtures are `structuredClone` factories
- [ ] Every invariant A4 named is a property-based test, or its exclusion is named in the
      summary with the reason it does not fit
- [ ] Mutation run covered the whole module — the Stryker install proposed and agreed
      first if it was missing — and every survivor is triaged as gap, accident, or
      equivalent; a run that did not happen names its reason in the summary: a declined
      install or a technical blocker, never silence
- [ ] Any product defect uncovered was reported with evidence and a filing offer — not
      fixed silently, not asserted around
- [ ] Any defective precedent you chose not to follow is named, not silently diverged from
- [ ] Closing summary delivered by category — counts and locations, not prose

Authoring adds:

- [ ] The verdict in A5 was yes, and for a stated reason
- [ ] Anything expressible as a type or a database constraint moved there instead of a test
- [ ] Each promise is exercised per equivalence class and at the boundaries between classes
- [ ] Test names read as promises

Review adds:

- [ ] Both directions covered: over-assertion **and** promises with no test
- [ ] Gaps ranked — worth adding versus over-testing
- [ ] Findings carry falsifiable evidence, not impressions

## References

- `references/property-based.md` — read when A4 produced an invariant, or when the input
  space is too large for examples.
- `references/contract-doc.md` — templates for the "Guarantees / Not guaranteed" line, and
  how to express a contract in the type system instead of prose.
