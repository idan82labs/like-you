# design-system.md template

The output of Phase 2.5 (visual extraction). One file per workspace. Read by every skill that produces visual output (`design-deck`, `draft-proposal`, `design-landing-page`, anything that renders to PDF/HTML/PPTX).

## Frontmatter

```yaml
---
generated_at: {ISO timestamp}
extracted_from:
  - "{source filename 1}"
  - "{source filename 2}"
extraction_method: pdfimages + pdftoppm + vision
tooling_used: poppler X.Y, ImageMagick X.Y
confidence: high | medium | low
language: he | en | mixed
direction: rtl | ltr
---
```

`confidence: high` requires ≥2 sources cross-checked. Single source → `medium`. Vision-only without asset extraction → `low`.

## Body sections (in this order)

### 1. Brand identity (5 lines max)

- Name
- Tagline (verbatim from extracted assets)
- Aesthetic in 1 sentence ("editorial restrained navy + white", "warm-paper one-accent", "brutalist mono")
- What it is NOT (helps skills avoid drift)

### 2. Persistent assets (CRITICAL — the reuse mechanism)

Table listing extracted assets that **skills MUST embed by path, not recreate**.

```markdown
| Asset | Path | Resolution | Use |
|---|---|---|---|
| Header banner (hi-DPI) | `raw/visual/assets/{slug}/banner-hires.png` | ~5000×600px | **Preferred** — crisp at A4 width and beyond |
| Header banner (original) | `raw/visual/assets/{slug}/{slug}-000.png` | source-embedded | Fallback if hi-DPI extraction failed |
| Logo only | `raw/visual/assets/{slug}/logo.png` | varies | Inline references, small contexts |
| Photos / decorative | `raw/visual/assets/{slug}/*` | source-embedded | Reuse as-is |
```

Then a code-block showing the embed pattern:

```html
<img src="../../raw/visual/assets/{slug}-banner.png" alt="..." style="width:100%;display:block;">
```

The point of this section: **stop the skill from regenerating logos in CSS.** Use the file.

### 3. Color tokens

CSS variable block with semantic names:

```css
--ink:     {hex}   /* heavy/banner navy/black — usually bg-only, NOT text */
--header:  {hex}   /* primary brand color — headings/emphasis text */
--body:    {hex}   /* body copy color */
--muted:   {hex}   /* secondary text */
--rule:    {hex}   /* dividers */
--paper:   {hex}   /* main background */
--paper-soft: {hex}   /* card/alt fill */
--accent:  {hex}   /* optional accent (talk decks only / casual contexts) */
```

**Cross-source verification table** is mandatory for confidence:high. Shows each token, where it was sampled, and whether sources agreed.

### 4. Typography

```markdown
| Role | Family | Weight | Size | Notes |
|---|---|---|---|---|
| Headlines (h1) | {Heebo \| Assistant \| ...} | {weight} | {pt} | |
| Subheads (h2) | ... | | | |
| Body | ... | | | |
| Numerical | {Inter Tight \| tabular} | tnum | | |
```

If the brand has actual font files in `raw/visual/fonts/` → reference them. Otherwise: best-guess + closest open-source analog. Always include CSS fallback stack ending in a system font.

### 5. Layout patterns

ASCII diagrams of recurring layouts: cover page, continuation page, table page, signature page. Mark which elements are MANDATORY (banner) vs OPTIONAL (logo top-right).

Also: direction (`rtl` / `ltr` / `bidi`), margin system (mm or rem), grid columns.

### 6. Numerical & content conventions

- How are numbers presented? (big, callout, in-line)
- Numbered sections format (1. / 1.1 / ١.)
- Bullets character (`•` / `–` / `▪`)
- Code/technical terms inline treatment (quoted? bold? mono?)
- English-in-Hebrew (or vice versa) rules

### 7. Iconography & decoration

What decorative elements the brand uses (and especially what it DOESN'T). If brand is minimal — say so explicitly so skills don't add icons.

### 8. Whitespace & density

Generous vs dense. Section break spacing. Maximum content density (rough %).

### 9. Forbidden visual moves

Hard list of things the brand never does. Examples:
- Gradients of any kind
- Heavy drop-shadows
- Coloured backgrounds beyond named tokens
- Stock photography
- Emoji in formal documents

These bans live HERE so every skill checks before emitting. Same role as `voice.md`'s "forbidden phrases" but for visuals.

### 10. How skills should use this file

5–7 numbered rules. Examples:
1. Read this file first before producing any visual output.
2. Embed assets from § Persistent assets by relative path — don't recreate.
3. Pin colors to the tokens — don't pick new hex codes.
4. RTL/LTR per `direction:` field.
5. HTML output: use Google Fonts CDN for declared families with system fallbacks.
6. PPTX output: drop the banner as the top element of every slide template.
7. Print/PDF output: use `@page { size: A4; margin: 0; }` + `print-color-adjust: exact`.

### 11. Variants & open questions

Note what's been sampled vs what's missing. Future re-runs fill in. Examples:
- "Talk-deck variant uses orange accent (#FF6B1A) — not yet sampled separately."
- "Spread layout for printed reports not extracted."

## Constraints on the template itself

- **Editable**: users will manually tune values. Frontmatter `confidence:` should change as they validate.
- **Linkable**: every skill that depends on this file references it by path. If renamed, all skills break — keep at `context/design-system.md` exactly.
- **Self-contained**: a downstream skill should be able to produce a brand-faithful output from THIS FILE + `raw/visual/assets/` alone, with no further extraction. Verify after writing: can a fresh `claude -p` session generate a brand-accurate HTML page from this file alone?
