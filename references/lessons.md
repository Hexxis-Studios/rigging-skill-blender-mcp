# Rigging failures and repair decisions

Use symptoms and matched comparisons to choose a repair. A method that improves one garment or pose may regress another; verify the affected surface and intended motion before accepting it.

## Hands pulling trousers or hip trim

Resting hands can sit close enough to the hips for automatic binding to give trouser vertices arm or hand weights. During walking or running, clothing then stretches toward the hand.

Inspect connected components, rest joint origins, existing influence families, and garment identity. Remove demonstrably wrong arm/leg influence locally and normalize retained weights. Review disconnected trim separately rather than deleting it.

Imported glTF joints provide origins, not reliable bone lengths. Blender may infer tails that extend far beyond the anatomy. Use parent/child joint origins for anatomical segment reasoning and handle leaves explicitly.

Texture samples can supplement region identification on suitable textured models. Use the original atlas resolution: downsampling can bleed colors across UV islands. Do not treat color as a general anatomy detector or assume a multicolored garment has one representative color.

## Missing source-authored weights

Before generating weights, inspect the asset's declared weight files and skeleton references. A converter may have used fallback correspondence despite authored data being available.

Verify source topology and bone identity before importing weights. Map unsupported leaf joints to appropriate driven joints deliberately. When source data is absent or incompatible, retain a documented compatible fallback instead of inventing a mapping.

## Garment fins and excessive stretching

Compare fitted source, pose conversion, ordinary skinning, and final contact/deformation stages. If the source silhouette is intact and disabling downstream contact removes a fin, repair that stage rather than sculpting the source around the artifact.

Nearest-face weight transfer from an appropriate deformation helper can improve binding without changing source geometry or UVs. Blender Data Transfer and BVHTree nearest-triangle interpolation are useful alternatives, but do not assume bit-identical results near skinny triangle boundaries. Compare resulting surface positions under deformation as well as weight differences.

Generic Corrective Smooth can damage pleats, fringe, or cuffs while leaving the underlying binding fault. Preserve silhouette details during diagnosis.

For large editable meshes, create destination groups first and fill a BMesh deform layer before publishing once. Avoid millions of individual group-add calls. Re-read the assigned weights to verify the operation.

## Inner bodysuit bound to an outer skirt helper

A single outfit object can contain an inner bodysuit and disconnected outer skirt. Binding the entire object to one skirt helper can stretch crotch/leg fabric. Inner faces can also contaminate outer-shell ray hits.

Inspect source components using a welded analysis copy where appropriate; preserve production topology and UV seams. Author stable part labels, retain the inner part's original leg rig through fitting and pose conversion, and bind only the outer skirt to its helper. Exclude inner faces from outer-shell helper construction.

## Local shirt intersections

Match the exact failing clip and phase. Compare body-visible and body-hidden views to establish actual skin penetration before changing weights. Broad shell expansion may hide one intersection while exposing another shoulder.

For a correspondence fault, transfer closest-triangle barycentric body weights in a bounded patch with a smooth transition to original weights. Preserve geometry, UVs, body rig, and visibility masks. When supporting multiple body shapes, compare them before selecting a shared patch.

Changed weights can alter converted T-pose positions even when raw source coordinates remain unchanged. Validate those stages separately. A neckline clearance defect can require a separate geometric repair rather than another weight override.

## Long hanging sleeves

Neighboring fabric vertices with torso versus forearm ownership can separate severely during A-to-T conversion. Height-only skirt classification can also capture hanging sleeves that need arm influence.

Watch for misleading improvements:

- Nearest-arm reassignment or geodesic arm/dress regions may reduce the maximum stretch but worsen P99 and many transition edges.
- Topology-local smoothing may improve a few outliers while spreading damage across an authored seam.
- Bone heat may leave no unbound vertices yet produce poor folds and body intersections in motion.
- Keeping more influences cannot fix a defect that persists with every original weight retained.
- Direct body-surface anchoring may look cleaner in idle but fail walking or running.

Compare maximum stretch, P99, severe-edge counts, and inspected motion together. When weight-only repairs repeatedly fail, investigate deliberate sleeve support, drape deformation, and contact rather than repeating the same binding family. These failures do not make automatic weights unsuitable for every other garment.

## Bind changes that pass math but fail motion

Replacing clothes-only bind data can introduce shoulder gaps; replacing body and garment binds together can distort arms and hands if the destination reference pose is incompatible. Passing a matrix round trip does not establish correct skinning.

Keep geometry, rest skeleton, bind transforms, and target-avatar reference coherent. A blended pose conversion is not automatically inverted by independently replacing each bone's bind matrix. Inspect the whole animated body after any conversion. If the cause remains unresolved, record that uncertainty rather than claiming a specific helper-bone or blendshape fault.

## Hair and headwear following the wrong bones

Proxy weights may contain eye/facial-helper influences inappropriate for rigid headwear or scalp. Map those regions to the intended head motion while preserving necessary neck transitions and lower drape rigs.

Long hanging hair may need a restricted nearest-body triangle transfer band. Retain scalp/head binding and loose regions outside that band; transferring an entire hairstyle to the torso creates a different error.

Use local source joints for regional thresholds. Unnecessary world-space round trips introduce small numeric changes that can break exact restoration. Test accessory removal at the same neutral orientation and confirm the original hairstyle is restored.

## Other faults that resemble bad weights

- **Curved torso clearance:** A pelvis-centered radial rule can push upper-back fabric into the body. Fit against actual central torso sections or local surface geometry and include hidden skin.
- **Retarget reference:** Generating a bind reference from a posed animation-only FBX can produce sideways feet or mismatched wrists. Preserve the publisher's actual reference and hierarchy settings. A model from the same package is not automatically compatible; retain an explicit auto-created reference when that is the source's intended setup.
- **BVH axes:** Correct Euler order and bone lengths do not prove correct pose. Check the loader's up-axis conversion and the order of handedness/rest-frame conversion against the original source pose.
- **Repeated refits:** Old animated local bone scales can survive hierarchy rebuilding and expand a body. Restore authored scales and old bindings through the actual refit lifecycle; repeat edits on the same instance.
- **Foot IK:** Permanent planted-contact weight can flatten authored foot flight. Inspect contact ownership before changing leg anatomy or source weights.
- **Source semantics:** A planted-foot upper-body dance should not fail a test merely because it contains no steps. Inspect the original motion before specifying movement expectations.
- **Normals:** Dark punctures can come from invalid or zeroed tiny-area normals rather than skinning. Inspect evaluated geometry and normals before rebinding.
- **Stale captures:** A screenshot can show the previous pose or an unsettled animation state. Verify evaluated time and orientation before judging the rig.

## Further reading

Check documentation against the installed software version:

- [MakeHuman matching groups and trumpet sleeves](https://static.makehumancommunity.org/assets/creatingassets/makeclothes/makeclothes_vertexgroups.html)
- [MakeHuman clothing construction](https://static.makehumancommunity.org/assets/creatingassets/makeclothes/clothes.html)
- [Blender Data Transfer](https://docs.blender.org/manual/id/5.0/modeling/modifiers/modify/data_transfer.html)
- [Unity bind-pose contract, when Unity is the destination](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Mesh-bindposes.html)
