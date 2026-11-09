# HyperFish upgrade — architecture summary & plan (written BEFORE editing)

## 1. What the repo is
* **Language / engine / framework**: JavaScript (ES modules) on **Three.js r160** (raw `ShaderMaterial`s, hand-written GLSL),
  wrapped by a thin **React 19 + Vite + Tailwind** shell (`src/App.tsx` only renders the DOM skeleton, then `boot()`s the engine).
* The engine is one huge module `src/engine/fishgen.js` (originally 20 `parts/`, concatenated by `tools/reassemble.mjs`)
  plus ~25 helper modules (`laws.js`, `gillgeo.js`, `mouthgeo.js`, `eyegeo.js`, `tailgeo.js`, `teethgeo.js`, `genome.js`, `validate.js`...).
  In this project `fishgen.js` is committed and is the single source of truth (the `parts/` are not used any more).

## 2. Where things are generated
| thing | where |
|---|---|
| body loft (spine, superellipse sections, skin coords `u`/angle in `aBody`) | `BodyModel` + `buildBodyGeometry` (fishgen.js) |
| head / jaw patches / mouth interior | `buildJawGeometry`, `buildMouthDetails`, `mouthgeo.js` |
| eyes | `buildEyes`, `eyegeo.js`, shader `FS_EYE` |
| gill cover (old: a raised plate *painted on the loft*) + slit ribbon | `gillgeo.js` (`buildOperculum`, `buildGills`) |
| fins & tail (rays, membrane) | `FinBuilder`, `addRidgeFin/addPectoral/addPelvic`, `tailgeo.js` |
| teeth / barbels / specials | `teethgeo.js`, `appendages.js`, `buildSpecials` |
| skin, scales, patterns, iridescence | **fragment shader `FS_BODY`** (+ `GLSL_PIGMENT.skinPigment`), uniforms in `makeUniforms` |
| materials | `makeMaterials` (body/jaw/fin/eye/simple/tooth/glow) |

## 3. How a fish is created
Seed string → `procgen.js` (`seed → RNG → clade → 59 continuous + 20 discrete trait axes`, clamp) → `mapping.js` (trait vector → engine genome paths
`body.* head.* fins.* skin.*`) → `validate.js` hard gate (8 anatomy invariants, resample on failure) → `buildFish(g, {quality})`.
Seed grammar: `A`, `A~SALT` (mutant), `A^B.SALT` (offspring). Same seed ⇒ same fish.

## 4. How many fish are handled
`QUALITY = {hero, med, lod}` tiers in `buildFish` (detail geometry — jaws, teeth, eyes, gills, fin rays — only when `detail`), `InstancedMesh`
for schools (`School`, `opts.instanced`), adaptive quality tiers (`TIERS`, `adaptQuality`), thumbnail queue for galleries. Detail items are *not*
built for the lod tier.

## 5. Gap analysis vs. the brief
Existing: good loft, laws, scale-field shader, eyes, fins with rays, teeth, validation.
Missing / weak (what this upgrade adds): separate hinged operculum pieces with their own textures; an *open* gill chamber with arches,
filaments, rakers, pseudobranch; ecomorph archetypes + constraint rules + sub-seeded hyper-genome; layered pattern system (11 layers);
lateral line pores, myomeres, scutes, spines, wear/age; richer iris; studio capture rig, contact sheet and automatic validation; LOD policy for the new parts.

## 6. Files
New (`src/engine/hyper/`):
* `rng.js` – cyrb hash sub-seeds (`sub(seed,'gill')`), RNG helpers
* `genome.js` – 9 ecomorph archetypes (mean+spread), blend weights, sampling, clamp, constraint rules, validation + re-roll → `hyper` param block per fish
* `textures.js` – CPU-baked per-part texture sets (albedo+height, normal+roughness+metal) for operculum variants (chrome crinkle / engraved reticulate / rust bumpy), cached by params
* `gillchamber.js` – hinged operculum pieces (operculum, preopercle, sub/interopercle, branchiostegal rays, spines, flap), cavity basin, arches, filaments, rakers, pseudobranch, head spines; shaders; open/breath animation
* `skin.js` – GLSL patch for `FS_BODY`/`FS_FIN`: pattern layers, lateral line pores, myomere chevrons, mucus streaks, wear/age, scutes, thin-film
* `rig.js` – studio rig, capture views, contact sheet, automatic validation, "Hyper" UI panel, perf stress test

Edited: `fishgen.js` (hooks in `buildFish`, `makeUniforms`, `FS_BODY` patch, `FS_EYE` patch, animate loop, boot), `gillgeo.js` (export sampler), `fish.css`, `index.html`, `README`.
