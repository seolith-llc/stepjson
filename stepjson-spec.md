# StepJSON — Open Specification (Version 1.0)

**Status:** 1.0 (stable) · **Published:** 2026-08-30 (revision 2; 1.0 first drafted 2026-08-22) · **Authors:** Shadowstep / Seolith LLC

**License:** This specification is licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — implement it freely, including
commercially, with attribution to Seolith LLC. `SPDX-License-Identifier: CC-BY-4.0`.
The specification is open; the Shadowstep step catalog and reference implementation are
not covered by this license.

StepJSON is a JSON-native, machine-readable dance-step notation. A StepJSON document
describes a short, repeatable unit of dance movement as timed keyframe poses on a
normalized two-dimensional skeleton, together with the cultural, historical, and rights
metadata needed to use it responsibly. It is designed to be renderer-agnostic: the same
document can drive a 2D silhouette, a 3D skeleton, or motion interchange formats like BVH.

## 1. Design principles

1. **Data over graphics.** Classical notation systems (Labanotation, Benesh,
   Eshkol-Wachman) encode movement as graphical scores. StepJSON encodes movement as
   plain data: JSON that validates against a schema, diffs cleanly in version control,
   and feeds any renderer.
2. **Provenance is mandatory.** Every document carries sources, rights status, and
   review attribution. Movement heritage deserves citation-grade traceability.
3. **Deterministic replay.** Identical input and beat position always evaluate to an
   identical pose. No timestamps, randomness, or environment dependence inside the
   motion data.
4. **Short loops.** A step is a loop-length vocabulary unit (≤ 16 counts by convention),
   not a choreographic whole. Larger works compose steps by reference.

## 2. The rig

StepJSON poses are expressed on a 19-joint, two-dimensional skeleton whose joint names
align with the MediaPipe Pose landmark subset, so camera-based motion capture maps
directly:

```
nose, leftEye, rightEye, leftEar, rightEar,
leftShoulder, rightShoulder, leftElbow, rightElbow,
leftWrist, rightWrist, leftHip, rightHip,
leftKnee, rightKnee, leftAnkle, rightAnkle,
leftFootIndex, rightFootIndex
```

**Coordinate system:** rig units, origin at the hip midpoint, `+y` up, and `+x` toward the
figure's own RIGHT side — so the `left*` joints sit at negative x (`leftShoulder` is at
about `-0.75`). How that maps to a screen is a renderer's choice, not the format's: the
reference renderer draws `+x` rightward, which presents the dancer mirror-image, the
follow-along convention of a studio mirror. Implementations that flip this axis without
intending to will render a dancer whose left and right are swapped.
**Normalization invariant:** `dist(leftHip, rightHip) = 1`. A standing adult figure spans
roughly y ∈ [-1.8, +2.1].

## 3. Document shape (step)

```jsonc
{
  "stepjson": "1.0",
  "id": "example-grapevine",                        // ^[a-z0-9-]+$
  "name": "Grapevine",
  "aliases": ["weave step"],
  "style": { "family": "folk-line-dance", "tags": ["travelling"] },
  "summary": "Sideways travelling weave: side, cross behind, side, cross in front.",
  "origin": {
    "region": "North America / Europe",
    "era": "20th century",
    "story": "…cited narrative…",
    "meaning": "…what the movement expresses…"
  },
  "sources": [
    { "kind": "encyclopedia", "title": "…", "publisher": "…", "url": "…", "accessed": "2026-08-22" }
  ],
  "rights": {
    "choreographyStatus": "social-dance-steps",
    "reviewedBy": "content-team",
    "reviewedAt": "2026-08-22"
  },
  "license": "proprietary",                  // proprietary | cc-by-4.0 | public-domain | work-for-hire
  "music": { "defaultBpm": 116, "meter": "4/4", "counts": 8 },
  "loop": { "beats": 8, "seamless": true },
  "difficulty": { "energy": 2, "complexity": 2, "coordination": 3 },   // 1–5 each
  "audience": ["all-ages", "classroom"],
  "annotations": [ { "atBeat": 2, "note": "Left foot crosses behind right." } ],
  "keyframes": [ /* see §4 */ ],
  "culturalNotes": [ { "text": "…", "sensitivity": "none" } ]
}
```

Required fields: `stepjson, id, name, style.family, summary, origin.region, sources,
rights.choreographyStatus, rights.reviewedBy, rights.reviewedAt, license, music.*,
loop.*, difficulty.*, audience, keyframes` (`rights.notes`, `origin.era`, `aliases`,
`style.tags`, `annotations`, `culturalNotes` are optional). Unknown keys are
rejected (strict schemas). Reference documents about songs, films, and performers are a
separate entity type and MUST NOT contain media URLs or lyrics.

## 4. Keyframes

Each keyframe is `{ "beat": number, "pose": <pose overlay>, "rootOffset": [x, y]?,
"ease": "linear"|"easeInOut"|"hold", "facing": "front"|"left"|"right"|"back"? }`.

- **Pose overlays** list any subset of the 19 joints as `[x, y]` pairs. Values are
  **absolute positions in rig units, not deltas**. An omitted joint takes its **base
  pose** value; the canonical base is the standing pose (all joints at rest per §2).
  Overlays may be full 19-joint poses (e.g. from motion capture) or partial (e.g.
  footwork-only authoring).
- The first keyframe is at beat 0; all beats are `< loop.beats`. Between keyframes,
  positions interpolate linearly by default; a keyframe's `ease` applies to the segment
  **ending** at that keyframe. `hold` freezes the segment at the previous keyframe's pose.
- **rootOffset** translates the whole figure in rig units (for travelling steps) without
  altering the pose. `loop.seamless: true` means the final segment interpolates from the
  last keyframe to the first keyframe's pose at beat `loop.beats`; with
  `loop.seamless: false` the last keyframe's pose is HELD for the remainder of the loop.
- **facing** is metadata (a 2D rig cannot infer it); renderers that project to 3D may
  consume it.

## 5. Provenance and rights

- `sources[]` (≥ 1): every factual claim in `origin` should be traceable. Kinds:
  encyclopedia, book, journal, video, expert, other.
- `rights.choreographyStatus` taxonomy: `social-dance-steps | folk-traditional |
  historical-reconstruction | original-ours | user-created | licensed | review-required`.
  Social dance steps, folk vocabulary, and short routines are not protectable
  choreographic works (per the U.S. Copyright Office's guidance, Circular 52); documents
  MUST NOT restage identifiable protected choreographic wholes.
- `sensitivity`: `none | context-recommended | consultation-required`. Steps rooted in
  living traditions should carry a `context-recommended` cultural note. Content marked
  `consultation-required` must not be published without documented consultation with
  practitioners.
- Captured (webcam) steps: `choreographyStatus: "user-created"` with an expert-kind
  source recording the capture provenance.

## 6. Share links

A choreography document may be encoded into a URL-safe string:
`"v1." + base64url(canonical_json(document))` where canonical JSON sorts keys recursively
and emits no whitespace; base64url uses `-_` and no padding. Decoders MUST enforce an
8192-character total cap, reject non-canonical base64url characters, and schema-validate
before use.

## 7. Choreography documents (compositional layer)

A choreography composes steps by reference (not by value):

```jsonc
{
  "choreojson": "1.0", "id": "…", "title": "…", "author": "anonymous",
  "createdAt": "2026-08-22", "license": "proprietary",
  "transitionBeats": 1,
  "entries": [
    { "stepId": "example-grapevine", "repeats": 2, "tempoPct": 100,
      "facingOverride": "left", "mirror": false,
      "poseOverlay": { "leftWrist": [0, 1.5] } }
  ],
  "musicRef": "optional-track-id"
}
```

Required fields: `choreojson, id, title, createdAt, entries, transitionBeats`
(`author` defaults to `"anonymous"`, `license` to `"proprietary"`, and `musicRef` is
optional). Bounds: `title` ≤ 80 characters, `author` ≤ 60, `entries` 1–32,
`transitionBeats` 0–4, and per entry `repeats` 1–16 and `tempoPct` 50–150.
`poseOverlay` on an entry merges into every keyframe of that entry's step.

Entries play consecutively; `transitionBeats` cross-fades poses at segment boundaries.
Renderers may add presentation-time transforms (tempo, direction/reverse, body-type
proportions, mirror) without altering documents.

## 8. Interchange

- **BVH:** a StepJSON step or choreography maps to a Z-rotation-only BVH skeleton
  (rest pose = the standing rig, scale 0.35 rig-units-to-meters, root at the hips).
  Planar by construction; segment lengths approximate to the rig's rest lengths.
- **glTF 2.0:** the same skeleton exports as an animation-only glTF asset — joint nodes
  carrying the rest offsets, one animation with a root `translation` channel plus a
  per-joint `rotation` channel (the planar local angle as a quaternion about Z), LINEAR
  sampling over a shared time input — standard glTF 2.0, verified against three.js
  `GLTFLoader`. The asset is animation-only (joint nodes, no mesh or skin), so importers
  show an animated node hierarchy rather than a rendered figure.
- **Joint convention (both formats):** PROXIMAL — a joint node sits at its segment's near
  end and its rotation orients that segment, so rotating a joint moves everything from it
  outward. Because the rig is positional rather than angular, an angle-only export is
  exact for poses that preserve rest segment lengths and approximate otherwise.
- **Rig conventions when porting:** preserve the hipWidth=1 normalization or record the
  scale factor; poses are hip-relative (root translation comes from `rootOffset`).

## 9. Conformance

Schema validation is **necessary but not sufficient**. A conforming step document also
satisfies these rules, which JSON Schema cannot express and which the reference
implementation enforces:

1. Keyframe beats are unique and the earliest is beat `0`. Documents SHOULD list keyframes
   in ascending beat order; implementations sort before evaluating, so order is a
   readability convention rather than a validity rule.
2. Every `beat` is `< loop.beats`, and every `annotations[].atBeat` is `< loop.beats`.
3. `loop.beats` is an integer multiple of `music.counts` (a loop spans whole count cycles).
4. Every pose coordinate is finite, and every resolved pose preserves the normalization
   `dist(leftHip, rightHip) = 1` (within a small tolerance) — an overlay that moves one hip
   must move the other consistently.
5. `sources[].url`, when present, is a well-formed absolute URL.

A validator that checks only the JSON Schemas will accept documents the reference
implementation rejects. Publishing tools SHOULD apply both.

## 10. Versioning

`"stepjson": "1.0"` is feature-frozen. Additive changes will bump to 1.1 with backward
compatibility; breaking changes bump the major version. Validators should reject unknown
future versions explicitly.

## 11. Example

See `examples/side-step.step.json` for a complete, validating step document; §3 above is an abridged illustration. A minimal valid keyframe set:

```json
"keyframes": [
  { "beat": 0, "pose": {} },
  { "beat": 4, "pose": { "leftAnkle": [-0.85, -1.7] }, "rootOffset": [-0.3, 0],
    "ease": "easeInOut" }
]
```

---

*This specification is released for implementation. Machine-readable JSON Schemas for both
document kinds ship alongside it in `schema/`, with worked examples in `examples/`. The
reference implementation (TypeScript schemas, compiler, evaluators, and the BVH and glTF
exporters) is part of the Shadowstep project and is not covered by this license.*
