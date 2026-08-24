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

`quality-loop` is distributed through the `qa-vault` marketplace catalog. Installing it is **self-contained** — you do not need any other `qa-vault` plugin (such as `qa-vault-skills` or `codelore`) installed first. Pick the section for your tool.

### Claude Code

1. **Add the marketplace** (one-time):

   ```
   /plugin marketplace add qa-vault/marketplace
   ```

   This fetches the catalog of `qa-vault` plugins from GitHub — no code is installed yet. (If you already added it for another `qa-vault` plugin, skip this step.)

2. **Install the plugin**:

   ```
   /plugin install quality-loop@qa-vault
   ```

   Claude Code will ask where to install:
   - **User** — available in every project on your machine (recommended for personal use)
   - **Project** — only active in this project, shared with teammates via `.claude/settings.json`
   - **Local** — only for you, only in this project

3. **Verify** — type `/` and you should see `setup-quality-loop`, `run-quality-gate`, `triage-code-review`, and `promote-quality-rules` (each annotated `(quality-loop)`).

**Updates:** Claude Code auto-updates installed plugins at startup.

### Codex CLI

Codex has its own plugin marketplace system; the flow mirrors Claude Code's and is fully independent of any other plugin.

> **Requires Codex CLI 0.122+.** The `url` source variant this catalog uses shipped in stable 0.122 (2026-04-20); earlier 0.121.x releases accept only `local` plugin sources. Upgrade to 0.122 or later.

1. **Add the marketplace** (one-time):

   ```
   codex plugin marketplace add qa-vault/marketplace
   ```

   (If you already added it for another `qa-vault` plugin, skip this step.)

2. **Install the plugin** — inside Codex, open the plugin browser:

   ```
   /plugins
   ```

   Find `quality-loop` under the `qa-vault` marketplace and toggle it on to install. (`/plugins` is an interactive browser — it does not accept inline arguments.)

3. **Verify** — type `$` in the Codex composer to open the skill-mention popup; `setup-quality-loop`, `run-quality-gate`, `triage-code-review`, and `promote-quality-rules` should be listed. Invoke one explicitly with `$<skill-name> <your request>`, or let Codex auto-detect when your prompt matches a skill's description.

**Updates:** refresh with `codex plugin marketplace upgrade qa-vault` periodically.

### First run

In the project you want onboarded, ask your agent to *"set up the quality loop"* — the setup skill drives everything from there.

## License

Apache-2.0 — see [LICENSE](LICENSE).

---

*Semgrep is a registered trademark of Semgrep, Inc. Greptile is a trademark of Tabnam, Inc. (d/b/a Greptile). quality-loop is an independent project, not affiliated with, sponsored, or endorsed by either company.*
