<<<<<<< HEAD
# AQEM AI Skills

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
[![Version](https://img.shields.io/badge/version-0.1.0-green.svg)](#)
[![Skills](https://img.shields.io/badge/skills-2-blueviolet.svg)](#skills)

AI agent skills for generating perfect images (and soon video) with [AQEM AI](https://aqemai.com).
They drive the **AQEM AI MCP connector** — picking the right model for the brief, building
strong prompts, spending credits responsibly, and auto-saving results to your AQEM Assets.
Works in Claude (Claude Code / Claude apps with the connector enabled).

## Prerequisite — connect AQEM AI

These skills call the AQEM MCP connector. Add it once:

1. Open **Settings → Connectors → Add custom connector**
2. URL: `https://aqemai.com/api/mcp`
3. Sign in with your AQEM account

The connector exposes two tools the skills use: `generate_image` and `get_credit_balance`.

## Install

### Claude Code marketplace

```
/plugin marketplace add aqem-ai/skills
/plugin install aqem@aqem
```

### Manual (clone)

```bash
git clone --depth 1 https://github.com/aqem-ai/skills.git
```

Then point your agent at the cloned folder, or copy the skill directories into your
agent's skills path.

## Skills

| Skill | Invoke | Description |
|---|---|---|
| [`aqem-generate`](./aqem-generate) | `/aqem:generate` | General image generation across AQEM's models (Flux Fast, Flux Kontext, Nano Banana 2/Pro, GPT Image 2), image editing/restyle, and credit-aware defaults. Video coming soon. |
| [`aqem-photoshoot`](./aqem-photoshoot) | `/aqem:photoshoot` | Brand-quality product & marketing visuals with 10 modes (studio, lifestyle, Pinterest pin, hero banner, ad pack, virtual try-on, conceptual, restyle, …) and per-mode prompt enhancement. |

## Models (via the connector)

| Model | Cost | Best for |
|---|---|---|
| `prunaai/flux-fast` | 1 cr | Fast drafts, the default for general work |
| `prunaai/flux-kontext-fast` | 2 cr | Richer detail / style, exploratory passes |
| `prunaai/p-image-edit` | 2 cr | Editing / inpainting an existing image |
| `google/nano-banana-2` | 12 / 17 / 26 cr (1K/2K/4K) | Photorealism |
| `google/nano-banana-pro` | 25 / 25 / 50 cr | Premium, highest-fidelity photorealism |
| `openai/gpt-image-2` | 2 / 8–9 / 26–33 cr (low/med/high) | On-image text, strict instruction-following |

The skills pick the right model automatically and ask before any expensive (12+ credit) run.

## Roadmap

- **Video** (`aqem-video`) — image-to-video and text-to-video when AQEM's video models ship.
- **Server-side prompt enhancer** — move per-mode prompt templates into the AQEM backend so the
  skills pass raw intent and the server assembles studio-grade prompts (the Higgsfield approach).
- **Marketplace cards** (`aqem-marketplace-cards`) — listing main/secondary/A+ images.

## License

MIT — see [LICENSE](./LICENSE).
=======
# aqem-skills
>>>>>>> 6c9b30bbad387ddcc0af199124cfadc9d668054d
