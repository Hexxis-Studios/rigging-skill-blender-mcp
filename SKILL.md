---
name: blender-mcp-character-rigging
description: Rig and repair character models through Blender MCP, including armatures, skin weights, clothing, hair, pose conversion, and deformation checks. Use when binding a character or diagnosing stretching, cross-limb influence, broken retargeting, or garment deformation.
---

# Blender MCP character rigging

Produce an editable rig that deforms correctly in the requested motions. A created armature, successful parenting operation, normalized weights, or valid game-engine avatar is an intermediate result, not rig acceptance.

Use this workflow to bind characters and repair existing humanoid rigs, clothing, and hair. Discover the connected Blender MCP server's inspection, Python execution, and screenshot capabilities rather than assuming tool names. Choose the binding method for the model's anatomy and intended motions.

Read [references/lessons.md](references/lessons.md) before choosing a repair; it separates working methods from rejected experiments. For MCP execution and reusable Python patterns, read [references/mcp-and-python.md](references/mcp-and-python.md).

## 1. Establish the actual target

Inspect before editing:

- Blender version, MCP capabilities, active file, scene, mode, selection, and target object names.
- All relevant mesh objects, their armature modifiers and target rigs, object transforms, parent transforms, constraints, shape keys, and existing animation data.
- Bone hierarchy, deform flags, names, rest matrices, and the skin's vertex-group mapping. Separate controls/helpers from the deform skeleton.
- Units, physical height, up/forward axes, rest pose, expected export format, supported influence count, and intended motions.
- Connected mesh components and material/UV boundaries: a single object can contain hands, trousers, lining, bodysuit, skirt, eyes, or disconnected trim.
- Authored weights or bind rigs included with the source. Read declared weight-file references before generating replacements.

Reuse a suitable existing rig and authored skin. Do not rebuild the skeleton merely because an automatic tool is available. Preserve bone names and clip identifiers used by downstream consumers. For a new skeleton, place joints from the actual anatomy and establish the destination's rest-pose contract before binding; do not copy another model's absolute joint coordinates.

Save recoverable source work, then use a named candidate collection/file. Keep a verified checkpoint and a concise record of changes. Never clear a live scene. An explicitly disposable headless scene may be reset. Do not overwrite the accepted source during an experiment.

## 2. Reproduce and isolate the failure

Record the exact object, body recipe, clip, frame/subframe or normalized time, viewing direction, and enabled modifiers. Keep them identical for comparisons. A similarly named dance is not the same test.

Compare the earliest available stages:

1. Original source mesh and rest skeleton.
2. Fitted mesh, before pose conversion.
3. Converted rest pose, before clearance/contact.
4. Ordinary armature deformation, without secondary cloth/contact.
5. Final evaluated mesh with downstream deformation enabled.

Use body-visible/body-hidden and original-rig/cloth-off controls to identify whether an apparent hole is skin penetration, missing geometry, shading, or a later deformation defect. Change one responsible stage at a time. Do not sculpt away a fin introduced by runtime collision or reweight a clean source to compensate for an importer error.

State the hypothesis and observable prediction before the edit. If the same repair family repeatedly fails the matched test, retain its measurements, restore the accepted state, and choose a different measured cause. Avoid an unbounded smoothing, clearance, or solver-parameter loop.

## 3. Choose a bounded binding method

### Existing body or ordinary fitted clothing

- Use source-authored weights when they match the topology and skeleton. Map unsupported source leaf joints deliberately to appropriate driven joints; do not create plausible-looking mappings from names alone.
- Automatic weights are a candidate. Check the deform-bone set and mesh structure first, then test joint bends and real motion. Bone heat can produce zero unbound vertices while giving visibly bad folds and intersections.
- For contaminated skin, identify the actual surface region before editing. Nearby resting hands can wrongly attract hip/trouser vertices; lower hanging sleeves can legitimately need arm influence. Position alone is not semantic ownership.
- Prefer a local patch with a smooth boundary and preserve original weights elsewhere. Normalize the retained deform weights after removing a wrong influence.
- Smooth along mesh connectivity only where a measured discontinuity warrants it. Euclidean neighbors can cross from trousers to hands, opposite legs, or inner to outer shells. Even topology-local smoothing can worsen authored sleeve seams; replay the failed motion before expanding it.

### Weight transfer

Transfer from a compatible body surface or deformation helper in a common coordinate frame and matching rest/fitted pose. Restrict candidate source surfaces by anatomy/part where nearest geometry is ambiguous.

Nearest triangle plus barycentric interpolation is a useful starting method: interpolate the three source vertices' weight dictionaries, merge contributions by bone name, blend into the bounded destination patch, enforce the target's influence budget, and renormalize. Reject or explicitly handle missing hits and degenerate triangles. Check transfer distance against this model's scale. Do not blindly assign nearest-vertex weights across a sleeve/torso boundary.

For fitted clothing, sample the relevant full body, including hidden skin. Body visibility masks are not fitting geometry. If the same garment must support multiple body shapes, compare local correspondence across those shapes before choosing shared weights.

If truncating influences, record discarded mass before renormalization. Use the destination's actual influence limit; do not assume four. A normalized result can still have lost important influence.

### Skirts, robes, and hanging sleeves

Use deliberate support/deformation for free fabric. A skirt helper may be appropriate for an outer skirt; retain original torso/arm skinning and separately rig an inner bodysuit. Identify regions from source topology and anatomy, not height alone. Exclude inner-body faces from outer-shell helper construction.

Do not assume a long trumpet sleeve can be fixed by replacing its weights with nearest-arm weights. That can reduce the worst stretch while worsening many other edges. Evaluate deliberate sleeve/drape support and contact when local weight repairs fail the full motion test.

### Hair and headwear

Head/scalp and rigid headwear should follow the intended head motion rather than accidental eye/facial helpers. Preserve an appropriate lower neck transition or existing drape rig where needed. Long hanging hair can need a bounded torso-correspondence band; retain the scalp/head binding and loose regions outside that band. Do not transfer the whole hairstyle to the body.

## 4. Keep pose conversion and bind data coherent

Mesh coordinates, bone rest matrices, skin bind transforms, and object transforms form one contract. Do not apply transforms, change bone rest pose, or swap bind matrices independently on an already animated skin. Work on the candidate and prove rest and posed equivalence after a coordinated conversion.

- Imported glTF joints supply joint origins; Blender may infer misleading bone tails. Use parent/child joint origins for anatomical segment reasoning, in the same space as mesh positions. Handle leaves explicitly.
- Reset old pose/action influences before judging neutral anatomy or rebinding. A detached action does not necessarily remove NLA, drivers, constraints, or nonidentity pose transforms.
- Changing A-pose to T-pose can change vertex positions when weights change, even if source coordinates are unchanged. Compare source geometry and converted geometry as separate invariants.
- A matrix round trip passing does not establish visually correct deformation. A source-bind substitution can pass mathematical checks yet distort animated arms/hands when it conflicts with the destination reference pose. Inspect the full body after conversion.
- When retargeting, preserve the publisher's actual bind reference and hierarchy settings. Same bone names or a model from the same package do not prove the same reference pose.
- For BVH, inspect the loader's real up-axis conversion and the order of any handedness conversion/rest-frame conjugation. Euler-order agreement alone missed a wrong source pose.

## 5. Validate deformation, then export

Structural checks:

- Intended deform groups map to existing deform bones; indices are valid. Separate deliberate mask/control groups from skin weights.
- Every intended skinned vertex has finite, nonnegative, nonzero deform coverage with an appropriate normalized sum and influence limit.
- No unintended geometry, UV, material, shape-key, topology, skeleton-name, or clip changes. For a source-index weight payload, verify exact vertex order and topology, not merely counts.
- Duplicate UV-seam vertices receive the intended source correspondence. Do not weld production geometry merely to inspect connectivity.

Motion checks:

- Inspect neutral front/side/back plus elbows, shoulders, wrists, hips, knees, ankles, and fingers relevant to the task. Test left and right sides independently.
- Replay the exact failing pose and sample multiple phases of intended walk/run, crouch, reach/overhead, seated or other task-specific motion. Use full playback as well as held poses.
- Inspect the evaluated skin, not only bone movement or the undeformed mesh. Track edge stretch against rest lengths: worst, P99, number over a chosen limit, and absolute expansion in scene units. A lower maximum with a worse P99 can be a regression.
- Separate rigid translation/root motion from deformation. Verify planted contacts only when the source motion actually promises them.
- Inspect body and clothing together and separately, including back views and silhouette details. Preserve pleats, fringe, cuffs, soles, palms, and layered ordering.
- For configurable avatars, include representative frame/age/proportion extremes and repeated edits on the same instance. Old bone scales surviving a rebuild caused repeated-edit growth that fresh-instance checks missed.

Choose tolerances for the model's units, size, topology, and intended motion rather than copying fixed distance or body-height thresholds. Near-zero rest edges need separate handling before computing ratios.

Export only the intended mesh, skeleton, and requested clips. Query the installed exporter options. Reimport into a disposable scene/process and verify facing, scale, rig, materials, clips and deformations. UV/normal seams may split vertices on round trip, so compare triangles and mapped attributes rather than raw vertex count alone. When an engine is the delivery target, test that imported asset with its actual retargeting/controller before claiming engine readiness.

Report the editable file and export paths, what changed, the exact tested poses/motions, structural results, inspected views, and remaining defects. If motion or visual review was unavailable, identify that limit explicitly. A rig is accepted only for the evidence actually obtained.
