# Fluidity Engine, Performance & Correctness

Deep reference for continuously-changing visual values, high-frequency render loops, canvas, and streaming/real-time data. Read when the task involves charts, counters, ranges, gesture tracking, or any 60fps hot path. Everything here is distilled from Liveline (Benji's canvas charting library); adapt values to your stack.

## The core primitive: chase a target, never render raw input

The mechanical heart of "fluid" is one idea applied uniformly: **never render the raw incoming value — keep a persistent displayed value that eases toward a target every frame.** Value, axis range, badge width, color, scroll position, opacity, panel height — all of them chase a target. This is what turns discrete, jumpy input into continuous motion.

```js
// Authoritative incoming value — this stays exact and business-owned
targetRef.current = incomingValue

// Presentation value follows the target inside the frame loop
displayRef.current = lerp(displayRef.current, targetRef.current, speed, dt)
```

Do not write the interpolated presentation value back into business/logical state. Truth and presentation are separate.

### Move the whole as one — shared interpolation is what "breathing" means

The cohesion isn't per-widget; it's that **every animated quantity eases toward its target with the *same* interpolation**. Value, axis range, badge width, tick labels, gridline positions, glow opacity — all driven by the same `lerp` with the same `speed` and the same measured `dt`, in one loop. When they share the interpolation, a change ripples through the whole surface at one tempo and it reads as a single organism reflowing, not a cluster of independent parts each updating on its own clock.

> "That's why it feels like one thing breathing rather than a bunch of parts updating independently."

Practically: keep one frame loop, thread one `dt` into every lerp it drives, and derive dependent geometry (label positions, badge width) from the *displayed* values, not the raw targets — so nothing races ahead of the value it annotates.

## Frame-rate-independent interpolation

A naive `current += (target - current) * 0.08` converges twice as fast at 120fps as at 60fps — the *personality* of the motion drifts with the hardware. Convert the per-frame fraction into a continuous exponential decay parameterized by real elapsed time:

```js
/**
 * Frame-rate-independent exponential lerp.
 * `speed` is the fraction approached per 16.67ms (one 60fps frame).
 */
function lerp(current, target, speed, dt = 16.67) {
  const factor = 1 - Math.pow(1 - speed, dt / 16.67)
  return current + (target - current) * factor
}
```

Requirements:

- Thread a **measured** `dt` (`performance.now()` deltas) through every lerp.
- **Clamp** large deltas (`MAX_DELTA_MS = 50`) so a stall or tab-refocus doesn't teleport everything in one giant step.
- Stop the loop when idle or hidden (see below).
- Snap at a sub-pixel or domain-appropriate epsilon.
- No per-frame allocation; no per-frame layout reads.

## Adaptive smoothing — fast for small changes, slow for big ones

A fixed lerp speed forces a compromise between responsiveness and smoothness. Vary speed with the size of the gap:

```js
const gapRatio = Math.min(gap / previousRange, 1)
const speed = baseSpeed + (1 - gapRatio) * adaptiveBoost  // e.g. 0.08 + up to 0.2
```

Small ticks snap responsively (`+0.2`); large swings stay slow and dramatic (near the `0.08` base). Small updates feel instant; big ones feel weighty. Use this only when it improves perception without hiding meaningful change.

## Snap to target at an imperceptible epsilon

Exponential lerps asymptote but never arrive, causing sub-pixel churn (shimmering text, `99.9999` values, wasted work). Snap once the error stops being *visible* — ideally in pixel space:

```js
const pxThreshold = 0.5 * currentRange / chartHeight  // half a pixel
if (Math.abs(display - target) < pxThreshold) display = target
```

## Performance is part of the feel

Smoothness that buckles under load isn't smoothness. Bound the cost so the fluidity never breaks.

### Bypass the framework only on measured hot paths

A true 60fps value update should not go through the framework's render cycle. Write to the DOM/canvas/native view directly and keep the draw-loop closure identity-stable:

```js
// 60fps value display — no React re-renders
valueElRef.current.textContent = formatValue(displayValue)
// Config lives in a ref so the rAF loop is never recreated on prop change
configRef.current = props
```

During a drag, mutate `element.style.transform` directly and reconcile to framework state only on drop. **But** keep ordinary state changes declarative — only go imperative where profiling shows a genuine hot path (pointer transforms, animated chart drawing, rapidly-updating counters, canvas, gesture tracking).

### Never read layout inside the render loop

Reading `getBoundingClientRect` / `offsetWidth` inside a rAF loop forces synchronous reflow (layout thrash) that tanks frame rate. Measure out of band with a `ResizeObserver` (or platform equivalent), cache the geometry in a ref, and read the ref in the loop. Read pointer geometry once per *event*, never per frame. Batch reads and writes.

### Stop invisible work

```js
if (document.hidden) { rafRef.current = 0; return }  // kill the loop entirely
// restart on 'visibilitychange', from a fresh timestamp
```

Stop the loop when no value is changing, the element is offscreen (when practical), the document/app is hidden, the component unmounts, or reduced motion disables the effect. Restart from a safe timestamp so a large accumulated delta doesn't teleport the presentation. A background dashboard tab should cost zero CPU.

### Device pixel ratio — handle and clamp

```js
const dpr = Math.min(window.devicePixelRatio || 1, 3)  // clamp: 5x phones would allocate 25x pixels
ctx.setTransform(dpr, 0, 0, dpr, 0, 0)                 // draw in logical coords, crisp on retina
```

Resize backing buffers only when dimensions actually change.

### Allocation and lookup

In frame loops: avoid per-frame array/object creation (reuse buffers; compact arrays with a write-index instead of `filter()`); use binary search for hover/hit-testing over large sorted data, not O(n) scans; avoid unnecessary formatting work; stop once presentation reaches the target. At 60fps, per-frame allocation causes GC pauses that show up as stutter.

## Stress-test against worst-case data, not the happy path

An animation that only looks good on calm, ideal input is untested. The smoothing decisions above (adaptive speed, snap epsilon, clamped `dt`, monotone interpolation) all break in different ways under adversarial data — so drive the loop with the worst case before you trust it:

- **Sharp reversals** — a value that climbs then plunges. This is the classic breaking point for interpolation: an overshooting spline draws a dip below a low that never happened, and a slow adaptive speed lags visibly behind the turn.
- **Isolated spikes on a flat baseline** — one outlier against calm data stresses the axis-range chase and the snap epsilon.
- **Rapid oscillation** — fast up/down/up exposes queued/stale animations and shimmer that never settles.
- **Irregular / bursty arrival with gaps** — clustered updates then silence stress the frame loop's idle-stop/restart and the `dt` clamp (a gap must not teleport the presentation on resume).

> "A chart that only looks good on calm data isn't much use… Sharp reversals are the classic breaking point."

Feed synthetic worst-case sequences through the same code path in the VERIFY step; a chart tuned only on real, calm feeds is not verified.

## Correctness is an aesthetic

Visual smoothness must never falsify information. For line charts and interpolated data:

- A plain cubic / Catmull-Rom spline **overshoots** — it draws dips below a real low or spikes above a real high that never happened, which is actively misleading. Use **Fritsch-Carlson monotone cubic** interpolation, which guarantees the curve never exceeds the local min/max.
- Keep exact tooltips and accessible values authoritative — distinguish *visual* smoothing from *data* smoothing.
- Do not hide missing, delayed, or uncertain data behind animation.

A beautiful but misleading visualization is incorrect. Here, being correct *is* being polished.
