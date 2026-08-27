# Water Lab — continue here

Interactive hydraulics and hydrology simulators for post A/L engineering
students, by Thushan Chamika. **One self-contained file: `water-lab.html`**,
around 600 kB, holding every simulator, its theory text and its styling.

**Read [`WATER_LAB_V2_MASTER_PLAN.md`](WATER_LAB_V2_MASTER_PLAN.md) before
writing any code.** It is the authority: the architecture, the v2 design system,
the pre-built engine APIs, and the work items in the order they must land. This
file only says where the work stopped.

There is also a skill called **`water-lab-v2`**. If it is available, invoke it
first. It carries the file's conventions, the tested engine APIs and the
verification harness, and it applies even to small bug fixes.

---

## The one next action

**Pick the next unchecked work item in phase order and do exactly one of them.**

Phase 3 (networks, transients, pumps) is finished. The last commit,
`19c2e67 Water Lab: graphics pass on thrust, NPSH and impeller (P3 remainder)`,
landed WL-1.3, WL-3.3 and WL-3.4.

So the next item is **Phase 4, hydrology**, starting with:

- **WL-4.3 · `thiessen` REBUILD** — "Areal Rainfall Studio, three methods"
  (2 sessions). Arithmetic mean, Thiessen polygons and isohyetal maps side by
  side, with the three answers compared. Use the pre-built `GeoField` helpers:
  `idwGrid`, `marchingSquares`, `chainSegments` and `bandArea`. Do not pass a
  large `eps` to `chainSegments`, the default 1e-6 is correct.

Then WL-4.4 `designstorm`, WL-4.2 `idf` and WL-4.5 `infil`, WL-5.3 `scs`, and
WL-5.1/5.2/5.4 together. Section 5 of the master plan has the full briefs.

---

## Session protocol, condensed

The full version is section 0.2 of the master plan. Follow it, do not improvise.

1. Pick the **next unchecked work item** in phase order. One item is one session
   unless it is marked multi-session.
2. `grep -n "function simXxx"` to find the target, read enough to know where it
   ends, and replace the **whole function body** when the item says REBUILD.
3. Adding a sim means five edits: `simNewthing()`, its `SIMS` entry, its
   `CATALOG` entry, an icon case in `svgIcon()`, and theory in `T` if specified.
4. Removing a sim means deleting all five, then grepping the key to prove there
   are zero references left.
5. Never rename an existing key. If a key is superseded, keep an alias in
   `route()`.
6. Keep the whole file under about 1.3 MB.

---

## Verifying

Browser APIs will not run under Node, but parsing catches typos, so syntax check
after **every** session:

```bash
awk '/<script>/{f=1;next}/<\/script>/{f=0}f' water-lab.html | tail -n +2 > /tmp/app.js
node --check /tmp/app.js
```

Use the last script block, and adjust the awk if there are several.

Then the manual smoke list, which nothing automates:

- the hub loads
- every changed sim opens from its card
- sliders respond
- leaving a sim stops its animation, CPU drops
- no console errors
- the back link works
- hash deep links work, for example `#/thiessen`

---

## House rules that get work rejected

- **No brown and no teal** in UI chrome. Soil and earth inside canvas scenes use
  the olive-grey set defined in section 1.2 of the plan.
- **No em dashes** in user-facing copy. Plain, confident microcopy, sentence
  case.
- Particles are pooled and capped: at most 160 per sim, radius 1.2 to 2 px, alpha
  fading with life.
- Animations must stop when the user leaves the sim.
- Use the pre-built engines rather than rewriting their maths. `MOCEngine` for
  water hammer, `solveNetwork` for pipe networks, `GeoField` for interpolation
  and contouring. Their exact signatures are in section 0.4.

---

## Where this sits

| | |
|---|---|
| Repo | `thushanch/curious-lk-labs` (private) |
| Branch | `waterlab-graphics-p3` |
| This lab | `waterlab/water-lab.html` plus the master plan |
| Deployed | `/waterlab/` on the Vercel site, via a redirect from `index.html` |

`waterlab/index.html` exists only to redirect to `water-lab.html` so the folder
resolves as a URL. Do not put content in it, and do not rename `water-lab.html`,
because the master plan and the skill both refer to it by name.

---

## One hazard

This folder lives inside OneDrive, and during one session OneDrive silently
restored an older snapshot and undid finished edits. `water-lab.html` is a single
600 kB file, so a revert loses a whole session's work in one go. Commit early,
and after a large edit confirm the change is really on disk before moving on.
