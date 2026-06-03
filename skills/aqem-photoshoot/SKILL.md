---
name: aqem-photoshoot
description: |
  Generate brand-quality product & marketing images with AQEM AI. The entry point
  for professional product visuals: studio shots, lifestyle scenes, Pinterest pins,
  hero banners, ad creative, virtual try-on, conceptual/CGI product, and restyles.
  Use when: "product photo", "studio shot", "lifestyle image", "Pinterest pin",
  "hero banner", "website header", "carousel", "ad creative", "Meta/TikTok ads",
  "paid social", "virtual try-on", "model wearing my product", "person holding
  product", "closeup with hands", "levitating/floating/splash product",
  "CGI/surreal product", "restyle this", "seasonal/holiday version", or any
  product, brand, or paid-social creative.
  Modes: product_shot, lifestyle_scene, closeup_with_person, moodboard_pin,
  hero_banner, social_carousel, ad_creative_pack, virtual_model_tryout,
  conceptual_product, restyle. Credit-aware; saves results to AQEM Assets.
  NOT for: plain no-product text-to-image (use aqem-generate), account/billing,
  marketplace listing cards (use aqem-marketplace-cards), or video.
allowed-tools: mcp__aqem__generate_image, mcp__aqem__get_credit_balance
---

# AQEM Product Photoshoot

Brand-image generation via the AQEM MCP connector's `generate_image` tool. This skill
picks the right **mode**, builds a professional prompt from the user's intent, routes to
the best AQEM model, and returns clean image URLs. The mode is the quality lever — it
turns a casual ask ("make ads for my candle") into a structured photography prompt.

> Tool names assume the **AQEM AI** connector is connected (`generate_image`,
> `get_credit_balance`, host-namespaced e.g. `mcp__aqem__generate_image`). See Step 0.
>
> Interim note: AQEM has no server-side product enhancer yet, so **this skill owns the
> prompt templates** (below). When an AQEM `photoshoot` backend enhancer ships, pass the
> raw intent + mode and let it assemble the prompt instead.

## Step 0 — Bootstrap

1. **Connector check.** If `generate_image` is unavailable: *"Add the AQEM AI connector:
   Settings → Connectors → `https://aqemai.com/api/mcp`, then sign in."* Stop until connected.
2. **Credit check.** Product modes route to premium models (nano-banana / gpt-image-2), which cost
   12–50 credits each, and multi-output modes call the tool several times. Call `get_credit_balance`
   first and warn the user if the run will exceed their balance.

## UX Rules

1. Be concise. Final reply = the image URLs + one short line. No JSON, no IDs, no model names,
   no prompt dumps.
2. Detect language, reply in it. Mode names + flags stay English.
3. Ask **at most 4** short, labeled questions before generating — never open-ended. Skip any whose
   answer is obvious (uploaded image, prior turn, stated use case).
4. State the credit cost up front when a run is expensive (multi-output or 4K).
5. Multi-output modes (`social_carousel`, `ad_creative_pack`, or `--count N`) = call `generate_image`
   once per slide/variant, varying angle/light/composition so they aren't copies.
6. Results auto-save to the user's AQEM Assets.

## Modes

| Mode | When the user wants… |
|---|---|
| `product_shot` | Product on neutral / studio / white / catalog background |
| `lifestyle_scene` | Product in a real environment — hands, action, atmosphere |
| `closeup_with_person` | Tight crop with hands / partial face — beauty, holding, demonstrating |
| `moodboard_pin` | Vertical Pinterest-native pin (2:3), moodboard feel |
| `hero_banner` | Wide website / email / campaign header |
| `social_carousel` | 3–10 connected slides for IG / LinkedIn / Facebook |
| `ad_creative_pack` | Coordinated pack of static ad variants for Meta / TikTok / Pinterest / Google |
| `virtual_model_tryout` | Product worn / used by an AI-rendered model |
| `conceptual_product` | Surreal / CGI / levitating / splash / sculptural product |
| `restyle` | Transform an existing image's aesthetic, mood, or season |

## Mode selection (intent, not keyword)

Prefer the more specific mode when two apply.

- neutral / clean / white / studio / catalog / Shopify → `product_shot`
- scene / in use / kitchen / outdoor / cafe / gym → `lifestyle_scene`
- hands holding / face with product / beauty application / demonstrating → `closeup_with_person`
- Pinterest / pin / vertical pin → `moodboard_pin`
- hero / banner / website header / landing / email header / wide → `hero_banner`
- carousel / slides / multi-slide / swipeable → `social_carousel`
- ads / ad pack / paid social / Meta / TikTok / Pinterest ads → `ad_creative_pack`
- model wearing / try-on / on body / lookbook / fashion → `virtual_model_tryout`
- levitating / floating / splash / frozen / surreal / CGI / sculptural → `conceptual_product`
- modify an EXISTING image's vibe/season without changing the subject → `restyle`

**Tie-breakers:**
- "Pinterest pin of my product on a kitchen counter" → `moodboard_pin` (platform wins)
- "Hero banner showing my product in use" → `hero_banner` (format wins)
- "Carousel of my product in different scenes" → `social_carousel` (multi-slide wins)
- "Closeup of someone applying my serum" → `closeup_with_person` (specific genre wins)

## Model routing (which AQEM model per mode)

- **Photoreal product / lifestyle / try-on / closeup / hero** → `google/nano-banana-2`
  (step up to `google/nano-banana-pro` for hero/premium briefs). Use `resolution 2K` for hero/print.
- **Ad packs, pins, or anything with on-image text / headline / price / logo** → `openai/gpt-image-2`
  (ask the quality tier if not given: `[Low ~2cr / Medium ~8–9cr / High ~26–33cr]`).
- **Conceptual / CGI / surreal** → `nano-banana-2` (or `gpt-image-2` if it carries text).
- **Cheap exploratory drafts before committing credits** → `prunaai/flux-kontext-fast` (2 cr), then
  re-run the winner on a premium model.
- **`restyle` / editing a supplied image** → `prunaai/p-image-edit` (2 cr) for light edits;
  `nano-banana-2` when the restyle needs a full re-render.

## Pre-generation interview (≤4 labeled questions, skip the obvious)

**A — uploaded a product photo, "make me images":**
1. How many? `[1 / 3 / 5]`
2. Style/mood? `[Clean studio / Lifestyle / Conceptual / With a model]`
3. Where used? `[Shopify / Instagram / Pinterest / Paid ads / Website hero]`
4. Brand colors to match? (skip if obvious)

**B — uploaded photo + named use case** (mode obvious): ask only the gaps — count, offer/hook, what to emphasize.

**C — text only, no product photo:** ask them to upload a product photo (much higher fidelity); if not,
get category/packaging/color/features; then style + destination.

**D — "redo / change vibe" on an existing image** → `restyle`: aesthetic? `[Clean girl / Quiet luxury /
Cottagecore / Dark academia / Y2K]`; seasonal? `[Christmas / Valentine's / Halloween / Black Friday / None]`.

**E — model wearing a product** → `virtual_model_tryout`: model archetype (suggest 2–3); environment
`[Studio / Outdoor / Street style / Editorial / Home]`; framing `[Full body / 3-quarter / Waist up / Closeup]`.

## Prompt building (per-mode recipes — the quality core)

Build a concrete, sensory prompt from the interview answers. Keep < ~200 tokens. Templates:

- `product_shot`: "{product} centered on a {seamless white / soft gradient / textured stone} surface,
  studio softbox lighting, gentle reflection, sharp focus, commercial product photography."
- `lifestyle_scene`: "{product} in {real setting}, natural {window/golden-hour} light, shallow depth of
  field, lived-in props, authentic candid feel."
- `closeup_with_person`: "close crop of hands {using/holding/applying} {product}, soft natural light,
  skin texture, blurred background, beauty-ad aesthetic."
- `moodboard_pin`: vertical 2:3, "{product} in a {aesthetic} flat-lay / styled scene, cohesive palette,
  Pinterest moodboard mood, airy negative space."
- `hero_banner`: wide 16:9, "{product} hero composition with clean negative space on the {left/right}
  for headline text, premium lighting, brand-forward."
- `ad_creative_pack` / `social_carousel`: lock one visual system (palette, light, framing) across all
  variants; vary angle/crop/background per call. Put any headline/price text in quotes (→ gpt-image-2).
- `virtual_model_tryout`: "{model archetype} {wearing/using} {product} in {environment}, {framing},
  editorial fashion lighting, natural pose."
- `conceptual_product`: "{product} {levitating / mid-splash / sculptural} on {bold color} backdrop,
  dramatic studio light, hyperreal CGI, frozen motion."
- `restyle`: "restyle into {aesthetic}{ + season}; preserve the product, change mood, palette and props."

For on-image text always quote the exact words and route to `gpt-image-2`.

## Aspect ratio per mode (defaults)

`product_shot` 1:1 · `lifestyle_scene` 4:5 · `closeup_with_person` 4:5 · `moodboard_pin` 2:3 ·
`hero_banner` 16:9 · `social_carousel` 4:5 · `ad_creative_pack` 1:1 or 4:5 · `virtual_model_tryout` 3:4 ·
`conceptual_product` 1:1 · `restyle` = source ratio. Override only if the user asks.

## Generation

One `generate_image` call per image. For multi-output, loop and vary the prompt:

```
generate_image(
  prompt   = "<enriched per-mode prompt>",
  model    = "google/nano-banana-2",   # or gpt-image-2 for text/ad packs
  aspect_ratio = "<per-mode default>",
  resolution   = "2K",                 # nano-banana hero/print only
  quality      = "medium",             # gpt-image-2 only, after asking
)
```

If the user gave a product image to reference or edit, route through `p-image-edit` (light edit) or
include the upload as the working image; describe only what changes.

## Delivery

Print the URLs as a short bulleted list + one line. No JSON, no IDs, no model names, no prompt text.

```
3 lifestyle shots ready (nano-banana-2, ~36 credits, 214 left):
- <url>
- <url>
- <url>
Want a hero-banner crop or an ad-pack version?
```

## Errors

- **Tool unavailable** → connector not added (Step 0).
- **`Not enough AQEM credits`** → relay needed/available + top-up link `https://aqemai.com/dashboard`;
  offer fewer variants or a cheaper model (flux-kontext-fast draft).
- **Blocked / NSFW / IP** → drop trademarks/public figures, rephrase positively, retry.
- **Generation failed** → summarize, retry or simplify.

## What this skill does NOT do

- Plain text-to-image with no product/brand → **aqem-generate**.
- Marketplace listing cards / A+ content → **aqem-marketplace-cards** (future).
- Video / animation → coming soon to **aqem-generate**.
- Buying credits or account changes → point to the dashboard.
