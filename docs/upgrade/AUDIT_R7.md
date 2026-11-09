# AUDIT R7 — Baseline audit for Round 7 (head re-sculpt)

Project state audited: fishes3 main @ 5e06364 (the R3 "HyperFish" edition),
reassembled byte-exact (fishgen.js SHA ecfd39de…, gillchamber.js SHA 6b9b82ed…).
Method: every claim in the Round 7 brief was checked against the actual code
first, then against baseline renders (docs/upgrade/r7-baseline/). Nothing is
marked fixed here; this document only records what is true at baseline.

## A0. THE GATES DID NOT RUN AT BASELINE (new finding, blocks everything)

All four Node gates (verify, randaudit, pectest, tailtest) crashed at import
time on the shipped code:

1. `hyper/rig.js` `installRig()` runs at `fishgen.js` import (top-level call)
   and does `document.head.appendChild(...)` — the headless harness stub
   (tools/headless.mjs) has no `document.head`. TypeError, gate dead.
2. After stubbing `head`, the next crash: `document.querySelector` is not a
   function (rig.js builds its whole UI panel set at install time).

So the R3 round's "gates green" could not have been produced by running these
tools against this exact tree in Node. Fixes applied (harness/install level
only — no validator touched, no threshold changed):

- tools/headless.mjs: stub gained `head`/`body` with no-op `appendChild`.
- hyper/rig.js: `installRig` now returns early when `document.querySelector`
  is unavailable (headless). The rig is browser UI; in a real browser nothing
  changes.

Baseline gate results after the fix: PENDING (recorded below when the runs
complete; screenshot renders had to wait for CPU, see A8).

## A1. GILL COVER = RECTANGULAR BAND — CONFIRMED in code

`src/engine/hyper/gillchamber.js` header, line 3–4: the cover/chamber
footprint is defined as "the footprint uFront(θ) … uEdge(θ), θ ∈ [θlo, θhi]"
in body skin coordinates — a rectangle in (u, θ) space, which renders as a
band/collar on the head, exactly as the brief's screenshots show. The cover
also bakes its own texture set (hyper/textures.js), decoupled from the head
skin field, so pattern/palette mismatch at the cover is structural, not a
tuning miss.

## A2. MOUTH = SEAM + HINGED SKIN PATCH, FLAT INTERIOR — CONFIRMED in code

`src/engine/mouthgeo.js` header, lines 1–2: "The mouth is a SEAM in the head
surface (lips meet, resting opening = 0) plus a hinged lower jaw … there is
NO lip ring/tube/torus." Line 48 calls the jaw construction "the lower-jaw
patch edge" — the lower jaw is a patch of head skin on a hinge, i.e. the
trap door in the brief. The cavity is `buildCavity()` capped by
`CAVITY_MAX = 0.35` head lengths (laws.js, enforced in lawaudit.js:55 and
fishgen.js:3355 "Mouth cavity deeper than 0.35 head length"). There are no
lip volumes, no maxilla ridge mesh, no mouth-corner fold — the brief's item 2
is accurate in full.

## A3. HEAD = FRONT OF THE BODY LOFT (CONE) — CONFIRMED by architecture

There is no head mesh, landmark set or head-specific section profile: the
head is the front stations of the single superellipse body loft, with parts
(mouth seam, eye, cover band) placed onto it. No forehead/orbit/cheek/throat
volumes exist as geometry. (bm.landmarks3D exists for rig/screenshot
framing, not for sculpting.)

## A4. EYES — layered eyeball exists; flat read is a shading/depth issue

eyegeo.js does build a layered eyeball (sclera, limbal funnel, iris, pupil
cup, cornea, adipose, lid) and hyper/skin.js adds iris fibres — so the
geometry stack the brief says to KEEP is present. What is missing (confirmed
by absence in code): no view-dependent parallax offset of the iris behind
the cornea, no lens highlight model beyond the shared env probe, and the
eye sits ON the loft surface without a recessed socket/rim mesh — the
"disc/ring" read in screenshots follows from placement + shading, not from
missing layers.

## A5. GILL FILAMENTS = FLAT BLADES — CONFIRMED verbatim

`hyper/gillchamber.js:324`: "flat tapered blade: 5 rings × 4 verts,
4 segments". Two rows per arch, 3–4 arches. No lamellae geometry (a shader
striation only), no clumping, no per-axis adaptation of count/length beyond
the gill-plan scalars in hyper/genome.js.

## A6. PIXEL BLOCKS — prime suspect confirmed; env-probe theory rejected

The brief guessed the CanvasTexture near fishgen.js:2177 — that texture is
the studio environment probe (equirect, LinearMipmapLinear + Linear, mipmaps
on): not the skin, and not blocky by construction. The head-skin pattern
field in hyper/skin.js is computed with `floor()` cell hashing throughout
(hBit line 16, hVor voronoi line 17, scale lattice `floor(g)` line 39, bar/
spot quantisers lines 45/49) with NO fwidth-based anti-aliasing — at head
close-ups these cells land on screen as hard pixel blocks, worst where the
head's vertex density is lowest and UV area per triangle is largest (snout,
cheek). Secondary suspect: the baked per-part textures in hyper/textures.js
at the default bake size. Fix direction: analytic AA (fwidth) on the pattern
cells + sample-at-scale-centre, per WP5.

## A7. CHAMBER DEFAULT STATE — CONFIRMED

The hyper rig's default cover state is the macro display override (cover
flared); the brief's requirement (closed/breathing by default, gill-open as
a labelled view) is not the shipped default.

## A8. Baseline renders — CONFIRMED visually (docs/upgrade/r7-baseline/, 10 seeds × side/quarter/open)

Rendered with tools/r7shots.mjs (new; headshots.mjs hangs in this
environment — playwright's screenshot waits forever on font load in
chrome-headless-shell, so r7shots captures via CDP). Inspected directly:

- head_Q1CW5_open.png (cartilaginous, ventral mouth): eye is a literal
  cartoon target — thick pale ring, gold ring, black dot. Open mouth is a
  flat red slab hanging below the head with small white spikes along its
  edge. Head is a smooth cone. Sparkly pixel noise across the snout/cheek.
  Gill slits read as pale scratch stripes.
- head_QOG9__open.png (elongated ambush): the gill cover is a huge dark
  crumpled-foil slab whose material visibly does not match the head skin —
  the band/barrel defect and the cover/skin mismatch in one image. Open
  mouth is a long flat red wedge running back most of the head length
  toward the cover, with sparse white triangles floating along its edge.
  Eye is a white disc with a black dot. Rectangular purple pattern blocks
  on the back (pattern quantisation visible at render distance).

Every red-flag class in the brief §11.5 that applies to these two seeds is
present at baseline. Nothing in A1–A7 is an exaggeration.

## Baseline gate results (after the A0 harness fixes — the shipped tree ran none of them)

- verify.mjs: **OVERALL PASS** — uniqueness 10,000/10,000, coherence
  1000/1000. Two single-occurrence warnings in the census tail (1× anal-fin
  area below the subcarangiform envelope, 1× EYE_OUTSIDE_HEAD flag) — logged
  as warnings by the tool, not failures.
- pectest.mjs: **PASS** (violations: 0).
- tailtest.mjs: **PASS** (violations: 0).
- npm run build: **PASS** (dist/index.html 1,572,012 bytes).
- randaudit.mjs: **PASS** — standard 0/300, wonder RAW 0/300, wonder SHIPPED
  0/295 fish with violations (all five coherence classes at zero).
