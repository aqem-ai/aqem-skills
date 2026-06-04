---
name: aqem-generate
description: |
  Generate images (and soon video) with AQEM AI. Spends the signed-in AQEM
  user's credits and renders an interactive image card.
  Use when: "generate an image", "make a picture", "create art", "draw me",
  "make a thumbnail", "design a poster/banner/flyer", "logo concept",
  "edit this image", "inpaint", "change the background", "image to image",
  "restyle this photo", "make it photorealistic", "text on image",
  "product shot", "make a wallpaper", "ai art", and (coming soon)
  "animate this photo", "image to video", "make a short video".
  Picks the right AQEM model for the brief, asks at most a couple of short
  questions, is credit-aware, and saves results to the user's AQEM Assets.
  Chain with aqem-photoshoot for branded product/marketing visuals.
  NOT for: account/billing/credit-topup actions, non-generation chat,
  marketplace listing copywriting, or training a face identity.
allowed-tools: mcp__aqem__generate_image, mcp__aqem__get_credit_balance
---

# AQEM Generate

Generate images with AQEM AI through the AQEM MCP connector. This skill decides the
best model + framing for the brief, enriches the prompt, spends credits responsibly,
and returns a clean result. The connector's `generate_image` tool does the real work
(auth, generation, credit spend, saving to Assets); this skill is the orchestration
and quality layer on top.

> Tool names: this skill assumes the **AQEM AI** MCP connector is connected. Its tools
> appear as `generate_image` and `get_credit_balance` (host-namespaced, e.g.
> `mcp__aqem__generate_image`). If they are not available, see **Step 0**.

## Step 0 — Bootstrap

Before generating:

1. **Connector check.** If the AQEM `generate_image` tool is not available, tell the user:
   *"Add the AQEM AI connector first: Settings → Connectors → Add `https://aqemai.com/api/mcp`,
   then sign in."* Stop until it's connected.
2. **Credit check (only when it matters).** For premium models (nano-banana, gpt-image-2 high),
   call `get_credit_balance` first so you don't fail mid-generation. For the 1–2 credit defaults,
   skip the check and just generate.

## UX Rules

1. **Be concise.** No JSON, no raw IDs, no model internals in chat. Deliver the image + one line.
2. **Don't over-ask.** Pick a sensible default model and generate. Ask at most 1–2 short questions,
   and only when genuinely missing (e.g. gpt-image-2 quality tier — see below). Use labeled options.
3. **Quality default first.** Don't push cheaper models or pre-estimate cost unless the user asks to
   save credits. Start with the right tool for the job.
4. **Be credit-aware, not stingy.** State the credit cost only when it's high (≥12) or when the user
   asks. Never silently spend big — see the gpt-image-2 rule.
5. **Detect the user's language** and reply in it. Model IDs and flags stay English.
6. **Results auto-save** to the user's AQEM Assets (grouped by session). Don't re-upload or re-describe.

## Workflow

1. **Detect intent and pick a mode** (see Modes). The mode shapes how you write the prompt.
2. **Pick a model** from the decision tree below.
3. **Enrich the prompt** using the mode template + prompt-engineering rules. Turn casual input into a
   concrete, sensory prompt. (Interim: this skill enriches client-side. A future AQEM server-side
   enhancer will take this over — when present, pass the user's raw intent and let it expand.)
4. **Ask the one gating question only if required** (gpt-image-2 tier; or a missing reference image
   for an edit).
5. **Generate** by calling `generate_image` with `prompt`, and `model` / `aspect_ratio` / `quality` /
   `resolution` as needed.
6. **Deliver** the card + a one-line summary (what it made, credits spent, balance if low).

## Model decision tree (intent → model)

Map the *intent*, not the surface keyword. When two fit, prefer the more specific.

- **Quick draft / brainstorm / "just make something" / iterating fast** → `prunaai/flux-fast`
  *(1 credit — the DEFAULT for everything unless a reason below applies).*
- **Detailed scene, richer style, more coherent composition** → `prunaai/flux-kontext-fast` *(2 cr).*
- **Edit / inpaint / change part of an existing image** (user supplied an image) → `prunaai/p-image-edit` *(2 cr).*
- **Photorealism / "make it look real" / lifelike product or person** → `google/nano-banana-2`
  *(12 cr @1K, 17 @2K, 26 @4K).*
- **Premium / hero / highest-fidelity photorealism** → `google/nano-banana-pro` *(25 cr @1K/2K, 50 @4K).*
- **Text rendered inside the image** (poster, logo with words, signage, UI, infographic) **or** strict
  instruction-following → `openai/gpt-image-2` *(low 2 cr / medium 8–9 / high 26–33).*

**Cost tie-breakers:** default to the cheapest model that satisfies the brief. Step up to nano-banana
or gpt-image-2 only when realism or in-image text genuinely requires it.

### gpt-image-2 quality rule (mandatory)
If you choose `gpt-image-2` and the user didn't state a tier, **ask first** with the costs:
`Quality? [Low ~2 cr / Medium ~8–9 cr / High ~26–33 cr]`. Never guess the tier — high can cost 30+ credits.

### Resolution rule (nano-banana only)
`resolution` applies only to `nano-banana-2` / `nano-banana-pro`: `1K` (default, cheapest), `2K`, `4K`.
Omit it unless the user explicitly asks for high-res — 4K can double the cost.

## Modes (prompt-enrichment presets)

A mode decides how you turn the user's words into a strong prompt. Pick by intent:

| Mode | When | Prompt shaping |
|---|---|---|
| `general` | default, open-ended art/scene | subject + setting + style + light; pick a clear medium |
| `photo` | realistic photo of a thing/person/scene | camera (lens, angle), natural lighting, depth, texture; route to nano-banana |
| `product` | a product on a clean or styled background | clean studio or lifestyle setting, soft light, sharp focus; for *branded* work prefer aqem-photoshoot |
| `poster_text` | needs words/typography in the image | state the exact text in quotes, layout, font feel; route to gpt-image-2 |
| `thumbnail` | YouTube/social thumbnail | bold focal subject, high contrast, space for overlay, 16:9 |
| `edit` | modify a supplied image | describe ONLY what changes, not the whole image; route to p-image-edit |
| `restyle` | change an image's vibe/season/aesthetic | name the target aesthetic; preserve subject, change mood |

> Quality lever: most of "perfect images" is the prompt the model receives. Be specific and sensory,
> not literal-vague. See **Prompt engineering** below.

## Aspect ratio

Default `1:1`. Pick by destination: `16:9` cinematic/desktop/thumbnail, `9:16` phone/story/reel,
`4:5` IG feed, `4:3`/`3:4` classic. Pass via `aspect_ratio` (string, e.g. `"16:9"`).

## Editing an existing image (image-to-image)

`generate_image` accepts an optional `image_url` (a public link to the source image).

**Edit quality — unless the user already chose, ASK which tier (state the credits):**
- **High** — `google/nano-banana-pro` + `resolution: "2K"` (~25 credits) — best fidelity
- **Quality** — `google/nano-banana-2` + `resolution: "2K"` (~17 credits) — great photoreal edits
- **Medium** — `prunaai/p-image-edit` (~2 credits) — fast & cheap

Then call `generate_image` with `image_url`, the chosen model (and `resolution: "2K"` for the nano tiers).

- **User gave a URL, or it's a previous AQEM generation** → pass it as `image_url` **silently**.
  Don't ask — you already have the link. (Iterative editing "now make her smile / change the
  background" just feeds the last result's URL back in.)
- **User pasted/attached a LOCAL file with no URL** → get it to a URL using the FIRST path that's
  available, in this order:

  1. **Shell upload (seamless — preferred when you have it).** If you have a Bash/terminal tool AND can
     resolve the attached file's local path, upload it directly — no drop zone, no extra step:
     ```bash
     curl -s -F "file=@/path/to/the/image.png" https://aqemai.com/api/upload
     ```
     The response is JSON: `{"success":true,"url":"https://pub-...r2.dev/...png"}`. Take that `url` and
     immediately call `generate_image` with it as `image_url`. (This is the Higgsfield-style flow; it
     works wherever Claude can run commands — e.g. Claude Code.)
     > Use `curl` for the upload. Do NOT base64-encode the file and call `save_upload` yourself — that
     > is slow and can hang for large images. (`save_upload` exists only for the drop-zone widget.)
  2. **Drop-zone tool (no shell — e.g. Claude web).** If you can't run a shell, call **`upload_image`**:
     it shows a drop zone. The user drops the file; a follow-up message returns the real URL; then call
     `generate_image` with that URL as `image_url`.
  3. **Last resort:** ask the user to paste a public image link.

> ⚠️ **NEVER fabricate, guess, or construct an image URL** (e.g. an `image-proxy?path=...` link or any
> made-up filename). Only ever pass an `image_url` that came from the user verbatim or from
> `save_upload`/`upload_image`. A made-up URL will 404 and the generation will fail. If you don't have a
> real URL, use `upload_image` or ask for a link — do not invent one.

## Pre-generation interview (only if needed)

Skip anything obvious from context. At most 1–2 labeled questions:
- **Editing, only a local file, no URL** → ask for a link (see above).
- **Chose gpt-image-2, no tier** → ask the quality question above.
- **Ambiguous destination affecting ratio** → `[Square / Landscape 16:9 / Portrait 9:16]`.
Otherwise: just generate with sensible defaults.

## Prompt engineering (the quality core)

- **Structure:** subject + setting + style. Add camera (35mm, 85mm; low/overhead angle), lighting
  (rim light, golden hour, soft window light), medium (photo, oil painting, 3D render, anime).
- **Keep it < ~200 tokens.** Very long prompts distort output.
- **Image-to-image / edit:** describe *what changes*, not the input. Bad: re-describe the whole photo.
  Good: "replace background with a sunlit beach, keep the subject unchanged."
- **Negatives → phrase positively:** "no blur" → "tack sharp"; "no people" → "empty landscape".
- **In-image text:** put the literal words in quotes and describe placement; use gpt-image-2.
- **Safety:** avoid real public figures, trademarks/branded characters, and NSFW — these get rejected.

## Delivery

The card renders automatically. Add one short line, e.g.:
`Done — flux-fast, 1 credit spent. Want a variation or a higher-fidelity version?`
Mention remaining balance only when it's getting low. Offer a natural next step (variation, upscale via
a premium model, different aspect ratio). No JSON, no IDs, no enhanced-prompt dump.

## Errors

- **Tool not available** → connector not added; see Step 0.
- **`Not enough AQEM credits`** → relay the needed/available amounts and the top-up link
  (`https://aqemai.com/dashboard`). Offer a cheaper model (e.g. flux-fast) as an alternative.
- **`Generation failed: ...`** → summarize briefly, suggest a retry or simplifying the prompt.
- **Blocked / NSFW / IP** → rephrase positively, drop public figures / trademarks, try again.

## Video (coming soon)

When AQEM video models ship, this skill gains image-to-video and text-to-video:
prefer `--start-image` to anchor the first frame; the prompt then describes **motion** (dolly in,
slow push, the subject turns). Until then, route video requests to a friendly "video is coming very
soon to AQEM" note, or a still image if that satisfies the intent.

## What this skill does NOT do

- Account/billing actions or buying credits (point users to the dashboard).
- Branded product/marketing campaigns at scale → use **aqem-photoshoot**.
- Marketplace listing cards / A+ content → use **aqem-marketplace-cards** (future).
