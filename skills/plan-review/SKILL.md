---
name: plan-review
description: Generate a self-contained HTML page that reviews a proposed implementation plan against the real codebase. Cross-references plan claims against current code, flags risks and ripples, includes ASCII UI mockups for any UX-touching change, and renders a Plain English translation layer that explains technical trade-offs in non-jargon terms. Use whenever the user asks to review a plan, RFC, or spec against the codebase, says "plan review", "review this plan", "make sense of this plan", or runs the `/plan-review` slash command.
license: MIT
metadata:
  version: "0.1.0"
---

# plan-review

Generate a self-contained HTML page that puts a plan and the codebase side-by-side. Not a rendering of the plan as-is. A *review* of it: where it's grounded in the code, where it isn't, what could break, and what the trade-offs actually mean for a human reader.

Two things make this skill different from a generic visualizer:

1. **ASCII UI mockups** — when a plan touches UX (routes, components, screens, forms), before/after wireframes appear inline next to the code diff so you can see what the user will experience, not just what the AST will look like.
2. **Plain English layer** — every jargon-heavy section gets a callout that translates the technical content into terms a smart non-engineer would understand. Toggleable via a header switch (state in localStorage). Always leads with the trade-off, not the implementation.

## Workflow

### 1. Read the plan and the codebase

Don't skim. The whole value of this review is that it's *code-grounded* — fake claims are worse than no review. Follow the data-gathering steps in `commands/plan-review.md`:

1. Read the plan file in full.
2. Read every file the plan references, plus their importers and tests.
3. Map blast radius — grep for callers, schemas, public API consumers.
4. Tag UX-touching changes with `[UX]`.
5. Cross-reference: do the files/functions/types the plan names actually exist?

### 2. Build a fact sheet before generating

For every claim the page will make, cite the source — plan section or `file:line`. If you can't verify a claim, mark it uncertain. This is your source of truth during HTML generation. Do not deviate from it.

### 3. Sketch ASCII mockups for UX changes

For each `[UX]` change, draw a before/after ASCII wireframe of the affected screen *before* writing HTML. Read `references/ascii-ui-mockups.md` for conventions: box-drawing characters, sizing rules, the `<pre class="pr-mockup">` wrapper, and how to handle changes that touch multiple screens vs. one screen touched by multiple changes.

### 4. Write Plain English callouts

For each section that earns one (plan summary, every change-by-change panel, risk assessment, and every UX Flows screen), write a 2-4 sentence callout. See `references/plain-english.md` for the voice rules: lead with the trade-off, never use jargon you wouldn't say at a dinner table, never start with "this section explains".

### 5. Generate the HTML

Read these references in order, then generate:

- `references/html-structure.md` — the document shell, view switcher, section layout, header controls.
- `references/css-patterns.md` — theme tokens, depth system, status colors, mermaid zoom controls, overflow protection.
- `references/mermaid-theming.md` — current vs. planned architecture diagrams, theme variables, ELK layout, zoom/pan.
- `references/ascii-ui-mockups.md` — embedding ASCII in HTML cleanly.
- `references/plain-english.md` — voice and structure for callouts.
- `templates/plan-review.html` — full reference template you can read end-to-end before writing.

### 6. Write to disk and open

Write to `~/.agent/diagrams/<project-slug>-plan-review-<YYYYMMDD-HHmm>.html` (create the directory if it doesn't exist). Run `open <path>` to view. Tell the user the path and a one-line summary of what's in it.

## Design system

### Aesthetic

Pick one. Commit. Vary from recent runs.

**Constrained aesthetics (prefer these):**
- **Blueprint** — technical drawing feel, subtle grid background, deep slate/blue palette, monospace labels, precise borders.
- **Editorial** — serif headings (Instrument Serif, Crimson Pro), generous whitespace, muted earth tones or deep navy + gold.
- **Paper/ink** — warm cream `#faf7f5` background, terracotta/sage accents, informal feel.
- **Monochrome terminal** — green/amber on near-black, monospace everything.

**Forbidden:**
- Neon dashboard (cyan + magenta + purple on dark) — always reads as AI slop.
- Gradient mesh (pink/purple/cyan blobs).
- Inter + violet/indigo + gradient text. Avoid this combination entirely.
- `#8b5cf6` `#7c3aed` `#a78bfa` `#d946ef` accents — Tailwind defaults that signal zero intent.

### Typography

Pick a font pairing from `references/css-patterns.md`. Forbidden as `--font-body`: Inter, Roboto, Arial, Helvetica, system-ui alone. Load via `<link>` with a system fallback in the stack.

### Color tokens

Define both light and dark in `:root` and `@media (prefers-color-scheme: dark)`. Minimum tokens: `--bg`, `--surface`, `--surface-elevated`, `--border`, `--border-bright`, `--text`, `--text-dim`, plus 3-5 accent colors with full and dim variants. Name semantically (`--state-current`, `--state-planned`, `--state-risk`) not by hue.

**Required semantic colors for plan-review specifically:**
- `--state-current` — neutral / blue: today's code
- `--state-planned` — green or warm accent: proposed code
- `--state-risk` — amber: risks, gaps, missing rationale
- `--state-gap` — red: definite misses, broken assumptions
- `--state-pe` — soft tinted background for Plain English callouts (use `--accent-dim`)

## Boundaries

- Review is read-only analysis. NEVER modify plan artifacts or code.
- NEVER fabricate file paths, function names, line numbers, or behavior descriptions. Missing data → "Not documented in plan" placeholder.
- ASCII mockups are best-effort representations of what the user will see. If you don't know the screen well enough to draw it, mark the mockup as "speculative" with a small tag.
- Plain English callouts are best-effort translations. If a section's logic is genuinely hard to translate without losing meaning, say so in the callout itself ("Hard to summarize without losing accuracy — see code panel").
- Keep HTML under 600KB. Massive plans → paginate change panels or lazy-render the UX Flows view.

## Success criteria

- Single valid `.html` file, no external deps except Mermaid CDN + Google Fonts.
- All 9 Full Review sections present (or labeled placeholder if no source data).
- View switcher between Full review and UX Flows works, state persists in localStorage.
- Plain English toggle hides/shows all callouts, state persists in localStorage.
- Every `[UX]` change has a before/after ASCII mockup inline AND appears in the UX Flows aggregate view.
- Light mode default, dark mode via `prefers-color-scheme`.
- No text smaller than 14px. Body text 16px+.
- Constrained aesthetic applied, fonts and palette differ from recent runs.
- All claims about the codebase verified during the fact-sheet step. Unverifiable claims explicitly flagged.
- Open in browser via `open <path>` after generation.
