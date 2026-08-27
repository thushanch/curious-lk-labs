# CONTINUE — start here

This repo holds two separate simulator suites. Open the one you want to work on:

- **[civillab/CONTINUE.md](civillab/CONTINUE.md)** — CivilLab, civil engineering apps
- **[waterlab/CONTINUE.md](waterlab/CONTINUE.md)** — Water Lab, hydraulics and hydrology

Each of those files is written so you can hand it straight to Claude Code on a
phone and carry on without any other context.

---

## Repo facts

| | |
|---|---|
| Repo | `thushanch/curious-lk-labs` (private) |
| Working branch | `waterlab-graphics-p3` |
| Default branch | `main` |
| Live site | root `index.html` is the hub, deployable to Vercel as a static site |

Both suites are plain HTML, CSS and JavaScript. No build step, no framework, no
runtime dependencies. Everything works from `file://` with no network.

---

## Quick start from a phone

Open this repo in Claude Code and paste one of these:

> Read `civillab/CONTINUE.md` and carry on from the next action listed there.

> Read `waterlab/CONTINUE.md` and carry on from the next action listed there.

Do not skip the `CLAUDE.md` (CivilLab) or `WATER_LAB_V2_MASTER_PLAN.md`
(Water Lab) that those files point at. They carry the conventions, and work that
ignores them gets rejected.

---

## Deploying

The repo is already shaped for Vercel with no configuration to fill in:

- `index.html` at the root is the hub page linking to both labs
- `vercel.json` declares no framework and no build, serving the root directory
- `.vercelignore` keeps planning docs and the test harness out of the deployment
- `waterlab/index.html` redirects to `water-lab.html`, so `/waterlab/` resolves

Connect the repo in Vercel, accept the detected settings, and deploy. Routes:

```
/                      the hub
/civillab/             CivilLab landing page, 14 apps
/civillab/apps/<id>/   an individual app
/waterlab/             redirects to the Water Lab file
```

Nothing on the site calls out to another host, so there is nothing to configure
for fonts, analytics or APIs.

---

## Two hazards worth knowing before you touch anything

**1. This folder is inside OneDrive.** During one session OneDrive silently
restored an older snapshot of the working tree and undid finished edits. Git
caught it, but only because the changes were checked afterwards. If files you
just wrote appear to revert themselves, that is what happened. Moving the
project out of the OneDrive-synced path is the real fix.

**2. There is a second, redundant CivilLab repo.** `thushanch/civillab` was
created from the `civillab/` subfolder and is now behind this one. This repo is
the authoritative copy. The stray `civillab/.git` directory makes the subfolder
look like a nested repository, which will confuse tooling. To collapse it:

```bash
rm -rf civillab/.git
```

Then delete `thushanch/civillab` on GitHub if you do not want it. Neither step
is done yet, because both throw away history and that is your call.
