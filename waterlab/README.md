# Water Lab

Interactive hydraulics and hydrology simulators for post A/L engineering
students, by **Thushan Chamika**, University of Moratuwa. Part of the
Curious LK Project, and a companion suite to
[CivilLab](https://github.com/thushanch/civillab).

**Live: https://thushanch.github.io/water-lab/**

---

## Run it

There is nothing to install and nothing to build. Open `water-lab.html` in any
browser, online or off. The whole suite is one self contained HTML file of
about 590 kB holding every simulator, its theory text and its styling.

```bash
git clone https://github.com/thushanch/water-lab.git
cd water-lab
# then just open water-lab.html
```

---

## What is in it

Forty one simulators running from a single pipe out to a whole modelled
catchment. Every one solves the real equations as you move the controls, so
the numbers on screen are answers rather than illustrations.

| Module | Covers |
|---|---|
| 01 Foundations | EGL and HGL along a pipeline, venturi, thrust blocks, jets, siphons |
| 02 Networks and transients | Free network editor solved by Hazen-Williams, water hammer by the method of characteristics |
| 03 Pumps | Series and parallel, NPSH and cavitation, impeller behaviour |
| 04 Rainfall | Areal rainfall by three methods, design storms, IDF, infiltration |
| 05 Runoff | Rational method, unit hydrographs, SCS curve number |
| 06 Open channel | Manning, specific energy, Froude, weirs and flumes |
| 07 Profiles | Gradually varied flow as an explainer rather than a drill |
| 08 Sediment | Sediment transport and scour |
| 09 Frequency | Flood frequency on your own data, distribution fitting, reliability |
| 10 Catchment | The flagship: a whole catchment modelled end to end with run comparison |

---

## Working on it

Two files matter:

- **[`CONTINUE.md`](CONTINUE.md)** — where the work stopped and what to do
  next. Start here, including from a phone.
- **[`WATER_LAB_V2_MASTER_PLAN.md`](WATER_LAB_V2_MASTER_PLAN.md)** — the
  authority. Architecture, the v2 design system, the pre-built engine APIs and
  every work item in the order it must land. Read it before writing any code.

Syntax check after every session, because browser APIs will not run under Node
but parsing still catches typos:

```bash
awk '/<script>/{f=1;next}/<\/script>/{f=0}f' water-lab.html | tail -n +2 > /tmp/app.js
node --check /tmp/app.js
```

Then the manual smoke list: the hub loads, every changed sim opens from its
card, sliders respond, leaving a sim stops its animation, no console errors,
the back link works, and hash deep links such as `#/thiessen` resolve.

---

## House rules

- No brown and no teal in the UI chrome. Soil and earth inside canvas scenes
  use the olive grey set in section 1.2 of the plan.
- No em dashes in user facing copy. Plain, confident microcopy, sentence case.
- Particles are pooled and capped at 160 per sim.
- Animations stop when the user leaves the sim.
- Never rename an existing sim key. If one is superseded, keep an alias in
  `route()`.
- Keep the whole file under about 1.3 MB.

---

## Licence

All rights reserved for now. Ask before reusing the content.
