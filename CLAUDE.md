# CLAUDE.md — AQEM AI Skills

## What this is

Agent skills that drive the **AQEM AI MCP connector** (`https://aqemai.com/api/mcp`) to generate
images (and soon video). The connector — not the skills — does auth, generation, credit spend, and
saving to the user's Assets. The skills are the **orchestration + quality layer**: model selection,
prompt building, UX, and credit-awareness.

```
aqem-generate    →  general image gen + editing across all AQEM models
aqem-photoshoot  →  brand/product/marketing visuals via 10 modes
aqem-video       →  (future) image-to-video + text-to-video
```

## Architecture

- **One backend, two tools.** Everything routes through the connector's MCP tools:
  `generate_image(prompt, model?, aspect_ratio?, quality?, resolution?)` and `get_credit_balance()`.
  Never invent other tools or call `aqemai.com` with raw HTTP from a skill.
- **Source of truth for models/costs is the tool's own description.** The catalog in README/skills is a
  convenience mapping (intent → model), not the live database. If costs change server-side, the tool
  description wins.
- **Results auto-save** to the user's AQEM Assets (session-grouped by a time window). Skills must not
  re-upload or duplicate.

## The quality lever

AQEM has **no server-side prompt enhancer yet**, so the skills currently **own the per-mode prompt
templates** (see `aqem-photoshoot/SKILL.md` → "Prompt building"). This is the single biggest driver of
output quality. The planned upgrade is to move these templates into the AQEM backend (a `mode` param on
`generate_image`, or a dedicated `photoshoot` endpoint) so skills pass raw intent and the server
assembles the studio-grade prompt — matching how Higgsfield's product-photoshoot works. When that ships,
flip the skills from "write the prompt" to "pass intent + mode."

## SKILL.md conventions (the 300-line rule)

Each `SKILL.md` loads into the agent's context when it triggers — every line costs tokens and latency.
Aim for **under 300 lines**.

**The test:** if removing a section would NOT break the agent's ability to decide *what to do next*, move
it to `references/`. If it WOULD, it stays.

- **Stays:** frontmatter (`name`, `description` with rich triggers + `NOT for:` boundaries,
  `argument-hint`, `allowed-tools`), stage flow, decision trees (model + mode), per-turn UX rules,
  short pointers.
- **Moves to `references/`:** long flag tables, prompt galleries, error trees, anything needed only once
  after the path is chosen.

## Triggering discipline

The `description` frontmatter is what makes a skill auto-fire. Keep it stuffed with **literal phrases a
user would actually type**, plus explicit **`NOT for:`** boundaries so skills don't misfire into each
other (`aqem-generate` = plain gen; `aqem-photoshoot` = product/brand). Route, don't overlap.

## Credit-awareness rules (apply in every skill)

- Default to the cheapest model that satisfies the brief (`flux-fast` for general).
- For `gpt-image-2`, **ask the quality tier** before spending (low ~2 / medium ~8–9 / high ~26–33 cr).
- For `nano-banana`, omit `resolution` (1K) unless the user asks for high-res (4K ≈ double cost).
- Warn before any run that exceeds the user's balance; for multi-output runs, state the total up front.
- On `Not enough AQEM credits`, relay needed/available + the dashboard link, and offer a cheaper option.

## Self-contained skills

Each skill folder is independent — no `../` references. Any `references/*.md` must be reachable from that
skill's own `SKILL.md`, so a skill can be installed standalone.

## Versioning

Repo-wide version lives in: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, and each
`SKILL.md` frontmatter `version:`. Keep them in sync (currently `0.1.0`).

## Adding a skill

New top-level `aqem-<name>/SKILL.md` folder. Follow the frontmatter convention (rich `Use when:`
triggers, `Chain` rules, `NOT for:` boundaries). Register it in `.claude-plugin/marketplace.json`
(`plugins[0].skills[]`) and in `README.md`.

## Related

- AQEM MCP connector: `https://aqemai.com/api/mcp` (see the app's `app/api/mcp/route.ts`).
- Design inspiration: [Higgsfield AI Skills](https://github.com/higgsfield-ai/skills).
- [Agent Skills spec](https://agentskills.io/specification.md).
