# Component API, Accessibility, Theming & Rendering Hygiene

Deep reference for component-API and design-system work. Read when creating or refactoring component interfaces, tokens, or themes. Distilled from cmdk and ConnectKit; adapt to the repo's conventions.

## Respect the existing API style first

Before changing a component API, inspect surrounding components and repo conventions. Do **not** replace config objects, render props, compound components, or controlled state merely because another pattern is nicer in the abstract. Change the API only when there's a demonstrated problem: repeated edge-case flags, unstable identity, inaccessible defaults, duplicated behavior, impossible composition, excessive setup, a performance bottleneck, or inconsistent state ownership.

## Compound components over config arrays or render props

When you *do* design a flexible component, expose behavior as composed children, not a `data` array or an `onRender` prop. Config-object APIs inevitably grow edge-case fields (`image`, `subtitle`, `hideWhen…`); render props collapse into centralized if-else chains. Composition gives the consumer full control of rendering while the library owns behavior:

```jsx
// Consumer composes and controls rendering; library filters/sorts/navigates
<Command>
  <Command.Input />
  <Command.List>
    <Command.Group heading="Recent"><Command.Item>…</Command.Item></Command.Group>
  </Command.List>
</Command>
```

## Let the DOM be the source of truth (for lists/menus)

For a filtering list/menu, keep every item mounted in the tree and let each remove *itself* from the DOM based on state; read ordering and selection back from the live DOM. This lets composition work (you can filter items nested inside opaque child components) and keeps item order = render order = DOM order with no index bookkeeping. Filter + sort **synchronously** on keystroke (never flash an unfiltered frame), but batch bulk lifecycle churn (mounting thousands of items) into one deferred pass keyed by intent, so you don't filter/sort/emit thousands of times.

## Stable identity

Track selected or animated entities by **stable identifiers**, never by list index, when filtering, sorting, reordering, or concurrent rendering can occur. An index-based highlight jumps to a stale row as the list changes underneath it; a value key survives filtering and re-sorting. (Index tracking also breaks under Strict-Mode double effects.)

## Strong defaults

A component should work well with minimal configuration: accessible roles and labels, keyboard behavior, reduced-motion behavior, coherent easing/duration, sensible focus management, stable identity, interruption handling, predictable controlled/uncontrolled behavior. Good defaults matter more than many options — most consumers never customize.

## Accessibility edge cases

Verify, when relevant:

- **IME composition** — ignore navigation keys while composing (`e.nativeEvent.isComposing || e.keyCode === 229`) or you break CJK/Japanese/Korean input.
- **`aria-activedescendant`** (or platform equivalent) — for a combobox/listbox, focus stays in the input while the "active" option is announced; the screen reader tracks selection without moving DOM focus. Wire `role="combobox"` + `aria-expanded` + `aria-controls` on the input and a `role="listbox"` / `role="option"` (`aria-selected`) tree.
- **Focus retention** across transitions; **scroll-into-view** for keyboard selection (`scrollIntoView({ block: 'nearest' })`, plus the group heading when selecting the first item in a group; pair with `scroll-padding` so items don't hide under sticky chrome).
- **Pointer = selection parity** — `onPointerMove` selects the hovered item so mouse and keyboard highlight are unified (no dead zone).
- **Touch hover false positives** — gate `:hover` effects behind a pointer/hover media query.
- **Disabled and busy** states; **announcements** for authoritative values (`role="progressbar"` with `aria-valuenow/min/max` for async work).

## Theming and token systems

**Derive visual roles, not semantic meaning.** It's good to derive a whole palette from one accent (fill stops, grid, glow, badge, tooltip — light/dark). But **never** derive success / warning / danger / destructive meaning from brand color — semantic colors must stay fixed, because meaning must not bend to aesthetics.

**Role-based fallback chains.** Every visual property reads through a chain `specific → role → body`, so a consumer can restyle everything with one accent or surgically override one element with zero component changes:

```css
color: var(--component-primary-color, var(--body-color));
--hover-background: var(--component-primary-hover-bg, var(--background));
```

The design system's surface area is its *token list*, not its component tree. When geometry (`--radius`), shadow, gradient, and font are all tokens, a single theme can change the product's entire personality (up to a full OS re-skin) without new markup.

**Progressive enhancement** (only where it degrades safely, never as a dependency of the core interaction): wide-gamut P3 color (emit hex, then a P3 value inside `@supports (color: color(display-p3 1 1 1))`); blur/translucency (with reduced-transparency and contrast variants); advanced masks; view transitions; high-refresh rendering; seed-based default avatars derived from a hash so every id gets a stable, attractive identity with no image load.

## Squircles, not rounded rectangles

CSS `border-radius` produces a circular-arc corner with a visible curvature discontinuity where the arc meets the edge. For app-icon-grade smoothness, use a real superellipse (continuous curvature) — a hand-authored SVG path or a squircle utility. Drawing it as an SVG cutout (an outer rect minus an inner superellipse, filled with the background) also lets effects sweep *behind* the mask.

## Shipping discipline

Prefer: no new runtime dependency when unnecessary; tree-shakeable modules (`sideEffects: false`); no per-component copy of shared motion constants; minimal adoption friction (ideally drop-in with no required config or CSS import); documentation and examples for shared primitives; bundle cost appropriate to the feature. Package size is a feature — the less friction to adopt, the more it gets used.

## Safari & sub-pixel hygiene

Browser workarounds are conditional, applied to *verified* affected elements — not universal rules. Use `backface-visibility: hidden`, temporary `will-change: transform`, `translateZ(0)`, or slight image scaling (`scale(1.01)` to kill a 1px seam on rounded images) **only when the defect is observed** and the fix verified. Do **not** apply `will-change` broadly or permanently — it raises memory and compositing cost. Hide auto-fit text (`visibility: hidden`) until measured to avoid a reflow flash. Document any non-obvious workaround in a code comment.
