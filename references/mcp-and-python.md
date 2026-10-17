# Blender MCP execution patterns

## Discover the connector, then operate explicitly

Blender MCP implementations differ. Discover the actual tools for scene inspection, Python execution, and screenshots. Do not invent a tool namespace or assume Python executes in the currently visible Blender instance. First query the active file/version/object list and confirm the expected scene. If Blender access is unavailable, provide the bounded script or explain the missing connection; do not claim edits occurred.

Start with a read-only inventory. Then send small, named, bounded edits to the identified candidate. Prefer Blender's data API for deterministic changes; operators often depend on mode, active object, selection, and view context. Where an operator is needed, explicitly establish its context and restore the user's state afterward.

A timeout does not prove rollback. Inspect scene state and output receipts before retrying. Long library operations should emit completed-item receipts and resume from validated hashes, not rerun mutations blindly. Do not terminate another Blender instance or unrelated process to recover your job.

Do not execute a scene-resetting historical batch script inside a live MCP session. Factory resets/object-wide deletion belong only in confirmed disposable scenes. Keep the user's Blender open.

## Read-only skin coverage inventory

Run through the connector's actual Python execution facility. This example inspects original mesh data, not deformation quality; it does not select, modify, save, or repair objects. Inspect masks/control groups separately. It inventories all mesh objects so an agent must still choose its intended target.

```python
import bpy, json, math

report = {"blender": bpy.app.version_string,
          "file": bpy.data.filepath, "meshes": []}
for obj in bpy.context.scene.objects:
    if obj.type != 'MESH':
        continue
    modifiers = [m for m in obj.modifiers if m.type == 'ARMATURE']
    rig = obj.find_armature()
    deform_names = ({b.name for b in rig.data.bones if b.use_deform}
                    if rig else set())
    group_names = {g.index: g.name for g in obj.vertex_groups}
    uncovered, invalid, unnormalized = [], [], []
    maximum_influences = 0
    for vertex in obj.data.vertices:
        entries = [g.weight for g in vertex.groups
                   if group_names.get(g.group) in deform_names]
        if any(not math.isfinite(w) or w < 0 for w in entries):
            invalid.append(vertex.index)
            continue
        total = sum(entries)
        positive = sum(w > 0 for w in entries)
        maximum_influences = max(maximum_influences, positive)
        if total <= 0:
            uncovered.append(vertex.index)
        elif abs(total - 1.0) > 1e-4:
            unnormalized.append(vertex.index)
    report['meshes'].append({
        'name': obj.name, 'vertices': len(obj.data.vertices),
        'armature': rig.name if rig else None,
        'armatureModifiers': [{'name': m.name,
            'target': m.object.name if m.object else None,
            'viewport': m.show_viewport,
            'vertexGroups': m.use_vertex_groups,
            'envelopes': m.use_bone_envelopes} for m in modifiers],
        'uncoveredCount': len(uncovered), 'uncoveredSample': uncovered[:20],
        'invalidCount': len(invalid),
        'unnormalizedCount': len(unnormalized),
        'maximumDeformInfluences': maximum_influences,
        'groupsNotMatchingDeformBones': sorted(
            set(group_names.values()) - deform_names)})
print(json.dumps(report, indent=2))
```

Interpret the result: nonmatching groups may be intentional masks, and envelope deformation or multiple armature modifiers needs separate analysis. Absence of coverage on an intentionally unskinned prop is not a rig failure. Numeric coverage does not diagnose wrong anatomical ownership.

## Efficient candidate weight assignment

For a new, explicitly identified candidate mesh, create the vertex groups first. Prepare a per-vertex list of `(group_index, weight)` pairs in the exact destination vertex order. Validate names, bounds, finiteness, sums and influence limits before assigning. Do not copy an index from another object's vertex groups: indices are local to each object.

```python
import bmesh

# candidate is an explicitly resolved candidate MESH object in Object mode.
# weights_by_vertex is already validated and matches candidate.data.vertices.
# This full replacement is for a candidate's complete group data only.
# If it has mask/control groups, preserve those entries or patch only skin keys.
bm = bmesh.new()
try:
    bm.from_mesh(candidate.data)
    bm.verts.ensure_lookup_table()
    deform = bm.verts.layers.deform.verify()
    for index, weights in enumerate(weights_by_vertex):
        values = bm.verts[index][deform]
        values.clear()
        for group_index, weight in weights:
            if weight > 0:
                values[group_index] = weight
    bm.to_mesh(candidate.data)
    candidate.data.update()
finally:
    bm.free()
```

This is an assignment pattern, not a weight generator or a script to run unadapted. Re-read resulting groups and verify against the prepared payload before saving. Retain unrelated masks when editing existing assets.

## Nearest-triangle transfer procedure

1. Resolve source/destination meshes and intended anatomical regions explicitly. Convert coordinates to one chosen space using the actual mesh matrices; do not mix Blender local coordinates with engine-space exports.
2. Use source triangles with a stable face-index mapping and source vertex weight dictionaries. Build `mathutils.bvhtree.BVHTree` from that exact geometry.
3. Query the destination point's nearest allowed triangle. A `None` hit or degenerate triangle requires explicit handling. Do not silently bind it to the first bone.
4. Compute barycentric coordinates at the closest point. Clamp only tiny floating-point excursions and renormalize; large violations indicate an error worth investigating.
5. For each of the three source vertices, accumulate `barycentric_weight * bone_weight` into a dictionary keyed by the source bone's identity. Map bone names to destination groups deliberately.
6. Blend with original destination weights using a bounded patch factor. Preserve original weights at zero factor. Merge duplicate influences before choosing the top supported influences; measure dropped mass, then normalize.
7. Validate source order/topology, payload hash, transfer distances and candidate skinning. Replay the failed motion and inspect transition seams.

Recorded Data Transfer authoring used the mapping enum `POLYINTERP_NEAREST` in Blender 5.1. Query installed RNA/modifier properties before using it on another version. The BVH method was an independently compared alternative, not guaranteed bit-identical output.

## Evaluated mesh and action checks

After setting a real pose/frame, update the view layer/dependency graph, inspect `obj.evaluated_get(depsgraph)`, and use its `to_mesh()` / `to_mesh_clear()` pair in `try/finally`. Test vertex-order/topology correspondence before comparing evaluated vertices by original indices. Topology-changing modifiers require a different mapping or isolated structural test.

Set frames with subframes when sampling fractional times. Record the active clip and actual evaluated time. Query the installed action API: newer Blender versions can use layered actions and action slots; old scripts that assume only legacy F-curves may lose or misassign clips. Do not blindly select the first slot when several targets exist.

Capture and restore any action, slot, frame/subframe, pose state, selection or mode changed by a diagnostic. Muting an action alone may leave NLA/constraints/drivers active. Restore the full state that was changed rather than assuming identity is the user's prior state.

For edge measurements, transform rest and deformed points consistently. Store per-edge rest lengths, absolute expansion, ratios for nondegenerate edges, severe counts, maximum and P99. The reference must correspond to the same topology and intended bind stage. Report a topology mismatch explicitly rather than obtaining plausible numbers from unrelated indices.

## Export and a minimal completion receipt

Discover the actual export operator's properties. Historical FBX flags such as `axis_forward='-Z'`, `axis_up='Y'`, or `add_leaf_bones=False` are starting examples only; they do not prove the correct facing or hierarchy for this target. Preserve requested clip names and existing extra clips unless removal was requested.

Keep a compact receipt containing source/candidate/export paths and hashes; Blender version; target mesh/rig names; modified regions; unchanged geometry/UV/skeleton contracts; coverage and influence results; dropped weight; exact motion samples; visual findings; round-trip result; and remaining defects. Mark an intermediate artifact as a candidate until its required checks pass.

## Scope of these examples

The snippets are reusable adaptations of recorded methods. The coverage inventory is a new diagnostic example; the full assignment pattern derives from the tested BMesh method. Neither is an automatic production rig repair. An agent must inspect its connected Blender version and validate results on the user's model.
