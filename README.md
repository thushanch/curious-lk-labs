# Curious LK · Engineering Labs

Two interactive simulator suites for engineering students, both by
**Thushan Chamika**, University of Moratuwa.

**Each lab now lives in its own repository.** This one is kept only as a way
in to both, and its copies of the two labs are frozen.

| Lab | Repository | Live |
|---|---|---|
| CivilLab | [thushanch/civillab](https://github.com/thushanch/civillab) | https://thushanch.github.io/civillab/ |
| Water Lab | [thushanch/water-lab](https://github.com/thushanch/water-lab) | https://thushanch.github.io/water-lab/ |

Work on the labs in their own repositories, not here.

---

## Water Lab

`waterlab/water-lab.html` — one self-contained HTML file, 41 simulators covering
hydraulics and hydrology, from a single pipe out to a whole modelled catchment.

Open `waterlab/water-lab.html` in any browser.

| Topic | Covers |
|---|---|
| 01 Foundations | EGL/HGL pipelines, venturi, thrust blocks, jets, siphons |
| 02 Pressure pipes | Network solver, series and parallel, Moody, water hammer |
| 03 Pumps | Operating point, pump stations, NPSH, impeller and specific speed |
| 04 Precipitation | IDF curves, areal rainfall, design storms, infiltration |
| 05 Rainfall–runoff | Rational method, unit hydrographs, SCS curve number |
| 06 Open channel | Manning, specific energy, Froude, weirs and flumes |
| 07 Profiles | Channel modeller, water surface profiles, jumps, sluice gates |
| 08 Sediment | Shields threshold, pier scour, reservoir trap efficiency |
| 09 Probability | Monte Carlo, return periods, frequency fitting, levee reliability |
| 10 Catchment studio | The Living Catchment, calibration arena |

`waterlab/WATER_LAB_V2_MASTER_PLAN.md` is the build plan and session ledger: it
records what each simulator does, the physics it implements, what has been
verified, and what is still outstanding.

## CivilLab

`civillab/index.html` — a multi-file suite of structural and geotechnical
simulators built on the University of Moratuwa Civil Engineering syllabus.
Shared physics engines live in `civillab/assets/js/`, one app per folder under
`civillab/apps/`.

Open `civillab/index.html`.

Covers beams, Mohr's circle, bending and shear stress, torsion, deflection,
buckling, trusses, influence lines, moment distribution, plastic collapse,
dynamics, and soil phase relationships and classification.

`civillab/CLAUDE.md` and `civillab/PLAN.md` carry the build brief and roadmap.

---

## A study aid, not a textbook

These simulations simplify things, and a few can mislead if taken literally.
Use them alongside your lectures to build a feel for the physics, then check
anything that matters with your lecturer. If something looks wrong, please say
so.

## Repository layout

This repository deliberately contains only the two projects above. It is
initialised inside a working folder that holds other, separate projects, so
`.gitignore` allows only `waterlab/` and `civillab/` through.

## Credits

Fonts in `civillab/assets/fonts/` are Inter and Poppins, used under the SIL Open
Font License.
