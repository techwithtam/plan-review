# plan-review

> A Claude Code skill that turns any implementation plan into a visual, code-grounded review you can actually read.

When you give Claude Code a plan (an RFC, a spec, a design doc, a `/plan` output), what comes back is usually a wall of file paths, function names, and architecture jargon. You read it, you nod, you say "looks good," and then halfway through implementation you realize there were three things in there you didn't actually understand. Or worse, the plan referenced behavior in the code that turned out to be wrong.

`/plan-review` runs the plan through a structured review and gives you back a single HTML page that puts everything side by side. You learn what's changing, what could break, what the trade-offs really mean, and where the plan is making claims that don't match the code.

## What it does

You point the command at a plan file:

```
/plan-review path/to/your-plan.md
```

It reads the plan in full. Then it reads every file the plan references, plus the files that import or depend on those files, plus the tests. It cross-references everything: if the plan says "the `createSession` function does X today," it actually opens `createSession` and checks. If something doesn't match, it flags it.

Then it writes a self-contained HTML page to `~/.agent/diagrams/` and opens it in your browser. One file. No build step. Mermaid and Google Fonts load from a CDN, but everything else is inline.

## What's in the page

Nine sections in the **Full review** view:

1. **Plan summary.** The intuition behind the plan, the scope (files touched, lines added/removed, new tests planned), and a Plain English overview at the top.
2. **Impact dashboard.** Counts of files modified, created, deleted, lines changed. Completeness indicators for tests, docs, and rollback. If the plan defines waves or phases, you see them as status pills.
3. **Current architecture.** A Mermaid diagram of how the affected subsystem works today. Only the parts the plan touches.
4. **Planned architecture.** Same diagram after the plan lands. New nodes glow green, removed nodes fade out, changed edges shift color. Same node names and layout as section 3 so the diff is visually obvious.
5. **Change-by-change breakdown.** Each proposed change as its own panel: current code on the left, planned code on the right, a Plain English callout explaining the trade-off, an ASCII UI mockup if the change touches user-facing code, and the rationale pulled from the plan. If the plan says what to do but never explains why, you see a `⚑ missing rationale` flag.
6. **Ripple analysis.** Every caller, importer, and downstream dependency. Color-coded by whether the plan addresses it, mentions it in passing, or misses it entirely.
7. **Risk assessment.** Edge cases, assumptions, ordering risks, rollback complexity, cognitive complexity. Each with a severity indicator.
8. **Good, Bad, Ugly, Questions.** A structured critique of the plan itself. Things it gets right. Gaps. Subtle concerns that will bite later. Ambiguities to clarify with the author before you start.
9. **Understanding gaps.** A closing dashboard showing rationale coverage as a progress bar, a list of cognitive debt flags, and an explicit "before you implement, document X and Y" recommendation.

And a second view, switched from the header:

**UX Flows.** Every screen the plan touches, aggregated. Before/after ASCII mockups side by side. A "what changes for the user" bullet list per screen. Backlinks to the code changes that touch each screen. Useful when the person reviewing the plan cares more about the user experience than the implementation.

## What makes it useful

Three things, specifically.

**It reads the actual code.** The whole review is grounded in what's in your repo right now. Before generating the page, the skill produces an internal fact sheet citing every claim against either a plan section or a `file:line` in the codebase. If a claim can't be verified, it's marked uncertain rather than presented as fact. So you don't get a confident, beautifully-styled page that turns out to be wrong about what the code does today.

**ASCII UI mockups for UX changes.** Whenever a change touches a route, page component, form, or anything user-facing, the page tags it `[UX]` and renders a before/after wireframe in box-drawing characters right next to the code diff. Then the UX Flows view aggregates every mockup by screen. So a non-engineer (a PM, a designer, a stakeholder) can look at the same page and understand what the user is about to experience, without reading a single line of code.

**A Plain English layer that you can toggle.** Every jargon-heavy section gets a callout that translates the technical content into the trade-off it represents. Lead sentence is always the trade-off, never the implementation: "Switching from cookies to JWTs. Means logins work across subdomains without extra setup, but harder to revoke a stolen token." One switch in the header turns every callout on or off at once, with the state saved across visits.

## Install

This is a Claude Code skill. It's also packaged as a plugin so it can be installed via the local marketplace flow.

### Option 1: Clone and symlink

```bash
git clone https://github.com/techwithtam/plan-review.git ~/plan-review
ln -s ~/plan-review/skills/plan-review ~/.claude/skills/plan-review
ln -s ~/plan-review/commands/plan-review.md ~/.claude/commands/plan-review.md
```

That's it. The `/plan-review` command and the `plan-review` skill are now available in any Claude Code session.

### Option 2: Local plugin marketplace

Add this to your `~/.claude/settings.json`:

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

## Usage

```
/plan-review path/to/your-plan.md
```

If the plan is for a different repo, pass the codebase root as a second argument:

```
/plan-review path/to/your-plan.md path/to/codebase
```

Here's what happens, step by step:

1. The skill reads the plan in full.
2. It reads every file the plan references, plus their importers and tests.
3. It maps the blast radius. What else in the code depends on the files about to change.
4. It tags every UX-touching change with `[UX]` and sketches a before/after ASCII mockup for each.
5. It produces a structured fact sheet to verify every claim against the real code.
6. It picks one of four constrained visual aesthetics (Blueprint, Editorial, Paper/ink, or Monochrome terminal) and varies the choice from run to run so the output never looks generic.
7. It writes the HTML to `~/.agent/diagrams/<project>-plan-review-<timestamp>.html` and opens it in your browser.

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

Fully self-contained. No dependency on any other skill.

## Customizing

Two things you'll want to tune as you use it.

**Aesthetic.** Edit `skills/plan-review/SKILL.md` to drop aesthetics you don't like or add your own. The forbidden colors list (Tailwind defaults, neon dashboard, gradient mesh) is also in there. Adjust it to taste.

**Plain English voice.** `skills/plan-review/references/plain-english.md` defines the voice rules. The included substitution list (replace "endpoint" with "URL," replace "middleware" with "code that runs before/after every request," and so on) is generic by design. Replace it with terms specific to your domain. The more domain-aware the substitutions, the better the callouts read for your team.

## Credit

The original `/plan-review` command and several of the patterns reused here (constrained aesthetics, Mermaid theming, the current-vs-planned diff layout) come from [cc-viz](https://github.com/zm2231/cc-viz) by Zain Merchant. This skill builds on that foundation and adds three things: the ASCII UI mockup layer with its dedicated UX Flows view, the Plain English translation layer with a header toggle, and the `[UX]` tagging heuristics in the data-gathering phase.

If you want a broader visualization toolkit (diff reviews, project recaps, slide decks, generic architecture diagrams), install cc-viz alongside this skill. They're designed to coexist.

## License

MIT
