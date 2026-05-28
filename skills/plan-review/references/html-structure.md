# HTML Structure

Complete spec for the plan-review document shell. One self-contained `.html` file.

## Document shell

```html
<!DOCTYPE html>
<html lang="en" data-theme="light" data-view="full" data-plain-english="on">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{PROJECT_NAME}} — Plan Review</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="{{FONT_LINK}}" rel="stylesheet">
  <style>/* all CSS inline — see css-patterns.md */</style>
</head>
<body>
  <header class="pr-header"><!-- title, view switcher, PE toggle, theme --></header>
  <div class="pr-layout">
    <aside class="pr-toc"><!-- sticky TOC + filter checkboxes --></aside>
    <main class="pr-main">
      <section class="pr-view" data-view="full"><!-- sections 1-9 --></section>
      <section class="pr-view" data-view="ux-flows" hidden><!-- UX Flows aggregate --></section>
    </main>
  </div>
  <footer class="pr-footer"><!-- timestamp, version --></footer>
  <script type="module"><!-- Mermaid + view switcher + PE toggle + theme --></script>
</body>
</html>
```

## Header

```html
<header class="pr-header">
  <div class="pr-header-left">
    <h1 class="pr-title">{{PROJECT_NAME}}</h1>
    <p class="pr-subtitle">{{MOTIVATION_ONE_LINER}}</p>
    <div class="pr-rationale-bar">
      <div class="pr-rationale-fill" style="--pct: {{RATIONALE_PCT}}"></div>
      <span>{{RATIONALE_DONE}}/{{RATIONALE_TOTAL}} changes have rationale</span>
    </div>
  </div>
  <div class="pr-header-right">
    <div class="pr-view-switcher" role="tablist">
      <button class="pr-view-btn pr-view-btn--active" data-view="full">Full review</button>
      <button class="pr-view-btn" data-view="ux-flows">UX Flows</button>
    </div>
    <button class="pr-pe-toggle" aria-pressed="true">
      <span class="pr-pe-dot"></span>Plain English: <strong>ON</strong>
    </button>
    <button class="pr-theme-toggle" title="Toggle theme">◐</button>
  </div>
</header>
```

## Full Review view

Two-column layout: sticky TOC left, main scrolling content right.

### TOC

```html
<aside class="pr-toc">
  <ol class="pr-toc-list">
    <li><a href="#s1">1. Plan summary</a></li>
    <li><a href="#s2">2. Impact dashboard</a></li>
    <li><a href="#s3">3. Current architecture</a></li>
    <li><a href="#s4">4. Planned architecture</a></li>
    <li><a href="#s5">5. Change-by-change</a></li>
    <li><a href="#s6">6. Ripple analysis</a></li>
    <li><a href="#s7">7. Risk assessment</a></li>
    <li><a href="#s8">8. Plan review</a></li>
    <li><a href="#s9">9. Understanding gaps</a></li>
  </ol>
  <hr class="pr-toc-sep">
  <fieldset class="pr-filters">
    <legend>Filter</legend>
    <label><input type="checkbox" data-filter="ux-only"> Show only UX changes</label>
    <label><input type="checkbox" data-filter="risk-only"> Show only risks</label>
  </fieldset>
</aside>
```

Sticky on desktop (`position: sticky; top: 0`). On mobile, collapse into a horizontal scrollable bar.

### Section 1 — Plan summary (hero depth)

```html
<section id="s1" class="pr-section pr-section--hero">
  <h2>Plan summary</h2>
  <div class="pr-pe-callout">
    <span class="pr-pe-label">In plain terms</span>
    <p>{{PLAIN_ENGLISH_SUMMARY}}</p>
  </div>
  <div class="pr-intuition">
    <h3>The intuition</h3>
    <p>{{2_OR_3_SENTENCE_ESSENCE}}</p>
  </div>
  <div class="pr-scope">
    <span>Scope:</span>
    <strong>{{N_MODIFIED}} modified</strong> ·
    <strong>{{N_CREATED}} new</strong> ·
    <strong>{{N_DELETED}} deleted</strong> ·
    <strong>+{{LINES_ADDED}}/-{{LINES_REMOVED}} lines</strong> ·
    <strong>{{N_NEW_TESTS}} new tests</strong>
  </div>
</section>
```

### Section 2 — Impact dashboard

```html
<section id="s2" class="pr-section">
  <h2>Impact dashboard</h2>
  <div class="pr-grid pr-grid--stats">
    <div class="pr-stat"><span class="pr-stat-value">{{N_MODIFIED}}</span><span class="pr-stat-label">Modify</span></div>
    <div class="pr-stat"><span class="pr-stat-value">{{N_CREATED}}</span><span class="pr-stat-label">Create</span></div>
    <div class="pr-stat"><span class="pr-stat-value">{{N_DELETED}}</span><span class="pr-stat-label">Delete</span></div>
    <div class="pr-stat"><span class="pr-stat-value">+{{LINES_ADDED}}/-{{LINES_REMOVED}}</span><span class="pr-stat-label">Lines</span></div>
  </div>
  <div class="pr-completeness">
    <span>Completeness:</span>
    <span class="pr-comp-item pr-comp-item--{{TESTS_STATUS}}">Tests</span>
    <span class="pr-comp-item pr-comp-item--{{DOCS_STATUS}}">Docs</span>
    <span class="pr-comp-item pr-comp-item--{{ROLLBACK_STATUS}}">Rollback</span>
  </div>
  {{#if has_phases}}
  <div class="pr-phase-pills">
    {{#each phase}}
    <span class="pr-phase pr-phase--{{status}}">{{id}} <small>{{title}}</small></span>
    {{/each}}
  </div>
  {{/if}}
</section>
```

### Sections 3-4 — Architecture diagrams

Wrap each Mermaid diagram in `.pr-mermaid-wrap` per `mermaid-theming.md`. Sections 3 and 4 use the same node names and layout direction so the visual diff is obvious.

### Section 5 — Change-by-change

```html
<section id="s5" class="pr-section">
  <h2>Change-by-change</h2>
  {{#each change}}
  <details class="pr-change" data-ux="{{is_ux}}" data-risk="{{has_risk}}" open>
    <summary>
      <span class="pr-change-id">{{n}}</span>
      <span class="pr-change-file">{{file}}</span>
      <span class="pr-change-title">{{title}}</span>
      {{#if is_ux}}<span class="pr-tag pr-tag--ux">UX</span>{{/if}}
      {{#if missing_rationale}}<span class="pr-tag pr-tag--gap">⚑ missing rationale</span>{{/if}}
    </summary>
    <div class="pr-change-body">
      <div class="pr-diff">
        <div class="pr-diff-side pr-diff-side--current">
          <h4>Current</h4>
          <pre><code>{{current_snippet}}</code></pre>
        </div>
        <div class="pr-diff-side pr-diff-side--planned">
          <h4>Planned</h4>
          <pre><code>{{planned_snippet}}</code></pre>
        </div>
      </div>
      <div class="pr-pe-callout">
        <span class="pr-pe-label">In plain terms</span>
        <p>{{plain_english}}</p>
      </div>
      {{#if is_ux}}
      <div class="pr-mockup-pair">
        <div class="pr-mockup">
          <h5>Before</h5>
          <pre class="pr-mockup-art">{{ascii_before}}</pre>
        </div>
        <div class="pr-mockup">
          <h5>After</h5>
          <pre class="pr-mockup-art">{{ascii_after}}</pre>
        </div>
      </div>
      {{/if}}
      <div class="pr-rationale">
        <strong>Rationale:</strong> {{rationale_or_missing_flag}}
      </div>
    </div>
  </details>
  {{/each}}
</section>
```

### Sections 6-9

Follow the spec in `commands/plan-review.md`. Section 6 uses `<details>` collapsed by default. Section 7 includes a section-level Plain English callout. Section 8 uses four cards with semantic left-border colors. Section 9 has a progress bar for rationale coverage.

## UX Flows view

```html
<section class="pr-view" data-view="ux-flows" hidden>
  <div class="pr-uxflows-header">
    <h2>UX Flows — every screen the plan touches</h2>
    <p>{{N_SCREENS}} screens affected · before/after side-by-side</p>
    <nav class="pr-uxflows-nav">
      {{#each screen}}
      <a href="#screen-{{slug}}">{{name}}</a>
      {{/each}}
    </nav>
  </div>
  {{#each screen}}
  <article id="screen-{{slug}}" class="pr-uxscreen">
    <h3>{{n}}. {{name}}</h3>
    <p class="pr-uxscreen-from">From: {{#each touched_by_change}}<a href="#change-{{id}}">Change {{id}}</a>{{/each}}</p>
    <div class="pr-mockup-pair pr-mockup-pair--large">
      <div class="pr-mockup">
        <h5>Before</h5>
        <pre class="pr-mockup-art">{{ascii_before}}</pre>
      </div>
      <div class="pr-mockup">
        <h5>After</h5>
        <pre class="pr-mockup-art">{{ascii_after}}</pre>
      </div>
    </div>
    <div class="pr-uxscreen-summary">
      <h4>What changes for the user</h4>
      <ul>
        {{#each user_facing_change}}
        <li>{{this}}</li>
        {{/each}}
      </ul>
    </div>
  </article>
  {{/each}}
</section>
```

## JS — view switcher, PE toggle, theme, filters, mermaid zoom

Inline. Module script. Three persisted prefs in `localStorage`:

```js
const STORE = {
  view: 'plan-review.view',          // 'full' | 'ux-flows'
  pe:   'plan-review.plain-english', // 'on' | 'off'
  theme:'plan-review.theme'          // 'light' | 'dark' | 'auto'
};
```

**Theme defaults to light. Always.** Set `data-theme="light"` on `<html>` at page load. Do NOT use `@media (prefers-color-scheme: dark)` — plan reviews are documents that get shared, and the default must be deterministic regardless of OS theme. Dark mode is opt-in only, via the header theme button. Persist in localStorage so the user's choice survives reloads on the same file URL.

View switcher toggles `<html data-view>`. CSS uses attribute selectors:
```css
html[data-view="full"]     [data-view="ux-flows"] { display: none; }
html[data-view="ux-flows"] [data-view="full"]     { display: none; }
```

PE toggle toggles `<html data-plain-english>`. CSS hides callouts when off:
```css
html[data-plain-english="off"] .pr-pe-callout { display: none; }
```

Filters use JS to toggle visibility on `.pr-change` elements based on their `data-ux` and `data-risk` attributes. Set `data-risk="true"` on any change that has a `⚑ missing rationale` flag, a high/medium severity risk card, or a discrepancy between plan claims and actual code. Both filters compose: a change must pass both active filters to remain visible.

Mermaid zoom controls per `mermaid-theming.md`.
