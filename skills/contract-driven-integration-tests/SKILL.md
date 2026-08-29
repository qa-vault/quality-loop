---
name: contract-driven-integration-tests
description: >-
  Hold several real parts of a system to what they jointly promise — a flow and its
  database, a component tree and its store — asserting the joint outcome, mocking only
  where the slice's own code ends, and provoking the failures the real infrastructure
  exists to catch. Use whenever writing, adding, or reviewing integration tests in
  JavaScript or TypeScript — vitest, jest — including "write an integration test",
  "test this flow against the database", "cover the store and components together" —
  and whenever a test's level is in question: "unit or integration?", "do we need the
  real database?". Also before mocking a store, a database, or an adjacent module in
  any test that renders more than one component or crosses a module boundary. Extends
  contract-driven-unit-tests to slice scope and overrides its mocking rule there. Not
  for end-to-end tests that drive the full application the way a live user does.
---

# Contract-Driven Integration Tests

## Why this exists

The three test levels are not more and less thorough versions of the same check. They
are three different questions, and the difference is which door the test enters the
system through: a unit test walks straight into one function; an integration test
enters at the boundary of several real parts working together; an end-to-end test
enters through the same interface a live person uses.

Integration suites go wrong at that door, in three characteristic ways. A test enters
at the slice boundary but mocks the parts inside it — a unit test wearing an
integration name, green while the wiring it claims to prove is out of the game. A test
enters at the right door but asserts every included function's behaviour along the
way — a complicated unit test that re-proves contracts already proven where they are
cheap. And a test keeps swallowing adjacent concerns until it is a slow, brittle
almost-E2E that breaks for five different reasons in five weeks.

So the job is not "write an integration test". It is: work out what this collaboration
of parts jointly promises, then hold one test to exactly that — with everything inside
the collaboration real, and everything outside it mocked at the seam.

**REQUIRED BACKGROUND:** this skill extends `contract-driven-unit-tests` and assumes
its discipline in full. Promise versus accident, the two questions every assertion
must pass, test names that read as promises, arrange–act–assert, fixture factories,
test independence, the refactor-failure triage, the product-defect protocol, and
"existing conventions do not get a vote" all apply here unchanged and are not
restated. This document states what changes at slice scope — and one of its rules
deliberately **overrides** that skill's mocking rule, as marked below.

## Phase 0 — decide the level before anything else

Every engagement opens with one question, asked before any file is read for style and
before any test is sketched:

**What exactly must break for this test to fail?**

- *One function computed, compared, or decided wrongly* → a **unit** test. Hand the
  work to `contract-driven-unit-tests`.
- *The system's data ended up in the wrong state after several parts worked
  together* — a status changed but its history row did not, a record appeared but the
  aggregate did not move — → an **integration** test. This skill.
- *The user's own path, gesture, or screen misbehaved* → an **end-to-end** test. Out
  of this skill's scope; say so and stop rather than approximating it here.

The level is an answer to that question — never an inheritance from whatever test file
happens to sit nearby. A repository whose nearest spec is an integration test is not
evidence that this behaviour needs one, and the reverse holds equally.

Two routings deserve their own line because each direction of error is common:

- A pure rule reachable without any infrastructure is unit territory even when it is
  *used by* a flow this skill tests. Test the rule's full decision table there, where
  examples are cheap; the integration test then exercises the flow through one or two
  representative values, proving the rule is *wired in*, not re-proving the table.
- A promise that only holds because several parts commit together — atomicity, a
  storage constraint, a derived value recomputed by shared state — cannot be proven at
  unit level against a mock, however thorough the unit suite looks. That promise is
  this skill's subject.

When the question is asked of a mixed task — "test the card transition logic" — the
answer is usually *both levels, split*: the rule as a unit suite, the application of
the rule as one integration test. Splitting is the routing working, not overhead.

## The subject is a vertical slice

An integration test covers a **vertical slice**: the set of parts that together
implement one business capability — the `orders` modules, their tables, their store —
as opposed to a horizontal layer (all controllers, all stores). The slice is the unit
of value; the layers are how it happens to be built.

Three things surround a slice, and each is handled differently:

- **The slice itself** — its modules, its tables, its store, its components. Real,
  all of it. This is what the test exists to prove.
- **Neighbouring slices** — modules the slice calls that implement a *different*
  capability: reporting called after an order, notifications sent after a change.
  Mocked at the module seam. Their behaviour is their own suite's job.
- **Обвязка (scaffolding)** — what runs *before* the slice's business logic gets
  control, deciding whether it runs at all: the authorization layer is the canonical
  case. Neutralized, not tested: hand the flow a user with maximal rights so every
  gate opens, and leave the gates' own logic to the suite that owns them. (GitLab's
  integration suites stub their `can` check to an unconditional *yes* for exactly this
  reason.) One access-denied smoke test is defensible when the wiring of the gate is
  itself the slice's promise; a matrix of role, ban, and verification cases inside a
  business-flow suite is scope leak.

A warning that saves real time: the slice concept may not exist anywhere in the
repository's structure. Directory-per-feature layouts express it; layer layouts smear
one slice across the tree. When the structure does not show the slice, map it by hand
in Phase A — the map is the deliverable that makes every later decision checkable.

## The real-parts rule

Everything inside the slice runs real, and the two most-tempting substitutions are
named because each quietly converts the test into something else:

**The database is not mocked.** The caller of a flow does not see an API layer and a
storage layer; it sees one system and one result, and the test takes the same view. A
mocked database answers *success* to anything, so three whole classes of promise
become unprovable: integrity constraints (the storage-level rule that a customer
reviews a product once), multi-write consistency (the new row and the recalculated
aggregate committing as one), and conflict behaviour (two near-simultaneous requests,
one predictably refused). Those are not edge cases of the slice's contract — at this
level they usually *are* the contract, and they are why the real database is here.

**The store is not faked, and neither is the browser.** When an action in one node of
a component tree must show up in another node through shared state, the store's real
logic is the thing under test — a fake cannot recompute derived values the way the
real code does. And a component tree is code written *for a browser*: run it in one.
jsdom is a mock of the browser, and this skill's own rule applies to it — use Vitest
Browser Mode with a real browser instance, not a simulation of one. A project
configured for jsdom is a convention, and conventions do not get a vote; propose the
browser-mode setup as an instrument (below) rather than inheriting the simulation
silently. This is not E2E: the test still enters through the component and the store —
the slice's door — not through the application shell a user walks through.

## The mock line — where your code ends

> **This section overrides `contract-driven-unit-tests` at slice scope.** That skill
> teaches: mock the system boundary; mocks of internal collaborators encode the call
> graph and turn refactors into failures. At integration level, applied verbatim,
> that rule produces the failures this skill exists to prevent — "мocking `can` would
> test the mock" is how authorization matrices end up inside business-flow suites, and
> "running the neighbour validates the full path" is how one slice's test starts
> owning another slice's behaviour. The boundary that matters here is the **slice
> boundary**, not the process boundary.

The line is drawn by ownership, never by convenience: **mock where the slice's own
code ends** — and nowhere before it.

| Seam | Treatment | Why |
|---|---|---|
| The network / third-party services | Intercepted at protocol level — MSW | Foreign system; the request that crosses the wire *is* the observable side effect |
| Neighbouring slices | Mocked at the module seam (partial mock keeping the rest of the module real) | Their contract is their own suite's job; asserting *that they were invoked with the right data* is this slice's promise, their internals are not |
| Обвязка (auth and friends) | Neutralized — max-rights user or an unconditional-yes stub | It runs before the subject; testing it here is a different scope |
| Everything inside the slice — modules, store, database, components | Real | This is the subject; substituting any of it un-tests the wiring |

Three named traps, each observed in practice:

- **The store-method mock.** Mocking the store's action instead of the network request
  inside it is shorter to write and matches many codebases' habits — and it removes
  the store's logic from the game entirely. The test keeps its integration name and
  silently becomes a unit test of two components against a fake. The mock belongs one
  layer deeper, at the fetch the store performs.
- **"Mocking the collaborator would test the mock."** True at unit scope; at slice
  scope, check *whose slice the collaborator belongs to* first. Inside the slice the
  sentence is correct — run it real. For обвязка and neighbours it is the trap itself:
  the стаб of `can` is not a mock of the subject, it is the removal of a different
  subject from the frame.
- **"Running the neighbour validates the full path."** It does — and it also makes
  this suite fail when the *neighbour's* schema, provider, or logic changes, which is
  precisely the brittleness integration suites are notorious for. The full path
  belongs to E2E; this test proves the slice hands the neighbour the right data, at
  the seam.

When a needed seam is unclear — a module that looks shared between slices, a helper
that might be обвязка or might be the subject — that is a Phase A question to raise,
not a coin to flip silently.

## How this works

The same two-phase seam as the unit skill, for the same reason: the model of the joint
contract is built before any test is read or written, so gaps show up by subtraction
rather than staying invisible behind whatever assertions already exist.

```
PHASE A — investigate the slice
    map → contract sources → joint behaviours → classify → mock line → verdict
    Output: a model of the slice's contract. No tests involved yet.

PHASE B — act on the model
    authoring: write the tests that cover the model
    review:    compare existing tests against the model, in both directions
```

Delegating Phase A to a fresh agent follows the unit skill's five rules unchanged
(carry decisions not implementation, bound the scope, falsifiable evidence, an
investigation manifest, verify before acting) — with the bias reason doubled: a
session that wrote any module *inside* the slice will classify that module's accidents
as promises from memory. The delegated scope rule for this skill reads: the slice's
own source, its consumers' call sites, neighbours' and обвязка's public signatures —
never neighbours' implementations.

### A1. Map the slice

Name the business capability, then list what implements it: modules, tables, store
slices, components. Then name what surrounds it: every neighbouring slice it calls,
and every piece of обвязка that runs before it. Grep the call sites; do not assume.
When the repository's structure does not express the slice, this map is built by hand
and shown — it is the reference every mock-line decision points back at.

### A2. Survey the contract sources

Collect what is already committed to about this slice's behaviour, from **every**
source available, not the first one found: schema definitions (zod, JSON Schema,
OpenAPI, GraphQL SDL, `.d.ts`), database constraints in migrations, API docs, and
tests at other layers that pin a behaviour. Two rules govern the survey:

- **Conflicts between sources are findings, not noise.** Documentation ages; a spec
  and a client that disagree on field names mean one of them is wrong in production.
  Surface the conflict for a human decision — never silently pick the side that makes
  the tests pass. `references/contract-testing.md` covers turning the resolved
  contract into an executable schema both sides test against.
- **Never derive an external service's contract from one observation.** Calling the
  service once and building the mock from what came back bakes a guess in as a
  contract — what came back is one shape the service *can* produce, not the set of
  shapes it *may* produce. The mock's shape comes from the surveyed sources; where no
  source exists, that absence is itself a finding to raise.

### A3. List the joint observable behaviours

Everything a consumer of the slice could notice, with the emphasis on what makes this
level worth its cost — outcomes no single unit exhibits alone:

- several writes that must land together, and what is observable when one of them
  fails mid-flight;
- values derived across parts — an aggregate recomputed by a trigger of shared state,
  a badge computed from a store another component mutated;
- what crosses each seam of the mock line: which request, with which payload, how many
  times;
- behaviour under storage refusal: the constraint violation, the conflicting
  concurrent write;
- what the slice returns to its caller, and what it throws for each refused input.

Be exhaustive here and judgemental at classification, not the other way round.

### A4. Classify: promise or accident

The unit skill's heuristics apply verbatim (typed/schema'd → promise; depended-on →
promise; defensible decision → promise; data-structure fallout → accident; internal
operation counts → accident), with one addition that earns its place at this level:

**A unit's own contract, already proven in that unit's suite, is not re-promised
here.** The discount table, the validation matrix, the formatting rule — each belongs
to its unit's tests, one example per equivalence class, where failure is cheap and
precise. The slice's promise is that the unit is *wired in and its result lands*:
one representative value through the flow proves that. Re-running the table through
the database re-proves what is already proven, at fifty times the cost, and makes the
integration suite fail for reasons a unit test would have localized in seconds.

### A5. Draw the mock line

For each item in A1's map, write down: real or mocked, at which seam, and why — one
line each. This is where the section above becomes concrete and checkable. A mocked
module inside the slice, or a real neighbour outside it, must survive being read here
with its stated reason, or it is wrong.

### A6. Verdict and scope check

Does the slice warrant an integration test at all? Yes when A3 produced at least one
genuinely joint promise — something no unit suite can prove against a mock. No when
every listed behaviour is one unit's contract plus plumbing; that verdict routes the
work back to Phase 0's unit branch.

Then the scope check: count the entities in play — parts of the slice, tables, seams.
**5±2 is the working bound**, three to five the comfortable range. Six or more says
split the test along its joint promises — or, if the scenario is genuinely one
indivisible user journey, promote it honestly to E2E rather than growing an
integration test into one. Diagnostics that a test has swallowed too much, useful in
review: the setup dwarfs the assertions; "why could this fail while every other test
is green?" has many answers; the same test gets repaired for the fifth time for the
fifth different reason.

**Output of Phase A**: the slice map, the surveyed sources with any conflicts, the
classified behaviour table (same fixed columns as the unit skill: *behaviour |
classification | heuristic | evidence (file:line) | open question*), the mock line,
and the verdict. Compact and checkable; show it before writing tests — the mock line
and the classification are what a human should review.

## Standing up the instruments

Checked after the verdict, before the first test — and governed by the unit skill's
proposal rule: a dependency is always the engineer's call, the proposal is not
optional, and a declined install goes into the summary as the reason a step did not
run. What is never acceptable is the silent fallback — stubbing `fetch` because MSW
is not in `package.json`, inheriting jsdom because the config already says so. The
project not having the dependency is a fact about the project, not an argument about
the test.

- **MSW** — the priority instrument for the network seam, at protocol level
  (`onUnhandledRequest: 'error'`, always — an unexpected request is a finding). In
  Node suites via `msw/node`'s `setupServer`; in browser-mode suites via
  `msw/browser`'s `setupWorker` from a setup file.
- **Vitest Browser Mode** — for any slice containing components:
  `@vitest/browser-playwright` provider plus `vitest-browser-react` (or the framework
  equivalent), with locator-based assertions (`expect.element`). Propose it whenever
  the frontend leg of a slice is in scope and the project still runs components in
  jsdom.
- **A real test database and an isolation strategy** — transactions rolled back,
  reset-plus-seed, or per-test data ownership, combined per context; the catalogue,
  the trade-offs, and the shared-database hazards are in
  `references/db-isolation.md`. The strategy is chosen *before* tests are written,
  because it shapes every fixture.

## Phase B (authoring) — write the tests

Each test proves one joint promise from the model, named as that promise, laid out
arrange–act–assert. Beyond the unit skill's rules, four are specific to this level:

**Assert every half of a joint promise.** "Status and history change together" is one
promise with two observable halves; asserting the status and trusting the history
turns the test into half a proof. The same applies to the returned value versus the
committed state: a flow that returns the new aggregate *and* commits it is only proven
when both are read — the return from the call, the state from a fresh query.

**Provoke the failures the real infrastructure is here for.** The happy path plus a
green suite is the documented stopping point of an agent that has already agreed to
use the real database — and it uses none of what the real database was brought in to
prove. For every constraint and conflict promise in the model, there is a test that
*makes the storage refuse*:

- *the duplicate*: perform the write twice; assert the domain error on the second
  attempt **and** that the committed state is exactly what the first write left —
  the rollback is the other half of the promise;
- *the concurrent conflict*: fire the conflicting calls together —
  `Promise.allSettled([op(), op()])` — and assert the invariant over the outcome:
  exactly one fulfilled, the state advanced exactly once. This is not a flaky test
  waiting to happen: against a correct implementation the assertion is deterministic,
  because it constrains the *set of outcomes*, not the interleaving. "Concurrency
  tests are too flaky to write" describes racy assertions on timing, not this
  pattern;
- *the refused input at the boundary*: the value the CHECK constraint rejects, the
  missing foreign row — asserting the error **and** that nothing was persisted.

Integration tests being expensive is not a reason to trim these; it is the reason the
happy path alone is not worth the suite's runtime. The investment is already made —
spend it on what only this level can prove.

**Count the calls that cross a seam.** At the mock line, the outgoing call is the
promised side effect, and *exactly one* of it is usually part of the promise — one
confirmation email, one charge, one event recorded. `toHaveBeenCalledTimes(1)` at a
seam is contract, not implementation; the duplicate send is a behaviour change no
other assertion notices.

**Keep the fixtures at the minimum the slice needs.** Seed data is the realistic
precondition — the product that must exist before its review — prepared per test and
owned by it, never inherited from a previous run or another test. Setup that has grown
to screens of preparation is the A6 diagnostic firing mid-authoring: stop and re-check
the scope before finishing the test.

## Phase B (review) — judge the existing tests

Both directions, from the model, exactly as the unit skill prescribes: joint promises
with no test are the gaps only model-first review can see; assertions with no promise
behind them freeze accidents. Three review findings are specific to this level, each
mapping to a section above:

- **the level mismatch** — a unit's decision table swept through the slice, or a
  "unit" test whose subject is a joint outcome (route each to its level);
- **the disguised unit test** — an integration-named test whose slice internals are
  mocked (store methods, own modules): the wiring it claims to prove is not in the
  game;
- **the scope leak** — обвязка matrices inside the flow suite, a real neighbour, a
  test failing for its fifth unrelated reason.

Report per test: what it claims, what it actually exercises given its mocks, and
whether its assertions cover every half of the claimed joint promise.

## Mutation testing at this level

The unit skill closes every engagement with a Stryker run. Here the economics invert:
a mutation run re-executes the suite per mutant, an integration suite carries real
I/O per test, and parallel mutation workers sharing one test database poison each
other's state — so a full run is **opt-in, not the default closer**. The default
verification at this level is the provoked-failures set plus the manual form of
Question 2 where doubt remains: disable the promise (drop the constraint in a scratch
schema, comment the second write) and watch the test notice.

Opt in to a real mutation run for a slice whose correctness budget justifies hours —
money paths, permission-adjacent flows — and then honestly: mutants scoped to the
slice's own modules, one isolated database per worker (or concurrency 1), timeout
factors sized for I/O variance, and every survivor triaged with the unit skill's
promise/accident/equivalent protocol. What is never acceptable is the silent middle:
neither run nor declared. The summary states which verification closed the
engagement — provoked failures, manual disabling, or a mutation run — and why.

## When the work uncovers a product defect

The unit skill's protocol applies unchanged: no silent product fixes, the report in
consumer terms with evidence, the filing offer, and never bending a test green around
a broken promise (`it.fails` / held out pending the decision). Two defect routes are
characteristic of this level and worth naming: a contract-source conflict from A2
(spec and client disagree — one side is wrong in production even though every suite is
green), and a consistency promise the flow observably does not keep (the write lands,
the aggregate does not move). Both are product findings, not test problems.

## Close with a summary

The unit skill's category table, with this level's rows added where they apply:

| Category | What to state |
|---|---|
| Level routing | anything re-routed to unit or promoted to E2E, in one line each |
| Slice map & mock line | where they are recorded; any seam decided against the default, with its reason |
| Tests created / changed | file, count, the joint promises covered |
| Provoked failures | which constraint/conflict promises have a refusing test — or which lack one and why |
| Contract sources | sources surveyed; conflicts found and escalated |
| Verification | what closed the engagement: provoked failures, manual disabling, or an opt-in mutation run (score and triage) |
| Defects found | each: the promise broken, the evidence, the filing offer made |
| Open questions | unresolved seams and classifications awaiting a human call |
| Precedent diverged from | the convention not followed, in one sentence |

## Before you finish

- [ ] Phase 0 answered explicitly: what must break for each test to fail, and the
      level chosen from that answer — not from the nearest test file
- [ ] Pure rules routed to unit level; their tables not re-proven through the slice
- [ ] Phase A output exists and is visible: slice map, surveyed sources, classified
      behaviours, mock line, verdict
- [ ] Everything inside the slice runs real — database, store, own modules; any
      exception is named in the summary with its reason
- [ ] Component slices run in a real browser (Vitest Browser Mode), or the summary
      names the declined install / blocker — never an inherited jsdom by silence
- [ ] The mock line: network at protocol level (MSW, `onUnhandledRequest: 'error'`),
      neighbours at the module seam, обвязка neutralized (max-rights user) — and no
      mock inside the slice
- [ ] No silent fallback instruments — every missing tool proposed, and a declined
      install recorded in the summary
- [ ] Every joint promise asserted in all its observable halves (return value and
      committed state; both sides of writes that land together)
- [ ] Every constraint and conflict promise has a provoked-failure test: duplicate,
      concurrent (`Promise.allSettled`, exactly-one invariant), refused input — each
      asserting the error and the untouched state
- [ ] Calls crossing a seam are counted where "exactly once" is the promise
- [ ] Scope check passed: 5±2 entities; oversized tests split or honestly promoted
      to E2E
- [ ] Tests own their data: seeds created per test, cleaned by the test, safe on a
      shared database — strategy per `references/db-isolation.md`
- [ ] Contract-source conflicts surfaced as findings, never resolved by picking the
      convenient side; no external contract derived from a single observation
- [ ] Verification that closed the engagement is named; a mutation run that did not
      happen is a stated decision, not a silence
- [ ] Any product defect reported with evidence and a filing offer — not fixed
      silently, not asserted around
- [ ] Closing summary delivered by category — counts and locations, not prose

## References

- `references/db-isolation.md` — isolation strategies for the real test database
  (transactions, reset-plus-seed, per-test ownership, containers), shared-database
  hazards, and the provoked-failure recipes in runnable form.
- `references/contract-testing.md` — executable contracts between independently
  developed sides: schema-as-third-source, both-sides verification, the
  contract-source survey protocol, and when contract tests earn their cost.
