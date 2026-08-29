# Contract testing — the third source of truth

Read this in Phase A when the slice talks to an independently developed side — a
backend consumed by a separately built frontend, an external service, another team's
microservice — and in Phase B when the mock of such a side is being written.

## The failure it exists to catch

Two sides of one feature are built in parallel against a prose description of the API.
Both suites are green — because each side tested itself against its *own* expectations:
the backend against itself, the frontend against its own mock. The first time the two
actually meet, the seam breaks, and no test saw it coming, because no test ever
compared the sides to each other.

Prose invites this: a document can be read inattentively, interpreted two ways where
it under-specifies, and never reopened after a change. Two agents reading the same
paragraph will each silently resolve its ambiguities their own way, and no reviewer
reading only one half of the code can see the divergence.

## The mechanics

A **contract** is a fixed, *measurable* promise agreed by two or more sides: required
fields, types, permitted values, error codes — an executable schema, not a paragraph.
Forms it takes, roughly in order of how much checking they give for free:

- runtime schemas — zod, Effect.Schema, and equivalents: validate live values in
  tests on both sides;
- interface definitions — JSON Schema, OpenAPI, GraphQL SDL, gRPC/protobuf: shared,
  generatable, diffable;
- TypeScript types (`.d.ts`): compile-time only, silent about runtime shape — the
  weakest form that still counts.

The test is a third, independent check that belongs to neither side's internal logic.
Both sides verify against the *same* contract, separately:

- the provider's test confirms the endpoint's real response conforms to the schema;
- the consumer's test confirms its code (and its everyday mocks) expect exactly the
  shape the schema describes.

There is no direct backend↔frontend comparison — each converses only with the
contract, so a divergence on either side fails immediately, before any shared
environment exists. This is what makes mocking safe at scale: the mock stays cheap in
daily work, and its drift from reality stops being a discovery made weeks after
release.

## The survey protocol

The contract for a mock is *collected*, never guessed. Before writing handlers for an
external or separately owned side:

1. **Enumerate every source available** — docs, OpenAPI/schema files, published
   types, database constraints behind the API, tests at other layers that pin shapes,
   and (as one source among several, never alone) a live call to the service.
2. **Compare them.** Sources age at different rates; a spec and a client that
   disagree on field names mean one of them is wrong in production right now. A
   conflict is a *finding* — surface it for a human decision, and record which side
   the tests were written against and why. Silently mocking the side that makes the
   tests pass converts a production bug into a green suite.
3. **Never build the contract from a single observation.** One response shows one
   shape the service *can* produce — not the fields that are optional, the enum
   values that appear under other conditions, or the error bodies. A mock built that
   way encodes a contract that does not exist, and every test against it inherits
   the invention.
4. **Where no source exists at all**, that absence is the finding. The options —
   adopt-and-document the observed shape as a provisional contract, or chase the
   owning team for a real one — are the engineer's call, made visibly.

## Wiring the schema into the tests

Once a schema exists, both everyday suites get an assertion that keeps them honest at
the seam, cheaply:

```ts
// consumer side: the mock itself is validated, so it cannot drift from the contract
const chargeResponse = ChargeResponseSchema.parse(buildMockCharge())

// provider side: the real response is validated before any field-level assertions
const body = ChargeResponseSchema.parse(await res.json())
```

A mock that fails its own schema is the divergence caught at authoring time — the
whole point, collapsed into one line per side.

## When it earns its cost

The value grows with the distance between the sides. One person writing both halves
will likely notice a divergence themselves; parallel teams meet divergences at
integration time; separately released services (many teams, independent deploy
cycles) may only meet them in production — there, the contract check is close to the
only way to learn of a break early, and far cheaper than an end-to-end environment
spanning every service. Inside one slice owned by one team, a shared schema module
usually does the same job with less ceremony — the discipline that stays constant is
the survey protocol above, whenever any mock of a separately owned side is written.
