# plan-review

> A Claude Code skill that turns any implementation plan into a visual, code-grounded review you can actually read.

Drop in a plan (RFC, spec, design doc) and your codebase. `/plan-review` generates a self-contained HTML page that compares the two side-by-side. Where the plan is right, where it's wrong, what could break, and what the trade-offs actually mean in plain English.

Two things make this different from a generic plan visualizer:

1. **ASCII UI mockups.** Every change that touches user-facing code gets tagged `[UX]` and rendered as a before/after wireframe — inline with the code diff, plus an aggregated *UX Flows* view that groups every screen the plan touches.
2. **Plain English layer.** Every jargon-heavy section gets a callout that translates the technical content into the trade-off it represents. One header toggle hides or shows all of them.

---

## What you get

One `.html` file. No external dependencies (except Mermaid + Google Fonts via CDN). Opens in any browser.

The page has nine sections:

1. **Plan summary** — the intuition, the scope, plain-English overview.
2. **Impact dashboard** — files modified, created, deleted, lines changed, completeness indicators for tests/docs/rollback, phase pills if the plan has waves.
3. **Current architecture** — Mermaid diagram of how the affected subsystem works today.
4. **Planned architecture** — same diagram after the plan lands, with new/removed/changed nodes visually called out.
5. **Change-by-change breakdown** — side-by-side code, plain-English callout per change, before/after ASCII mockup if the change touches UX, rationale (or a `⚑ missing rationale` flag if the plan doesn't say *why*).
6. **Ripple analysis** — callers, importers, downstream effects. Color-coded by whether the plan addresses them.
7. **Risk assessment** — edge cases, assumptions, ordering risks, rollback complexity, cognitive complexity. Each with severity.
8. **Good / Bad / Ugly / Questions** — structured critique of the plan itself.
9. **Understanding gaps** — rationale coverage bar, cognitive debt list, explicit "do this before implementing" recommendations.

And two views switched from the header:

- **Full review** — the nine sections above.
- **UX Flows** — every screen the plan touches, aggregated. Before/after ASCII mockups, "what changes for the user" bullets, backlinks to the originating code changes.

---

## Why this exists

Most plan reviews die one of two deaths:

- **Too technical.** The plan is full of file paths and function signatures, and the reviewer skims because nothing tells them *what's actually at stake*.
- **Too hand-wavy.** A high-level summary that ignores what the code actually does today, so claims about "current behavior" turn out to be wrong.

This skill threads the needle. It reads the plan, reads the real code, cross-references the two, surfaces gaps — and presents the result with a Plain English layer for the non-engineer in the room and ASCII mockups for anyone who needs to see the user-facing impact.

---

## Install

This is a Claude Code skill. It's also packaged as a plugin so it can be installed via the marketplace flow.

### Option 1 — Clone and symlink (fastest)

```bash
git clone https://github.com/techwithtam/plan-review.git ~/plan-review
ln -s ~/plan-review/skills/plan-review ~/.claude/skills/plan-review
ln -s ~/plan-review/commands/plan-review.md ~/.claude/commands/plan-review.md
```

That's it. The `/plan-review` command and the `plan-review` skill are now available in any Claude Code session.

### Option 2 — Local plugin marketplace

Add to your `~/.claude/settings.json`:

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

---

## Usage

In any Claude Code session, point the command at a plan file:

```
/plan-review path/to/your-plan.md
```

Optionally pass a codebase root if the plan is for a different repo:

```
/plan-review path/to/your-plan.md path/to/codebase
```

The skill:

1. Reads the plan in full.
2. Reads every file the plan references, plus their importers and tests.
3. Maps the blast radius — what else depends on the changing code.
4. Tags every UX-touching change with `[UX]` and sketches before/after ASCII mockups for each.
5. Writes a structured fact sheet to cross-check every claim against the real code.
6. Generates the HTML page in one of four constrained aesthetics (Blueprint, Editorial, Paper/ink, or Monochrome terminal — varied per run).
7. Writes to `~/.agent/diagrams/<project>-plan-review-<timestamp>.html` and opens it in your browser.

---

## Folder layout

```
plan-review/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── commands/
│   └── plan-review.md           # the /plan-review slash command
├── skills/plan-review/
│   ├── SKILL.md                 # skill entry point
│   ├── references/
│   │   ├── html-structure.md    # document shell, view switcher, section layout
│   │   ├── css-patterns.md      # tokens, depth system, status colors, zoom controls
│   │   ├── mermaid-theming.md   # current vs. planned architecture diagrams
│   │   ├── ascii-ui-mockups.md  # how to draw and embed wireframes
│   │   └── plain-english.md     # voice rules for translation callouts
│   └── templates/
│       └── plan-review.html     # working reference template
├── README.md
├── LICENSE
└── package.json
```

Fully self-contained. No dependency on `cc-viz` or any other skill.

---

## Customizing

Two things you'll want to tune as you use it:

- **Aesthetic.** Edit `skills/plan-review/SKILL.md` to drop aesthetics you don't like or add your own. The forbidden colors list (Tailwind defaults, neon dashboard, gradient mesh) is also in there.
- **Plain English voice.** `skills/plan-review/references/plain-english.md` defines the voice rules. Replace the example translations with terms specific to your domain — the more domain-specific the substitution list, the better the callouts read.

---

## Credit

The plan-review concept and several patterns (constrained aesthetics, Mermaid theming, current-vs-planned diff layout) originated in [cc-viz](https://github.com/zm2231/cc-viz) by Zain Merchant. This skill extends that lineage with three additions:

- ASCII UI mockup layer (inline + aggregated UX Flows view)
- Plain English translation layer with header toggle
- `[UX]` tagging heuristics during data gathering

If you want a broader visualization toolkit covering diff reviews, project recaps, slide decks, and architecture diagrams, install cc-viz alongside this skill.

---

## License

MIT
