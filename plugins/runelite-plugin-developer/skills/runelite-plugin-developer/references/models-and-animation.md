# Models, Animation, and Cosmetic Scene Objects

Use for 3D model construction, recoloring, animation, cosmetic followers/replacements, and projected model hit testing. These are observations from specific source revisions plus API constraints, not quality assessments or promises of Plugin Hub acceptance. Inspect the target dependency before adopting a technique; APIs and cache data change.

## Source Examples Inspected

Inspected on 2026-10-26; neither plugin was built or run in game during this research.

| Concern | Pet Thralls | Companion Pets |
| --- | --- | --- |
| Revision | `9aa81f979654627a9fa06044afaff41af0c82705` | `660eb1787b8f6135a05ef08a6349c84546c622dd` |
| Model construction | `PetFollower`: NPC composition model IDs, loaded parts, merge, vertex/color clones, recolors, lighting | `ModelUtility`: model IDs and recolors from `PetData`, loaded parts, merge, clones, lighting |
| Metadata | Live NPC composition plus explicit animation/lighting entries in `ThrallPet` | Bundled NPC JSON populates models, poses, scale, footprint, and lighting; separate animation height offsets |
| Scene object | `PetObject` extends `RuneLiteObjectController`; separate pose/action controllers | `PetObjectModel` wraps a `RuneLiteObject` with a custom `getModel()` |
| Movement | Copies the tracked NPC's current position and facing in `BeforeRender` | Computes cosmetic targets on game ticks; advances a bounded target queue and rotation on client ticks |
| Extra geometry | Extracts a ground symbol and builds an outline shell | Projects an oriented model AABB to a canvas polygon for mouse hit testing |

Source entry points:

- [PetFollower](https://github.com/timbo1359/pet-thralls/blob/9aa81f979654627a9fa06044afaff41af0c82705/src/main/java/com/petthralls/PetFollower.java), [PetObject](https://github.com/timbo1359/pet-thralls/blob/9aa81f979654627a9fa06044afaff41af0c82705/src/main/java/com/petthralls/PetObject.java), [ThrallMarkings](https://github.com/timbo1359/pet-thralls/blob/9aa81f979654627a9fa06044afaff41af0c82705/src/main/java/com/petthralls/ThrallMarkings.java).
- [Companion model construction](https://github.com/Mrnice98/Companion-Pets-Plugin/blob/660eb1787b8f6135a05ef08a6349c84546c622dd/src/main/java/com/example/Utilities/ModelUtility.java), [PetObjectModel](https://github.com/Mrnice98/Companion-Pets-Plugin/blob/660eb1787b8f6135a05ef08a6349c84546c622dd/src/main/java/com/example/PetObjectModel.java), [render projection](https://github.com/Mrnice98/Companion-Pets-Plugin/blob/660eb1787b8f6135a05ef08a6349c84546c622dd/src/main/java/com/example/Utilities/RenderUtility.java), [plugin lifecycle and metadata loading](https://github.com/Mrnice98/Companion-Pets-Plugin/blob/660eb1787b8f6135a05ef08a6349c84546c622dd/src/main/java/com/example/CompanionPetPlugin.java).

Both inspected repositories have BSD-2-Clause terms. This reference paraphrases techniques and copies no implementation. Follow the skill's license/attribution guardrails before copying source.

## Model Construction and Ownership

- Distinguish unlit geometry (`ModelData`), lit renderable geometry (`Model`), and the scene object's placement/animation/lifecycle. NPC definition IDs, model-part IDs, and animation IDs are different identifiers.
- Load all required parts before merging. `Client.loadModelData` can return null for loading or nonexistent assets; null alone does not distinguish those cases. Preserve the original appearance while assets are missing and bound retries without blocking. Pet Thralls uses a fixed frame-attempt cap; Companion's inspected loader has no explicit missing-part checks. Neither policy is a universal recipe.
- Establish array ownership before mutation. Loaded data can share arrays; `shallowCopy()` still shares arrays. Clone vertices for geometric transforms, colors for recoloring, textures for retexturing, and transparencies when modifying them. Lighting shares several arrays with its input; it does not create a fully independent model. A merge is not evidence that every mutable array is privately owned.
- Apply appearance metadata deliberately: recolors use Jagex HSL values, scale 128 is neutral, and width/height need not match. Verify lighting/contrast conventions for the metadata source instead of copying another plugin's numeric adjustments. When relevant and supported, resolve transformed NPC compositions rather than assuming the base definition describes the visible form.
- Prefer current definitions for fields actually exposed by the API. Supplemental tables need provenance, cache-version awareness, and bounded loading cost. Companion's inspected `npc_data.json` is 28,062,697 bytes and is parsed during startup. This is observed file size and code structure, not a measured runtime cost or a recommended approach.

## Animation and Transformation Order

- Both examples animate an unscaled base before display scaling. Test that order for the model/animation pair: scaling first can distort animation translations. Keep a stable base and avoid repeatedly scaling it in `getModel()`; static and missing-animation paths also need correct scaling.
- Treat transformed models as temporary. `Client.applyTransformations` clones vertices but returns a shared result valid only until the next transformation call, including calls made internally by the client. Other arrays can remain shared. Do not retain animated results across frames or interleaved transformations, or assume vertex independence permits editing every field.
- Use `AnimationController` progression/completion semantics supported by the resolved API. Separate looping poses from one-shot actions when needed, avoid restarting unchanged poses every frame, and define missing-clip fallbacks. Check skeleton compatibility and layering; an animation ID alone does not guarantee suitability for another model.
- Pet Thralls ticks pose/action controllers explicitly and clears completed actions. Companion switches a single animation back to idle/walk through frame comparisons. Verify legacy and Maya animation behavior before relying on frame counts or deprecated helpers; these examples use different animation state models.

## Placement, Rendering, and Interaction

- Choose `RuneLiteObject` for behavior it already supplies; consider `RuneLiteObjectController` for necessary custom control. Verify availability and semantics against the resolved dependency instead of treating either example's class choice as mandatory.
- Keep model/client operations on the client thread. Track the correct `WorldView`, local location, plane, orientation, and terrain height. Re-register custom objects on world-view changes when required; remove them on despawn, logout/hop, world-view unload, and shutdown. Do not reuse scene-local coordinates in a new scene or assume all actors belong to the top-level view.
- Keep frame work to necessary motion, animation, and drawing. Reuse built models until relevant appearance inputs change. Mirroring an existing actor and simulating an independent cosmetic follower solve different problems; do not add pathing solely to mirror an actor already positioned by the client.
- Pet Thralls hides the original NPC through a render callback only while its replacement is showing, while preserving UI rendering. Keep replacement failure reversible. Hiding a renderable differs from changing the actual NPC or its server behavior.
- Companion projects an oriented AABB to a canvas polygon for local mouse interaction. This approximates the silhouette; verify world view, height, scale, orientation, and offscreen behavior. Avoid unnecessary model/animation evaluations for picking and respect temporary-model lifetimes when evaluating multiple objects. Local cosmetic menu actions must remain distinct from server game actions.

## Geometry Requiring Extra Verification

`ThrallMarkings` identifies disconnected flat faces near the model's lowest point as the ground symbol and hides other faces by collapsing triangles. Its outline uses a grown, recolored, untextured shell with face-winding changes and optional displacement along computed normals. It excludes transparent faces and handles some inside-out components.

These algorithms depend on topology. Ground tolerances, vertex welding on merge, face ordering, signed-volume assumptions, and array getter write-through are not universal guarantees. Inspect the resolved implementation before direct array edits; the plugin's read-back probes do not establish general API support. Keep optional effects separable so failure can leave the base model visible.

Test small geometry fixtures where practical. Ask the user to check animation, scaling, terrain, world-view transitions, transparent parts, and replacement fallback in game. This research establishes no live rendering or performance result.

## RuneLite API Contracts

Checked against RuneLite `d955988815db9138241ee99991880f68a626496b` on 2026-10-26; recheck the target dependency:

- [Client loading and transformation contracts](https://github.com/runelite/runelite/blob/d955988815db9138241ee99991880f68a626496b/runelite-api/src/main/java/net/runelite/api/Client.java).
- [ModelData ownership and lighting](https://github.com/runelite/runelite/blob/d955988815db9138241ee99991880f68a626496b/runelite-api/src/main/java/net/runelite/api/ModelData.java), [Mesh transform units](https://github.com/runelite/runelite/blob/d955988815db9138241ee99991880f68a626496b/runelite-api/src/main/java/net/runelite/api/Mesh.java).
- [AnimationController](https://github.com/runelite/runelite/blob/d955988815db9138241ee99991880f68a626496b/runelite-api/src/main/java/net/runelite/api/AnimationController.java), [RuneLiteObject](https://github.com/runelite/runelite/blob/d955988815db9138241ee99991880f68a626496b/runelite-api/src/main/java/net/runelite/api/RuneLiteObject.java), [RuneLiteObjectController](https://github.com/runelite/runelite/blob/d955988815db9138241ee99991880f68a626496b/runelite-api/src/main/java/net/runelite/api/RuneLiteObjectController.java).
