---
name: setup-quality-loop
description: >-
  Use when installing or onboarding the quality loop into a project — "set up the quality loop",
  "install the semgrep gate and greptile review process", "onboard this repo onto quality-loop" —
  or when QUALITY-LOOP.md exists in the repo but the local toolchain (Semgrep, Greptile access) is
  not wired up. Verifies or installs Semgrep CE, connects the user's own Greptile account (repo
  app, in-repo review config, MCP server; on Claude Code also Greptile's official plugin), creates
  the project rules directory seeded only from the project's own declared invariants, writes the
  QUALITY-LOOP.md marker, and dry-runs the gate. Conversational and review-gated — every file
  written into the repo is approved by the user first.
---

# Set Up the Quality Loop

## Overview

Onboard a project onto the **quality loop**: a two-layer quality process where a deterministic gate (Semgrep with the project's own rules, plus the project's existing formal checks) runs locally before every PR, an AI reviewer (Greptile) reviews every PR with whole-codebase context, and findings crystallize into rules over time — always through the user's approval.

Full setup happens once per repository; each collaborator's machine may still need local wiring. When setup finishes, the loop is runnable end to end: the gate passes on the current codebase, the reviewer is connected, and the repo carries a marker file that declares the process.

**On an already-onboarded repo** (marker present, toolchain not wired locally), every step below becomes verify-don't-recreate: confirm rules, review config, and marker are in place rather than re-proposing them; do fresh only the local wiring — Semgrep install, MCP registration, key storage. This path expects zero repo writes.

## Operating discipline

These rules govern every step below:

- **Review-gated writes.** Every file this skill creates or modifies in the user's repository — rules, fixtures, review config, the marker — is shown to the user and approved before it lands. Setup is a conversation, not a script.
- **Discover, don't assume.** Tool interfaces change. Establish what Semgrep and Greptile offer *today* from their current documentation and the interfaces present in this environment (`--help`, MCP tool listings, official docs) — never from memory of version-specific commands, flags, tool names, or config schemas. See `references/interface-discovery.md` for the discovery protocol.
- **Clean rule provenance.** Never seed from generic security patterns, registry rulesets, or anything not derived from this project's own declared conventions. Never fetch registry rule YAML as authoring material. Every rule you propose must trace to an invariant this project declared about itself.
- **Nothing version-specific lands in the repo.** Interface details (endpoints, tool names, CLI flags) are discovered per-session, never recorded in project files.

## Step 1 — Deterministic layer

1. **Verify or install Semgrep CE.** Check whether Semgrep is available locally. If not, install it by the method its current official documentation prefers, and confirm the install with a trivial invocation. Community Edition running locally is the target — no account or platform login is required for the loop.
2. **Discover the project's existing formal checks.** Read the project's own scripts and CI configuration to learn what already gates quality here: linters, typecheck, formatters, test commands. The quality gate wraps *all* formal checks, with Semgrep as the layer this process adds — it does not replace what the project already runs.
3. **Create the rules home.** Create `.semgrep/rules/` as the project's rules directory. Rules and their test fixtures live here, versioned with the code.
4. **Seed rules from declared invariants only.** Ask the user (and read the project's spec, conventions, and agent-context files) for invariants the project has *declared*: "all HTTP goes through our wrapper", "colors come from tokens, never hardcoded", "no secrets in client-exposed variables". For each invariant that is expressible as a syntactic or structural match, propose a rule:
   - authored from scratch against that invariant, in the rule schema the current Semgrep documentation declares;
   - accompanied by test fixtures exercising both the violating and the clean pattern, passing the current fixture-test mechanism;
   - presented to the user individually — the user picks which rules land. No declared invariants means the directory starts empty; that is a valid outcome. The ratchet will grow it.

## Step 2 — AI-review layer

Guide the user through connecting Greptile — their own account, never a shared or bundled one:

1. **App installation.** The Greptile app must be installed on the repository's host (GitHub/GitLab) and granted access to this repo, with indexing completed. This is a dashboard action only the user can perform; walk them through it per current Greptile documentation and wait for confirmation.
2. **In-repo review config.** Write the baseline review configuration in the repository, in whatever in-repo mechanism the current Greptile documentation prescribes. The expected baseline is a valid, near-empty config that establishes the file as the anchor where promoted guidance will accumulate (via the promote-quality-rules skill) — any initial guidance must trace to a declared invariant and is user-approved like every other write.
3. **Status signal.** If the current Greptile offering supports a per-review status check on the host, enable it — it doubles as the loop's review-completion and skip-detection signal.

## Step 3 — Wire the richest interfaces

Wire up the richest interface set available, so later skills can use the best channel and degrade gracefully. One rule governs the whole step: **each interface gets exactly one carrier** — the same server registered twice (by a vendor plugin and again by hand) is a configuration error, not extra richness.

1. **Plugin-first.** Where the vendor ships an official plugin for this harness, guide the user to install it from the harness's plugin marketplace — the plugin is the carrier of the vendor's MCP server, so no manual registration of the same server happens alongside it. Only where no official plugin exists for the harness, register the official MCP server manually via the harness's MCP registration mechanism, with the endpoint and auth the current vendor documentation declares — at the user/session level, never in project-tracked files.
2. **Read the credential contract from the carrier.** How the API key must be supplied is defined by whichever carrier is in play: an installed plugin declares its expectation in its own manifest/config (typically an environment variable it interpolates); a manual registration follows the vendor's current docs. Never assume a header, flag, or variable name from memory — a key supplied the wrong way produces a server that half-works or silently fails to start.
3. **Provision the reviewer API key — a gate, not a courtesy.** Without the credential the interface tier is dead, so this step blocks setup completion. Offer two scopes, defaulting to project-local:
   - **Project-local (default):** the key lives in the project's untracked local configuration — the harness's per-project environment mechanism. Different projects can use different reviewer accounts, nothing leaks through the repo, and the key sits next to the repo its app installation belongs to.
   - **User-global:** the harness's user-level configuration or shell profile, when the user prefers one account everywhere.
   Prepare the exact snippet for the chosen scope, and expect the harness to require the user to apply it by hand — agent writes to harness settings are often denied, and that is the normal flow, not an error.
4. **Verify reachability — including duplicates.** Confirm the tier actually works: list the tools and make one live authenticated call — an invalid key fails with an auth error; a valid one returns a well-formed response, even an empty one. Check for double registration: the same server configured both by a plugin and manually is a conflict — surface it and offer to remove the manual one. If the tier cannot be established, the loop still works over the host API alone — tell the user which fidelity they are running at, and continue.

## Step 4 — The marker

Write `QUALITY-LOOP.md` at the repository root, exactly this content — the user may amend it at approval (repository link included so a collaborator without the plugin can find the process):

```markdown
# Quality Loop

This repository runs the **quality-loop** process: a two-layer quality gate for AI-assisted development.

- **Deterministic layer** — Semgrep with this project's own rules (plus the project's linters and typecheck) runs locally before every PR. A PR that fails the gate is never opened.
- **AI review layer** — Greptile reviews every PR with whole-codebase context; the implementing agent critically triages every comment and answers with argued verdicts.
- **Ratchet** — findings worth keeping become rules: formalizable patterns as Semgrep rules with test fixtures, recurring semantic ones as review guidance in the repo's review config. Every promotion is a human-approved diff; the ratchet only tightens.

## Principles

- Reviewers advise — the implementing agent judges critically — the user owns product behavior.
- Escalation follows impact domain (does it change observable product behavior?), never reviewer confidence.
- Every decision leaves an auditable trail in PR digest comments.
- No artifact is infallible: code, review, rules, and spec all improve through argued challenge.

## Tooling

Contributors using an AI agent get the full process as skills from the **quality-loop** plugin: https://github.com/qa-vault/quality-loop (marketplace: `qa-vault`). Without it, honor the invariants above manually: run the gate before opening a PR and treat review comments critically.
```

This is the plugin's only own artifact in the repository. Everything else setup writes belongs to the tools (rules, review config) and lives where those tools prescribe.

## Step 5 — Verification

1. **Dry-run the gate.** Run the full deterministic stack — Semgrep against `.semgrep/rules/` plus the discovered project checks — on the current codebase. If seeded rules fire on existing code, surface each hit to the user: fix the code now, adjust the rule, or consciously defer; the gate must end green before setup is declared done. Run the rule fixtures too — every seeded rule must pass its own tests.
2. **Offer an end-to-end test.** Offer to validate the review leg with a real, small PR through the full loop (gate → PR → review → triage). This may incur paid review usage — the user decides. Setup is complete without it; the first real PR will exercise the same path.

## Done means

Semgrep runs locally; `.semgrep/rules/` exists with only user-approved rules, each with passing fixtures; the gate is green on the current codebase; Greptile is installed, indexed, and configured in-repo; the richest available interfaces are wired with the reviewer credential verified by a live call; `QUALITY-LOOP.md` is at the root. Report the fidelity level (full MCP vs host-API-only) to the user as the last word.
