# CSS Patterns

Reusable patterns for the plan-review page. Inline all CSS in `<style>`. Two themes (light/dark), defined via CSS custom properties.

## Theme tokens

```css
:root {
  --font-body: 'Bricolage Grotesque', system-ui, sans-serif;
  --font-mono: 'Fragment Mono', 'SF Mono', Consolas, monospace;
  --font-display: 'Instrument Serif', Georgia, serif;

  --bg: #f8f9fa;
  --surface: #ffffff;
  --surface-elevated: #ffffff;
  --surface-recessed: #f1f3f5;
  --border: rgba(0, 0, 0, 0.08);
  --border-bright: rgba(0, 0, 0, 0.15);
  --text: #1a1a2e;
  --text-dim: #6b7280;

  /* Plan-review semantic colors */
  --state-current: #475569;        /* slate — today */
  --state-current-dim: rgba(71, 85, 105, 0.08);
  --state-planned: #059669;        /* emerald — proposed */
  --state-planned-dim: rgba(5, 150, 105, 0.08);
  --state-risk: #d97706;           /* amber — warning */
  --state-risk-dim: rgba(217, 119, 6, 0.08);
  --state-gap: #dc2626;            /* red — missed/broken */
  --state-gap-dim: rgba(220, 38, 38, 0.08);
  --state-pe: rgba(8, 145, 178, 0.06);   /* soft tint for Plain English */
  --state-pe-border: rgba(8, 145, 178, 0.25);
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #0d1117;
    --surface: #161b22;
    --surface-elevated: #1c2333;
    --surface-recessed: #0a0e14;
    --border: rgba(255, 255, 255, 0.06);
    --border-bright: rgba(255, 255, 255, 0.12);
    --text: #e6edf3;
    --text-dim: #8b949e;

    --state-current: #94a3b8;
    --state-current-dim: rgba(148, 163, 184, 0.12);
    --state-planned: #34d399;
    --state-planned-dim: rgba(52, 211, 153, 0.12);
    --state-risk: #fbbf24;
    --state-risk-dim: rgba(251, 191, 36, 0.12);
    --state-gap: #f87171;
    --state-gap-dim: rgba(248, 113, 113, 0.12);
    --state-pe: rgba(34, 211, 238, 0.08);
    --state-pe-border: rgba(34, 211, 238, 0.3);
  }
}
```

## Recommended font pairings

Pick one. Vary from recent generations. Forbidden as `--font-body`: Inter, Roboto, Arial, Helvetica, system-ui alone.

- DM Sans + Fira Code (technical, precise)
- Instrument Serif + JetBrains Mono (editorial, refined)
- IBM Plex Sans + IBM Plex Mono (reliable, readable)
- Bricolage Grotesque + Fragment Mono (bold, characterful)
- Plus Jakarta Sans + Azeret Mono (rounded, approachable)

Load via Google Fonts `<link>`. Include system fallback.

## Depth system

Three depths for visual hierarchy:

```css
.pr-section          { background: var(--surface); border: 1px solid var(--border); border-radius: 12px; padding: 24px; }
.pr-section--hero    { background: var(--surface-elevated); padding: 40px; font-size: 1.05em; box-shadow: 0 8px 32px rgba(0,0,0,0.04); }
.pr-section--flat    { background: transparent; border: none; padding: 16px 0; }
.pr-section--recessed{ background: var(--surface-recessed); }
```

Section 1 (Plan summary) gets `--hero`. Sections 3, 4, 5 are default. Section 6 (Ripple) is `--flat` and `<details>`. Section 9 (Understanding gaps) is `--recessed`.

## Plain English callout

```css
.pr-pe-callout {
  background: var(--state-pe);
  border-left: 3px solid var(--state-pe-border);
  border-radius: 6px;
  padding: 14px 18px;
  margin: 16px 0;
  max-width: 65ch;
}
.pr-pe-label {
  display: inline-block;
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--text-dim);
  margin-bottom: 6px;
}
.pr-pe-callout p {
  font-size: 15px;
  line-height: 1.6;
  color: var(--text);
}
html[data-plain-english="off"] .pr-pe-callout { display: none; }
```

## Status colors

```css
.pr-tag             { display: inline-block; font-size: 11px; font-family: var(--font-mono); padding: 2px 8px; border-radius: 4px; text-transform: uppercase; letter-spacing: 0.05em; }
.pr-tag--ux         { background: var(--state-planned-dim); color: var(--state-planned); border: 1px solid var(--state-planned); }
.pr-tag--gap        { background: var(--state-gap-dim); color: var(--state-gap); border: 1px solid var(--state-gap); }
.pr-tag--risk       { background: var(--state-risk-dim); color: var(--state-risk); border: 1px solid var(--state-risk); }

.pr-comp-item--green { color: var(--state-planned); }
.pr-comp-item--amber { color: var(--state-risk); }
.pr-comp-item--red   { color: var(--state-gap); }

.pr-phase--done        { background: var(--state-planned-dim); color: var(--state-planned); }
.pr-phase--in-progress { background: var(--state-pe); color: var(--text); animation: pulse 2s infinite; }
.pr-phase--todo        { background: var(--surface-recessed); color: var(--text-dim); }
.pr-phase--blocked     { background: var(--state-gap-dim); color: var(--state-gap); }
```

## Change diff (side-by-side)

```css
.pr-diff {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 16px;
}
.pr-diff-side {
  min-width: 0;             /* prevents grid blowout */
  overflow-wrap: break-word;
  background: var(--surface-recessed);
  border-radius: 8px;
  padding: 16px;
}
.pr-diff-side--current  { border-left: 3px solid var(--state-current); }
.pr-diff-side--planned  { border-left: 3px solid var(--state-planned); }
.pr-diff-side h4        { font-size: 12px; text-transform: uppercase; letter-spacing: 0.08em; color: var(--text-dim); margin-bottom: 8px; }
.pr-diff-side pre       { font-family: var(--font-mono); font-size: 13px; line-height: 1.5; overflow-x: auto; }

@media (max-width: 768px) {
  .pr-diff { grid-template-columns: 1fr; }
}
```

## ASCII mockup pair

See `ascii-ui-mockups.md` for the full spec. Key rule: monospace, preserve whitespace, prevent line wrap.

```css
.pr-mockup-pair {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin: 20px 0;
}
.pr-mockup {
  background: var(--surface-recessed);
  border-radius: 8px;
  padding: 16px;
}
.pr-mockup h5 {
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--text-dim);
  margin-bottom: 8px;
}
.pr-mockup-art {
  font-family: var(--font-mono);
  font-size: 12px;
  line-height: 1.35;
  white-space: pre;
  overflow-x: auto;
  color: var(--text);
}
.pr-mockup-pair--large .pr-mockup-art { font-size: 13px; line-height: 1.4; }

@media (max-width: 768px) {
  .pr-mockup-pair { grid-template-columns: 1fr; }
}
```

## Layout

```css
.pr-layout {
  display: grid;
  grid-template-columns: 220px 1fr;
  gap: 32px;
  max-width: 1400px;
  margin: 0 auto;
  padding: 24px;
}
.pr-toc {
  position: sticky;
  top: 24px;
  align-self: start;
  font-size: 14px;
}
@media (max-width: 900px) {
  .pr-layout { grid-template-columns: 1fr; }
  .pr-toc { position: static; overflow-x: auto; white-space: nowrap; }
  .pr-toc-list { display: flex; gap: 12px; }
}
```

## Mermaid zoom controls

Wrap every Mermaid in `.pr-mermaid-wrap`:

```html
<div class="pr-mermaid-wrap">
  <div class="pr-mermaid-controls">
    <button data-zoom="in">+</button>
    <button data-zoom="out">−</button>
    <button data-zoom="reset">⟳</button>
  </div>
  <div class="mermaid"> graph TD; ... </div>
</div>
```

```css
.pr-mermaid-wrap {
  position: relative;
  background: var(--surface-recessed);
  border: 1px solid var(--border);
  border-radius: 8px;
  overflow: hidden;
  cursor: grab;
  min-height: 360px;
}
.pr-mermaid-wrap.dragging { cursor: grabbing; }
.pr-mermaid-controls {
  position: absolute;
  top: 12px;
  right: 12px;
  display: flex;
  gap: 4px;
  z-index: 10;
}
.pr-mermaid-controls button {
  width: 32px; height: 32px;
  border-radius: 6px;
  background: var(--surface);
  border: 1px solid var(--border-bright);
  font-family: var(--font-mono);
  cursor: pointer;
}
.mermaid { transform-origin: center; transition: transform 0.2s; }
```

JS pattern (inline):

```js
document.querySelectorAll('.pr-mermaid-wrap').forEach(wrap => {
  let scale = 1, tx = 0, ty = 0, dragging = false, sx = 0, sy = 0;
  const svg = wrap.querySelector('.mermaid');
  const apply = () => svg.style.transform = `translate(${tx}px, ${ty}px) scale(${scale})`;
  wrap.querySelector('[data-zoom="in"]').onclick  = () => { scale *= 1.2; apply(); };
  wrap.querySelector('[data-zoom="out"]').onclick = () => { scale /= 1.2; apply(); };
  wrap.querySelector('[data-zoom="reset"]').onclick = () => { scale = 1; tx = 0; ty = 0; apply(); };
  wrap.addEventListener('wheel', e => {
    if (!(e.ctrlKey || e.metaKey)) return;
    e.preventDefault();
    scale *= e.deltaY < 0 ? 1.1 : 0.9;
    apply();
  }, { passive: false });
  wrap.addEventListener('mousedown', e => { dragging = true; sx = e.clientX - tx; sy = e.clientY - ty; wrap.classList.add('dragging'); });
  window.addEventListener('mousemove', e => { if (!dragging) return; tx = e.clientX - sx; ty = e.clientY - sy; apply(); });
  window.addEventListener('mouseup', () => { dragging = false; wrap.classList.remove('dragging'); });
});
```

## Overflow protection

Three rules that prevent grid/flex blowout when code snippets or ASCII mockups exceed their container:

```css
.pr-diff, .pr-mockup-pair, .pr-change-body { min-width: 0; }
.pr-diff-side, .pr-mockup, .pr-uxscreen { overflow-wrap: break-word; }
.pr-diff-side pre, .pr-mockup-art { overflow-x: auto; }
```

Never use `display: flex` on `<li>` for marker characters — use absolute positioning instead.

## Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  * { transition: none !important; animation: none !important; }
}
```

## Print

```css
@media print {
  .pr-toc, .pr-header-right, .pr-mermaid-controls { display: none; }
  .pr-section { break-inside: avoid; box-shadow: none; }
}
```
