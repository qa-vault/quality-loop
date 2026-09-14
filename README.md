# quality-loop

**A quality loop for AI-assisted development: question the plan, test the contract, gate the PR, triage the review, and turn what you learn into rules.**

[![Version](https://img.shields.io/badge/version-0.5.0-blue)](.claude-plugin/plugin.json)
[![License](https://img.shields.io/badge/license-Apache--2.0-green)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-plugin-cc785c)](#install)
[![Codex CLI](https://img.shields.io/badge/Codex_CLI-plugin-1f2328)](#install)

Seven skills that make an AI coding agent operate a disciplined quality process around its own code. Part of the [qa-vault](https://github.com/qa-vault/marketplace) plugin family built around [QA Vault](https://qa-vault.com), an MCP-native test-management platform.

## What it does

- **Questions plans and code before they cost you.** A skeptical reviewer surfaces non-obvious decisions and architectural choices in a spec or an implemented feature, fixes purely technical findings itself, and escalates only product-behavior questions to you.
- **Holds tests to the contract.** Unit and integration tests assert what the code promises, never how it works. Unit suites are verified by mutation testing, integration suites by provoked real-infrastructure failures.
- **Gates every PR deterministically.** Before any push or PR the agent runs Semgrep with your project's own rules plus your linters and typecheck. A PR that fails the gate is never opened.
- **Triages AI reviews critically.** Every Greptile comment is steelmanned, verified against code and spec, and answered with an argued verdict. Nothing is applied blindly.
- **Ratchets findings into rules.** Recurring findings become Semgrep rules or review guidance, through your approval only. The ratchet only tightens.

## Quick start

```
/plugin marketplace add qa-vault/marketplace
/plugin install quality-loop@qa-vault
```

Then, in the project you want onboarded, ask your agent:

```
set up the quality loop
```

The setup skill installs the gate, connects your Greptile account, seeds rules from your declared conventions, and writes the `QUALITY-LOOP.md` marker. From there the other skills trigger on their own when your prompt matches. Full steps for both harnesses are under [Install](#install).

## Skills

| Skill | When it runs | What it does |
|---|---|---|
| `exploratory-qa` | On request, in any repository: "QA this plan", "review this module critically", "what's non-obvious here?" | Examines a plan or an implemented feature with a skeptical eye. Resolves technical findings in the working tree (never commits), escalates product-behavior findings to you. |
| `contract-driven-unit-tests` | Whenever unit tests are written, changed, or reviewed, in any repository | Works out what the unit promises, holds assertions to exactly that, and closes with a mutation-testing check. |
| `contract-driven-integration-tests` | Whenever integration tests are written, changed, or reviewed, or a test's level is in question | Keeps everything inside the tested slice real, mocks only at its seams, and provokes the failures the real infrastructure exists to catch. |
| `setup-quality-loop` | Once per project | Installs the gate, connects Greptile, seeds rules from your conventions, writes `QUALITY-LOOP.md`. |
| `run-quality-gate` | Before every commit, push, or PR, and on demand | Runs Semgrep with your rules plus your linters and typecheck. Fixes findings autonomously until green; behavior-changing fixes escalate instead. |
| `triage-code-review` | When a review lands on a PR, or should have | Steelmans each comment, verifies it, decides technical items with argued verdicts, escalates product ones, and posts a one-comment digest. |
| `promote-quality-rules` | After a review cycle closes, or when your spec declares new invariants | Proposes Semgrep rules with test fixtures or review guidance for recurring findings. You approve every promotion. |

## The loop

1. **Specify.** You own the spec. It can declare invariants that become rules immediately.
2. **Question.** `exploratory-qa` pressure-tests the plan before code exists. Plans are the cheapest place to catch issues.
3. **Implement.** The agent writes code with the project's rules available. The tests it writes follow the contract-driven discipline.
4. **Gate.** `run-quality-gate` runs the deterministic layer locally. A red gate never becomes a PR.
5. **Review.** Greptile reviews the PR with whole-codebase context.
6. **Triage.** `triage-code-review` validates every comment. Product decisions go to you; technical ones the agent decides and logs in the PR digest.
7. **Promote.** `promote-quality-rules` turns findings worth keeping into rules, with your approval.

### Principles

- Reviewers advise, the implementing agent judges critically, you own product behavior.
- Escalation follows impact domain (could a user of the product notice?), never reviewer confidence.
- Every decision leaves an auditable trail in PR digest comments.
- No artifact is infallible: code, review, rules, and spec all improve through argued challenge.

## How it fits with the other qa-vault plugins

- [`codelore`](https://github.com/qa-vault/codelore) keeps implementation docs indexed and routes them into the agent's context. When a `docs/INDEX.md` exists, `exploratory-qa` loads the relevant docs as more material to question, including doc/code drift. Neither plugin requires the other.
- [`qa-vault-skills`](https://github.com/qa-vault/qa-vault-skills) covers the QA practice itself: manual test cases and Playwright e2e automation through the QA Vault MCP. quality-loop covers the code-quality side of the same development iteration.

## Prerequisites

- **Semgrep CE** installed locally. You bring it; quality-loop ships no Semgrep code and no rules.
- **A Greptile account** connected to your repository, for the review and triage legs.
- `exploratory-qa` and the two contract-driven testing skills need neither. They work in any repository.

Everything the loop adds lives in your repository and is approved by you.

**Data note.** The review leg sends your PR content to your own Greptile cloud account. Account for that under your NDAs and data policies.

## Install

`quality-loop` is distributed through the `qa-vault` marketplace catalog. Installing it is self-contained: no other `qa-vault` plugin is required.

<details>
<summary><strong>Claude Code</strong></summary>

1. **Add the marketplace** (one-time):

   ```
   /plugin marketplace add qa-vault/marketplace
   ```

   This fetches the catalog of `qa-vault` plugins from GitHub. No code is installed yet. If you already added it for another `qa-vault` plugin, skip this step.

2. **Install the plugin**:

   ```
   /plugin install quality-loop@qa-vault
   ```

   Claude Code asks where to install:
   - **User**: available in every project on your machine (recommended for personal use)
   - **Project**: only active in this project, shared with teammates via `.claude/settings.json`
   - **Local**: only for you, only in this project

3. **Verify**: type `/` and you should see the seven skills listed, each annotated `(quality-loop)`.

**Updates:** Claude Code auto-updates installed plugins at startup.

</details>

<details>
<summary><strong>Codex CLI</strong></summary>

> Requires Codex CLI 0.122 or later. The `url` source variant this catalog uses shipped in stable 0.122 (2026-04-20).

1. **Add the marketplace** (one-time):

   ```
   codex plugin marketplace add qa-vault/marketplace
   ```

   If you already added it for another `qa-vault` plugin, skip this step.

2. **Install the plugin**: inside Codex, open the plugin browser:

   ```
   /plugins
   ```

   Find `quality-loop` under the `qa-vault` marketplace and toggle it on. `/plugins` is an interactive browser and does not accept inline arguments.

3. **Verify**: type `$` in the Codex composer to open the skill-mention popup. The seven skills should be listed. Invoke one explicitly with `$<skill-name> <your request>`, or let Codex auto-detect when your prompt matches a skill's description.

**Updates:** refresh with `codex plugin marketplace upgrade qa-vault` periodically.

</details>

## License

Apache-2.0. See [LICENSE](LICENSE).

---

*Semgrep is a registered trademark of Semgrep, Inc. Greptile is a trademark of Tabnam, Inc. (d/b/a Greptile). quality-loop is an independent project, not affiliated with, sponsored, or endorsed by either company.*
