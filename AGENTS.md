# Agent Context: quality-loop

This repository is a **dual-ecosystem plugin** — installable natively in both Claude Code and Codex CLI via the `qa-vault/marketplace` catalog. When working on this codebase, keep both ecosystems in sync.

## Repository layout

```
quality-loop/
├── .claude-plugin/plugin.json          # Claude Code manifest
├── .codex-plugin/plugin.json           # Codex manifest ("skills": "./skills/")
├── skills/
│   ├── setup-quality-loop/
│   │   ├── SKILL.md
│   │   └── references/interface-discovery.md
│   ├── run-quality-gate/SKILL.md
│   ├── triage-code-review/
│   │   ├── SKILL.md
│   │   └── references/digest-and-escalation.md
│   └── promote-quality-rules/
│       ├── SKILL.md
│       └── references/promotion-criteria.md
├── README.md
├── LICENSE                             # Apache-2.0
└── AGENTS.md                           # this file
```

The `skills/` directory is the **single source of truth** for skill content. Both manifests reference it — never duplicate skill files. There is no `agents/` directory: every skill runs inline on any harness (the triage skill deliberately runs in the implementing session, not a subagent).

## What this plugin is

quality-loop makes an AI agent operate a two-layer quality process around its own code: a deterministic local gate (Semgrep with project-owned rules plus the project's formal checks), critical triage of Greptile AI reviews, and a human-approved ratchet that turns findings into rules. The plugin ships **markdown instructions only** — no binaries, no vendor code, no rules.

## Standing design constraints

These bind every current and future skill in this repo. They come from the design and legal review (see the design doc in the author's project records) — do not relax them without that context:

1. **No rule YAML in this repo, ever.** Not as examples, not in references, not in tests. Rules exist only in user repositories, authored fresh against each project's own conventions. (Legal provenance constraint — AI-regenerated registry rules in a distributed repo are the one real infringement vector.)
2. **Provenance rule, verbatim in skills that author rules:** never fetch registry rule YAML as authoring material; every rule traces to an invariant the user's project declared about itself.
3. **Staleness resistance:** no version-specific tool details in skill text — no CLI flags, MCP tool names, endpoints, or config schemas stated as facts. Skills instruct runtime discovery (help output, MCP listings, current official docs).
4. **Principles tone:** invariants and criteria for an intelligent executor, not step scripts where a script isn't needed.
5. **Each skill embeds its own discipline inline** — there is no shared conventions skill.
6. **User sovereignty:** every repo write is user-approved; promotions are proposals; the ratchet only tightens; escalation routes by impact domain, never reviewer confidence.
7. **Wording:** nominative vendor references only ("works with Semgrep", "your Greptile account"); no vendor logos; no guarantee-shaped claims; keep the README's affiliation-disclaimer footer.
8. **Document only what exists** — no roadmap or future-skill announcements anywhere in the repo.

## Release checklist

Every release, in this order:

1. Bump `version` in **both** `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json` (keep them identical).
2. If the plugin's description or skill set changed, update `README.md` accordingly.
3. PR the `qa-vault/marketplace` repo updating **both** catalog manifests: `.claude-plugin/marketplace.json` and `.agents/plugins/marketplace.json`.
4. Verify a clean install from the live marketplace discovers all skills on both harnesses.
