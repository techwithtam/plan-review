# plan-review

A Claude Code plugin and skill for visual plan reviews.

Drop in a plan (RFC, spec, markdown design doc) and the current codebase, and `/plan-review` generates a self-contained HTML page that puts them side-by-side — what the plan claims vs. what the code actually does, where the trade-offs are, and what could break.

Two things make this different from a generic plan visualizer:

1. **ASCII UI mockups** for every UX-touching change. Before/after wireframes appear inline next to the code diff and aggregate into a dedicated **UX Flows** view, so you see what the user will experience, not just what the AST will look like.
2. **Plain English layer.** Every jargon-heavy section gets a callout that translates the technical content into terms a smart non-engineer would understand. Toggleable via a header switch.

## What you get

A single `.html` file with:

- **9 sections**: Plan summary, Impact dashboard, Current architecture (Mermaid), Planned architecture (Mermaid), Change-by-change breakdown, Ripple analysis, Risk assessment, Good/Bad/Ugly review, Understanding gaps.
- **Two views**: *Full review* (sections 1-9) and *UX Flows* (aggregated mockups by screen). Switch via header tabs.
- **Plain English toggle** in the header (`Plain English: ON`). Hides/shows every translation callout in one click. State persists.
- **`[UX]` tag** on changes that touch user-facing code, with inline ASCII before/after wireframes.
- **`⚑ missing rationale` flags** on changes where the plan says *what* but not *why*.
- **Mermaid current-vs-planned diagrams** with zoom and pan.
- Constrained aesthetics — Blueprint, Editorial, Paper/ink, or Monochrome terminal. No AI-slop palettes.

## Install

This is a Claude Code plugin, distributable via the plugin marketplace.

### From this repo

```bash
git clone https://github.com/tamnguyenvan/plan-review ~/plan-review
```

Then add to your `~/.claude/settings.json`:

```json
{
  "plugins": {
    "marketplaces": {
      "plan-review": {
        "type": "local",
        "path": "~/plan-review"
      }
    }
  }
}
```

Or symlink for live editing during development:

```bash
ln -s ~/plan-review/skills/plan-review ~/.claude/skills/plan-review
ln -s ~/plan-review/commands/plan-review.md ~/.claude/commands/plan-review.md
```

## Usage

In any Claude Code session:

```
/plan-review path/to/plan.md
```

Or with a specific codebase root:

```
/plan-review path/to/plan.md path/to/codebase
```

The skill writes the generated HTML to `~/.agent/diagrams/<project-slug>-plan-review-<timestamp>.html` and opens it in your browser.

## Folder layout

```
plan-review/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest
│   └── marketplace.json         # Marketplace manifest
├── commands/
│   └── plan-review.md           # The /plan-review slash command
├── skills/plan-review/
│   ├── SKILL.md                 # Skill entry point
│   ├── references/              # Workflow references the skill reads
│   │   ├── html-structure.md
│   │   ├── css-patterns.md
│   │   ├── mermaid-theming.md
│   │   ├── ascii-ui-mockups.md
│   │   └── plain-english.md
│   └── templates/
│       └── plan-review.html     # Reference template
├── README.md
├── LICENSE
└── package.json
```

The skill is **fully self-contained**. No dependency on `cc-viz` or any other skill — every reference and template lives in this repo.

## Credit

The plan-review concept and several patterns (constrained aesthetics, Mermaid theming, mockup-pair layout) originated in [cc-viz](https://github.com/zm2231/cc-viz) by Zain Merchant. This skill extends that lineage with the ASCII UI mockup layer, the Plain English translation layer, and the UX Flows aggregate view.

## License

MIT
