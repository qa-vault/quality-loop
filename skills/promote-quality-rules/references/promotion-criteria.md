# Promotion Criteria — Calibration

Worked examples for the four-way weighing (severity · formalizability · recurrence · false-positive cost). Patterns are described in prose deliberately — rule code is authored fresh against the live schema at promotion time, never copied from anywhere.

## Formalizable — Semgrep-rule territory

- **"Raw HTTP call instead of our client wrapper."** Structural match on the forbidden call outside the wrapper's own file. High recurrence (generated code reaches for the standard library by default), low false-positive surface, cheap to fixture. The canonical promotion.
- **"Hardcoded color / spacing literal in components instead of design tokens."** Pattern match on literals in the component tree, excluding the token definitions themselves. Same profile: recurring, mechanical, precise.
- **"Secret-looking value in a client-exposed configuration channel."** Severity alone justifies proposing this on first occurrence — data-exposure class, zero tolerance for recurrence.
- **"Multi-step write without the project's atomicity mechanism."** Formalizable when the mechanism is nameable in code (a required wrapper or call pattern); severity is data integrity. If the boundary between "multi-step" and "single" needs judgment, split it: formalize the nameable core, leave the judgment rim to review guidance.

## Not formalizable — review-guidance territory

- **"This abstraction is wrong / this belongs a layer down."** Recognizing it requires understanding intent. Permanently reviewer territory; if it recurs, it becomes a *written* guidance line so the reviewer flags it consistently — never a syntactic rule that would misfire constantly.
- **"Error messages here should tell the user what to do next."** A quality bar, not a pattern. Guidance.
- **"Prefer narrow interfaces at module boundaries."** Architectural taste with genuine exceptions — a rule would need so many escape hatches it would teach suppression habits. Guidance.

## Proposable from a single occurrence

Security exposure, data loss/corruption, a violated architectural invariant the project explicitly declares. For these, recurrence likelihood is irrelevant — the cost of one more occurrence dominates the weighing.

## Usually not worth proposing at all

- A style nit with a wide false-positive surface (the "many exceptions to be tolerable" smell) — the CI tax outlives the annoyance it prevents.
- A pattern the project is actively migrating away from anyway — the rule would fire on scheduled work.
- Anything traceable to a one-off mistake rather than a habit — fixed is fixed.

## The tiebreaker

When the weighing is genuinely close, propose it and say the weighing is close — the user owns long-tail decisions, and a declined proposal costs one paragraph. What is *not* acceptable is silently deciding a borderline critical pattern wasn't worth the user's minute.
