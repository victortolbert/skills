---
name: refactoring-ui
description: Apply high-impact UI/UX refactoring techniques across typography, layout, color, shadows, and component details. Use when designing or refactoring user interfaces, reviewing visual design, or polishing components for production quality.
---

# Refactoring UI

Act as an expert UI/UX design agent. When designing or refactoring interfaces, apply the rules below to typography, layout, color, and components.

## Typography

- **Reading widths.** Constrain paragraphs to 70–80 characters per line (e.g. `max-width: 40ch`).
- **Font weights & tracking.** Use variable fonts to reach intermediate weights (e.g. 550) between medium and semi-bold. Tighten letter tracking for headlines over 24–30px.
- **Uppercase styling.** Widen letter tracking on fully uppercase text (labels, eyebrows). Use monospace for small eyebrow text for a highly designed feel.
- **Editorial emphasis.** Italicize specific words in headlines/copy for a conversational tone. For long text blocks consider print-inspired touches — drop caps inside a circle, small caps for opening words.

## Layout & Spacing

- **Room to breathe.** Use liberal vertical padding; widen panels to avoid cramped content and excessive wrapping.
- **Alignment.** Default to left-aligning headlines, navs, and voting options; avoid floating-center layouts.
- **Form structure.** Don't let inputs span the full width of large monitors. Split long forms into sections separated by light borders and generous spacing. Use a two-column layout: labels/context in the first third, inputs in the remaining two-thirds.
- **Canvas grids.** Add visual interest with decorative borders wrapping the site and containing each section — no custom graphics required.

## Color, Contrast & Shadows

- **Crisp shadows.** Don't use solid borders on elevated elements. Use Gray 950 at 5–10% opacity as an outer ring, bleeding into the drop shadow below. Layer multiple shadows — one saturated with a large blur, one darker with a short blur — for a natural effect.
- **Tailored grays.** Match gray temperature to the brand: true grays for warm brands, grays saturated with the primary hue (e.g. blue) for cool brands.
- **Difficult colors.** If a brand color fails contrast with white text, don't darken it into a muddy brown. Shift to a lighter, warmer alternative (closer to yellow) and apply dark text.
- **Background image legibility.** For text over photos, desaturate the photo and use `multiply` blend mode with a bold brand color overlay.

## Components & Details

- **Icons vs. text.** Solid icons read heavier than text. When paired side-by-side, make icons slightly lighter and smaller so they balance.
- **Containing icons.** If an icon lacks detail, don't enlarge it — contain it within a geometric shape (e.g. a circle) to give it a larger footprint.
- **Input fields.** Use off-white or gray backgrounds for inputs on white pages, removing the need for busy borders. Match input length to expected data length.
- **Data tables.** Don't repeat labels in every cell of a comparison chart — use a dedicated label column.
- **Buttons.** One primary action per page. De-emphasize secondary actions with a subtle border and inverted text.
