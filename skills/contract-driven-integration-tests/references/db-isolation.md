# Isolating the real test database

Read this at the instruments step, when the verdict says the slice needs integration
tests and the isolation strategy has to be chosen — it shapes every fixture, so it is
chosen before the first test is written, not retrofitted after the first flake.

## The strategies, and when each fits

They combine per context; a suite commonly uses two of them at once.

| Strategy | Mechanics | Fits when | Watch out |
|---|---|---|---|
| **Per-test data ownership** | Each test creates its own rows under its own unique keys, asserts only against them, deletes them in `afterEach` | Shared test database; suites that must run beside other suites or workers | Uniqueness must be real (timestamp+random ids, per-run prefixes); cleanup deletes only what the test created — never `TRUNCATE` on a shared database |
| **Transaction rollback** | Open a transaction per test, run the flow inside it, roll back in teardown | Fast suites over flows that do not themselves manage transactions | A flow that opens its own transaction (`BEGIN`/`COMMIT` in the code under test) cannot run inside the test's — nested behaviour differs; and committed-state assertions from a second connection see nothing |
| **Reset + seed** | Drop to a known schema and load minimal seed data before a suite or run | A dedicated database per suite run; CI jobs with their own instance | Between whole runs, not between tests — per-test resets are slow and hide ordering bugs; never against a database other suites share |
| **Database per worker / container** | Testcontainers or a template-database clone per parallel worker | Parallel CI at scale; the opt-in mutation run (workers must not share state) | Startup cost per container; keep schema application scripted, not manual |

The seed rule holds across all of them: seed data is the *minimal realistic
precondition* — the product that must exist before its review — created for this test,
not inherited from a previous run and not a dump of production-shaped everything.

## Shared-database hazards

Two failure modes account for most "flaky integration suite" reports:

- **File parallelism over one database.** Runners execute test files in parallel
  workers by default; two files that both `TRUNCATE` or both assert on global counts
  corrupt each other mid-test, non-deterministically. Either every file owns its data
  (first strategy above), or file parallelism is disabled for the integration project
  (`fileParallelism: false`), or workers get separate databases. Pick one explicitly;
  the default configuration has already picked *corruption* for you.
- **The fallback connection string.** A pool built on
  `TEST_DATABASE_URL ?? DATABASE_URL` silently targets a real database when the test
  variable is unset. Guard the suite: fail loudly in `beforeAll` when the dedicated
  test URL is missing, rather than letting cleanup code loose on whatever the
  fallback points at. The same applies per pool — a slice that constructs several
  pools (its own, a neighbour's) needs every one of them wired to the test instance.

## Provoked-failure recipes

The reason the real database is in the suite. Each recipe asserts two halves: the
refusal, and the state the refusal left behind.

**The duplicate** — the storage-level uniqueness promise:

```ts
it('refuses a second review from the same user and keeps the first intact', async () => {
  const productId = await seedProduct()
  await submitReview({ productId, userId: 7, rating: 5 })

  await expect(submitReview({ productId, userId: 7, rating: 1 }))
    .rejects.toBeInstanceOf(DuplicateReviewError)

  expect(await countReviews(productId)).toBe(1)
  expect(await readAggregate(productId)).toEqual({ avg_rating: '5.00', reviews_count: 1 })
})
```

**The concurrent conflict** — two near-simultaneous writes, one predictably refused.
Deterministic against a correct implementation, because the assertion constrains the
*set of outcomes*, never the interleaving:

```ts
it('lets exactly one of two concurrent submissions through', async () => {
  const productId = await seedProduct()
  const attempt = () => submitReview({ productId, userId: 7, rating: 4 })

  const results = await Promise.allSettled([attempt(), attempt()])

  const fulfilled = results.filter((r) => r.status === 'fulfilled')
  expect(fulfilled).toHaveLength(1)
  expect(await countReviews(productId)).toBe(1)
})
```

"Concurrency tests are flaky" is true of assertions on *timing* — who finished first,
how long it took. The exactly-one invariant has no timing in it: whichever call wins,
the set of outcomes is the same, and a suite that cannot afford this test is a suite
that cannot prove the promise the double-click depends on.

**Mid-flight failure** — the atomicity promise, proven by making a *later* write fail
and reading the *earlier* one back:

```ts
it('rolls back the order when an item insert is refused', async () => {
  const items = [validItem(), { ...validItem(), quantity: 0 }] // second violates CHECK

  await expect(createOrder(maxRightsUser(), items)).rejects.toThrow()

  expect(await countOrders(maxRightsUser().id)).toBe(0) // the first insert did not survive
})
```

The refused input must arrive *after* at least one write has landed inside the
transaction — a validation error thrown before any write proves input checking, not
atomicity.

## Cleaning up after the engagement

A database or container the engagement started is shut down when the engagement ends;
a suite's `afterAll` closes every pool it opened, including pools imported from
modules the slice pulled in, so runs exit cleanly instead of hanging the worker.
