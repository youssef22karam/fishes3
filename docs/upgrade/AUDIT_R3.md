# HyperFish — Round 3 baseline audit (BEFORE any R3 edit)

Method: `node tools/hr3_shots.mjs docs/upgrade/r3/before 1` renders the 12 fixed audit seeds (`tools/hr3_lib.mjs: AUDIT_SEEDS`) in
8 views (side, head34, mouth, gillOpen, eye, scale, fin, belly) with the existing rig (`FG.hyperRig.shot`) in headless Chromium / SwiftShader.
Files: `docs/upgrade/r3/before/<seed>_<view>.png`.  I looked at the images listed in each row; I did **not** look at every one of the 96 frames,
so the table lists what I saw, not a claim of completeness.

Environment notes: the repo really stores `fishgen.js` / `gillchamber.js` as `parts/` (README packaging note TRUE, README changelog "removed parts/" FALSE;
UPGRADE_PLAN.md "parts are not used any more" FALSE). R3 consolidates: the assembled files are now the committed source, `parts/` and `reassemble.mjs` are gone.

| # | Earlier defect from the brief | Still present? | Evidence |
|---|---|---|---|
| 1 | Gill cover reads as a cylinder / collar around the head | **YES, severe** | `R31_head34`, `R1_head34`, `R40_side`: the cover is a ring-shaped band whose rim leaves the head silhouette at the top and bottom; it reaches the belly line (θ = −1.5 rad) |
| 2 | Red filaments outside the head silhouette | **YES** | `R31_head34`, `R1_head34`: pink / red spikes stand out of the band, also in the default (closed-ish) side views `R31_side`, `R40_side` (red visible with the cover "closed") |
| 3 | Body or head reads as cone / bullet / tube, no forehead / nape / cheek | **YES** | `R31_side`, `R8_side` (conical head, straight dorsal line to the snout, no cheek bulge, no concave forehead) |
| 4 | Steps / seams where parts join | **YES** | the cover band is a separate shell with a visible skirt step against the head (`R31_head34`) |
| 5 | Flat red stripe for a mouth, teeth painted on | **YES (mouth) / not seen (teeth)** | `R19_mouth`, `R8_side`: closed mouth is a thin red line painted into the face; no lips, no rim highlight |
| 6 | Flat disc eyes | **YES** | `R1_head34` (flat yellow disc), `R19_mouth` (flat ring + dark disc, no bulge / lid / orbit rim) |
| 7 | Fins white fuzz / single-coloured / floating | **PARTLY** | `R8_fin`: tail is a flat comb-edged plate (single hue, serrated edge from ray quads), pectorals in `R31_side` are thin yellow slivers; some fins start off the surface |
| 8 | Cap / rim / vertical seam where the tail meets the peduncle | **YES** | `R8_fin`: round cap + hard vertical edge between peduncle and tail plate |
| 9 | Black halo under the belly | not seen in the frames I viewed | `R40_side`, `R31_side` have no dark rim; **not verified for all 12 seeds** |
| 10 | Pixelated blocks | **YES** | `R31_head34`, `R19_mouth`: rectangular steps on the snout / cheek and on the peduncle scale lattice (the scale lattice is evaluated in (u, θ) which pinches at the snout tip). The textures themselves were already linear-filtered; the blocks come from the lattice, not from nearest filtering |
| 11 | Matte "silver" that is not chrome | **PARTLY** | `R40_side` flank is a believable silver sheen; the cover "chrome" in `R31_head34` is a flat grey-blue |
| 12 | Fish clipped by the viewport | not seen in `side`; **YES in `R1_head34` / `R31_head34`** (head framed so the body is cut, intended for close-ups) | |
| 13 | Impossible hybrid anatomy | not individually inspected (hybrid frames not viewed) | |

Other defects seen: tail of `R40` is a hair-thin crescent with no membrane colour variation; the pectoral fin of `R40` is a single dark slab;
head surface of `R8` has regular bright vertical streaks (myomere pattern projected onto the cheek).

Functional checks: `npm run build` OK (1.57 MB single file). Page boots with no page errors. `FG.hyperRig.validate` runs.
