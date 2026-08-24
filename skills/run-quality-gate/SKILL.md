---
name: run-quality-gate
description: >-
  Use before every push and before opening or updating a pull request in a repository with
  QUALITY-LOOP.md — and on demand ("run the gate", "check my changes against the rules") — to run
  the deterministic quality layer: Semgrep with the project's own rules plus the project's linters,
  typecheck, and other formal checks. Findings are fixed autonomously until green (fixes that would
  change product behavior escalate instead); a PR that fails the gate is never opened or updated.
  Tool or config errors are a broken gate, never a pass.
---

# Run the Quality Gate

## Overview

The deterministic layer of the quality loop: every formal check the project has, run locally, before code leaves the machine. The gate is cheap, reproducible, and blocking — it runs before the expensive AI review, and nothing that fails it gets a PR.

**The invariant this skill exists to uphold: never open — or update — a PR that fails the deterministic gate.**

The gate runs before code leaves the machine: before every push, and always before opening or updating a PR. Run it earlier (pre-commit) whenever it is cheap enough to be routine — but the push boundary is the hard line.

## Operating discipline

- **Discover, don't assume.** Establish the current CLI shape from the tool's own help output and current official docs — never from remembered flags or commands. Interfaces drift; the gate must not.
- **The gate is the whole formal stack.** Semgrep with the project's rules is the layer this process adds; the project's own linters, typecheck, and other formal checks are equal citizens. Discover them from the project's scripts and CI configuration and run them all. A "green gate" means all of it passed.
- **Principles over scripts.** What follows defines outcomes and failure semantics; the exact commands are whatever today's tools declare.

## Running the gate

1. **Assemble the stack.** From the project's scripts/CI config: the lint, typecheck, and other formal check commands. From `.semgrep/rules/`: the project's rules, run with a machine-readable output mode and the exit mode where findings fail the run. Run the rules' own fixture tests via the current fixture mechanism as part of the stack — a failing fixture is a broken gate (you can no longer trust what that rule asserts), not a code finding.
2. **Run everything.** Order cheap-to-expensive so the first failures surface while the rest still run; run all of it regardless of early failures — one pass should surface the full picture, not the first stumble.
3. **Read results with the failure semantics below.**

## Failure semantics — the part that must never be fuzzy

**Findings mean the gate failed — fix and re-run until green.** Any tool error, by contrast (bad config, invalid rule, crashed scan), means the gate itself is broken: fix the gate or escalate; never report a broken gate as a clean pass.

Concretely, three distinct outcomes:

| Outcome | Meaning | Response |
|---|---|---|
| Clean exit, no findings | Gate green | Proceed |
| Findings reported | Gate failed — the code violates approved rules | Fix the code autonomously, re-run |
| Tool/config error (invalid rule, parse failure, crash, missing tool) | **Gate broken** — you know nothing about the code | Fix the gate (or escalate); a broken gate blocks exactly like a red one |

The third row is the one careless automation gets wrong. Distinguish "the scan ran and found nothing" from "the scan did not run" by exit semantics and output — the current CLI documents which exit codes mean findings versus infrastructure errors; check, don't guess.

## Why fixes are autonomous

Every rule in `.semgrep/rules/` was approved by the user — the gate enforces *their* delegated decisions, so fixing violations needs no escalation. Fix the code to satisfy the rule, re-run, repeat until green.

Two boundaries on that autonomy:

- **A fix that would change observable product behavior is not a mechanical fix.** If satisfying a rule requires changing what a user of the product could notice, stop and escalate — that decision belongs to the user (the loop's escalation-by-impact-domain principle applies inside the gate too).
- **Comply, then appeal.** If you believe a rule itself is wrong or harmful: first make the code pass the rule as written, then escalate an argued proposal to revise the rule. Never silently bypass, disable, or weaken a rule — no inline suppressions, no rule edits, no "temporary" exclusions on your own authority.

## Scope of a run

The gate's subject is the change at hand, but its instruments run at their natural scope — typecheck and linters however the project runs them, Semgrep across the tracked tree. Diff-aware modes, where current tooling offers them, are a speed optimization for large repos — never a narrowing of what green asserts. When the user asks for a full-project pass, run everything on everything.

A repo whose `QUALITY-LOOP.md` exists but whose rules home (`.semgrep/rules/`) is missing is setup drift — treat it as a broken gate and route to the setup-quality-loop skill.

## Done means

All formal checks green on the code that is about to leave the machine — stated plainly, with a one-line summary of what ran. If the gate is broken rather than red, that is the headline, not a footnote. If an appeal was raised against a rule, it is noted for the cycle digest (the triage skill carries it to the PR); if no review cycle follows in this session, surface the appeal directly in the summary to the user.
