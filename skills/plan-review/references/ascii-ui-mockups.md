# ASCII UI Mockups

When a plan touches UX, render the affected screen as an ASCII wireframe so the reader sees what the user will experience, not just what the code looks like.

## When to render a mockup

Tag a change `[UX]` and draw a mockup if any of these are true:

- The change modifies a route, page component, or layout file (`app/`, `pages/`, `routes/`, `views/`).
- The change modifies a user-visible component (form, modal, button, header, nav, card grid).
- The change adds, removes, or relabels user-facing copy.
- The change alters validation messages, empty states, loading states, or error screens.

When in doubt, draw it. The mockup is opt-in via the `[UX]` tag — better to over-draw than miss a screen.

## Mapping changes to screens

Two relationships to handle:

1. **One change touches multiple screens.** Render an ASCII mockup for each screen inside that change's panel, stacked vertically with screen-name labels.
2. **Multiple changes touch the same screen.** In the **Full review** view, render the mockup once per code change (so each change panel is self-contained). In the **UX Flows** view, render the screen **once**, listing all the code changes that touch it as backlinks.

Track this with a `screens[]` array per change in your fact sheet:

```yaml
changes:
  - id: 1
    file: app/auth/login.tsx
    is_ux: true
    screens: ['sign-in']
  - id: 3
    file: components/RememberMeCheckbox.tsx
    is_ux: true
    screens: ['sign-in']
screens:
  - slug: sign-in
    name: Sign in
    touched_by: [1, 3]
    ascii_before: |
      ...
    ascii_after: |
      ...
    user_facing_changes:
      - New "remember me" checkbox extends session from 1 day → 30 days
```

## Drawing conventions

Use box-drawing characters: `┌ ─ ┬ ┐ │ ├ └ ┘ ┤ ┴ ╭ ╮ ╯ ╰`

```
┌──────────────────────────────────┐
│ Page title                       │
├──────────────────────────────────┤
│ [Field label]  [_____________]   │
│                                  │
│ [ Button →]                      │
└──────────────────────────────────┘
```

**Rules:**
- One mockup = one screen viewport. Don't try to show 4 screens in one diagram.
- Show only what the user sees. Don't draw component boundaries, prop names, or implementation details.
- Use `[ Button text ]` for buttons and `[___________]` for inputs.
- Use `(•)` for filled radio / `( )` for empty, `[x]` for checked / `[ ]` for unchecked checkbox, `▼` for dropdown indicators.
- Use `…` (single ellipsis character, not three dots) for truncation.
- Use `(N)` for badge counts: `Inbox (3)`.
- For loading states: `▰▰▰▱▱▱` or `Loading…`.
- For empty states: write the empty-state message verbatim.

**Sizing:**
- Width: 38-50 characters wide for inline (change panel) mockups so they fit side-by-side without horizontal scroll on standard viewports.
- Width: 60-80 characters for UX Flows view mockups (more room).
- Height: as tall as needed — page scrolls.

**Annotations** when something is changing:
- Wrap added elements with `+` markers: `[+ ☐ Remember me +]`
- Wrap removed elements with `-` markers: `[- Old link -]`
- Highlight changed text with `«»` brackets: `[«Sign in with email»]`

Use these sparingly — the side-by-side before/after layout already conveys the diff.

## Embedding in HTML

Wrap in `<pre class="pr-mockup-art">`. CSS in `css-patterns.md` handles monospace, whitespace preservation, and overflow.

```html
<div class="pr-mockup">
  <h5>Before</h5>
  <pre class="pr-mockup-art">┌──────────────────────────────────┐
│ Sign in                          │
│                                  │
│ Email     [__________________]   │
│ Password  [__________________]   │
│                                  │
│ [ Sign in → ]                    │
└──────────────────────────────────┘</pre>
</div>
```

**Indentation trap:** because `white-space: pre` preserves leading whitespace, the opening `<pre>` content must start immediately after the tag with no leading newline or spaces. Use template literals or write the mockup left-aligned in the source HTML.

## Speculative mockups

If you don't have enough information to draw an accurate before/after (e.g., the change touches an existing screen you can't fully inspect), mark the mockup as **speculative**:

```html
<div class="pr-mockup pr-mockup--speculative">
  <h5>Before <span class="pr-tag pr-tag--risk">speculative</span></h5>
  <pre class="pr-mockup-art">…best-guess sketch…</pre>
</div>
```

This signals to the reader that the mockup is your best interpretation, not a verified rendering. Better than skipping the mockup entirely — gives the reader something to react to.

## Don't draw if

- The change is purely backend (data layer, API logic, internal types) — even if the user *eventually* sees a difference, if no screen markup changes, skip the mockup. The Plain English callout can describe the user impact in prose.
- The change is to a component that has no visual representation (a context provider, a hook, a util function).
