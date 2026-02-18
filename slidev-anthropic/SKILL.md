---
name: slidev-anthropic
description: Create polished Slidev presentations using the Anthropic-inspired theme (slidev-theme-anthropic). Use when the user asks to create a presentation, slide deck, or slides using Slidev, or references the Anthropic theme. Covers project setup, slide authoring with cover/section/body slide types, correct CSS class usage, sequencing, typography, and content guidelines. Also use when editing or adding slides to an existing Anthropic-themed Slidev deck.
---

# Slidev Anthropic Theme

Create Slidev presentations using the `slidev-theme-anthropic` theme -- warm terracotta covers, muted earth-tone section dividers, clean white editorial body slides with two-column layouts.

## Project Setup

### New project

```bash
mkdir my-presentation && cd my-presentation
npm init -y
npm install @slidev/cli slidev-theme-anthropic
# Or from GitHub: npm install github:leonchike/slidev-theme-anthropic
```

Add scripts to `package.json`:

```json
{
  "scripts": {
    "dev": "slidev",
    "build": "slidev build",
    "export": "slidev export"
  }
}
```

### Frontmatter

Every `slides.md` starts with:

```yaml
---
theme: anthropic
title: Your Title Here
highlighter: shiki
drawings:
  persist: false
transition: fade
---
```

If the theme is not installed as a package and instead uses a local `style.css`, set `theme: none`.

## Slide Types

There are exactly three slide types. Use `<div>` wrappers with CSS classes -- never use Slidev's built-in `layout:` frontmatter.

### Cover Slide

First slide only. Full-bleed terracotta background, large serif title, brand at bottom-left.

```markdown
<div class="cover-slide">
<div>

# Presentation Title

</div>
<div class="brand">Team Name</div>
</div>
```

- Title: 3-8 words max
- No subtitles, bullets, or descriptions

### Section Slide

Introduces each topic. Full-bleed colored background, chapter pill, serif title at bottom-left.

```markdown
<div class="section-slide olive">
<div class="chapter-pill">Label</div>

# Section Title

</div>
```

Colors: `olive`, `terracotta`, `clay`, `oat` -- vary across sections, never repeat consecutively.

- Chapter pill: 1-2 word label
- Title: 2-6 words
- No body text, lists, or code

### Body Slide

All content goes here. White background, chapter header, two-column grid.

```markdown
<div class="body-header">
<div class="chapter-pill">Label</div>

# Page Title

</div>

<div class="two-col">
<div>

## Left Heading

Content here.

</div>
<div>

## Right Heading

Content here.

</div>
</div>
```

- Chapter pill must match the preceding section slide
- Always use the `two-col` grid -- no single-column layouts
- Each column gets its own h2
- Max ~120 words per column; add another body slide if needed

## Sequencing

Every topic: **Section Slide** (colored) then **Body Slide(s)** (white).

```
1. Cover Slide
2. Section Slide (olive)       → topic 1 intro
3. Body Slide                  → topic 1 content
4. Section Slide (terracotta)  → topic 2 intro
5. Body Slide                  → topic 2 content
6. Section Slide (clay)        → topic 3 intro
7. Body Slide                  → topic 3 content
8. Section Slide (oat)         → topic 4 intro
9. Body Slide                  → topic 4 content
```

Multiple body slides per section are fine -- keep the same chapter pill label.

## Typography

- **h1**: Only in cover-slide, section-slide, body-header. Never inside `two-col`.
- **h2**: Bold sans-serif column subheadings inside `two-col`.
- **h3**: Sub-sections within a column (use sparingly).
- **Body**: 1-3 short paragraphs per column.
- **Bold** (`**text**`): Key terms, component names.
- **Inline code**: Tool names, API paths, config values.

## Content Elements

**Code blocks**: Warm beige background, dark text. Keep under 15 lines. All syntax highlighting is overridden to uniform dark charcoal.

**Tables**: Minimal -- bold header border, thin row separators, no stripes. Max 8 rows.

**Lists**: Filled disc bullets. Use `- **Bold lead** -- description` pattern.

## Constraints

- Never use `layout:` frontmatter (`layout: cover`, `layout: section`, etc.)
- Never use dark code block backgrounds
- Never add emojis, icons, or decorative elements
- Never use h1 inside `two-col`
- Never apply inline color styles
- Never use `---` as a horizontal rule (it's a slide separator in Slidev)
- Never override `position`, `inset`, or `padding` on cover-slide or section-slide divs

## Reference

For the full CSS architecture (color tokens, full-bleed positioning details, Shiki override specifics), read [references/theme.css](references/theme.css).
