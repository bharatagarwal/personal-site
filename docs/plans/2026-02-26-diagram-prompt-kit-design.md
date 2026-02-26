# Design: diagram-prompt-kit

## What it is

A framework for getting consistent, accessible, editorial-quality SVG diagrams from LLMs. Encodes a visual philosophy — composable primitives, composition rules, colorblind-safe palette — so that reproducible output falls out naturally. The Manim of explanatory diagrams, operating through natural language instead of Python.

## What it isn't

A charting library. An SVG editor. A component library. An animation framework. It is prompt context + conventions + examples.

## Target user

Writers, bloggers, and documentation authors who use LLMs to produce explanatory diagrams and want consistent, accessible results without extensive iteration.

## Design principles

- **Model-agnostic**: Tested with Claude, principles apply broadly. Frame as general LLM guidance.
- **Framework with examples**: Ship structure (template, archetypes, rules) with palette and style configurable. The default Okabe-Ito aesthetic is one example, not the only option.
- **Manim-inspired**: Composable primitives + scene types + constraints. The LLM is the "runtime." Strong defaults, customizable.
- **Under 80 lines**: The prompt-context.md must be concise. Dense enough for an LLM to hold in working memory and produce correct output. Long context dilutes signal.

## Core artifact: prompt-context.md

Structured in five sections:

### 1. Primitives (like Manim's MObjects)

| Primitive | Description | Key attributes |
|-----------|-------------|----------------|
| Panel | White rounded rect with shadow and border, holds content | rx, fill, stroke, filter |
| Node | Labeled shape with a palette color | color from palette, data-role="node" |
| Arrow | L-shaped polyline with arrowhead marker | stroke, marker-end (not bezier curves) |
| Label | Text element at a specific size tier | title (17-19px), body (10-12px), annotation (8-10px) |
| Axis | Structural line with labeled endpoints | stroke #999-#ddd |
| Zone | Dashed boundary grouping related elements | stroke-dasharray, no fill or light fill |

### 2. Scene types (like Manim's Scene subclasses)

| Scene | Composition rule |
|-------|-----------------|
| Two-panel comparison | 2 Panels side by side, dashed divider, each Panel contains a visual + Labels |
| Feedback loop | 4 Nodes at cardinal positions, Arrows connecting in cycle, center Label |
| Coordinate space | 2 Axes crossing, content in quadrants, Zones overlaying |
| Vertical flow | Nodes stacked with Arrows between them |
| Causal chain | Nodes in horizontal sequence with Arrows |

### 3. Rules

- Namespace prefix on all IDs and class scopes (e.g., `gvl-shadow`, `.gvl .title`)
- `role="img"`, `aria-labelledby` pointing to `<title>` and `<desc>`
- Background: vertical linear gradient (`#f7f9fc` to `#ebeef4`)
- Drop shadows: `feGaussianBlur` filter on `<rect>` elements only, NOT on `<g>` (rasterizes text)
- Fonts: `sans-serif` throughout, no web font dependencies
- Positioning: absolute via `transform="translate(x,y)"`, not relative layout
- Groups ordered in DOM to match natural reading/reveal order
- `data-role` attribute on primitives (`node`, `arrow`, `label`, `panel`, `axis`, `zone`)
- Self-contained: no external references

### 4. Palette (default: Okabe-Ito)

| Role | Color | Hex | Usage |
|------|-------|-----|-------|
| Primary | Blue | `#0072B2` | Structural, intellectual |
| Warning | Orange | `#D55E00` | Critique, barriers, gates |
| Secondary | Amber | `#E69F00` | Economic, warm |
| Positive | Green | `#009E73` | Growth, earned |
| Tertiary | Pink | `#CC79A7` | Exclusion |
| Highlight | Sky blue | `#56B4E9` | Aspiration |
| Text dark | — | `#1a1a1a` | Titles |
| Text mid | — | `#555-#666` | Body, subtitles |
| Text light | — | `#888-#aaa` | Annotations |
| Structural | — | `#999-#ddd` | Axes, dividers |

### 5. Anti-patterns

- No freehand curves or organic illustration
- No complex bezier paths — use L-shaped polylines
- No layout requiring visual balancing by eye — use explicit coordinates
- No decorative elements that don't carry information
- No gradients on shapes (only on backgrounds)
- No `<style>` class names without namespace prefix

## Repo structure

```
diagram-prompt-kit/
├── README.md                      # Philosophy, quickstart, Manim comparison,
│                                  # future directions
├── prompt-context.md              # THE core artifact
├── findings.md                    # Empirical data on LLM SVG generation
├── palette/
│   ├── okabe-ito.json             # Default palette with semantic roles
│   └── customizing.md             # How to swap in your own palette
├── archetypes/
│   ├── two-panel-comparison.svg   # Annotated reference SVG
│   ├── feedback-loop.svg
│   ├── coordinate-space.svg
│   ├── vertical-flow.svg
│   └── causal-chain.svg
├── template.svg                   # Minimal SVG skeleton with all conventions
└── examples/
    └── (2-3 real-world diagrams produced using the system)
```

## Workflow in practice

1. User copies `prompt-context.md` into LLM conversation (or includes as project context)
2. User says: "Create a [scene type] diagram about [topic]. Nodes: [list]. Colors: [assignments]. Prefix: `xyz`. ViewBox: [dimensions]."
3. LLM produces SVG following primitives, scene type, rules, and palette
4. Minimal iteration — the context did the heavy lifting

## Out of scope (v1)

- Animation integration (GSAP + ScrollTrigger solves when needed)
- Composability documentation with external libraries
- MCP server wrapping the design system
- Multi-model testing (GPT-4, Gemini)
- Build-time tooling (SVG preprocessors, validators)
- Dark mode variants

## Future directions

- **Composability**: Document integration with Rough Notation (hand-drawn annotations), GSAP ScrollTrigger (scroll-driven reveal), RoughJS (sketch style), Anime.js (simpler animations), Leader Line (cross-element connections)
- **MCP server**: Serve the design system as tools (`get_archetype`, `validate_svg`, `get_palette`)
- **Color Theory MCP**: Integrate for palette customization and WCAG validation
- **Cross-model testing**: Validate findings with GPT-4, Gemini, and document model-specific behaviors
- **Blog post**: Write about the approach — "How I make diagrams with LLMs"

## Landscape context

### What exists
- Academic work on general LLM-to-SVG generation (SVGThinker, GenAI-DrawIO-Creator, "See it. Say it. Sorted.")
- GSAP + ScrollTrigger for SVG animation
- CSS `@scope` and Shadow DOM for SVG style isolation
- Okabe-Ito and ColorBrewer for accessible color palettes
- Tufte, Bertin, Munzner for information visualization theory

### What doesn't exist (the gap this fills)
- A design system specifically optimized for what LLMs produce reliably
- Documented empirical findings about LLM SVG generation capabilities and limitations
- A prompt template framework for consistent diagram output
- The Manim equivalent for editorial/explanatory diagrams via natural language
