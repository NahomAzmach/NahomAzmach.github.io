Andromeda Interpreting & Translation Services was started by Patty and Miguel, who moved from Mexico to the Seattle area in 2001 and noticed how much was getting lost between non-English speakers and everyone else. Patty became a certified interpreter through DSHS, Miguel followed, and the community kept coming back to them.

Their website was on Weebly. It worked, but it looked like it was on Weebly.

![The new home page: the brand, a typewriter hero, and the one action that matters.](assets/img/andromeda.webp)

## Content-first modernization

The rule for this rebuild was strict: **zero information loss.** Every line of their existing text, their history, and their color palette stays. I'm not rebranding them, I'm giving the brand they already have a much better house.

So before any code, I wrote a `master_idea.md` that acts as the single source of truth: the exact color palette (gold `#EBD391`, near-black `#1A1A1A`, off-white `#F1F1F1`) and a complete inventory of every piece of copy on every page, word for word. Everything built after that gets checked against it.

That doc turned out to be the most useful thing in the repo. When you're moving fast, especially with AI coding tools in the loop, it's really easy to "improve" someone's copy without meaning to. A written spec of what must not change keeps everyone honest, including me.

## Simple, but alive

The original's simplicity was a feature, so the structure stays simple: Home, About, Contact. What changed is how it feels:

- A continuous **typewriter effect** on the hero ("We translate and interpret.").
- Floating logo, hover-scaling icons, and small rotating elements with framer-motion, so it feels alive without being busy.
- The homepage's static info cards became an alternating **editorial layout**.

## History you scroll through

The About page used to be one static block of text. Now it's a parallax **vertical timeline** that walks through how the agency started, beginning in 2001.

![The About page's history timeline.](assets/img/andromeda-about.webp)

**Stack:** Next.js (App Router), React, TypeScript, framer-motion, CSS Modules, Vercel.
