# Working With an AI Agent on Design

Read when the workflow itself is human↔agent design iteration — one party critiques, another edits. Distilled from Agentation (Benji's visual-feedback tool for AI coding agents). This matters doubly because you are often the agent: hold yourself to the taste bar the human would.

## Point, don't describe

Prose descriptions of UI are lossy; references are not. Instead of "the blue button in the sidebar," give a concrete selector plus structured data the agent can `grep` for the exact code:

> **Bad:** "This section needs work."
> **Good:** "`.sidebar > button.primary` — this bullet list reads like docs, not a showcase. Use a 3-column card grid with icons, like Stripe's guidelines pattern. Creates visual rhythm and scannability."

Capture class names, selectors, and element positions as structured fields (`element`, `selector`, `boundingBox`, `cssClasses`), not natural language the agent must re-interpret.

## Give a coordinate system and a translation formula, not a picture

When communicating layout intent to an agent, don't dump pixel positions — emit a reference frame with rules it can apply mechanically:

- Horizontal position: `element.x - containerLeft` → use as `margin-left` / `left`.
- Width: `element.width / containerWidth × 100` → use as `width: X%`.
- Centered: if `|element.centerX - containerCenterX| < 20px` → use `margin-inline: auto`.

## Use relational language, not absolute coordinates

Describe intent in relationships: "below `Navigation`", "aligned to the trigger", "centered in `main`", "16px after the previous group", "right of `Sidebar` (24px gap)", "width relative to the parent". Read the parent's actual CSS context (flex-direction, grid columns, gap) so the agent edits in the layout's own terms. Detect *structural* intent (a row became a column; a group dissolved) rather than reporting raw coordinate diffs.

## The critique / fixer loop

A robust pattern: one agent **critiques** (adds annotations), another **fixes** (reads them, edits code, marks each resolved), and a human watches in a visible browser — *"like watching a self-driving car navigate."* Model feedback as a loop with acknowledgement and closure (acknowledge → change → resolve-with-summary; dismiss-with-reason; reply-to-thread), not one-way instruction. Iterate top-to-bottom through the surface, and after each action **verify it actually took effect** — assume nothing succeeded silently.

## The design-feedback rubric

Whether you're giving feedback to an agent or generating it as one, good design feedback is:

- **Specific and actionable** — "Stack the install command below the subheading at 16px," not "fix the layout."
- **Principled** — name the principle: visual hierarchy, Gestalt grouping, whitespace, emphasis, conversion design.
- **Comparative** — reference a known-good product: "like how Stripe / Linear / Vercel handles this."
- **Tight** — 2–3 sentences per note; ~5–8 notes per surface.

## You bring the taste

An agent "optimizes for working rather than feeling right" and often can't tell when something looks wrong. The correction only a human eye catches — that an arrow should *rotate* rather than have its coordinates morphed, that an easing feels sluggish, that something pops where it should glide — is the entire value add. Don't stop at "it works." Look at it, and ask whether it *feels* right.
