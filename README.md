# Skills For Fluid Interface Design

Skills that teach an AI coding agent — or you — how to build interfaces that feel **simple, continuous, and alive**.

They distill the design-engineering philosophy behind [Benji Taylor's](https://benji.org/) work: Family, Honk, Liveline, and ConnectKit, plus his writing on fluid interfaces and on collaborating with AI agents. The goal isn't software that merely works or looks nice — it's software that feels *coherent*, where every transition, value change, component, and interaction seems to belong to the same system.

> "Crafting a fluid interface is the result of hundreds of small, deliberate decisions woven together. A single fluid transition alone doesn't create a fluid interface." — Benji Taylor, [Family Values](https://benji.org/family-values)

## Install

```bash
npx skills@latest add bochenn/benjitaylor-skills
```

Or just copy the skill folder you want from [`.agents/skills/`](./.agents/skills) into your own project — each skill is a self-contained `SKILL.md` (plus, for `benji-design-v3`, a few reference files loaded on demand).

## Why use it?

**Agents don't have great taste.** They optimize for *working*, not for *feeling right*, and they often can't tell when something looks wrong — an `ease-in` on an enter animation, an element that pops in from `scale(0)`, a modal whose height snaps between steps, an arrow whose coordinates get morphed when it should simply rotate.

All those little things compound. They're the difference between an interface people love without knowing why and one that feels just... off.

These skills encode those decisions — the easing curves, the timing, the continuity rules, the performance discipline, the accessibility edge cases — so an agent (or you) reaches the right choice faster. They also carry the one thing agents lack: the instruction to stop at *"does this feel right?"*, not just *"does this work?"*

## The skills

This repo ships **three variants of the same philosophy** so you can pick the one that fits how you work. They cover the same ground — motion, continuity, morphing, performance, component APIs, theming, and working with an AI agent — but differ in shape and length.

- **[benji-design](./.agents/skills/benji-design/SKILL.md)** — Craft-first. Opinionated and evidence-grounded, with concrete curves, durations, and code examples pulled straight from Benji's open-source work. Reads like a design engineer talking. Single file.

- **[benji-design-v3](./.agents/skills/benji-design-v3/SKILL.md)** — **Recommended.** The balanced version: a lean ~200-line core that opens with a strict priority-order gate (correctness → accessibility → simplicity → conventions → performance → fluidity → delight), an inspect-then-verify workflow, and stack detection — then points to four [`references/`](./.agents/skills/benji-design-v3/references) files loaded only when a task needs them. Keeps the concrete voice, adds the safety rails, stays cheap on context.

- **[benji-design-code-audit](./.agents/skills/benji-design-code-audit/SKILL.md)** — Procedure-first. The most thorough: operating modes (`audit` / `fix` / `improve` / `build` / `system`), a full required workflow, severity classification, and a modification-report format. Best when you want an agent to run a disciplined, repeatable audit over a whole frontend.

Not sure which to start with? Use **`benji-design-v3`**.

## How to use it

Once installed, just ask your agent in plain language — it will pull in the skill:

```
> review this component using Benji's fluid-interface philosophy
> audit the animations in this screen and tell me what feels off
> make this drawer transition feel more fluid
> apply the design philosophy to the checkout flow
> build a command menu that feels alive
```

The skills work in any stack. They inspect your repo first — framework, styling system, animation library, existing tokens — and never add a dependency or impose a pattern the project didn't already reach for.

## What's inside

The skills are distilled from real, publicly shared work:

- **Benji's essays** — [family-values](https://benji.org/family-values), [honkish](https://benji.org/honkish), [liveline](https://benji.org/liveline), [morphing-icons-with-claude](https://benji.org/morphing-icons-with-claude)
- **Benji's code** — [liveline](https://github.com/benjitaylor/liveline) (frame-rate-independent interpolation, canvas/DPR, monotone splines) and [agentation](https://github.com/benjitaylor/agentation) (working with AI agents on design)
- **Family's [ConnectKit](https://github.com/family/connectkit)** — measure-then-morph modals, a curated easing vocabulary, token-based theming
- **Related craft references** — [cmdk](https://github.com/pacocoursey/cmdk) by Paco Coursey, for compound-component API design, accessibility, and DOM-as-source-of-truth patterns

## Credits

The ideas, quotes, and techniques here belong to **[Benji Taylor](https://benji.org/)**. This repository is an independent, unofficial distillation of his publicly shared writing and open-source code — it is not affiliated with or endorsed by him. Credit for the craft is his; any errors in the distillation are ours.
