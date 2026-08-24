---
name: promote-quality-rules
description: >-
  Use after a review cycle closes in a quality-loop project, when the user asks "should anything
  become a rule?" / "make a rule for this" / "can we catch this automatically next time", or when
  the spec declares new invariants worth formalizing. Judges findings for promotion — severity,
  formalizability, recurrence likelihood, false-positive cost — and routes them: formalizable
  patterns become Semgrep rules with test fixtures in the project's rules directory; recurring
  non-formalizable patterns become review guidance in the repo's review config. Every promotion is
  a proposed diff for user approval, deduplicated against existing rules; the ratchet only tightens
  — weakening or removing a rule is its own explicit escalation.
---

# Promote Quality Rules

## Overview

The ratchet: the mechanism by which the loop learns. A class of problem that cost an AI review cycle to catch once should be caught by a rule forever after — at zero marginal cost if it is formalizable, or as standing review guidance if it is not. Promotion is always a proposal; the user approves every rule, because every rule is a permanent tax on CI and on every future PR. Long-tail decisions belong to the human.

## Operating discipline

- **Judgment, not counting.** There is no occurrence threshold. A critical pattern — security, data integrity, an architectural invariant — is proposable from a single occurrence; a shrug-grade nit is not proposable after ten.
- **Provenance governs your initiative.** Rules you author come from scratch against this project's own convention or incident, in the schema the current Semgrep documentation declares, with fixtures exercising both the violating and the clean pattern via the current fixture mechanism — never from registry rule YAML as authoring material. Rules the user explicitly asks to bring in from an external pack are vendored, not authored — a legitimate path (the setup skill's external-pack route, license note included) that passes the same fixtures / dry-run / approval bar as everything else.
- **Discover, don't assume.** Current rule schema, fixture mechanism, and review-guidance format come from the tools' current documentation and interfaces, per-session — never from memory, per the setup skill's discovery protocol (`../setup-quality-loop/references/interface-discovery.md`).
- **Propose, never apply.** Every promotion lands as a reviewed diff (or an explicit approval for anything outside the repo). Nothing activates on your own authority.

## Step 1 — Harvest candidates

From the cycle that just closed:

- Triage verdicts — read from the cycle digest comments on recent PRs — especially accepted findings and *modified* ones (the reviewer saw something real; would a rule have caught it earlier?).
- Recurring patterns across PRs — where the MCP tier offers cross-review search or exposes which guidance produced which comments, use it; within one repo, the PR history itself is searchable.
- Spec invariants declared but not yet formalized (setup seeded rules from declared invariants once; this step keeps that channel open as the spec evolves).

## Step 2 — Judge each candidate

Weigh four criteria (calibration examples in `references/promotion-criteria.md`):

1. **Severity of consequences** if the pattern recurs unchecked.
2. **Formalizability** — is the pattern expressible as a syntactic/structural match, or does recognizing it require judgment?
3. **Recurrence likelihood** — generated code re-generates its patterns; a one-off typo does not recur, a habit does.
4. **False-positive cost** — a rule that cries wolf blocks CI on every PR; noisy rules erode the gate's authority. When a candidate needs many exceptions to be tolerable, it is guidance, not a rule.

Candidates that fail the weighing are dropped — silently if trivial, with a line in the promotion report if the user might reasonably disagree.

## Step 3 — Route what survives

- **Formalizable → Semgrep rule** in `.semgrep/rules/`: authored per the provenance discipline above, fixtures proving both directions, named after the convention it guards. The proposal is the rule file plus its fixtures, as a diff. Before presenting a rule, run its fixtures and run the rule over the current codebase: failing fixtures mean it is not ready to propose; hits on existing code are part of the proposal — fix now, adjust the rule, or consciously defer, the same trichotomy setup uses. An approved rule must never turn the gate red on arrival.
- **Non-formalizable but recurring → review guidance** in the repository's review config, in the current in-repo format the reviewer documentation prescribes: a scoped, plainly-worded instruction the reviewer applies on every future PR. Also a diff — the soft layer stays as versioned and auditable as the hard one.
- **Org-wide scope — only on the user's explicit ask:** where guidance should span repositories and the vendor offers a programmatic channel for it, create it in a pending/suggested state where the interface allows, and only after the user has approved the exact wording. Repo-scoped is the default; org-scope is an escalation, not an optimization.
- **Companion spec amendment, when warranted:** if the pattern exists because the spec or conventions are silent, propose the sentence that would stop the pattern at the source — a rule catches what a convention failed to prevent.

## Step 4 — Dedup before proposing

Check every proposal against what already exists: the rules directory, the review config, and — where visible — guidance already firing in reviews. An overlapping proposal becomes either a strengthening of the existing rule (proposed as its own diff) or nothing. Re-running promotion over the same cycle must surface nothing already approved or already covered — if it does, the dedup failed. A *declined* candidate may resurface on a re-run; that is the no-grudge policy working, not a dedup failure.

## Step 5 — Present

One proposal at a time, each with: the incident or convention it traces to, the weighing in one or two sentences, and the diff. The user approves, adjusts, or declines — all three are normal outcomes, and a decline is recorded nowhere; the ratchet holds no grudge.

**The ratchet turns one way.** Weakening, narrowing, or deleting an existing rule is never bundled into a promotion pass — it is its own explicit, argued escalation (yours via the gate skill's comply-then-appeal path: pass the rule as written, then argue the revision — or the user's own call), decided separately. When the user asks to weaken or remove a rule, handle it as its own standalone proposal under the same discipline — an argued case, a diff, their explicit approval.

## Done means

Every surviving candidate is proposed and resolved; approved diffs are committed through the project's normal flow with the gate green (rule fixtures pass); nothing was activated without approval; a re-run surfaces nothing already approved or covered. Report what was proposed, what the user decided, and what was consciously dropped.
