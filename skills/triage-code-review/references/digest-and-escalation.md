# Digest and Escalation Formats

Two fixed formats. Keep them boring and identical across cycles — the user reads digests diagonally, and consistency is what makes diagonal reading work.

## Cycle digest (one PR comment)

```markdown
## Quality-loop cycle digest

**Review:** <completed | skipped: <reason> | failed> · **Comments triaged:** <N>

| # | Finding | Verdict | Argument |
|---|---------|---------|----------|
| 1 | <one-line finding> | accepted / rejected / modified | <one-line argument> |

**Escalated:** <item → user's decision, one line each; or "none">
**Autonomous decisions outside review comments:** <one line each, e.g. a design choice made while fixing, not prompted by any comment; or "none">
**Process notes:** <degraded mode, gate-rule appeals raised, review anomalies; or "none">
```

One row per inline comment, in the order they appear on the PR. The argument column is the compressed form of the in-thread reply, not a replacement for it — rejections and modifications always get the full argument in-thread. Reactions on the PR must mirror this table one-to-one: a 👍 for every `accepted` and `modified` row, a 👎 for every `rejected` row. An escalated row records the user's decision as its verdict.

In degraded mode (no review), post the digest with `Comments triaged: 0` and omit the table.

## Escalation (product domain)

Escalate in the live conversation when the user is present in the session; mirror the outcome to the PR thread. When the user is not present, post to the PR thread and flag it in your summary:

```markdown
**Needs your decision — product behavior**

<2–3 plain-language sentences: what a user of the product would notice.>

Options:
1. <option> — <trade-off>
2. <option> — <trade-off>
3. <option, if a real third exists> — <trade-off>

Recommendation: option <N> — <reasoning>.
```

Plain product language in the description — no internal shorthand, no rule IDs, no reviewer jargon. Two options are fine when only two exist; never pad with a fake third. The recommendation is mandatory: escalation hands over the decision, not the thinking.

The user's decision goes into the digest's **Escalated** line verbatim, so the audit trail closes. If a decision is still pending when the digest posts, record `→ pending` and update the digest comment when the decision lands — the audit trail closes then, and your summary says so.
