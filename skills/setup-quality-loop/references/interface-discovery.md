# Interface Discovery Protocol

How to establish what the tools offer *today*, in this environment. Run discovery per session — never cache conclusions in project files, and never trust remembered tool names, flags, endpoints, or config schemas: both tools have a history of moving and renaming their interfaces.

## Semgrep

1. **Presence:** invoke the CLI with its help flag. Absent → install by the method the current official documentation prefers, then re-verify.
2. **Capabilities:** from the CLI's own help output, establish the current shape of: scanning with a local rules directory, machine-readable output, the findings-as-failure exit mode, and the rule fixture-test mechanism. The help output is the authority; official docs fill gaps.
3. **Rule schema:** when authoring rules, consult the current official rule-syntax documentation (or a machine-readable schema if the project of record publishes one). Accept that severity vocabularies and field sets evolve — author to what the installed version accepts, and prove it by running the fixtures.
4. **Boundary:** local scanning with local rules is the loop's home turf — it requires no login and sends nothing anywhere. Anything that fetches remote rulesets is outside this skill's writ (provenance rule) and changes the privacy posture; do not introduce it during setup.

## Greptile

Discover top-down, use the richest tier available, degrade without ceremony:

1. **MCP tier:** if a Greptile MCP server is available in this harness, list its tools and read their descriptions — that listing, not memory, defines what review-lifecycle data, comment metadata, and guidance-management operations exist right now. Identify the **carrier**: an official vendor plugin (which ships the server itself) or a manual registration. One server, one carrier — the same endpoint configured both ways is a conflict to surface and fix (keep the plugin's, remove the manual one), not extra capability.
2. **Credential contract:** how the key is supplied belongs to the carrier. An installed plugin declares its expectation in its manifest/config in the harness's plugin cache (typically an environment variable it interpolates); a manual registration follows the vendor's current docs. On harnesses without an official vendor plugin, manual registration is the legitimate path — discover how that harness's MCP configuration references secrets (environment interpolation, a secret store) from the harness's own documentation, never from another harness's habits. Prove the contract with one live authenticated call before declaring the tier up.
3. **CLI tier:** check for a Greptile CLI on PATH; if present, its help output defines its current verbs.
4. **Host-API tier (always present):** Greptile ultimately acts as a reviewer identity on the repository host. Everything essential — reading review comments, replying in threads, adding reactions, mention-triggering a review — works through the host's ordinary PR interfaces even with no Greptile-specific channel at all.
5. **Reviewer identity:** identify the review bot by pattern-matching comment authors on a recent reviewed PR (a bot-suffixed account whose name contains the vendor's), never by a hardcoded login — app slugs differ between installations and over time. On a fresh setup with no reviewed PR yet, defer identification to the first triage.
6. **Absence semantics:** a missing tier is degraded fidelity, not an error. Note what the session lost (e.g., without MCP there is no programmatic review-lifecycle visibility — completion and skips must be inferred from host-side signals) and proceed on the tiers that exist.

## Reporting

End discovery by telling the user, in one or two sentences, which tiers this session runs on and what that means for the loop's fidelity. If a tier the project previously had is now unreachable (revoked key, server down), say so — silent degradation hides problems the user can fix in a minute.
