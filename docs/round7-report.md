# ROUND 7 — Re-sculpt the head: report

Baseline audit: `docs/upgrade/AUDIT_R7.md` (sections A0–A8). The shipped
tree's four Node gates did not run at all before this round (harness stubs);
the A0 harness fixes (headless stub `document.head/body`, `installRig`
early-return without a DOM) touch no validator and no threshold.

## Strategy choice (prompt §2 allows A or B, with justification)

**Strategy B** — landmark-driven profiles + displacement fields inside the
existing loft. Every consumer of head geometry (mouthgeo lip line, gillgeo
sampler, eyegeo seating, the coherence validators, fin attachment) is built
on `bm.frame/section/point`; Strategy B keeps ONE manifold surface, so
skin-field continuity across head/cover/jaw is exact by construction and
every existing gate keeps its meaning. Strategy A (a separate welded SDF
head mesh) would have forked the surface every one of those systems
samples, and its weld seam would sit exactly where the red-flag list
forbids seams.

## Deliberate law changes (ordered by the prompt, recorded here)

1. **`laws.js` CAVITY_MAX .30 → .62.** The 0.30 cap forced the oral pocket
   to be a shallow wide bag — the flat red wedge in every open-mouth
   render. The genome schema already rolled `mouth.cavityDepth` in
   0.2–0.85; the law now lets the funnel reach a pharynx cap in the
   prompt's 0.45–0.75 HL design band. All enforcements read the constant
   (mouthLaw clamp, lawaudit, the fishgen mouth audit) — one change,
   consistently applied; the stale "0.35" message texts were corrected.
2. **mouthgeo header law: "NO lip ring/tube/torus" replaced.** The lips
   are now welded volumes (`buildLips`): a rounded roll swept along the
   seam, sunk ~30 % of its radius into the surface, split at the seam —
   static upper roll, jaw-weighted lower roll. This is not the forbidden
   floating torus: it is surface-hugging tissue with a skin-tissue tag,
   and the teeth rows stay rooted inside it.
3. **L1 (coherence eye measurement) — measurement correction, NOT a
   weakening.** `eyeOutside` located the eye by nearest-spine-point
   search, biased ≈ +0.014 u backward on short curved heads; three boxfish
   seeds provably legal by parameter margin (gillStart − (eye + R/2L) =
   +0.013…+0.014) were flagged. The organ info now carries its true seat
   station (`u: spec.u`); the predicate and thresholds are unchanged and
   the spine proxy remains the fallback.

## WP1 — landmark head sculpt (landmarks.js, sculpt.js)

- `head2Params(bm)`: sub-seeded (±15 % jitter) drivers from the generator's
  axes only — fineness/elongation, plan family, law cheek mass, eye size.
  Stations: forehead, nape, saddle, orbit dome, cheek, jaw undercut,
  throat, snout rounding.
- `sculptSection` reshapes the loft section (dorsal lobes, ventral
  undercut, cheek/throat width); `sculptPoint` adds the orbit dome and
  cheek bulge radially. The orbit dome carries an **eye-footprint notch**:
  it is exactly zero over the cornea's (u, ly) footprint, so the eye seat
  surface — and every measurement from it — is baseline-identical (three
  earlier dome designs clipped corneas or pushed eyes onto the gill
  region; the notch ended that class entirely).
- Hero builds add resolving stations across the head (`extraStations`).
- Law verified: dorsal-profile curvature sign changes 5–6 on all named
  non-eel, non-disc seeds (gate ≥ 3 in validate7); eels and disc plans
  exempt, as their own plan laws govern their silhouettes.
- Honest limit: at the shipped amplitudes the sculpt is a profile
  correction, not a caricature — it removes the cone/tube read; on
  elongated plans it is deliberately subtle.
- **Orbit-dome moat calibration (third verify run: 996/1000, EYE_SLICED
  on small-eyed boxfish):** the dome's eye notch started at 1.2× the
  cornea footprint — a small eye sat in a raised crater and the rim
  sliced 4–16 % of the cornea in the oblique ray-cast (CORNEA_VIS_MIN
  .97), worst for eyes the seat refinement had sunk to the protrusion
  floor. The moat is now 3.4× / 3.2× the footprint (the dome reads as a
  brow swell ≥ 3 R from the eye). A 120-boxfish scan through the full
  geometry audit went from ~40 % failing to 0.
- **Verify-driven correction (first full verify run: coherence 912/1000,
  "Mouth cavity breaks out of the head"):** sculptPoint's angular cheek
  term originally added up to +7 % radial at the hinge — outside the
  section model (bm.section/halfWidthAtY) that the cavity containment
  audit measures against (slack 1.02), so cheek-wall vertices riding
  the surface failed as "outside". The cheek MASS lives in
  sculptSection's width lobe (audit-visible); the angular term is now
  .012 — it only breaks the ellipse's uniformity. All ten named failing
  seeds re-probed at 0 out-vertices, and the shark gill-slit margins
  (same refill mechanism) recovered with it.

## WP4 — gill cover, chamber, gillplan (gillplan.js + gillchamber)

Already present in the R3 build and kept: the shield outline with the
swept-forward lower tip (`bounds()`), 4 cover pieces + flap +
branchiostegal membrane (5–8 rays), hinge + breathing in the vertex
shader, recessed basin, pseudobranch, rest state nearly flush
(1.4–5.6°, breathing 3–12° at 0.5–2 Hz).

New in this round:
- **gillplan**: the axis table as code. Kind by plan (cover / slits 5–7 /
  ventral / pores); cruiser (ram) fish get filament count ×1.3–1.5 and
  length ×1.3–1.5 with 20° flare class; bottom ambushers ×0.7–0.9;
  filter/planktivore flags → sieve rakers 80–180 (long, thin); crushing
  tooth class → knob rakers (short, thick); predator/ambusher → sparse
  rakers 8–20. Signatures are exposed on the head-system API for the
  census (validate7 / r7audit).
- **Filaments are no longer blades**: cross-section 4 → 6 verts, a true
  ellipse ~0.38 as thick as wide, width following an elliptical length
  profile; lamellae striation was already in the shader.
- **The cover wears the fish's own skin field**: cover-family vertices
  carry their skin coordinates (u, sin θ) in aK.zw; the plate shader
  blends 58 % toward the dorsal→belly skin gradient, caps metalness by
  the fish's own scale chrome (uIrid), floors plate roughness at .28
  (a mirror bake washed white in studio light), and flattens the bake's
  crinkle relief on the outer plate (it read as crumpled foil).
- **Chamber fold tightened**: the visibility fold now needs a real
  gill-open (smoothstep .04–.50 rad), and the fragment discard for
  chamber kinds was raised to vVis < .10 — in between, the folded
  chamber still rendered as a tiny red wad at the hinge pivot (a few
  red pixels at the cover's front edge in the first panels).
- **Census calibration**: the cruiser trigger threshold is
  H.derived.cruiser > .40 (not .55): at .55 the N=1000 census showed
  the ram boost almost never fired (cruiser-swim class mean filMul
  1.04, design band 1.3–1.5); the class table is re-verified in the
  final audit run.

## WP2 — mouths (mouthgeo buildLips + funnel)

- Lips as above (law change 2). Diameter 1.6–4.4 % HL by genome lip
  fullness; papillose forms (> .65) swell toward ~9 % with a knobbly
  radius modulation; ends taper into the corners; tissue tag 0 (skin).
  Colour note: lip vertices carry aBody.y as a true section-angle
  fraction — the skin shader recovers vert = −cos(2π·aBody.y); an early
  build mapped it linearly and the lips coloured as dorsal skin
  (near-black). Fixed by inverting the shader's own mapping.
- The pocket is a **funnel**: rings 5 → 7, cross-section shrink
  1 − .8·e^1.15 to the pharynx cap (was 1 − .25e), depth per law change 1.
  The baked occlusion field was re-weighted so depth (not just the
  lateral term) drives the lip→throat colour ramp — the funnel now
  darkens toward the pharynx instead of presenting one flat red wall.
- Teeth: unchanged geometry (already rooted ≥ ⅓ inside the jaw edge,
  envelope-fitted against the same jawEnv the validator checks); they
  now sit behind the lip rolls, which is where the prompt wants the
  gum line.

## WP3 — eyes (eyegeo + orbit rim)

- Round pupil radius .38 → **.50** of the iris radius (spec band
  .45–.65); slit pupils keep their niche shapes.
- IRIS_RECESS .03 → **.055 R**: the iris plane sits deeper behind the
  limbus, so it shifts against the cornea with the view vector — real
  geometric parallax, no shader trick.
- The orbit rim (R3's tube) was coloured as pale bone (kind 2) — in
  renders a perfect cream ring, the "disc eye" flag. It is now kind 16,
  a **skin fold**: body-coloured from the same palette uniforms.
- Iris fibres: the eye shader's striation frequencies (79/131/37 base,
  and the hyper path's uIrisFib 80–200 per fish) already sit in the
  spec's fibre band; the hyper metallic iris ring + pupil ruff are in
  the same shader.
- **Symmetry by construction (same verify run, two iterations):** the
  per-side seat refinement samples the surface independently; on the
  sculpted head the two sides diverge beyond float residue (the 'med'
  loft is faceted between stations and the sculpt's gradients are
  steep near the eye). Averaging the two refined centres fixed the
  symmetry audit but sat one boxfish eye too deep (EYE_SLICED, 995/1000
  on the second run). Final construction: the raw specs are
  symmetrised first, the seat refinement runs ONCE (first side), and
  its scalars (dl, lf, skin-curvature set) are applied mirrored to
  the twin — the proven refined seat depth, exactly mirrored. The
  three failing seeds (two boxfish EYE_SLICED, one symmetry) all pass
  round2Check post-fix with centre deltas of exactly 0.

## WP5 — skin continuity + pixel blocks (hyper/skin.js)

- Continuity: body, cover (WP4 blend), lips/jaw (skin tissue in the
  shared body shader) now all draw from the fish's own palette field.
  The full 12-layer pattern stack still evaluates only in the body
  shader — pattern *motifs* (bars/spots) do not yet continue onto the
  cover; the cover carries the correct countershading gradient and
  palette. Recorded as a residual, not hidden.
- Pixel blocks: the aliasing came from cell-quantised pattern terms
  evaluated past pixel scale. Layer 1 (speckle) and layer 5 (spots) now
  fade by their own cell-space fwidth; the bars layer softens its edge
  by fwidth of the bar coordinate; the wear cell-tint fades at pixel
  scale too. The measurable test ships in tools/r7audit.mjs
  (--shots): 8-px-grid vs off-grid edge-energy ratio per render,
  gate ≤ 1.30, computed by tools/pngread.mjs. Honest calibration: the
  BASELINE renders also score ≈ 1.0–1.11 on this metric — the blocks in
  the baseline playtest shots were pattern cells reading as rectangles
  at close zoom, not a grid-aligned raster artefact, so the metric is a
  guard against introducing grid artefacts (final panel: 1.00–1.06),
  not a before/after proof. The fades target the actual mechanism
  (cells at pixel scale); fine speckle is still visible on some seeds
  at head close-ups — listed in the residuals.

## §9 validators

`src/engine/validate7.js` (profile curvature, lips present, cavity
within law, pupil band, iris recess, eye-size envelope aligned to
eyeLaw, gill signature/kind) and `tools/r7audit.mjs` (named seeds at
hero + population census + axis-correlation table + cavity-depth
distribution + blockiness). Results: see "Verification" below.

## Verification (final code, all runs on the delivery tree)

- **verify: OVERALL PASS** — uniqueness 10,000/10,000 distinct;
  coherence **1000/1000**; mesh audit 1000/1000. (Three earlier runs
  failed and were fixed at the construction, never at the validator:
  912/1000 cavity breakout → cheek-term amplitude; 995/1000 eye
  symmetry → post-refinement averaging replaced by mirrored
  refinement; 996/1000 EYE_SLICED → orbit-dome moat 3.4×/3.2×.)
- **randaudit: PASS** — standard 0/300, wonder SHIPPED 0/294
  (wonder RAW 2 EYE_SLICED — the long-standing marginal hybrid class
  the production filter rejects).
- **pectest PASS · tailtest PASS · vite build PASS.**
- **r7audit: PASS** — named seeds all checks ok; population N=1000:
  0 check failures, **859 distinct gill signatures in 1,000 fish**;
  axis table: ambusher filMul ≈ .85 (band .7–.9), filter/planktivore
  mean plan rakers 127 (band 80–180), cruiser qualifiers at 1.3–1.5;
  cavity-depth median .55 HL.
- **Boxfish eye scan:** 120 random boxfish through the full geometry
  audit — 0 eye violations (was ~40 % EYE_SLICED before the moat fix).
- **Blockiness (WP5 guard metric):** final panel 18/18 renders at
  1.00–1.06 grid/off-grid edge ratio (gate ≤ 1.30).
- **Packaging:** fishgen.js, hyper/gillchamber.js and (new) mouthgeo.js
  are generated from ≤29 KB parts by tools/reassemble.mjs with SHA-256
  verification; reassembly byte-exact; every committed file ≤ 30 KB.

## Residuals, stated plainly

- Cover carries gradient + palette, not the pattern motifs (above);
  on chrome-baked fish it still reads paler than the flank in studio
  light — better than the baseline foil slab, not yet invisible.
- The mouth interior darkens to the pharynx now and the teeth sit
  behind the lip rolls, but the far cheek wall can still read flat
  from a pure side open view, and the lip rolls catch a strong wet
  highlight line — material tuning, not structure.
- Fine speckle remains visible on some seeds at head close-ups
  (WP5 note).
- WP1 sculpt is conservative on elongated plans.
- Wonder-hybrid material failures from earlier rounds (glossy plastic
  tube bodies on some hybrids) are outside this round's head scope.
