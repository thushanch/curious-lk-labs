# WATER LAB v2 — MASTER REVISION PLAN
**Owner:** Thushan Chamika · **File under revision:** `water-lab.html` (single-file app, ~8,430 lines)
**Purpose of this document:** a complete, self-sufficient build plan. Any capable model (e.g. Claude Sonnet) must be able to open this plan plus the HTML file and execute one work item per session, in order, without asking design questions. Every physics formula needed is written here so nothing is invented.

---

## 0. HOW TO EXECUTE THIS PLAN (read first, every session)

### 0.1 Architecture of the existing file (do not change these conventions)
- One HTML file. No external libraries, no backend, no build step. Fonts via Google Fonts link only.
- All CSS in one `<style>` block; all JS in one `<script>` block at the end of `<body>`.
- **Registry:** `const SIMS = { key: {title, module, blurb, build, theory?} }` (currently line ~8151). `build` is a function returning `{node, cleanup}`. `cleanup` MUST cancel every `requestAnimationFrame` and remove window listeners.
- **Catalog:** `const CATALOG = [ {n:'01', t:'Module name', sims:[{k:'key', name, d, flag?}]} ]` (currently line ~8265). Cards without `k` render as "Soon". Numbering in the catalog is display-only; hash routes use keys (`#/manning`).
- **Router:** `route()` reads `location.hash`, calls `renderSim(key)` or `renderHub()`.
- **Shared helpers already present (reuse, do not duplicate):**
  `el(tag,cls,html)`, `fmt(x,d)`, `clamp`, `slider(label,min,max,step,value,unit,cb)`, `numField`, `group(title)`, `setupCanvas(canvas,w,h)` (HiDPI scaling), `fitWidth(panel,ratio,max)`, `predictCard(question)`, `eqStrip(lines)`, `K()/N()/U()` (equation typography spans), `paintBG`, `waterGrad`, `earthGrad`, `glowStroke`, `rrect`, `insetPanel`, `flowDashes`, `ro(label)` (readout rows), `zpInv(p)` (Acklam inverse-normal — reuse for all frequency work), `yvOf(T)` (Gumbel reduced variate), `probPaper(...)`, theory content object `const T = {...}` (line ~8034).
- Sim builder functions are named `simXxx()` and live between lines ~387 and ~8030. Each is self-contained.

### 0.2 Session protocol for the executing model
1. Open this plan. Pick the **next unchecked work item** in Section 5 (phase order). One item = one session unless the item says "multi-session".
2. `grep -n "function simXxx"` to find the target function; view enough lines to know its boundaries; replace the whole function body rather than patching fragments when the item says REBUILD.
3. When adding a new sim: (a) write `simNewthing()`, (b) add SIMS entry, (c) add CATALOG entry, (d) add an icon case in `svgIcon()` if that switch exists (grep `svgIcon`), (e) add theory in `T` if specified.
4. When removing a sim: delete its `simXxx` function, its `SIMS` entry, its `CATALOG` entry, its `T.*` theory entry, and its `svgIcon` case. Then `grep` the key to confirm zero remaining references.
5. **Syntax check after every session** (browser APIs won't run in node, but parsing catches typos):
   `awk '/<script>/{f=1;next}/<\/script>/{f=0}f' water-lab.html | tail -n +2 > /tmp/app.js && node --check /tmp/app.js`
   (Use the LAST script block; adjust the awk if there are several.)
6. Manual smoke test list: hub loads; every changed sim opens from its card; sliders respond; leaving the sim stops animation (CPU drops); no console errors; back link works; hash deep-link works.
7. Never rename existing keys. If a key is superseded, keep an alias in `route()` (see WL-CAT).
8. Respect the brand: **no brown, no teal** in UI chrome. Soil/earth inside canvas scenes uses the olive-grey set defined in 1.2. No em-dashes in user-facing copy; plain, confident microcopy; sentence case.
9. Budget: keep the total file under ~1.3 MB. Phase 0 removals free significant room before the big builds.

### 0.3 Traceability — Thushan's instructions mapped to work items
| Instruction (verbatim gist) | Sim (key) | Work item | Verdict |
|---|---|---|---|
| Full graphics should be revised | all | WL-0 kit + per-item | Global reskin system |
| 1.1 pipes more customizable | egl | WL-1.1 | REBUILD (segment editor) |
| 1.3 flow visual more appealing | thrust | WL-1.3 | ENHANCE |
| PipeFlow Studio: junction coordinates, add pipes, solve, head interpretation | pipeflow | WL-2.1 | REBUILD (free network editor) |
| 2.2 can't understand anything, try pump icons | serpar | WL-2.2 | REBUILD (physical scene) ⚠ see note |
| 2.4 remove | threeres | WL-CAT | REMOVE |
| 2.5 show water hammer physically, animation | hammer | WL-2.5 | REBUILD (MOC animation) |
| 3.3 NPSH: explain why, animation | npsh | WL-3.3 | REBUILD-lite |
| 3.4 explain what happens, accurate illustrations | impeller | WL-3.4 | ENHANCE |
| 3.5 remove | affinity | WL-CAT | REMOVE |
| 4.1 OK but more enhanced visuals | sandbox | WL-SUPER | SUPERSEDED by SuperApp ⚠ |
| 4.3 add isohyetal maps + compare methods | thiessen | WL-4.3 | REBUILD (3 methods) |
| 4.4 name each method, explain, calc table | designstorm | WL-4.4 | ENHANCE |
| Module 5: every app theory + graphics + calc table | rational, uh, scs, scsuh | WL-5.x | ENHANCE all |
| 5.3 CN vs vegetation, canopy icons, animation | scs | WL-5.3 | REBUILD scene |
| 5.5 remove | timearea | WL-CAT | REMOVE |
| Module 6: every app animated/graphic upgrade | all mod 6 | WL-6.x | ENHANCE all |
| 6.1 friendlier interface | manning | WL-6.1 | ENHANCE UX |
| 6.2 slide width changes OR weir | specific-energy | WL-6.2 | ENHANCE (add width contraction) |
| 6.3 graphics should clarify meaning | froude | WL-6.3 | ENHANCE |
| 6.4 more developed weir/flume | weir | WL-6.4 | REBUILD-lite (add flume, submergence) |
| 7.1 water particles move with velocity | openchannel | WL-7.1 | ENHANCE |
| 7.2 remove app, make theory explainer with animations | profiles | WL-7.2 | REBUILD as explainer |
| Remove whole module 8 | culvert,gutter,pond,sewer,afflux | WL-CAT | REMOVE ×5 |
| Keep 9.1, 9.2, 9.5; enhance graphics | shields, scour, traplife | WL-9.x | KEEP + polish ⚠ |
| 10.3 enhance graphics | freqfit | WL-10.3 | ENHANCE |
| 10.4 remove; new table/CSV app, all methods | freqlab | WL-10.4 | REBUILD v2 |
| 10.5 remove OR make physically sensible | reliability | WL-10.5 | REBUILD-lite (default keep) ⚠ |
| Module 11 remove all, keep 11.5 elsewhere; add RMSE/MAE, cal+val, watch training | calib (+musk,nash,greenampt,recession,resyield removed) | WL-11.5 | RELOCATE + REBUILD |
| New SuperApp: full rainfall-runoff portal, real water-cycle animation, charts | new key `catchment` | WL-SUPER | NEW FLAGSHIP |

**⚠ Interpretation decisions (already made; change only if Thushan overrides):**
- **"Keep 9.5":** Module 9 has only three sims (shields, scour, traplife). Interpreted as keep first, second and last, i.e. keep all three. Nothing removed in module 9.
- **"2.2 pump icons":** 2.2 is the series/parallel *pipe* trainer. Interpreted as: the abstract diagram is unreadable, so rebuild it as a physical scene driven by a clearly drawn pump symbol, with animated flow and piezometers. (The series/parallel *pump* app is 3.2 `pumpbuild`; it stays as-is apart from the global reskin.)
- **"10.5 remove or fix":** default is REBUILD-lite because a physical cross-section already exists; spec below makes it unmistakably physical. Fallback removal instructions included.
- **"4.1 OK, enhance" + "add New SuperApp":** the SuperApp is the enhanced 4.1. The two-bucket `sandbox` engine is the seed; the SuperApp replaces it in a new final module and `#/sandbox` redirects to `#/catchment`. Do not maintain two versions.
- **Module 8 removal:** complied fully. Note for later: the level-pool routing engine inside `simPond` is good code; it may be salvaged into a future SuperApp reservoir toggle, but not now.

---

### 0.4 PRE-BUILT BY FABLE (2026-08-03) — as-built APIs, use these exactly
P0a, P0b (kit portion), the P1 C-1 engine, and every numerically risky engine for P2/P3/P4 are **already inserted and tested**: the block sits in `water-lab.html` between the CHANGELOG comment and the `SIM 1 — EGL` header (search `V2 KIT`). CSS for `.dtable`/`.dt-bar` is already in the `<style>` block. 42 node asserts passed against the embedded copy (Gumbel/LP3/GEV quantile recovery, MOC Joukowsky peak + 4L/a period, network solver vs analytic series case + loop mass balance, catchment closure < 1e-12 and forest peak damping 45%, marching-squares single closed loop). **Do not rewrite these; consume them.** The old `FREQ` object (used by `simFreqFit`) is still present; migrate freqfit to `Freq` during P2, then delete `FREQ`.

Signatures (exact):
- `SC` scene tokens (olive-grey soils, no brown); `REDUCED` prefers-reduced-motion flag.
- `Anim(drawFn(t,dt))` → `{start, stop, frame, running}`. One per sim; call `.stop()` in cleanup. Auto-pauses when tab hidden; renders a single frame under reduced motion.
- `Particles(cap≤160)` → `{spawn(x,y,life), step(dt, velFn(x,y,p)→[u,v,kill?]), draw(ctx,{col,r}), clear, count}`.
- drawKit: `arrow(ctx,x0,y0,x1,y1,{col,w,head})`, `chip(ctx,x,y,text,{align:'left|center|right',bg,fg,font})→bbox`, `callout(ctx,fromX,fromY,toX,toY,text,o)`, `dimLine(ctx,x0,y0,x1,y1,label,o)`, `hatchRect(ctx,x,y,w,h,{fill,col,sp})`, `flowField(ctx,pts,phase,{sp,speed,col})`, `paintField(ctx,W,H)` light hydrology background (deep theme keeps existing `paintBG`).
- `Chart2(canvas,{mL,mR,mT,mB})`: set `ch.size(ctx,W,H)` after `setupCanvas`, assign `ch.x=ch.scaleLinear([a,b])` (also `scaleLog`, `scaleGumbel([Tmin,Tmax])`, `scaleZ`), then per frame `ch.clear('deep'|'field')`, `ch.axes({xLabel,yLabel,topTicks,xGrid,grid})`, `ch.line(pts,{col,w,dash})`, `ch.area(pts,y0,o)`, `ch.bars(pts,o)`, `ch.barsDown(pts,{max,frac})` (hyetograph from top), `ch.points(pts,{r,ring})`, `ch.legend([{label,col,dash}])`, `ch.cursor(xv)`, `ch.hoverChip(px,py,lines)`, `ch.clip(fn)`, `ch.X(v)/ch.Y(v)` px mappers, `off=ch.onMove(cb({x,y,px,py}|null))` (call `off()` in cleanup).
- `DataTable({cols:[{key,label,type:'num'|'text',ro,step,d,warn(row)}], rows, onChange(rows), filename, maxRows, features:{addRow,delRow,paste,csvImport,csvExport,sort}})` → `{wrap, rows, setRows(r), refresh}`. `setRows` does not fire onChange. Paste accepts Excel TSV; CSV import maps by header else position.
- `theoryV2({what, eqs:[...], worked:{cols, compute(state)→rows}, practice, trap})` → `{node, update(state)}`. Renders inside the existing `.thy` details style; keep old `theoryPanel` for untouched sims.
- `csvParseText(text)` (sniffs , ; tab; handles quotes/BOM), `csvDownload(filename, rowsOfArrays)`.
- Math: `erf`, `normCdf`, `lnGamma`, `gammaFn`, `pearson(a,b)`, `niceStep`, `fmtTick`.
- `Freq` (for P2): `Freq.fit(data)` → `{stats, logStats, allPositive, methods:{normal,gumbel,ln2,lp3,gev}}` each method `{label, col, q(T)}` (lp3 adds `cs, clamped`; gev adds `xi, alpha, kappa`); `Freq.empirical(data, 'weibull'|'gringorten'|'cunnane')` → desc-sorted `[{Q,P,T,rank}]`; `Freq.gof(data,qFn,pp)` → `{rmse, mae, ppcc, n}` (rmse/mae on log10); `Freq.seGumbel(T,sd,n)`; `Freq.zT`, `Freq.kGumbel`, `Freq.kLP3(T,cs)`.
- `MOCEngine({L,a,D,f,V0,H0,N=33})` (for WL-2.5) → `{step(tau)→{t,H,Q}, reset, H, Q, dt, dx, N, H0v, Q0, B, joukowsky}`. Valve `tau` 1→0; τ≤0 is full closure. Steady state pre-initialized with linear friction slope.
- `solveNetwork({nodes:[{id,type:'J'|'R',z,H(R only),demand m³/s}], pipes:[{a,b,L,D,C}]})` (for WL-2.1) → `{ok, res, iters, H:{id→head}, press:{id→head−z}, Q:[signed a→b], r:[pipe resistances]}` or `{ok:false, err, island?}`. Hazen-Williams SI, damped Newton, numerical Jacobian.
- `SOILS` {sand,loam,clay}; `stormSeries(depth,durH,'uniform'|'front'|'back'|'alt')` → per-minute mm array (alt = alternating block).
- `CatchmentEngine({depth,durH,pattern,soil,forest 0..1,kq,kb,antecedent,G0,horizonH,areaKm2})` (C-1, DONE) → `{series:{t,P,inter,infil,overland,et,C,S,G,Rq,qq,qb,Qmm,Qm3s}, ledger:{sumP,sumQ,sumET,dS,closure}, peak, peakT, params}`. As-built physics: interception cap `0.5+2.5·forest` mm; `f0·(1+0.8·forest)`; quickflow `kq·(1−0.3·forest)` (forest roughness); ET only when P=0, canopy first. C-2 binds the scene to `series`, C-3 the scrubber, C-4 the charts, C-5 compare-runs; the engine itself needs no changes.
- `GeoField` (for WL-4.3): `idwGrid(pts[{x,y,v}], [x0,y0,x1,y1], nx, ny, p=2)` (snaps to gauge within half a cell), `marchingSquares(grid, level)` → segments, `chainSegments(segs)` → polylines (nearest-first, default eps 1e-6 — do not pass a large eps), `bandArea(grid, lo, hi, mask?)`.

---

## 1. WL-0 — GLOBAL DESIGN SYSTEM v2 ("full graphics revised")
*Phase 0. Build the kit first; every later item consumes it. Multi-session: 0a helpers, 0b removals + catalog, 0c reskin pass on kept-but-unmentioned sims.*

### 1.1 Two canvas themes
Every sim canvas declares a theme at the top of its draw code:
- **deepPanel** (existing dark `#0e2233` panel): hydraulics modules 1, 2, 3, 6, 7, 8(sediment). Upgrade, don't replace: water bodies use `waterGrad` plus a new animated surface (sum of 2 sines, amplitude ≤ 3 px, phase from the shared clock) and a specular line (white, 8% alpha, 1.5 px, along the surface). Add faint caustic shimmer: 5 to 8 slow quadratic curves inside water, `--pale` at 5% alpha.
- **fieldPanel** (NEW, light): hydrology modules 4, 5, 9(risk), 10(modelling). Background `#F4F7F9` to `#E8EFF3` vertical gradient, faint topographic contour lines (reuse the body-background contour motif at 25% alpha) so hydrology sims feel like the brand's "paper and terrain" world. Ink on light: labels `#1C2A33`, secondary `#5b6870`.

### 1.2 Scene color tokens (add to `:root` and mirror as a JS object `SC`)
```
--sc-water-deep:#0F2C44; --sc-water:#1E78B0; --sc-water-lite:#9FC4E6;
--sc-soil-dark:#4F4B41; --sc-soil:#6E6A5E;  --sc-soil-lite:#8A8474;   /* olive-grey earth, replaces any brown */
--sc-sand:#B0A488;      --sc-veg-deep:#2E6B4F; --sc-veg:#3E9A6E; --sc-veg-lite:#7FBF9E;
--sc-sky-hi:#DCEAF4;    --sc-sky-lo:#F6F7F4;   --sc-cloud:#FFFFFF;
--sc-danger:#D2452F;    --sc-warn:#E8913C;     --sc-ok:#3E9A6E;
```
Semantic canvas lines stay: EGL `--egl` orange, HGL `--water` blue, critical depth `--crit` red dashed, normal depth `--norm` green dashed. Never repurpose these four.

### 1.3 New shared JS modules (insert directly after existing helpers, ~line 385)
Write these once, small and dependency-free. Signatures are contracts; later specs call them by name.

**a) `Anim` — one rAF per sim.**
```
const anim = Anim(drawFn);       // drawFn(t, dt) seconds
anim.start(); anim.stop();        // stop() is called from cleanup
```
Internals: single rAF loop, dt clamped to 50 ms, auto-pauses on `document.hidden`, and if `matchMedia('(prefers-reduced-motion: reduce)')` matches, runs drawFn once per input change instead of continuously (expose `anim.frame()`).

**b) `Particles(n)` — pooled tracers.** `p.spawn(x,y,life)`, `p.step(dt, velFn)` where `velFn(x,y)->[u,v]`, `p.draw(ctx, style)`. Global budget: ≤ 160 per sim, radius 1.2–2 px, alpha fades with life, color default `#CFE6F5`. Used by WL-1.3, 2.2, 2.5, 3.3, 6.x, 7.1, 9.x, SuperApp.

**c) `drawKit` additions:** `arrow(ctx,x0,y0,x1,y1,{w,head,col})`; `dimLine(ctx,x0,y0,x1,y1,label)` engineering dimension with end ticks and a centered label chip; `chip(ctx,x,y,text,{bg,fg})` rounded label; `callout(ctx,x,y,tx,ty,text)` thin leader line + chip; `hatchRect(ctx,...)` 45° hatch for solid ground; `flowField(ctx,path,phase)` animated dash arrows along a polyline (generalizes `flowDashes`).

**d) `Chart2` — the chart kit (this is the single biggest quality lever).**
```
const ch = Chart2(canvas, {mL,mR,mT,mB});
ch.x = ch.scaleLinear([x0,x1]) | ch.scaleLog([..]) | ch.scaleGumbel([Tmin,Tmax]) | ch.scaleZ([..]);
ch.y = ...same;
ch.clear(theme); ch.axes({xLabel,yLabel,xTicks,yTicks, topTicks});
ch.line(pts,{col,w,dash}); ch.area(pts,y0,{col,alpha}); ch.bars(pts,{col,w});
ch.barsDown(pts,{...})            // hyetograph hanging from the top axis
ch.points(pts,{col,r,ring}); ch.legend([{label,col,dash}]);
ch.cursor(x)                       // vertical time-cursor line
ch.hover(cb)                       // crosshair + nearest-point readout chip
```
`scaleGumbel(T)` maps via reduced variate `y = -ln(-ln(1-1/T))` and draws top-axis ticks at T = 2,5,10,25,50,100,200,500. `scaleZ` maps via `zpInv(1-1/T)` for normal-probability paper. Hydrograph convention everywhere: rainfall bars hang from the top (barsDown, `--sc-water-lite`), flow area filled from the bottom (`--water` line over 18% fill), baseflow shaded `--ocean` 25%.

**e) `DataTable(spec)` — editable data + calc tables.**
```
DataTable({cols:[{key,label,type:'num'|'text',ro?}], rows, onChange, features:{addRow, delRow, paste, csvImport, csvExport, sort}})
```
Paper styling: white card, `--contour` hairlines, Poppins 500 headers, tabular numerals, edited-cell flash. `paste` accepts TSV from Excel. `csvImport` uses `<input type=file>` + FileReader, sniffs delimiter, lets the user map columns (simple two-select row). `csvExport` builds a Blob download. **Every hydrology sim (modules 4, 5, 9-risk, 10) ends its right column or page bottom with a live calc `DataTable` bound to current inputs.**

**f) `theoryPanel` v2.** Restructure `theoryPanel(T.x)` rendering into fixed sections, in order: **What this is** (2–3 sentences) · **Governing equations** (eqStrip) · **Worked example, live** (a small DataTable auto-filled from the CURRENT slider state, so theory and sim never disagree) · **Where you meet it in practice** (one Sri Lankan example where natural: tanks, Mahaweli, monsoon design storms) · **Common trap** (one misconception). Existing `T` entries are progressively rewritten into this shape as each sim is touched; new entries must use it from day one.

### 1.4 Motion rules
One shared clock (`t` from `Anim`). Ambient motion is quiet: surface sines, dash drift, particle drift. Reserve *fast* motion for the physics event the sim teaches (the jump roller, the hammer wave, the choke). Everything eases; no linear pops. `prefers-reduced-motion` honored by `Anim` automatically. Nothing blinks faster than 2 Hz.

### 1.5 Canvas typography
Labels: Inter 12 px `#9FB4C4` on deepPanel / `#5b6870` on fieldPanel. Values: Poppins 600, tabular-nums. Every plotted line gets either an inline end-label or a legend chip; no unlabeled colors anywhere. Units always shown, thin-space before unit.

### 1.6 WL-CAT — removals and the new catalog (part of Phase 0)
**Delete completely (follow 0.2 step 4):** `threeres`, `affinity`, `timearea`, `culvert`, `gutter`, `pond`, `sewer`, `afflux`, `musk`, `nash`, `greenampt`, `recession`, `resyield`. Also delete the old `sandbox` build ONLY in the same session that lands WL-SUPER (redirect `#/sandbox` → `#/catchment` inside `route()`), and the old `profiles` build only when WL-7.2 lands (key reused).

**New CATALOG (renumber display only; keys unchanged):**
```
01 Foundations · Energy & Momentum ........ egl, venturi, thrust, jet, siphon
02 Pressure Pipe Systems & Networks ....... pipeflow★, serpar, moody, hammer
03 Pumps & Pump Systems ................... pump, pumpbuild, npsh, impeller
04 Precipitation & Design Rainfall ........ idf, thiessen, designstorm, infil
05 Rainfall–Runoff & Design Flows ......... rational, uh, scs, scsuh
06 Open Channel: Uniform & Critical ....... manning, specific-energy, froude, weir, compound
07 Profiles & Rapidly Varied Flow ......... openchannel★, profiles, jump, sluice, stepmethod
08 Sediment & Scour ....................... shields, scour, traplife
09 Probability & Risk for Design .......... montecarlo, risk, freqfit, freqlab★, reliability
10 Catchment Modelling Studio ............. catchment★★, calib
```
Update hero stats to compute counts from CATALOG (never hard-code "54" again). Tagline for module 10 card row: "The portal: one living catchment, and the honest way to test a model against it."

---

## 2. PHASE 1 — THE SUPERAPP: `catchment` (WL-SUPER, flagship, multi-session)
*"Full rainfall-runoff process with enhanced animation, control lots of things, real-like water cycle, charts for processes, like a major portal." This is the centerpiece. Build it in five sub-sessions C-1 … C-5, in order. Do not start C-2 before C-1's asserts pass.*

### 2.1 Layout (fieldPanel)
```
+------------------------------------------------------------+---------------+
|                 SCENE CANVAS (16:9, full width)             |  CONTROLS     |
|  sky · clouds · rain · canopy · hillslope cross-section     |  Storm        |
|  soil column · groundwater · river · outlet gauge           |  Land         |
+------------------------------------------------------------+  Soil         |
|  TIME BAR: play/pause · speed 1–60× · scrub · t readout     |  Antecedent   |
+----------------------------------------------------------------------------+
|  CHART DECK (shared x = time, one synced cursor across all)                |
|  [1] hyetograph (bars down) over hydrograph (quick stacked on baseflow)    |
|  [2] storages: canopy C, soil S/Smax, groundwater G                        |
|  [3] live budget ribbon: this-step P → interception/infil/overland/ET      |
|  [4] mass-balance ledger card: P = Q + ET + ΔS  (closure %)                |
+----------------------------------------------------------------------------+
```

### 2.2 C-1 · Engine (pure functions, no canvas yet)
Event-mode conceptual model, step `dt = 1 min` of simulated time. State: canopy store `C` (mm), soil store `S` (mm), groundwater `G` (mm), quick-flow reservoir `Rq` (mm). All fluxes in mm per step over catchment area `A` (km², fixed 25).

Per step, in this exact order (P = rain this step from the storm pattern):
1. **Interception:** `dC = min(P, Ccap - C)`; throughfall `Pt = P - dC`. `Ccap = 0.5 + 2.5·forest` mm (forest ∈ 0…1).
2. **Infiltration capacity:** `fcap = fc + (f0 - fc)·(1 - S/Smax)` (mm/h → per-step). Soil presets: Sand f0=100, fc=20, Smax=100; Loam f0=60, fc=10, Smax=150; Clay f0=25, fc=3, Smax=180. Forest bonus: `f0 *= (1 + 0.5·forest)`.
3. **Split:** `infil = min(Pt, fcap)`; `overland = Pt - infil` → add to `Rq`.
4. **Soil:** `S += infil`; saturation excess: any `S > Smax` spills to `Rq`. Percolation `perc = kperc·S·dt_h` (Sand .08, Loam .05, Clay .02 h⁻¹) → `G`.
5. **ET (only when P = 0):** `et = PET·(S/Smax)·(1 + 0.6·forest)` with `PET = 0.10 mm per 10 min` scaled to dt; remove from C first, then S.
6. **Flows:** quickflow `qq = kq·Rq·dt_h` (kq slider 0.6–2.4 h⁻¹); baseflow `qb = kb·G·dt_h` (kb slider 0.01–0.10 h⁻¹). `Q_mm = qq + qb`; convert for display: `Q_m3s = Q_mm/1000 · A·1e6 / (dt·60)`.
7. Log every flux into `series[]` for the charts, and accumulate the ledger.

**Storm builder inputs:** total depth (10–200 mm), duration (0.5–6 h), pattern: uniform | front-loaded | back-loaded | alternating-block (build from a simple i = a/(d+t)^0.75 curve, rank blocks, alternate around the center). Antecedent wetness slider sets initial `S/Smax` (0.1–0.9) and `G` (5–60 mm).

**Acceptance C-1 (console.assert, run once at build):** with dt=1 min, 24 h horizon: `|ΣP − (ΣQ + ΣET + ΔC + ΔS + ΔG + ΔRq)| / ΣP < 0.005`. Forest 0→1 at fixed storm must (a) reduce peak `Q_m3s` by ≥ 20 %, (b) raise the post-storm baseflow fraction. If either fails, tune kq/Ccap/f0-bonus, not the structure.

### 2.3 C-2 · Scene painter (static first)
Paint order, one function per layer, all driven by current state:
1. **Sky:** `--sc-sky-hi→lo` gradient; during rain, darken 12 % and desaturate.
2. **Sun & ET wisps:** soft disc top-left; when ET > 0, spawn 1–3 upward wisp particles per second from canopy and soil surface (alpha 0.25, slight sway).
3. **Clouds:** 3–5 blobs (overlapping ellipses, `--sc-cloud` at 85 %), drifting at 6 px/s; count and darkness scale with remaining storm depth.
4. **Rain:** streak particles, count ∝ current intensity (map 0–120 mm/h → 0–140 streaks), 12° slant, splash tick on ground contact. This is the single loudest motion in the scene; everything else stays calm under it.
5. **Terrain cross-section:** a hill sloping to the river at right. Canopy band: procedural crowns (3-lobe arcs, two greens, crown count and radius ∝ forest slider; below 0.15 draw stumps + pale grass). Trunks `--sc-soil-dark`.
6. **Soil column:** below ground surface, `--sc-soil` body; moisture shown as a `--sc-water` tinted fill rising from the column base proportional to `S/Smax`, with a dotted wetting-front line during infiltration; small downward arrows while `infil > 0`, size ∝ rate.
7. **Groundwater:** darker aquifer band; water-table line elevation ∝ `G`; slow horizontal seep arrows toward the river when `qb > 0`.
8. **Overland flow:** when `overland > 0`, a translucent water ribbon slides down the surface into the river, thickness ∝ rate, with 6–10 surface glints.
9. **River (right 22 % of canvas):** water depth and width ∝ `Q_m3s^0.4`; animated surface sines; 20–40 stream particles at speed ∝ Q; tiny gauge hut + stage needle at the outlet.
Label chips (drawKit.chip) on: canopy store, soil store %, water table, Q at outlet. `prefers-reduced-motion`: rain becomes a static hatch density, particles frozen, state still redraws per step.

### 2.4 C-3 · Time control + binding
`Anim` clock drives the engine at `speed` sim-minutes per real second (default 12×). Play/pause; scrubber sets the engine index into the precomputed run (recompute the whole run instantly on ANY control change — the horizon is ≤ 24 h × 1 min = 1,440 steps, trivially fast; never simulate lazily). The scene renders the state at the cursor index; charts draw the full series with `ch.cursor(t)` synced.

### 2.5 C-4 · Chart deck (Chart2)
Chart 1: barsDown hyetograph (mm/10 min) + hydrograph area, quickflow stacked in `--water` on baseflow in `--ocean`; legend chips; hover readout (t, P, Q, base %). Chart 2: three storage lines C, S, G with right-edge end labels. Chart 3: budget ribbon: a single horizontal stacked bar for the cursor step (interception | infiltration | overland | ET), animating widths. Ledger card: ΣP, ΣQ, ΣET, ΔStorage, closure % in a 2×2 stat grid; closure turns `--sc-warn` if > 1 %.

### 2.6 C-5 · Compare runs + polish (the thesis feature)
"Pin run A" button freezes the current hydrograph as a dashed `--sc-veg-deep` overlay with a small legend chip naming its settings (e.g. "forest 80 %, loam"). Change anything, watch run B against it. One-click scenario chips: **Deforest it** (forest→0.05), **Pave nothing / Reforest** (forest→0.9), **Wet season start** (antecedent→0.85). This turns the SuperApp into a forest-hydrology argument machine. Theory panel `T.catchment`: what each store is, why forests trade peak for baseflow, and the honest caveat that this is a teaching model, not a calibrated one — for calibration, walk next door to the Arena.
Retire `sandbox` per WL-CAT in this session. Catalog entry: module 10, `flag:true`, name "The Living Catchment", d: "Every millimetre accounted for, from cloud to gauge."

**Acceptance C-5:** 60 fps on a mid laptop with ≤ 160 particles; compare-run overlay correct after 5 rapid control changes; ledger closes ≤ 0.5 %; deforest chip visibly raises peak and drops baseflow within one click.

---

## 3. PHASE 2 — FREQUENCY SUITE (module 09)

### 3.1 WL-10.4 · `freqlab` REBUILD — "Flood Frequency Studio, Your Data" (2 sessions)
Replace `simFreqLab` entirely. This is the practitioner tool: user data in, every method out.

**Data panel (left column, DataTable):** columns Year | Q (m³/s). Features: addRow, delRow, sort by year, paste TSV, CSV import with column mapping, CSV export, and a "Load sample" menu (Sample A: 32-yr wet-zone-style record; Sample B: 18-yr short record to show uncertainty; write plausible synthetic values inline). Validation chips: n < 10 → red "too short to fit"; non-positive Q → block LP3/LN with an explanatory chip; duplicate years flagged. n updates live.

**Options row:** plotting position: Weibull `P=m/(n+1)` | Gringorten `P=(m−0.44)/(n+0.12)` | Cunnane `P=(m−0.4)/(n+0.2)` (m = rank, largest = 1). Axis toggle: Gumbel paper (`scaleGumbel`) | Normal paper (`scaleZ`). y: Q linear | log.

**Methods (compute all, one color each, legend chips):** work on `x = Q` for Normal/Gumbel, `y = log10 Q` for LN2/LP3. Sample stats: mean, s (n−1), skew `Cs = n·Σ(y−ȳ)³ / ((n−1)(n−2)·s³)`.
1. **Normal:** `Q_T = x̄ + z_T·s`, `z_T = zpInv(1 − 1/T)` (reuse existing Acklam fn).
2. **Log-Normal 2p:** `log10 Q_T = ȳ + z_T·s_y`.
3. **Gumbel (method of moments):** `K_T = −(√6/π)·(0.5772 + ln(ln(T/(T−1))))`; `Q_T = x̄ + K_T·s`.
4. **Log-Pearson III:** `K_T` by Wilson–Hilferty with `k = Cs/6`: `K = z + (z²−1)k + (z³−6z)k²/3 − (z²−1)k³ + z·k⁴ + k⁵/3`; `log10 Q_T = ȳ + K·s_y`. Clamp |Cs| ≤ 3 with a warning chip.
5. **GEV by L-moments (v2.1 flag, build last, skip if session runs long):** ascending sort; `b0 = mean`, `b1 = Σ((i−1)/(n−1))·x_i/n`, `b2 = Σ((i−1)(i−2)/((n−1)(n−2)))·x_i/n`; `λ1=b0, λ2=2b1−b0, λ3=6b2−6b1+b0`; `t3=λ3/λ2`; `c = 2/(3+t3) − ln2/ln3`; `κ = 7.859c + 2.9554c²`; `α = λ2·κ/(Γ(1+κ)(1−2^{−κ}))`; `ξ = λ1 − α(1−Γ(1+κ))/κ`; `Q_T = ξ + (α/κ)(1 − (−ln(1−1/T))^κ)`. Implement `lnGamma` with the standard Lanczos g=7, 9-coefficient series.

**Outputs:**
- **Probability plot:** data points (rings) at plotting positions; every fitted curve drawn T = 1.05…500; top axis T ticks; hover shows nearest point (year, Q, T_empirical).
- **Quantile table (DataTable, read-only):** rows T = 2, 5, 10, 20, 25, 50, 100, 200, 500 × one column per method; a highlight toggle marks the row for the user-chosen design T.
- **Goodness-of-fit table:** per method, computed at each observation's plotting-position T: RMSE and MAE on `log10 Q`, plus PPCC (Pearson r between sorted observed Q and fitted quantiles). Best RMSE gets a `--sc-ok` "closest fit" chip, with the caption: "closest to the plotted record, which is not the same as true."
- **Design card:** pick T and method → big Poppins number, and for Gumbel also the standard error `SE = (s/√n)·√(1 + 1.14K + 1.10K²)` shown as ±1 SE.
- Export results CSV (quantile + GoF tables).
**Theory `T.freqlab` (v2 shape):** why AMS, what a plotting position claims, why the four/five answers differ in the tail, and why a 30-yr record cannot pin a 100-yr flood. Sri Lanka note: this is exactly the workflow behind design floods for tank spillways.
**Acceptance:** Sample A: methods agree within ~10 % at T=10 and diverge visibly at T=200; CSV round-trip (export → import) reproduces the table; deleting rows live-updates everything; no NaN leaks to screen (fmt guards).

### 3.2 WL-10.3 · `freqfit` ENHANCE (1 session)
Keep the synthetic-record teaching intent. Upgrades: use `Chart2.scaleGumbel` (proper top-T axis); per-distribution color + dash from a legend; keep the disagreement envelope but fill with 10 % `--egl`; add a small residual strip under the main plot (obs − fitted vs T, per selected distribution); add an n-slider (15…80) so re-rolls show sampling noise shrinking; T=100 answers as aligned stat cards, not text. Reuse ALL math from WL-10.4's method functions — extract those into shared `Freq = {...}` (replacing the current `FREQ` object) so freqfit, freqlab and reliability import one implementation.

### 3.3 WL-10.5 · `reliability` REBUILD-lite (1 session) — default is KEEP
Make the physics impossible to miss:
- Cross-section stays (river, levee, town) but becomes the live stage: press **Simulate design life** → n years tick past (0.15 s each); each year draws an EV1 stage sample `h = u − β·ln(−ln(rand))` rising up the levee face as a water fill; overtopping years splash over the crest, flood-tint the town, and add a tally chip. Failure count vs predicted `R = 1−(1−1/T)^n` shown side by side, so the dial and the disaster meet.
- Crest slider physically raises the levee polygon; freeboard drawn with `dimLine` from design stage to crest.
- Cost panel: crest cost rises with height (simple quadratic), expected damage falls (= annual exceedance × damage value × years); total-cost curve with the optimum marked. Three sliders: damage value, cost rate, design life.
- Theory `T.levee` v2: risk equation, why freeboard is "a probability wearing metres", and the honest limits (stationarity!).
**Fallback if Thushan says remove:** follow 0.2 step 4 for `reliability`; move the risk equation card into `risk` (10.2) theory.
*(`montecarlo` and `risk` are untouched except the Phase-0 reskin sweep.)*

---

## 4. PHASE 3 — NETWORKS, TRANSIENTS, PUMPS (modules 1, 2, 3)

### 4.1 WL-2.1 · `pipeflow` REBUILD — free network editor (2 sessions)
The current studio has fixed topologies. Rebuild around a canvas editor.
**Editor:** toolbar modes: Select | +Junction | +Reservoir | +Pipe | Delete. Click empty canvas → node at a 20 px snap grid (coordinates shown live, editable in the panel: x, y in metres via a scale of 1 px = 2 m). +Pipe: click node A then node B (rubber band while choosing). Select any element → side panel edits it: junction {z (m), demand (L/s)}; reservoir {H (m), drawn as the classic tank glyph}; pipe {L (auto = geometric length × scale, manual override toggle), D (mm), C_HW}. Delete removes element + incident pipes. "Save/Load" = JSON in a textarea (copy-paste persistence, no backend). Presets: **Two-loop classic** (7 nodes) and **Hill town branch** (reservoir + 6 junctions with elevations).
**Solver (Newton nodal, Hazen–Williams SI):** unknowns = heads at junctions; reservoirs fixed. Pipe flow `Q_ij = sgn(ΔH)·(|ΔH|/r)^{1/1.852}` with `r = 10.67·L/(C^{1.852}·D^{4.87})` (Q m³/s, D m). Residuals `F_i = ΣQ_in − ΣQ_out − demand_i`. Numerical Jacobian (central, δ = 1e−4 m), damped Newton (halve step while ‖F‖ grows), floor `|ΔH| ≥ 1e−6` to kill the zero-flow singularity, ≤ 60 iterations. Guards before solving: every node must reach a reservoir (BFS) else red chip "island: no fixed head"; total demand vs supply sanity note.
**Head interpretation (the point of the rebuild):** after solve — nodes fill on a diverging color scale of pressure head `p/γ = H − z` (red < 0, `--sc-warn` 0–10 m, blue-green ≥ 10 m) with the value beside each node; pipes get animated dash arrows sized by |Q| with a Q label chip; convergence sparkline (‖F‖ vs iteration). **HGL path tool:** click a node sequence → a Chart2 panel draws ground z and HGL along the path, with negative-pressure segments flagged. Iteration table (DataTable): iter, max residual, worst node.
**Theory:** continuity + energy as the whole story; why the solver is just "guess heads, enforce continuity"; where Hardy–Cross fits historically (one paragraph; the old convergence-race feature is retired to keep the editor focused).
**Acceptance:** two-loop preset converges < 15 iterations; deleting a pipe and re-solving works; a drawn triangle network with one reservoir solves; sub-atmospheric node flags red; JSON round-trip restores the network exactly.

### 4.2 WL-2.5 · `hammer` REBUILD — real MOC animation (2 sessions)
Replace the schematic viewer with a genuine Method-of-Characteristics simulation so the wave is *watched*, not asserted.
**Numerics (Wylie–Streeter form):** N = 33 nodes, `Δx = L/(N−1)`, **`Δt = Δx/a` exactly** (Courant = 1). Constants `B = a/(g·A)`, `R = f·Δx/(2·g·D·A²)`. Steady init: `Q = Q0` everywhere, H linear from reservoir `H0` down the friction slope. Each step, for interior nodes:
```
Cp = H[i−1] + B·Q[i−1] − R·Q[i−1]·|Q[i−1]|
Cm = H[i+1] − B·Q[i+1] + R·Q[i+1]·|Q[i+1]|
H[i] = (Cp+Cm)/2 ;  Q[i] = (Cp−Cm)/(2B)
```
Upstream reservoir: `H[0]=H0`, `Q[0]=(H0−Cm)/B`. Downstream valve closing over `tc`: `τ(t)=max(0, 1−t/tc)` (offer linear | equal-percentage `τ^2`); `Cv = (Q0·τ)²/(2·H0v)` (H0v = steady head at valve); `Q[N−1] = −B·Cv + √((B·Cv)² + 2·Cv·Cp)`; `H[N−1] = Cp − B·Q[N−1]`; τ=0 → Q=0, H=Cp.
**Controls:** L (200–3000 m), a (300–1400 m/s), D, f, V0, H0, tc, speed/slow-mo, pause/scrub.
**Scene (deepPanel):** the pipe in elevation, colored along its length by H (blue → white → `--sc-danger` above Joukowsky, violet-dark below steady, hatch-flash where H < −10 m with a "column separation risk" chip). Wave fronts get a bright leading edge marker so you *see* the pressure wall travel, hit the reservoir, and reflect. Pipe wall drawn with exaggerated local dilation ∝ (H − Hsteady). Valve glyph animates closed over tc. Reference chips pinned on the H-scale: **Joukowsky ΔH = a·V0/g** and steady H.
**Charts:** H(t) at the valve and at midpoint (two lines), with vertical gridlines at t = 2L/a, 4L/a, 6L/a labeled "round trips"; a comparison chip: `tc` vs `2L/a` → banner "effectively sudden" when tc ≤ 2L/a (and the observed peak ≈ Joukowsky), else "slow closure, ΔH ≈ 2·L·V0/(g·tc)".
**Prediction card:** "If you close the valve twice as fast — but still faster than 2L/a — what happens to the peak?" (Answer the sim delivers: nothing.)
**Acceptance:** instant closure peak within 3 % of a·V0/g (f→0); oscillation period = 4L/a within one Δt; slow closure (tc = 6·L/a) peak clearly below Joukowsky; stable for 60 s sim time at every slider extreme.

### 4.3 WL-2.2 · `serpar` REBUILD — physical scene (1 session)
Kill the abstract diagram. New scene (deepPanel), two tabs sharing one control set:
- **SERIES:** a drawn supply tank + a bold pump glyph (circle, chevron impeller, subtle spin) pushing through three visibly different pipes end-to-end (D1 > D2 > D3 drawn to scale). Piezometer standpipes at the four joints show the HGL stepping down; the steepest drop sits over the skinniest pipe. Particle tracers speed up in narrow pipes (continuity, felt).
- **PARALLEL:** same tank + pump into a manifold that splits into the three pipes side by side, rejoining right. Standpipes at the two manifolds show ONE shared Δh; particle density per branch shows the flow split.
- **Readout panel:** series card "one Q, losses add: Σh = h1+h2+h3" with a stacked loss bar; parallel card "one Δh, flows add: Q = Q1+Q2+Q3" with a flow-split donut; equivalent-pipe D_eq readout for each mode. Physics: Darcy–Weisbach `h = f·(L/D)·V²/2g`, f from Swamee–Jain `f = 0.25/[log10(k/(3.7D) + 5.74/Re^{0.9})]²`; parallel split solved by bisection on Δh.
- The pump is iconographic (fixed head boost slider 5–40 m), not a curve lesson — that belongs to 3.1. Caption under the pump: "the pump pays for every metre the pipes burn."
**Acceptance:** halving D3 in parallel visibly starves that branch's particles and its donut slice; in series it steepens only its own HGL step.

### 4.4 WL-1.1 · `egl` REBUILD — customizable pipeline (1 session)
From fixed three-diameter line to a **segment editor**: a list of 2–6 segments, each row {L, D, material → k (PVC 0.0015 mm | steel 0.045 | concrete 0.3 | old cast iron 1.0), ΣK fittings}, add/remove segment buttons; ends = two reservoirs with level sliders; optional booster pump toggle (adds Hp at a chosen joint); optional partly-closed valve (K slider) at a chosen joint; a "high point" y-offset per joint so the profile can climb.
**Solve:** unknown Q from energy: `H1 − H2 + Hp = Σ_seg (f_i·L_i/D_i + ΣK_i)·V_i²/2g`, `V_i = 4Q/(πD_i²)`, f by Swamee–Jain each iteration — bisection on Q (monotone). Then draw the longitudinal profile: ground/pipe, **EGL** (orange) dropping per segment + step drops at fittings/valve, **HGL** = EGL − V²/2g, sub-atmospheric zones (HGL below pipe) shaded `--sc-danger` 15 % with the min-pressure callout, pump = vertical EGL jump. Per-segment loss bar chart. Velocity chip per segment turns `--sc-warn` outside 0.6–3 m/s.
**Acceptance:** adding a fat segment barely moves Q; pinching one segment dominates losses (its EGL slope visibly steepest); raising the high point until HGL dips below it triggers the cavitation shading exactly there.

### 4.5 WL-1.3 · `thrust` ENHANCE (½ session)
Add: particle tracers sweeping through the bend along the arc (color by speed); a toggleable control-volume overlay (dashed box, ṁV in/out arrows, p·A arrows at the two faces); animated head-to-tail vector composition building the resultant when any input changes (300 ms ease); keep everything else. One caption: "pressure does most of the shoving" pinned to whichever term is larger.

### 4.6 WL-3.3 · `npsh` REBUILD-lite — the WHY, animated (1 session)
Keep the head-ladder (it is good). Add the physical half:
- **Scene:** supply tank → suction pipe → pump eye → short discharge. Under the pipe, a live **pressure-head profile line** falls from atmospheric at the tank, drops along friction, and dives at the eye (local acceleration). A dashed vapor-pressure line sits below; the gap between them IS the margin, shaded green.
- **Cavitation moment:** when NPSHa < NPSHr, bubble particles nucleate at the eye, ride into the impeller, and collapse with 1-frame white flashes; the pump emits a subtle shake; the pump-curve inset visibly droops; verdict banner SAFE / MARGINAL (< 0.5 m) / CAVITATING.
- **Controls that tell the story:** water temperature 10–95 °C → vapor head from the table {10:0.12, 20:0.24, 30:0.43, 40:0.75, 50:1.26, 60:2.03, 70:3.18, 80:4.83, 90:7.15, 95:8.62 m} (interpolate); altitude 0–2000 m → `Hatm = 10.33·(1 − z/8434)`... use `10.33·exp(−z/8434)`; static lift slider (negative = flooded suction); suction-line hf via L, D, K; Q slider moving along an NPSHr(Q) curve `NPSHr = 2 + 6·(Q/Qref)²`.
- `NPSHa = Hatm − Hvap − z_lift − hf_suction`, both bars racing on the ladder.
**Theory v2:** why the *suction* side kills pumps, what NPSHr physically is (the machine's internal pressure dip), the 0.5–1 m margin rule, and the classic Sri Lankan case: a riverbank pump in April, hot water + low river = the same pump that ran fine in December now rattles.
**Acceptance:** at 20 °C sea level the demo is SAFE; raising temperature alone to ~80 °C triggers cavitation with no other change (this is the "hot water quietly steals the budget" beat, now visible).

### 4.7 WL-3.4 · `impeller` ENHANCE — what actually happens (1 session)
Keep the morph. Add meaning: (a) state the number with units every time: `Ns = N·√Q / H^{0.75}` (rpm, m³/s, m) with live substitution in the eqStrip; (b) internal **flow-path particles** through the morphing impeller: radial machines throw particles outward through narrow passages, axial machines pass them straight through — the particle paths morph WITH the geometry; (c) an efficiency-envelope chart (η vs Ns, the classic hill: radial ≈ 10–80, mixed ≈ 80–150, axial ≈ 150–300 in these SI-rpm units) with the current point riding it and region bands labeled; (d) three application cards that light up by regime: borehole/high-lift (radial), irrigation main (mixed), flood-drainage/low-lift (axial — the Sri Lankan flood pump); (e) a one-line story caption that updates: "high head, little water → a narrow disc spinning hard" ↔ "little head, a river of water → a propeller in a duct."

---

## 5. PHASE 4 — HYDROLOGY (modules 04, 05)
*House rule from Thushan, applied to BOTH modules: every app gets (a) theory in the v2 shape, (b) upgraded fieldPanel graphics, (c) a live calculation DataTable at the bottom bound to current inputs.*

### 5.1 WL-4.3 · `thiessen` REBUILD — "Areal Rainfall Studio, three methods" (2 sessions)
One catchment polygon, 3–10 draggable gauges with editable depths (click gauge → mini input; add/remove gauge buttons). Three method tabs + a Compare tab:
1. **Arithmetic mean:** all gauges inside the boundary, equal chips, `P̄ = ΣP/n`.
2. **Thiessen (exists, keep the drag-live Voronoi):** polygons clipped to the catchment, each labeled with its weight `A_i/A`; calc table: gauge, P, A_i, weight, P·w.
3. **Isohyetal (NEW):** build an IDW field on a ~96×64 grid over the catchment bbox: `P(x) = Σ(P_i/d_i^p)/Σ(1/d_i^p)`, power p slider 1–3, `d` floored at half a cell. Contour it with **marching squares** at a clean interval (auto: span/6 rounded to 5 or 10 mm); fill bands `--sc-water-lite`→`--sc-water-deep`; label each isohyet on its line. `P̄ = Σ(A_band·P̄_band)/ΣA_band` where band area = in-catchment cell count and `P̄_band` = mean of its two bounding isohyets (edge bands use the band's cell-mean). Calc table: band range, cells→km², band mean, product.
4. **Compare:** three stat cards + a delta table (method, P̄, % vs isohyetal) + one honest paragraph: arithmetic for flat dense networks, Thiessen when gauges are trusted points, isohyetal when you can draw the storm (orographic Sri Lanka: isohyetal earns its keep on the wet-zone escarpment).
**Acceptance:** dragging one gauge live-updates all three tabs; a bullseye storm (one hot gauge centered) makes arithmetic ≠ Thiessen ≠ isohyetal by obvious margins; marching squares produces closed, non-crossing contours for random gauge layouts.

### 5.2 WL-4.4 · `designstorm` ENHANCE — named methods + calc tables (1 session)
Restructure into a method picker where each method is NAMED, explained, computed and tabulated:
- **Uniform block** (constant i = P/D) · **Triangular (Yen & Chow)** (peak position slider r 0.3–0.6) · **Alternating block** (from the IDF: depths `D_k = i(t_k)·t_k`, increments, rank, alternate around the peak; table shows every column of the classic exam layout: block, duration, cumulative depth, increment, rank, arranged position) · **SCS 24-h Type distributions** (dimensionless mass-curve lookup, 13 points each, linear interp — include Type I, IA, II, III tables inline as data) · optional **Chicago** flag for v2.1.
- Keep the existing payoff and finish it: run every pattern through ONE small SCS-CN + triangular-UH transform (fixed teaching catchment) → peak-flow comparison bars titled "same depth, four floods."
- Theory v2: what a hyetograph shape claims about a storm, why alternating block is conservative, what the SCS types encode climatically, and which Sri Lankan storms front-load (convectional afternoon) vs long-duration (monsoonal/cyclonic — Ditwah-class systems are exactly why duration matters more than the label).

### 5.3 WL-4.2 `idf` + WL-4.5 `infil` (½ session each)
Global reskin + house rule only. `idf`: Chart2 axes, curve family legend, live calc table (T, d → a·T^m/(d+t)^n substitution rows). `infil`: keep Horton machinery; add the calc table (t, f, F cumulative, rain, ponded?) and a soil-type preset row; wetting shading via `--sc` soil colors.

### 5.4 WL-5.3 · `scs` REBUILD — the CN-vegetation scene (2 sessions)
The centerpiece of module 5, and Thushan's own research made teachable.
**Scene (fieldPanel): two identical hillslopes side by side, "Bare/Reference" vs "Yours".**
- **Vegetation slider (0–100 %)** on the right slope draws procedural canopy: crown count, radius and two-tone green density scale with the slider; below 15 % show stumps and pale grass; litter dots appear above 40 %. HSG selector A–D restyles the soil column (sandy pale → clay dark).
- **CN engine:** composite from an inline table (HSG A/B/C/D): Woods-good 30/55/70/77 · Pasture-good 39/61/74/80 · Row-crop 67/78/85/89 · Bare-fallow 77/86/91/94 · Impervious 98/98/98/98. The vegetation slider morphs the right slope's CN from Bare-fallow → Woods-good for the chosen HSG (linear in the slider; show the moving CN number big). AMC toggle: `CN_I = 4.2·CN_II/(10 − 0.058·CN_II)`, `CN_III = 23·CN_II/(10 + 0.13·CN_II)`.
- **Storm button:** identical rain animates on both slopes (P slider 20–200 mm). Canopy droplets visibly hold interception on the vegetated slope; infiltration arrows sink larger; the runoff ribbon leaving each slope has thickness ∝ its Q. Result chips under each slope: CN, S, Ia, Q, runoff coefficient.
- **Math:** `S = 25400/CN − 254` mm, `Ia = 0.2S`, `Q = (P−Ia)²/(P−Ia+S)` for P > Ia else 0.
- **Calc table:** both slopes' full substitution, side by side.
- **Research mode (small toggle, off by default):** replaces the slider mapping with the biomass curve `CN = CN_bare − (CN_bare − CN_min)·(1 − e^{−k·AGBD})` with AGBD 0–250 Mg/ha, k = 0.02, CN_min = the Woods-good value — captioned "a research-grade way to let satellites set this slider." One line, no thesis dump.
- **P–Q curve inset:** Q vs P for CN 40/60/80/95 with the current state riding its curve — the nonlinearity, felt.
**Acceptance:** slider 0→100 % at P = 100 mm, HSG C: Q drops from ~64 mm toward ~25 mm continuously; AMC III visibly punishes the vegetated slope's advantage; both ribbons and chips always agree with the table.

### 5.5 WL-5.1 `rational`, WL-5.2 `uh`, WL-5.4 `scsuh` (1 session together)
- `rational`: keep the paint-the-land idea; add the calc table (sub-area, cover, C, A, C·A → composite C; `tc` by Kirpich `tc = 0.0195·L^{0.77}·S^{−0.385}` min with L m, S m/m; i read off the IDF at tc with the substitution row; `Qp = 0.278·C·i·A` m³/s, i mm/h, A km²). Animate the "storm must outlast tc" beat with a duration slider that greys the answer when d < tc.
- `uh`: the convolution table IS the lesson: DataTable where row i shows every `P_j·U_{i−j+1}` term; stepping the animation highlights the anti-diagonal being summed while the hydrograph bar grows. Rain-block colors match their scaled UH copies (already partly there; make table and canvas share colors exactly).
- `scsuh`: show the funnel as four → two → one: input cards (A, L, Y, CN) flow into `tl` and `Tp` dials, then the universal shape. Use the SCS lag formula in original US units, converting internally: `tl = L_ft^{0.8}·(S_in + 1)^{0.7} / (1900·√Y%)` h, `S_in = 1000/CN − 10`; `Tp = D/2 + tl`; `qp = 0.208·A_km²/Tp` m³/s per mm of runoff. Calc table shows the unit conversions honestly (m→ft row included, labeled "the formula is imperial; we translate").

---

## 6. PHASE 5 — OPEN CHANNEL (modules 06, 07)
*House rule for module 6: every sim gains ambient animated water (surface sines + tracer particles) and at least one physical-event animation.*

### 6.1 WL-6.1 · `manning` ENHANCE — friendlier (1 session)
- Shape gallery: four cards with mini cross-section icons (rect | trapezoid | triangle | circle) instead of a select. Chosen shape renders large with live water fill, animated surface, and `dimLine` dimensions (b, y, 1:m side slopes, D).
- Two big mode tabs: **Find Q (given y)** and **Find y (given Q)**. Find-y runs Newton on `Q(y)` and ANIMATES the iterations: a small table (iter, y, Q(y), error) types itself row by row at 4 rows/s while a marker slides on the y axis — the solver becomes a lesson.
- Results as stat cards: Q, V, Fr, τ0 = γRS. Lining check: permissible-velocity chips (grass 1.5 | earth 1.0 | gravel 1.8 | concrete 6.0 m/s) — exceed → `--sc-warn` card "your channel is eroding."
- Slope entry accepts S0 or "1 in x" (linked inputs). n presets with photos-as-words (smooth concrete 0.013, earth clean 0.022, weedy 0.030, natural stony 0.040).
**Acceptance:** switching shapes preserves Q intent in Find-y mode; iteration table always converges < 12 rows for sane inputs; Fr chip flips regime color at 1.0.

### 6.2 WL-6.2 · `specific-energy` ENHANCE — width AND hump (1 session)
Add a disturbance selector: **Hump Δz** (existing) | **Width contraction b₂** (NEW) | **Both**.
- Width mode: plan-view inset shows the flume narrowing b₁→b₂; unit discharge jumps to `q₂ = Q/b₂`, so the E–y diagram now draws TWO curves (q₁ solid, q₂ lighter) and the state point hops between them at the contraction; `E_min,2 = 3/2·(q₂²/g)^{1/3}`.
- Choke logic unified for all modes: required `E₂ = E_min,2 + Δz`; if `E₁ < E₂` the section chokes: upstream depth rises (animated backwater fill + rising y₁ marker on curve 1) until energy suffices, throat runs critical (yc marker pulses). Banner: FLOWS THROUGH / AT THE LIMIT / CHOKED, with the energy deficit number.
- Readouts: y₁, y₂ (subcritical + alternate shown ghosted), yc₂, Fr₁, Fr₂. Mini table: section, b, q, y, V, E.
**Acceptance:** with Δz = 0, narrowing b₂ alone can choke the flow; the two-curve diagram and the flume animation always agree on which branch the throat rides.

### 6.3 WL-6.3 · `froude` ENHANCE — make it click (1 session)
Keep the ripple tank; add three clarity devices:
1. **The race strip:** a side lane where two arrows sprint each second: wave celerity `c = √(gy)` vs flow `V`, lengths to scale, numbers attached. Fr = V/c printed as the score.
2. **Upstream-message test:** a button drops a pulse at a downstream marker; the sim tracks whether any ring reaches the upstream flag. Subcritical → flag lights green "message received"; supercritical → rings sweep away, flag grey, caption "the flow outruns its own news."
3. **The wedge, measured:** for Fr > 1 draw the Mach-style wedge with a protractor arc and `β = asin(1/Fr)` substituted live; slow-motion toggle (0.25×) for ring geometry.
Regime banner colors match the global semantic tokens.

### 6.4 WL-6.4 · `weir` REBUILD-lite — a real measurement lab (2 sessions)
Structure menu with four devices, side elevation drawn properly for each, nappe animated (particle sheet over the crest, trajectory parabolic, aeration gap shown for sharp crests):
1. **Rectangular sharp-crested:** `Q = (2/3)·Cd·√(2g)·b_eff·H^{3/2}`; suppressed b_eff = b; contracted `b_eff = b − 0.1·n·H` (n = contractions selector 0/1/2). Cd default 0.62, Rehbock toggle `Cd = 0.611 + 0.075·H/P`.
2. **V-notch:** `Q = (8/15)·Cd·√(2g)·tan(θ/2)·H^{5/2}`, θ ∈ {30°, 60°, 90°}, Cd = 0.58. Caption: the low-flow specialist.
3. **Broad-crested:** `Q = Cv·1.705·b·H^{3/2}` (critical control on the crest shown: yc marker ON the crest).
4. **Parshall flume (NEW):** throat W selector {0.5, 1, 2, 3 ft} drawn in plan + elevation; free-flow `Q_cfs = 4·W_ft·Ha_ft^{1.522·W^{0.026}}`, computed in US units and converted (state this in the calc table exactly like scsuh does). **Submergence:** Hb slider; ratio chip Hb/Ha; > 0.7 → banner "submerged: free-flow rating invalid", rating curve greys.
- Shared: drag the head ruler on the canvas to set H (existing feel, keep); rating curve (Chart2, log-log toggle) with the live point; sensitivity card `dQ/Q = 1.5·dH/H` (or 2.5 for V-notch) — "a 2 mm reading error costs you X %"; calc table with full substitution.
**Acceptance:** all four devices give continuous Q as H sweeps; Parshall submergence kills the reading exactly at 0.7; V-notch beats rectangular for resolution below ~5 L/s in the sensitivity card.

### 6.5 WL-6.5 `compound` (¼ session) — reskin sweep + zone shading for the DCM panels (main channel vs floodplain colors), the rating-curve kink annotated with a callout, floodplain-n slider labeled with land descriptions.

### 6.6 WL-7.1 · `openchannel` ENHANCE — living water (1 session)
Add tracer particles to the flagship: surface tracers + mid-depth tracers advected with `u(x) = q/y(x)` per reach (q changes at width transitions), spacing auto so ≤ 120 alive, opacity fade at reach ends, subtle vertical bob from the surface sine. Speed is the message: particles visibly sprint through supercritical reaches and crawl through backwater. Toggle chip "tracers". Also: dam/gate glyphs get the v2 draw style; EGL/HGL toggle if not present. Touch nothing in the solver.

### 6.7 WL-7.2 · `profiles` REBUILD — from drill to explainer (2 sessions)
Delete the flash-quiz. New build, same key: **"Water Surface Profiles, Explained"** — a scrolling set of animated panels inside the sim page (not slides; sections with their own small canvases):
1. **The two depths:** one canvas, yn (green dash) and yc (red dash) with sliders for S0 and Q; the five slope classes M/S/C/H/A appear as the lines cross/merge; the current class is named live.
2. **The three zones:** pick a class → band diagram; hover a zone → the GVF fraction `dy/dx = (S0 − Sf)/(1 − Fr²)` renders with the SIGN of numerator and denominator shown as ⊕/⊖ chips and the resulting arrow (rising/falling toward what). This is the reasoning engine, made mechanical.
3. **Cause gallery:** six cards, each a looping mini-animation of a real trigger integrating the actual GVF equation (RK4, reuse the openchannel integrator if importable): M1 behind a dam · M2 to a free overfall · M3 under a sluice · S1 after a jump at a slope break · S2 entering a steep chute · A2 on an adverse apron. Each card names the profile, the control, and one sentence of why.
4. **The classification map:** an interactive flowchart (SVG in DOM, not canvas): answer two questions (slope class? y vs yn, yc?) by clicking → the profile name lights. 
5. **Try me:** three-question mini-check (kept tiny; radio cards, instant feedback) so the old drill's value isn't fully lost.
Theory panel is thin here (the page IS theory); include only "Common trap: Fr decides which way disturbances propagate, so subcritical profiles are computed upstream, supercritical downstream."
*(`jump`, `sluice`, `stepmethod` get only the Phase-0 reskin sweep + tracer particles where cheap.)*

---

## 7. PHASE 6 — SEDIMENT & SCOUR polish (module 08) (1 session total)
These three are already the file's graphical high bar; align them with the kit, then add one device each:
- **WL-9.1 `shields`:** magnifier inset on the hero grain (zoomed force diagram: drag, lift, weight, contact) that composes vectors as θ changes; crosshair trace on the Shields diagram showing the state's path as sliders move; unify particle style with `Particles`.
- **WL-9.2 `scour`:** add time-development mode: `ys(t) = ys_eq·(1 − e^{−t/T})` with a scrubber (T mapped from flow intensity), sediment particles ejected from the hole while it deepens, and a foundation-risk meter (depth remaining) replacing the flash with a calmer countdown bar; keep the HEC-18 equation strip as the live substitution.
- **WL-9.3 `traplife`:** add a capacity-vs-year Chart2 line under the long-section, synced to the scrubber; make inflow/sediment sliders re-derive the Brune point live; one added chip: "sluicing" toggle that halves trap efficiency (gates open in monsoon) to show management mattering. Sri Lankan framing in theory: this is the biography of every ancient tank.

---

## 8. PHASE 7 — CALIBRATION ARENA v2 + FINAL SWEEP

### 8.1 WL-11.5 · `calib` RELOCATE + REBUILD (2 sessions) — module 10, beside the SuperApp
Keep the 4-parameter model and response-surface DNA. Additions, in priority order:
1. **Cal/Val split:** a draggable divider on the observed record (default 70/30). Metrics computed and displayed for BOTH windows in a two-column scorecard: **NSE** `1 − Σ(o−s)²/Σ(o−ō)²`, **KGE** `1 − √((r−1)² + (α−1)² + (β−1)²)` with its components r, α = σs/σo, β = μs/μo shown small, **RMSE**, **MAE**, **PBIAS** `100·Σ(s−o)/Σo`. Validation column gets the emphasis styling; a gap chip `ΔNSE = NSE_cal − NSE_val` turns `--sc-warn` above 0.15 with the caption "you memorized the wet season."
2. **Watch it train:** an **Auto-calibrate** button runs an animated random-search/DE-lite: population 12, 40 generations, ~6 evals per frame; every candidate flashes as a dot on the response surface, the best-so-far trail draws in `--egl`, the simulated hydrograph morphs toward the observed one, and a convergence chart (best NSE vs evaluation) grows. Pause/step buttons. Objective selector: NSE | KGE | RMSE — re-running with a different objective lands somewhere different, which IS the lesson.
3. **Equifinality kept and sharpened:** after auto-cal, the "equally good" cloud (all evals within 0.02 of best) highlights on the surface; a chip counts them.
4. **Overfit preset:** one click loads a record whose calibration window is wet and validation dry; auto-cal then demonstrates the gap chip firing. Theory v2: what each metric rewards, why validation is the only honest number, and the bridge line: "the Living Catchment next door is this model's world; here you learn why fitting it is not the same as knowing it."
**Acceptance:** auto-cal reaches NSE_cal ≥ 0.85 on the default record within 480 evals; metrics match hand-checked values on a 10-point toy series (assert in console once); divider drag recomputes both columns live.

### 8.2 Final sweep (1 session)
Reskin any sim not touched by name (`venturi`, `jet`, `siphon`, `moody`, `pump`, `pumpbuild`, `montecarlo`, `risk`, `jump`, `sluice`, `stepmethod`): apply theme, Chart2 for any chart, semantic line colors, legend chips, chip/callout labels, surface animation on any standing water. No feature changes. Then the master QA pass below, update the hero copy (counts auto-derived; lead line can stay "Learn hydraulics by breaking things."), and bump a visible version tag in the footer: "Water Lab v2".

---

## 9. MASTER QA CHECKLIST (run at the end of every phase)
1. `node --check` on the extracted script passes.
2. Hub: counts correct, no dead cards, every card routes.
3. Every changed sim: open → interact → leave → reopen; CPU idle after leaving (cleanup verified); no console errors or NaN on screen at slider extremes.
4. Deep links work for all keys, `#/sandbox` redirects to `#/catchment` (after Phase 1).
5. Mobile 380 px: stage stacks, canvases fit (`fitWidth`), tables scroll horizontally, no clipped controls.
6. `prefers-reduced-motion`: sims render and update per input without continuous animation.
7. Mass/energy sanity spot-checks: SuperApp ledger ≤ 0.5 %; hammer period 4L/a; freqlab CSV round-trip; Newton network residual < 1e−6 m.
8. Brand: no brown/teal chrome, no em-dashes in copy, every plotted color labeled.

## 10. PHASE ORDER & SESSION LEDGER (tick as completed)
- [x] **P0a** Shared kit: Anim, Particles, drawKit, Chart2 — DONE by Fable 2026-08-03, see §0.4
- [x] **P0b** DataTable + theoryPanel v2 + scene tokens/themes — DONE by Fable (CSS + JS inserted); theme reskins of individual sims happen inside their own items
- [x] **P0c** WL-CAT removals + new CATALOG + hero counts — DONE 2026-08-04. 13 sims retired (functions, SIMS, CATALOG, T, ICONS all cleared, zero residual refs); CATALOG rebuilt to the 10-module structure; hero and footer counts now derived from CATALOG.
- [x] **P1 C-1** SuperApp engine + asserts — DONE by Fable (`CatchmentEngine`, closure < 1e-12, forest damps peak 45%)
- [x] **P1 C-2** SuperApp scene painter — DONE 2026-08-04 (hillslope: sky/rain particles, canopy scaled by forest slider, soil moisture fill, groundwater table, overland sheet with flowField, channel with live stage, live flux callouts)
- [x] **P1 C-3** time control + binding — DONE 2026-08-04 (play/pause, 24 h scrubber, 1x–8x speed, chart click seeks)
- [x] **P1 C-4** chart deck — DONE 2026-08-04 (hyetograph barsDown + stacked quickflow/baseflow hydrograph; storages C/S/G; ledger readouts with closure %)
- [x] **P1 C-5** compare runs, retire sandbox, polish — DONE 2026-08-04 (up to 4 ghost runs with Clear-fell / Reforest presets; `sandbox` deleted; `ALIAS` map redirects #/sandbox → #/catchment, #/pond and #/resyield → #/tankcascade)
- [x] **NEW · `tankcascade`** Tank Cascade Simulator (ellangawa), module 10 — DONE 2026-08-04. Daily season model, 2–5 tanks, spill/seepage/issue cascade downward, "abandon the upper tanks" toggle. Not in the original plan; added at Thushan's request as the first of the new-app series. Engine `cascadeRun` lives beside the sim, not in the kit.
- [x] **P2** WL-10.4 freqlab v2 — DONE 2026-08-04. Editable Year/Q table (paste, CSV in/out, sort, add/delete) with live validation chips; Sample A (32 yr) and Sample B (18 yr) loaders; Weibull/Gringorten/Cunnane; Gumbel and normal paper; Q linear or log; all five methods from `Freq`; quantile table (9 design T) and GoF table (RMSE/MAE on log₁₀, PPCC, closest-fit marker); design card with Gumbel ±1 SE; results CSV. Acceptance met: Sample A methods spread 1.8% at T=10 and 54.4% at T=200. Legacy `FREQ` maths deleted; the remaining `FREQ` is a 12-line shim over `Freq`, to be removed with WL-10.3.
- [x] **P2** WL-10.3 freqfit enhance — DONE 2026-08-04. Rebuilt on `Chart2.scaleGumbel` with a proper top-T axis; the sampled-from truth is now drawn as a thick navy line so the fits can be scored against it; per-method colour and dash; disagreement envelope filled 10% egl; residual strip (obs − fit in log₁₀) under the main plot with a method selector; n-slider 15–80; Q₁₀₀ answers as stat cards with percentage error against truth. `FREQ` shim and `probPaper` deleted: `Freq` is now the only frequency implementation in the file. Acceptance met: mean |error| in Q₁₀₀ falls from 83 to 73 m³/s going from n=20 to n=80 over 12 records.
- [x] **P2** WL-10.5 reliability — DONE 2026-08-04, decision was REBUILD-lite as planned (not removal). Animated design-life run: one EV1 stage sample per year at 0.15 s, water rises to it, overtopping years splash the crest and flood-tint the town with a running tally against the formula's prediction. Freeboard drawn with `dimLine`; scene moved to the light field theme with `SC` olive-grey soil (legacy brown `earthGrad` no longer used here); cost curve rebuilt on `Chart2` with build, expected damage and total, optimum marked. Added an earthwork cost-rate slider. Acceptance met: over 400 simulated design lives the empirical chance of at least one flood is 84%, against 84% from R = 1−(1−P_f)^n.

**Phase 2 complete.** `Freq` is the single frequency implementation in the file; `FREQ` and `probPaper` are gone.

### Off-plan changes (2026-08-04, at Thushan's request)
- **`tankcascade` removed** the same day it was built. `#/pond` and `#/resyield` now alias to `catchment`. If it is ever wanted back, the daily cascade engine (`cascadeRun`) is in the transcript for that session.
- **Homepage carries no progress counts.** The hero stats block (simulators / modules / flagships) and the per-module "N live" chips are gone, along with the footer count line. The hero now offers two ways in instead: the catchment flagship and one channel. `smoke.js` asserts no digit-plus-"simulators/modules/live" string can reappear on the hub, so a future session cannot quietly add them back.
- **Module labels now derive from `CATALOG`** via a `MODOF` lookup in `renderSim`, instead of the stale `module:` string stored on each SIMS entry. Before this, twenty-five sims claimed module numbers that P0c had renumbered away. The flagship icon tint now keys off `s.flag` rather than matching the word "Flagship" in that string. `smoke.js` opens all 41 sims and asserts each page's eyebrow matches its catalog module. → WL-10.3 freqfit (1) → WL-10.5 reliability (1)
- [x] **P3** WL-2.1 pipeflow — DONE 2026-08-13. Rebuilt as a free network editor per §4.1: toolbar modes (Select / +Junction / +Reservoir / +Pipe / Delete), 20 m snap grid, click-A-then-B pipe drawing with a rubber band, per-element inspector (junction z + demand, reservoir H, pipe D / C / auto-or-manual L), delete removes incident pipes, JSON save/load textarea, presets Two-loop classic and Hill town branch. Solved on the kit's `solveNetwork`; junctions filled on a p/γ ramp (red < 0, warn 0–10, water above) with the value under each node; pipe width and dash speed scale with |Q| and the arrows point downhill in head; HGL path tracer draws ground z against HGL on `Chart2` and rings any node where the grade line dives under the ground; pipe and node result tables with CSV export. Equations substitute live for whatever is selected.
- [x] **P3** WL-2.5 hammer — DONE 2026-08-13. Rebuilt on `MOCEngine` per §4.2. The whole run is precomputed (Δt = Δx/a, Courant 1, 33 nodes) so the scrubber is exact and free; the pipe is coloured segment by segment by head and bulges with it, hatching where H < −10 m; the steepest gradient node is marked so the wave front is visibly seen travelling and reflecting; reference lines for steady H, H₀ + aV₀/g and −10 m; valve glyph closes over t_c on a linear or equal-percentage law; H(t) at valve and midpoint on `Chart2` with 2L/a gridlines labelled by round trip. Banner calls "effectively sudden" when t_c ≤ 2L/a and otherwise quotes Michaud, with the simulated rise beside the closed form.
- [x] **P3** WL-2.2 serpar — DONE 2026-08-13. Abstract diagram replaced by a physical scene per §4.3: drawn supply tank, spinning pump glyph, three pipes drawn to their real diameter, standpipes whose water climbs to the local HGL. Series steps the HGL down joint by joint with the steepest step over the skinniest pipe; parallel shows one shared Δh across both manifolds. Darcy-Weisbach with Swamee-Jain f, bisection on Q. Tracer speed follows each pipe's velocity so continuity is felt. Stacked loss bar (series) and flow-split donut (parallel), equivalent single pipe for both.
- [x] **P3** WL-1.1 egl — DONE 2026-08-13. Rebuilt as a segment editor per §4.4: 2–6 segments each with L, D, material (PVC / steel / concrete / old cast iron), ΣK and a rise at its end so the profile can climb; two reservoir levels; optional booster pump (vertical EGL jump, drawn with an arrow) and part-closed valve (step drop). Bisection on Q with Swamee-Jain f each iteration. Longitudinal profile with ground body, pipe drawn to diameter, EGL and HGL, sub-atmospheric zones shaded and the minimum-pressure callout placed exactly where it happens, plus a per-segment table (V, Re, f, h_f, fittings, min p/γ) with velocity warnings outside 0.6–3 m/s.
- [ ] **P3 remainder** WL-1.3 thrust (½) → WL-3.3 npsh (1) → WL-3.4 impeller (1)
- [x] **P4** WL-4.3 areal — DONE 2026-08-14. Three methods on one gauge set: arithmetic, Thiessen (Voronoi cell areas counted on a 140² lattice clipped to the basin) and isohyetal (`GeoField.idwGrid` → `marchingSquares` → `chainSegments`, with `bandArea` giving the area between successive isohyets and a masked basin test). Compare tab bars all three. Draggable gauges, add/remove, weighting table. The excess-rainfall chart maps each method's areal depth onto the SCS P–Q curve, so the disagreement is expressed in runoff rather than millimetres. **Performance gotcha, do not undo:** `chainSegments` is O(n²) in segment count, and calling it inside `recompute()` locked the page during drags. Contours are now traced lazily in `isoFor()`, cached on a gauge-configuration key, and only when the current tab actually shows them; the IDW grid is 44×34, not 64×48.
- [x] **P4** WL-4.4 designstorm — DONE 2026-08-14. Each of the four named patterns is drawn as its own hyetograph card with the φ-index loss shaded off the bottom of every block, so the excess hyetograph is visible rather than implied. Hydrograph deck shows the selected pattern solid with the other three ghosted behind it, hyetograph hanging from the top axis in the same two tones. Hour-by-hour table (rain, loss, excess, cumulative). Theory names each method and explains why φ punishes gentle patterns hardest.
- [x] **P4** WL-5.3 scs — DONE 2026-08-14. Field theme. The ground itself is the graphic: canopy density and crown size fall as CN rises while roofs and tarmac spread in from the right, rain particles fall, a wetting front descends through the soil in proportion to F, and an overland sheet slides off the surface with thickness set by Q. Three-way split bar (Iₐ / F / Q) over the storm. P–Q chart on Chart2 with ghost CN curves, the 1:1 ceiling and the initial-abstraction band shaded. Calc table + worked example.
- [x] **P4** WL-5.1 rational — DONE 2026-08-14. Aerial catchment view: procedural canopy, roofs and a road grid mixed in the proportion C implies (deterministic pseudo-random so it does not shimmer), drainage network converging on the outlet, and the longest flow path highlighted as the thing that actually sets t_c. IDF chart with neighbouring return periods ghosted, land-cover comparison table showing what each cover would give with everything else fixed.
- [x] **P4** WL-5.4 scsuh — DONE 2026-08-14. Map sketch showing where each of the four numbers is measured: the divide scaled by A and stretched by L, contours tightening with Y, the longest path drawn with flow arrows, a slope wedge, and a CN tint. Hydrograph on Chart2 with the triangular equivalent (t_b = 2.67 t_p) overlaid and the unit rain block D hanging from the top. Ordinate table at t/t_p, worked example stepping through the imperial lag equation.
- [ ] **P4 remainder** WL-4.2/4.5 (1) → WL-5.2 uh (1)
- [x] **P5** WL-6.4 weir lab — DONE 2026-08-14. Rebuilt in elevation rather than the old front view: a real nappe trajectory springing off the crest with the aerated pocket drawn under it, splash particles where it lands, approach-flow tracers, and dimension lines for H, p and tailwater. Tailwater slider drowns the structure via Villemonte, and the rating chart then shows the free and submerged branches together. Two new types beyond the original three: Cipolletti, and a throated flume (no step to silt up, tolerates ~80% submergence against a weir's ~20%, which is the whole reason flumes get chosen). Rating table quantifies how a 5 mm head error propagates.
- [ ] **P5 remainder** WL-6.1 (1) → WL-6.2 (1) → WL-6.3 (1) → WL-6.5 (¼) → WL-7.1 tracers (1) → WL-7.2 explainer (2)
- [ ] **P6** WL-9.x polish (1)
- [ ] **P7** WL-11.5 calib v2 (2) → final sweep + QA (1)

### Off-plan changes (2026-08-13, at Thushan's request)
- **UI v2.1.** Header nav (Home / Apps / About) with an active-link state. Equation strips are now titled cards ("Equations, live") with substituted numbers highlighted, per-line horizontal scroll instead of overflow, and a `.note` class for prose lines. Study-aid strip added under the footer. Card, button and control polish.
- **Home page reshaped.** The numbered per-module dump is gone from the home page. It now runs: animated hero (a CSS-animated inline SVG of the whole water cycle, from cloud through canopy and soil to a gauged river, with a readout card drawing its own hydrograph) → animated "Created by Thushan Chamika" signature card → **Popular right now**, ten cards → **Browse by topic**, ten tiles with no numbers. Full listing moved to `#/apps`, one topic to `#/m/<nn>`. `POPULAR` pins catchment, manning, openchannel, calib and reliability as requested, then pipeflow, hammer, freqlab, specific-energy, weir. Animations are pure CSS so `prefers-reduced-motion` disables them.
- **About page** at `#/about`, modelled on the Physics Study Aid about page: person card, Curious LK quote block, study-aid disclaimer, help-me-make-it-better, dedication, and email / LinkedIn / Facebook contact rows. Self-contained, no external assets.
- **Note on "gif".** Thushan asked for animated GIFs. Both the hero and the signature are inline animated SVG plus CSS keyframes instead: crisp at any size, a few KB rather than a few hundred, theme-aware, and they honour reduced-motion. Swap to real GIFs only if the file must survive a context that strips CSS animation.

### The 2026-08-13/14 request, closed out
Everything asked for on those two days is now built and verified, except as noted below.
- **Remove or improve 8.3 / 7.5 / 5.3 / 5.4.** All four were read and judged worth keeping rather than removing: each teaches something no other sim in the file covers, and all four already showed their equations. All four were improved instead. `stepmethod` now draws the profile as the table fills, marks every computed section on the water surface, dimensions Δx between the last two, and puts that step's arithmetic in the equation strip. `traplife` moved to the field theme with a decade-banded sediment wedge advancing on the dam, silt visibly escaping over the crest as TE falls, and capacity plus trap efficiency on one Chart2 with half-life and useful life marked.
- **`moody` was deliberately left alone.** It is a chart navigator that already reads well, and nothing in the request pointed at it. Not an oversight.
- **Still open:** WL-4.2/4.5, WL-5.2 `uh`, WL-6.1/6.2/6.3/6.5, WL-7.1, WL-7.2, the P3 remainder (thrust, npsh, impeller), P6 and P7. These were never part of the 13/14 August ask; they are the rest of the original plan.

### Verification on a machine with no Node
This machine has neither Node nor Python, so `scripts/check.sh`, `test_engines.js` and `smoke.js` cannot run. Two replacements were built and live in the session scratchpad:
- `balance.ps1` — extracts the last `<script>` block and checks bracket balance with a small tokenizer that understands strings, template literals, comments and regex literals. Catches the class of damage a bad splice does.
- `smoke.ps1` + `harness.js` — injects a smoke harness into a copy of the file and runs it under **headless Edge** (`--headless=new --dump-dom`), which is a real browser engine. It walks all six routes, asserts the home page shows ten popular cards, ten topics and zero full-app cards, then builds every sim, drives every slider to both extremes and back, cycles every select, clicks every non-animation button, toggles every checkbox, scans the rendered text for NaN/Infinity, asserts every sim renders an equation strip, calls `cleanup()`, and checks CATALOG/SIMS/POPULAR integrity. Two gotchas worth remembering: strip the Google Fonts `<link>`s from the test copy or offline headless stalls for minutes waiting on them, and never pass `--virtual-time-budget`, which turns every `setInterval` in the file into a tight loop.

Result 2026-08-13 after all four pipe rebuilds: **41/41 sims built, 0 errors, 0 NaN, 0 sims without equations, all 6 routes ok.**

### Original verification (when Node is available)
```
bash scripts/check.sh water-lab.html        # syntax, duplicate functions, registry counts
node scripts/test_engines.js water-lab.html # 42 numerical asserts on the embedded kit
node scripts/smoke.js water-lab.html        # jsdom: opens EVERY sim, drives every control, checks cleanup
```
`smoke.js` needs `npm install jsdom`. It catches what `node --check` cannot: a sim that throws on open, a slider that breaks at its extreme, NaN reaching the screen, a catalog card pointing at a sim that no longer exists. As of 2026-08-04 all three pass: 41 sims open clean, 28/28 smoke asserts, 42/42 engine asserts. The smoke file now also carries the acceptance criteria for WL-10.3, WL-10.4 and WL-10.5, so a later session cannot silently break the pedagogy those sims were built to deliver.

**Total ≈ 34 focused sessions.** Rule of thumb per session: read the item, read the target function, build, syntax-check, smoke-test, tick the box, note anything learned in a `<!-- CHANGELOG -->` comment at the top of the script block.
