---
name: triage-code-review
description: >-
  Use when an AI code review has landed on a pull request in a quality-loop project
  (QUALITY-LOOP.md present) — "process the review", "triage the greptile comments", "address the
  review comments", "the review is in" — or when a PR was opened and the review should have
  arrived. Detects skipped or failed reviews, steelmans each comment, verifies it against code and
  spec, escalates product-behavior items to the user, decides technical ones autonomously with
  argued verdicts, applies accepted fixes, replies in-thread, sets thumbs reactions, and posts a
  one-comment cycle digest. Never blindly applies review comments — supersedes any vendor auto-fix
  loop inside the quality-loop process.
---

# Triage the Code Review

## Overview

The AI reviewer advises; you judge; the user owns product behavior. This skill turns a Greptile review into argued verdicts: every comment is critically validated against the code, the spec, and the facts — then accepted, rejected with an argument, or modified. Product-domain items escalate to the user; technical items you decide yourself; everything leaves a trail in one digest comment.

**What this skill is not:** an auto-fix loop. Vendor tooling that applies reviewer suggestions until the reviewer is happy inverts this process's philosophy — inside a quality-loop project, review comments pass through critical validation here, always, even at the reviewer's highest confidence.

## Operating discipline

- **Discover, don't assume.** Establish the interfaces available this session (MCP tools listing → CLI → host API) per the setup skill's discovery protocol (`../setup-quality-loop/references/interface-discovery.md`); use the richest, degrade gracefully. Identify the reviewer bot by author pattern (a bot-type account whose login contains the vendor name), never a hardcoded login.
- **You are judging your own code — respect the hazard.** The same session that wrote the code judges its review. The compensator is procedural, and it is not optional: **before judging any comment, argue it as if it came from a stronger reviewer than you; the verdict comes only after checking the claim against the code, the spec, and observable facts.** "That's not how I meant it" is not a verdict.
- **Nothing is infallible.** A comment can be wrong; so can the code; so can the spec or a gate rule the comment collides with. When a comment contradicts the spec, run the three-way analysis — code error, review error, or spec error — and treat a suspected spec error as a product-domain finding to escalate as an improvement proposal, not a point automatically won by either side.

## Stage 1 — Await and detect

A review is expected once the PR opens. It can also legitimately never arrive — drafts, size limits, and account spend caps all skip reviews, sometimes silently.

Prefer the richest signal available for review state: programmatic review-lifecycle queries (completion, failure, *skipped*) where the MCP tier offers them; otherwise host-side signals — the reviewer's status check if enabled, its status reactions, the arrival of review comments. Reviews land on the order of minutes, not seconds — poll rather than block, and prefer an explicit skipped/failed signal over timeout inference. Before declaring absence, check the checkable causes (draft status, size limits). Then branch:

- **Arrived** → Stage 2.
- **Draft PR** → the review is deferred, not absent; resume this skill when the PR is marked ready — no degraded-mode digest.
- **Failed** → one retrigger via the mention mechanism the vendor's current docs describe, then a second waiting window; still nothing → treat as absent.
- **Skipped, or absent after the window** → **degraded mode**: proceed on the deterministic gate alone, record the fact and the reason in the digest (a timeout maps to `skipped: timeout`), and tell the user in your summary — a silently missing review layer is a process event, not background noise.

## Stage 2 — Read

Collect the review summary and every inline comment. Where the MCP tier is available, take the structured enrichments it offers (severity/type fields, suggested changes as data, whether a comment is already addressed, which guidance rule produced it — that last one feeds the promotion skill). Where it is not, parse the reviewer's comments via the host API — they are ordinary PR comments from the bot author.

## Stage 3 — Validate, steelman first

For each comment, in this order:

1. **Steelman.** State the strongest version of the comment's claim, as if written by a reviewer stronger than you, with the failure it implies spelled out concretely.
2. **Verify.** Check that claim against the actual code, the spec, and observable facts (run the code or tests where that settles it). Reviewer confidence and severity calibrate **how thoroughly you check — they never decide who decides.**
3. **Verdict:** accept / reject / modify (the comment points at something real but the proposed fix is wrong — describe the better fix). Every reject and modify carries an argument grounded in code, spec, or facts; "seems unimportant" is not an argument. For product-domain comments (Stage 4's classification), the verdict is not yours to issue — carry your conclusion into the escalation's Recommendation line; the user's answer becomes the recorded verdict. Steelman and verification apply to every comment regardless of domain; only the verdict forks.

## Stage 4 — Classify by impact domain

Escalation follows one axis only: **could a user of the product notice this change?** Not reviewer confidence, not diff size, not whether the current spec mentions it.

- **Product domain → escalate** using the format in `references/digest-and-escalation.md`: observable behavior changes, acceptance criteria, API contracts, data schemas, business rules — and suspected errors in the spec or in an approved gate rule. When you cannot confidently classify, escalate; doubt resolves toward the user.
- **Technical domain → decide yourself:** changes to *how* without changing *what* — refactoring, naming, structure, tests, performance with identical semantics.

## Stage 5 — Act

- Apply accepted technical fixes; treat modified verdicts as your better fix for the real problem the comment found.
- **Re-run the quality gate on the result** (fixes are code too) — use the run-quality-gate skill; the never-open-a-failing-PR invariant extends to never *updating* a PR into a failing state.
- Reply in-thread on every rejection and modification with the argument — the reviewer learns from replies, and the thread is where a human reads the reasoning later.
- **Set 👍 on accepted and modified comments — a modified verdict means the claim was right, and the in-thread reply is what corrects the fix — and 👎 on rejected ones. Each reaction mirrors a digest entry: reactions are the reviewer's training signal, deliberate per-comment verdicts, never blanket.**
- For escalated comments, the user's decision is the verdict: reply in-thread noting the outcome, set the reaction matching it (👍 if the comment's concern was upheld, 👎 if dismissed), and record it in both the verdict table and the Escalated line.

## Stage 6 — Digest

Post one PR comment using the digest template in `references/digest-and-escalation.md`: a verdict line per comment, escalations with their outcomes, autonomous decisions taken outside review comments, and process notes (degraded mode, gate-rule appeals carried from the gate skill). The digest is the user's asynchronous audit surface — they read it diagonally and can challenge any verdict after the fact; its rejection share over time is the meta-signal for whether reviewer or judge is systematically wrong.

## Done means

Every comment has a verdict with an argument; product-domain items are with the user (or resolved by them); accepted fixes are applied and the gate is green again; threads and reactions mirror the verdicts; the digest is posted. Your summary to the user leads with what was escalated and whether the review arrived at all — and notes whether any verdicts look promotion-worthy, since they are the promote-quality-rules skill's raw material.
