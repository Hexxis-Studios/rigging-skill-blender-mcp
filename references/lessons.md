# Recorded rigging lessons

These are distilled observations from character work through October 17, 2026. Accepted means the stated source-specific test passed; it is not a universal guarantee. Rejected means the candidate was removed from accepted assets. Counts document the scope of evidence and are not required test quotas.

## Existing humanoid auto-rig repair

**Observed:** Resting hands and trousers/hip trim shared incorrect influences. Walking/running pulled clothing toward the hands, sometimes leaving flying fragments.

**Working method:** Inspect connected components, actual rest joint origins, existing influence families, and garment identity. Remove demonstrably wrong arm/leg influence locally and normalize retained weights. Preserve garment geometry and bone names. Small disconnected fragments needed their own binding review rather than deletion.

**Important correction:** Imported glTF bone tails were inferred and could extend dozens of metres on scaled rigs. Distance-to-tail heuristics selected wrong bones. Parent/child joint origins were a better anatomical segment source. Leaf endpoints still required a bounded, anatomy-aware decision.

**Additional evidence:** Full-resolution source texture samples helped distinguish trousers from adjacent skin on these particular textured models. Downsampled atlas colors bled across UV islands, and a single median color missed multicolored garments. Texture classification was supplementary source-specific evidence, not a general anatomy detector.

**Limits:** Connectivity-based local weight smoothing reduced measured tears in the older humanoid cohort. Nineteen models received sampled-pose and visual review; extra retained emotes were not individually polished. Do not copy its fixed height bands, color thresholds, iteration count, or movement recipe into an unrelated rig.

## Source weight files can be better than a new bind

**Observed:** A converter omitted declared MakeHuman shoe weight files and used fallback correspondence. Six source shoes already carried authored skinning.

**Working method:** Read each source's declared weight-file reference, verify its source topology and bone map, retain hashes/credits, and map source toe leaves onto the actual driven toe joints. Keep a recorded compatible fallback when source data is absent or incompatible.

**Lesson:** Before autorigging, audit what the asset already supplies. A plausible fallback and valid avatar did not prove natural motion.

## Nearest-surface helper binding

**Observed:** Garment fins were much larger after downstream deformation than in the source silhouette. Generic smoothing damaged pleats/fringe.

**Working method:** Export fitted source and actual helper geometry in the same rest state. Blender Data Transfer nearest-face interpolation authored helper weights without replacing source geometry, UVs, identity, or credits. Retaining the fine contact pass filled sampled skin holes while the new helper binding reduced large fins.

**Evidence:** Initial two-style tests showed the large fringe fin removed and a white-gown fin reduced. Later admission covered 43 styles / 455,138 split vertices with exact source topology/UV/order checks and broader motion views. Hanging sleeves and some running folds were still unfinished.

**Tooling improvement:** BVHTree nearest triangles and barycentric interpolation completed the larger authoring batch faster than many-group Data Transfer. Retained references showed up to 88.72 micrometres surface discrepancy under the recorded deformation probes. This was not bit-identical, especially near skinny triangle boundaries, and still needed native motion validation. Preserve completed output bytes when resuming.

**Assignment improvement:** Create all destination group names, fill a BMesh deform layer, and publish once rather than making millions of separate group-add calls. An 11,271-vertex authoring check matched admitted weights exactly. That establishes the assignment operation, not runtime performance.

## Layered outfits require separate binding regions

**Observed:** Ballet/skating garments contained an inner bodysuit and disconnected outer skirt. A garment-wide skirt rule sent crotch/leg fabric to the outer cloth helper. Inner faces also contaminated outer-shell ray hits.

**Working method:** Inspect welded-analysis source components without welding the production mesh. Author source-stable region labels. Retain the inner part's original leg rig through pose conversion and fitting; bind only the outer skirt to its helper and exclude inner faces from helper construction.

**Evidence:** Four fits / 1,860 movement frames plus 12 reviewed views removed the skating strip and retained the tutu silhouette. Other layered outfits still need their own region review.

## Local shirt correspondence can repair a local defect

**Observed:** A default shirt pierced the body in a held dance. Broad collar expansion fixed one view but exposed another shoulder. Per-wall and arm-exclusion variants did not consistently repair it. A different dance used as a test was initially mistaken for the failing clip.

**Working method:** Match the exact clip/phase and use body-on/off controls. Transfer nearest-triangle barycentric skin weights in a bounded central upper-shirt patch, using both relevant fitted source frames, and smoothly retain original weights elsewhere. Preserve source geometry, UVs, body rig and mask.

**Evidence:** 1,487 of 8,426 source vertices changed. Ten body recipes / 8,278 movement frames and 40 reviewed captures supported this specific repair. Converted T-pose positions changed because weights participate in pose conversion; unchanged raw source positions did not imply unchanged converted positions.

**Limits:** This did not authorize whole-wardrobe reweighting. A later older-body neckline defect needed a separate bounded geometric clearance repair, rather than another weight override.

## Hanging sleeves: rejected approaches, unresolved support

**Observed:** In one gown, adjacent source vertices about 12 mm apart carried torso versus forearm influence and separated by about 310 mm during A-to-T conversion. Worst source-stage stretch was about 25.96 times rest. A height-only skirt mask also captured low hanging sleeves.

**Rejected trials:**

- Geodesic arm/dress seeds and arm-segment weights reduced the worst stretch to about 20.39 times but worsened P99 to 4.23 times from 1.31 times.
- A later boundary-led cuff remap worsened P99 to 11.17 times and increased edges above three times rest from 281 to 2,339.
- Local five-ring discontinuity smoothing improved the maximum but worsened P99 and the number of severe edges.
- Automatic bone heat left no unbound vertices and improved the maximum to about 7.33 times, yet all six inspected idle/walk/run views retained poor folds/intersections. Numeric fit success was insufficient.
- Retaining every original influence left the worst stretch unchanged at about 25.95768 times. Influence truncation was therefore not the cause of that worst defect.
- Direct source-proxy fitting, full source-rest cloth/support variants, and body-surface anchoring did not pass walking/running art. None was accepted as a general garment solution.

**Decision:** Do not repeat weight-only smoothing, bone heat, whole-bind swaps, or nearest-arm reassignment as a generic repair of this known drape problem. A new candidate needs deliberately authored support/deformation/contact and matched proof. These observations do not mean bone heat or sleeve regions are unsuitable for every other character.

## Bind/rest-pose changes can pass math and fail art

**Observed:** Clothes-only source A-pose bind restoration passed numeric motion checks but introduced shoulder holes. Restoring body and skins together passed matrix/fit checks but made animated arms/hands stretch and fan out. All inspected body-art views failed.

**Decision:** Both were rolled back. The exact incompatibility with that generated Humanoid pipeline was unresolved. Do not invent a proven helper-bone or blendshape cause. Do not present the rejected source-bind swap as a recipe for repairing all A-to-T rigs.

**General lesson:** Keep geometry, rest skeleton, bind transforms, and target-avatar reference coherent. A blended pose conversion is not automatically inverted by independently replacing each bone's bind matrix. Validate the entire animated body after a conversion, not only cloth distances.

## Hair/headwear skinning

**Observed:** Proxy hat/scalp weights included eye/facial helpers. Long hanging hair intersected clothing when its entire shape followed only the head.

**Working method:** Map scalp/rigid headwear to the intended head with a bounded lower neck transition. For nearby hanging hair, use a restricted nearest-body triangle transfer band and retain the scalp and loose regions outside it. Actual head/body shell fitting is separate from weight ownership.

**Evidence:** Headwear controls included exact hairstyle restoration after removal; hair validation covered 52 styles on two frames plus selected age/coat cases and 248 inspected views. This does not certify every long-hair animation.

**Precision lesson:** Use local source joints for regional thresholds. A world-space round trip at a translated/rotated studio position changed 286 weights by up to 2.384e-7 and broke exact-restoration comparison. Compare the same neutral orientation and avoid unnecessary coordinate conversions.

## Failures outside weights that masqueraded as bad rigging

- **Runtime contact:** Source/converted silhouettes were intact; contact-off views removed fins while exposing penetration. Sculpting the source would have hidden the wrong stage's fault.
- **Torso fitting:** A pelvis-centered radial clearance rule pushed curved upper-back fabric into the body. Height-dependent fitted central skin sections fixed the sampled backs. It was not missing source geometry or lower-helper weighting.
- **Retarget reference:** Discarding a publisher's source-avatar dependency and generating a reference from an animation-only FBX caused sideways feet and mismatched wrists. Restore the actual publisher reference/hierarchy settings. Another package intentionally used an auto-created reference; substituting a same-package model introduced a roughly two-metre offset. Preserve explicit source configuration rather than always forcing a shared avatar.
- **BVH axes:** Euler agreement and bone-length checks passed while the actual yoga pose was wrong. The official loader's automatic Z-up conversion had been omitted before mirror/rest conversion. Corrected output matched the source loader and visible pose.
- **Repeated refits:** Old animated local bone scales survived hierarchy rebuilding and expanded otherwise valid bodies. Reset authored scales/old bindings as part of the actual refit lifecycle, then repeat edits on the same instance.
- **Foot IK:** Permanent contact weight flattened authored foot flight. Release/reapply contact based on the intended motion and support; do not reshape legs to hide an IK fault.
- **Source semantics:** A planted-foot upper-body dance was mistakenly tested as a stepping dance. Compare the original source before demanding movement it never contained.
- **Normals:** Dark facial punctures came from valid tiny-area normals being normalized to zero, not skinning. Inspect evaluated geometry/normals before rebinding or changing anatomy.
- **Stale evidence:** Offscreen or same-call engine captures sometimes showed a stale pose. Let actual animation frames settle and verify recorded time/orientation. A good rig cannot be judged from the wrong frame.

## Evidence provenance

The extraction used project handoff/source-cleanup records, the character-authoring skill references, Blender authoring/validation scripts, and October 15–17 garment, shirt, headwear and hair validation reports. No private models, source archives, machine paths, save files, or project code are required by this portable skill. It contains distilled workflow guidance rather than copies of those assets.

Primary references recorded during that work, for optional further reading (version-specific APIs still require live inspection):

- [MakeHuman matching groups and trumpet sleeves](https://static.makehumancommunity.org/assets/creatingassets/makeclothes/makeclothes_vertexgroups.html)
- [MakeHuman clothing construction](https://static.makehumancommunity.org/assets/creatingassets/makeclothes/clothes.html)
- [Blender Data Transfer documentation used in the investigation](https://docs.blender.org/manual/id/5.0/modeling/modifiers/modify/data_transfer.html)
- [Unity bind-pose contract, if Unity is the target](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Mesh-bindposes.html)
