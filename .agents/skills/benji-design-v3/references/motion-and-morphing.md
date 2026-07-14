# Motion, Morphing & Layout Continuity

Deep reference for transitions, multi-step surfaces, navigation, and gestures. Read when building or refining how the UI moves between states. Distilled from Family and ConnectKit; adapt to your stack.

## Enter, exit, and presence

**Express origin and destination.** When continuity matters, elements enter from a plausible source and leave toward a plausible destination; reversible transitions follow compatible paths; anchored surfaces (popovers, menus) transform *from their trigger*, not from center. Modals with no spatial trigger may stay centered.

**Commit removal after the visual exit — when safe.** For non-critical, trackable objects: begin the exit state → preserve layout if needed → advance/complete the exit → commit removal. Do **not** delay destructive state, security state, network cancellation, or business logic to wait for decoration. Logical removal and visual exit may be decoupled.

**Asymmetric timing.** Enter and exit need not match. A common, good pattern: outgoing content begins first; incoming content waits briefly, then plays slightly shorter; the exiting element is positioned independently so layout can morph beneath it. This avoids both the empty flash (enter too late) and the muddy double-image (full overlap).

```js
const D = 0.22
// enter: waits 25% of the window, then plays for 75%
animate: { duration: D * 0.75, delay: D * 0.25, ease: [0.26, 0.08, 0.25, 1] }
// exit: full duration, pinned so it doesn't hold layout open
exit:   { duration: D, position: 'absolute', left: '50%', x: '-50%', ease: [0.26, 0.08, 0.25, 1] }
```

## Container morphing (measure-then-transition)

The single most valuable trick for step-based surfaces (dialogs, trays, inspectors, wizards): don't animate layout with a motion library. **Measure each step's content outside the frame loop, push its size into explicit dimensions / CSS custom properties, and let a plain transition morph them** while content crossfades independently on top.

```js
// Measure the new content on mount
const contentRef = useCallback((node) => {
  if (node) setDimensions({ width: `${node.offsetWidth}px`, height: `${node.offsetHeight}px` })
}, [stepId])
// Apply as CSS variables on the container
<Container style={{ '--w': dimensions.width, '--h': dimensions.height }} />
```
```css
.surface {
  width: var(--w); height: var(--h);
  transition: width 200ms var(--ease-standard), height 200ms var(--ease-standard);
}
```

- Remeasure after data, locale, font, or viewport changes to prevent clipping and stale measurements.
- **Honesty:** animating width/height can trigger layout. It's fine for small, contained, measured surfaces — but do **not** describe it as compositor-only. If profiling shows jank, prefer the repo's layout-animation primitive.
- Never `transition: all` — name exact properties.

## Depth-aware navigation

When navigation has hierarchy, encode direction so users feel where they are:

```js
enter = currentDepth > prevDepth ? 'scale-up-in'   /* child grows in from ~scale(0.85) */ : 'fade-in'
exit  = currentDepth < prevDepth ? 'scale-down-out' /* parent shrinks in from ~scale(1.1) */ : 'fade-out'
```

Lateral peers use a lateral/neutral transition. Reduced motion preserves hierarchy *without* spatial travel. Do not apply arbitrary zoom to flat navigation.

## Shared-element continuity

When the same entity appears on two screens, animate it *between* them (a card that morphs into a full screen; a graphic that flies between onboarding slides via a shared `layoutId`; a button that glides across a tray and morphs into it). Use when the repo has a reliable shared-layout mechanism and focus/accessibility stay stable. Avoid when it causes complex layering, stale snapshots, or inaccessible duplicate content.

## Text and icon morphing (optional craft)

Use only when they clarify state and stay readable.

**Text.** When a label changes meaning (e.g. "Continue" → "Confirm"), morph the shared letters and transition only the differing ones. It makes the change both noticeable and smooth, reinforcing awareness of what the action did.

**Icons.** Rotate when two states are the same shape at a different angle (arrow-right vs arrow-down): *"When you morph coordinates, the lines bend and warp. When you rotate, it just works."* Morph coordinates only when the geometry truly changes. Preserve stroke weight and visual center. Do **not** force every icon into one primitive model unless the system was deliberately designed that way (Benji's 21-icon set used a fixed "three lines + rotation groups" structure precisely because it was designed end-to-end for morphing — that is a system-level decision, not a default).

## Easing vocabulary and springs

**Reuse a small named vocabulary** (search the repo first). Representative roles and curves from ConnectKit:

| Curve | Role |
| --- | --- |
| `cubic-bezier(0.26, 0.08, 0.25, 1)` | Content crossfade — soft ease-out |
| `cubic-bezier(0.16, 1, 0.3, 1)` | Emphasized reveal — expo-out |
| `cubic-bezier(0.25, 1, 0.5, 1)` | Button / state change — quart-out |
| `cubic-bezier(0.76, 0, 0.24, 1)` | Spatial move / reorder — symmetric ease-in-out |
| `cubic-bezier(0.175, 0.885, 0.32, 0.98)` | Element entrance — near ease-out-back |
| `cubic-bezier(0.15, 1.15, 0.6, 1)` | Tactile overshoot only (sheet up, checkmark) |

Reserve overshoot (a control value **> 1**) for physical/tactile moments; never for ordinary content or navigation.

**Springs — for the physical few.** Use for drag/swipe, momentum-driven dismissal, interruptible position changes, gesture retargeting, and the one intentionally-weighted object. Start critically damped or low-bounce; add overshoot only when the gesture/personality justifies it.

For gesture-driven motion: track the current presentation position; capture release velocity; hand velocity into the spring; project momentum when choosing a destination; retarget from the current value+velocity (never restart from zero); apply rubber-band resistance beyond bounds; use pointer capture; guard multi-touch and gesture ambiguity.

## Two input models, two systems

The motion that feels right depends on the input device — don't force one system onto both:

- **Pointer (desktop):** cards scale/crossfade in place; hover can select.
- **Touch:** a bottom sheet slides up from the thumb with a slight overshoot (`~300ms`, small delay); carousels become native scroll-snap; hit-areas get transparent padding (`scale(1.4)`). Gate hover effects behind a pointer/hover media query so taps don't trigger false hovers.
