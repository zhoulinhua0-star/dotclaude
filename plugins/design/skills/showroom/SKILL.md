---
name: showroom
description: Build a live, clickable preview page of design options for one part of a frontend (layout, visual style, copy, animation or transition), open it in the browser, and let the user pick one and copy an exact implementation instruction back. Preview only; nothing in the app is changed. Use when the user runs /showroom, or asks to "give me options", "preview options", "show me some variants", "let me pick" or "what could this look like" for a UI part, animation or transition, often with a screenshot. Not for requests to implement a change directly.
argument-hint: "<the part of the UI to explore>"
---

# Showroom

The user wants to see several real options for one part of their UI side by side, click through them, and choose. Your job is to build that showroom, not to change the app. They'll paste their choice back later and implementation happens then, in a normal turn.

Part to explore: $ARGUMENTS (plus any screenshot the user attached). If it's unclear which part they mean, ask one short question before building.

## Principles

- **Preview only.** Don't edit, create or delete anything in the project and don't commit. Everything you make goes in the session scratchpad (or the system temp dir if there's no scratchpad). The user is choosing, and a half-implemented option in their tree is noise they'd have to clean up.
- **Real, not generic.** Read the actual component(s) and styles first. Copy the real CSS, design tokens, fonts, copy text and assets into the preview so each option looks like the actual site. A generic mock makes every option look worse than it would in place and makes the choice meaningless.
- **Current first.** Option 0 is the part as it ships today, so every option is judged against reality.
- **Distinct options, about 4–6.** Each option should be a different idea (structure, hierarchy, motion concept, tone), not the same idea with a new color or a 4px nudge. Use fewer for a tiny tweak and more only when the design space is truly wide. Mark one **Recommended** with a one-sentence reason.
- **Honest copy.** If options rewrite product copy, check claims against the code and whatever product docs exist (README, CLAUDE.md, docs/). Badge anything that promises more than the product does as **Overclaims**, with the reason. Never invent features.
- **Say what's simulated.** If the preview leaves out surrounding UI, fakes data or handles navigation differently from the real app, add a note. Don't show controls that do nothing, such as a text field that ignores input.

## Quality bar

Bring your best aesthetic judgment to every option. Each one should look like something a top product designer would be proud to ship, not a filler variation. Pay attention to spacing rhythm, type scale, contrast, alignment and detail.

Make every clickable thing inside an option feel good to press:
- instant hover feedback
- a quick, tactile press, such as a slight scale-down or depress
- a clear `:focus-visible` ring
- a smooth settle back

Snappy ease-out timing works best (about 100–200ms). Keep animations on `transform` and `opacity` where you can, give each motion a sensible `prefers-reduced-motion` fallback, and badge an option **Heavier on low-end** when it relies on large blurs, filters, backdrop-filter over moving layers or layout-changing animation.

Polish within the site's design language so the pick can actually ship. One or two options may break out of it on purpose. Badge those **Bold**.

## Steps

1. **Read.** Find the component(s), styles and assets for the part. Note the exact current copy, tokens (colors, fonts, radii, shadows), existing motion, and any variants or modes the part has (e.g. tabs or audience switches). Collect the file paths that implementation would touch.
2. **Design** the options, using the principles and quality bar above.
3. **Build** from [template.html](template.html). Copy it to `<scratchpad>/showroom-<part-slug>/index.html`, copy needed assets next to it, and fill in only the marked slots:
   - fonts
   - `window.SHOWROOM`: sections, options with their `html` (or `<template>`), `css`, optional `script` for JS motion, `pick` text, badges and notes

   Each preview renders in its own iframe, so paste the project's CSS as it is in the repo. `:root` tokens, `body` styles and breakpoints behave as on the real site, including in the Tablet and Mobile widths. Don't rewrite the chrome. It already provides the sticky "Preview only" bar, option cards, live stage, Replay, a viewport switch, variant and toggle controls, and the **Copy my choice** button with a clipboard fallback. Use one section per distinct part being explored. Use `variants` when the part has modes, so the user picks once per mode. Write each `pick` so it stands alone: the exact copy, or a precise one-line description of the design or motion.
4. **Serve and open.** Run `python3 -m http.server <free port>` from that folder in the background. Open `http://localhost:<port>` (localhost, not 127.0.0.1) in a new tab with the browser tools available, preferring Claude in Chrome. If no browser tool is available, just give the URL.
5. **Check it.** Click a few options, press Replay on a motion option, take a screenshot, and read the console. Fix anything broken before replying.

## Reply

Keep it short:
- the URL
- the number of options per section
- the recommendation and why
- anything flagged (overclaims, heavier motion, simulated parts)
- end with: "Pick, press **Copy my choice**, and paste it here."

When the user pastes the copied block, the showroom is done. Implement exactly the IDs it names, as a normal task.
