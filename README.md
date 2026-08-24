# quality-loop

A two-layer quality loop for AI-assisted development: a deterministic pre-PR gate built on your local Semgrep CE install and your project's own rules, plus critical triage of Greptile AI code reviews — closed by a human-approved ratchet that turns recurring findings into rules.

Skills for AI coding agents, installable in Claude Code and Codex CLI.

## The loop

1. **Specify** — you own the spec; it can declare invariants that become rules immediately.
2. **Implement** — the agent writes code with the project's rules available to it.
3. **Gate** — before any PR, the agent runs the deterministic layer locally: Semgrep with your project's rules plus your linters and typecheck. A PR that fails the gate is never opened.
4. **Review** — Greptile reviews the PR with whole-codebase context.
5. **Triage** — the agent critically validates every review comment: steelman first, verify against code and spec, then an argued verdict. Product-behavior decisions escalate to you; technical ones the agent decides and logs in a one-comment PR digest.
6. **Promote** — findings worth keeping become rules through your approval: formalizable patterns as Semgrep rules with test fixtures, recurring semantic ones as review guidance in your repo's review config. The ratchet only tightens.

## Principles

- Reviewers advise — the implementing agent judges critically — you own product behavior.
- Escalation follows impact domain (could a user of the product notice?), never reviewer confidence.
- Every decision leaves an auditable trail in PR digest comments.
- No artifact is infallible: code, review, rules, and spec all improve through argued challenge.

## Skills

| Skill | When it runs |
|---|---|
| `setup-quality-loop` | Once per project: installs the gate, connects your Greptile account, seeds rules from your declared conventions, writes the `QUALITY-LOOP.md` marker |
| `run-quality-gate` | Before every commit, push, or PR — and on demand |
| `triage-code-review` | When a review lands on a PR (or should have) |
| `promote-quality-rules` | After a review cycle closes, or when your spec declares new invariants |

## What you bring

You install Semgrep CE yourself; you connect your own Greptile account. quality-loop ships no Semgrep or Greptile code and no rules — everything the loop adds lives in your repository, approved by you.

## Data note

The review leg sends your PR content to your own Greptile cloud account — account for that under your NDAs and data policies.

## Install

Claude Code:

```
/plugin marketplace add qa-vault/marketplace
/plugin install quality-loop@qa-vault
```

Codex CLI: install from the same `qa-vault/marketplace` catalog with your Codex version's plugin mechanism.

Then, in the project you want onboarded, ask your agent to *"set up the quality loop"*.

## License

Apache-2.0 — see [LICENSE](LICENSE).

---

*Semgrep is a registered trademark of Semgrep, Inc. Greptile is a trademark of Tabnam, Inc. (d/b/a Greptile). quality-loop is an independent project, not affiliated with, sponsored, or endorsed by either company.*
