---
description: Generate a visual HTML plan review — current codebase state vs. proposed implementation plan, with ASCII UI mockups for UX changes and a Plain English translation layer.
---

Load the `plan-review` skill, then generate a comprehensive visual plan review as a self-contained HTML page comparing the current codebase against a proposed implementation plan.

Follow the `plan-review` skill workflow. Read `skills/plan-review/SKILL.md`, then the references it points to, before generating. Use a constrained aesthetic (blueprint, editorial, paper/ink, or monochrome terminal). Vary fonts and palette from any recent diagrams.

**Inputs:**
- Plan file: `$1` (path to a markdown plan, spec, or RFC document)
- Codebase: `$2` if provided, otherwise the current working directory

**Data gathering phase** — read and cross-reference these before generating:

1. **Read the plan file in full.** Extract:
   - The problem statement and motivation
   - Each proposed change (files to modify, new files, deletions)
   - Rejected alternatives and their reasoning
   - Any explicit scope boundaries or non-goals

2. **Read every file the plan references.** For each file mentioned in the plan, read the current version in full. Also read files that import or depend on those files — the plan may not mention all ripple effects.

3. **Map the blast radius.** From the codebase, identify:
   - What imports/requires the files being changed (grep for import paths)
   - What tests exist for the affected files (look for corresponding `.test.*` / `.spec.*` files)
   - Config files, types, or schemas that might need updates
   - Public API surface that callers depend on

4. **Tag UX-touching changes.** Any change that modifies a route, page component, screen, form, modal, or anything user-facing gets a `[UX]` tag. Heuristics: file paths containing `app/`, `pages/`, `components/`, `routes/`, `views/`, or filenames ending in `.tsx`/`.vue`/`.svelte` that render user-visible markup. Err on the side of tagging — the ASCII mockup is opt-in via the data anyway.

5. **Cross-reference plan vs. code.** For each change the plan proposes, verify:
   - Does the file/function/type the plan references actually exist in the current code?
   - Does the plan's description of current behavior match what the code actually does?
   - Are there implicit assumptions about code structure that don't hold?

**Verification checkpoint** — before generating HTML, produce a structured fact sheet of every claim you will present in the review:
- Every quantitative figure: file counts, estimated lines, function counts, test counts
- Every function, type, and module name you will reference from both the plan and the codebase
- Every behavior description: what the code currently does vs. what the plan proposes
- For each, cite the source: the plan section or the file:line where you read it
Verify each claim against the code and the plan. If something cannot be verified, mark it as uncertain rather than stating it as fact. This fact sheet is your source of truth during HTML generation — do not deviate from it.

**For UX changes — produce ASCII mockups before writing HTML.** For each `[UX]` change, sketch a before/after ASCII wireframe of the affected screen. Use box-drawing characters `┌ ─ ┬ ┐ │ ├ └ ┘`. The mockup is a *user-facing* artifact: show what the user sees, not the component tree. Read `skills/plan-review/references/ascii-ui-mockups.md` for conventions, sizing rules, and the wrapper pattern for embedding ASCII inside HTML without breaking the layout.

**Page structure** — the output HTML must include:

### Top of page — view switcher

Two views in the same HTML file, switched via tab buttons in the header:
1. **Full review** (default) — sections 1-9 below
2. **UX Flows** — aggregated view of every before/after ASCII mockup, grouped by screen (not by code change). Each screen lists which code changes touch it, with backlinks. Includes a "What changes for the user" plain-English summary per screen.

State (selected view) persists in `localStorage`.

### Top of page — Plain English toggle

A header control labeled **`Plain English: ON`** (or `OFF`). It hides or shows every "In plain terms" callout on the page in one click. State persists in `localStorage`. Default is ON.

### Sections (Full review view)

1. **Plan summary** — *hero depth*. Lead with the *intuition*: what problem does this plan solve, and what's the core insight behind the approach? Then the scope: how many files touched, estimated scale of changes, new modules or tests planned. A reader who only sees this section should understand the plan's essence. Include a **Plain English callout** at the top (see `references/plain-english.md`).

2. **Impact dashboard** — files to modify, files to create, files to delete, estimated lines added/removed, new test files planned, dependencies affected. Include a **completeness** indicator: whether the plan covers tests (green/red), docs updates (green/yellow/red), and migration/rollback (green/grey for N/A). If the plan defines phases or waves, render **phase pills** with status (`todo`/`in-progress`/`done`/`blocked`).

3. **Current architecture** — Mermaid diagram of how the affected subsystem works *today*. Focus only on the parts the plan touches. Wrap in `.pr-mermaid-wrap` with zoom controls (+/−/reset buttons), Ctrl/Cmd+scroll zoom, and click-and-drag panning. See `references/css-patterns.md` for the full pattern.

4. **Planned architecture** — Mermaid diagram of how the subsystem will work *after* the plan is implemented. Use the same node names and layout direction as section 3 so the differences are visually obvious. Highlight new nodes with a glow or accent border, removed nodes with strikethrough or reduced opacity, changed edges with a different stroke color.

5. **Change-by-change breakdown** — for each change in the plan, a panel containing:
   - **Side-by-side code:** current (left) vs. planned (right). Apply `min-width: 0` on grid/flex children and `overflow-wrap: break-word` on panels.
   - **Plain English callout** below the code, before the mockup. Lead with the trade-off in non-technical terms.
   - **`[UX]` ASCII mockup** if the change touches UX. Before/after side-by-side, wrapped per `references/ascii-ui-mockups.md`.
   - **Rationale:** extract _why_ the plan chose this approach. Pull from the plan's reasoning, rejected alternatives section, or inline justifications. If the plan says _what_ but not _why_, render a **⚑ Missing rationale** flag.
   - Flag any discrepancies where the plan's description of current behavior doesn't match the actual code.

6. **Dependency & ripple analysis** — *compact, collapsible*. What other code depends on the files being changed. Table or Mermaid graph showing callers, importers, and downstream effects. Color-code: covered by plan (green), not mentioned but likely affected (amber), definitely missed (red).

7. **Risk assessment** — styled cards for:
   - **Edge cases** the plan doesn't address
   - **Assumptions** the plan makes that should be verified
   - **Ordering risks** if changes need to be applied in a specific sequence
   - **Rollback complexity**
   - **Cognitive complexity** — areas where the plan introduces non-obvious coupling, action-at-a-distance behavior, or invariants that exist only in the developer's memory. Each flag gets a brief mitigation suggestion.
   - Each risk gets a severity indicator (low/medium/high).
   - Include a section-level **Plain English callout** summarizing the biggest risk.

8. **Plan review — Good / Bad / Ugly / Questions:**
   - **Good**: Solid design decisions, things the plan gets right.
   - **Bad**: Gaps — missing files, unaddressed edge cases, incorrect assumptions.
   - **Ugly**: Subtle concerns — complexity, maintenance burden, things that work initially but bite at scale.
   - **Questions**: Ambiguities that need the plan author's clarification.
   - Styled cards with green/red/amber/blue left-border accents. Reference specific plan sections and code files. If nothing to flag in a category, say "None found".

9. **Understanding gaps** — closing dashboard:
   - Count of changes with clear rationale vs. missing rationale (bar chart or progress indicator)
   - List of cognitive complexity flags with severity
   - Explicit recommendation: "Before implementing, document X and Y"
   - Makes cognitive debt visible *before* the work starts.

### Sections (UX Flows view)

Visible only when "UX Flows" tab is active. Renders:

- A **jump-to nav** at the top listing every screen touched.
- For each screen: large heading, before/after ASCII mockups side-by-side, a "What changes for the user" paragraph (plain English, no jargon), and backlinks to the code Changes that touch this screen (e.g., "Linked changes: Change 1 ↗  Change 3 ↗").

**Visual hierarchy**: Sections 1-4 dominate the viewport on load (hero depth for summary, elevated for architecture diagrams). Sections 6+ are reference material — flat or recessed depth, compact layout, collapsible where appropriate.

**Output:**
- Single self-contained HTML file. All CSS and JS inline. No external dependencies except CDN-loaded Mermaid (per `references/mermaid-theming.md`) and Google Fonts.
- Write to `~/.agent/diagrams/<project-slug>-plan-review-<timestamp>.html` (create the directory if missing) and open in the browser via `open <path>`.
- Tell the user the path and what's in the page.

Ultrathink.

$@
