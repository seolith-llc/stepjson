# StepJSON

**A JSON-native, machine-readable notation for dance steps.**

A StepJSON document describes a short, repeatable unit of dance movement as timed keyframe
poses on a normalized two-dimensional skeleton, together with the cultural, historical, and
rights metadata needed to use it responsibly. It is renderer-agnostic: the same document can
drive a 2D figure, a 3D skeleton, or motion interchange formats like BVH and glTF.

- **[Read the specification →](stepjson-spec.md)** (version 1.0, stable)
- **Validate against it →** [`schema/1.0/step.schema.json`](schema/1.0/step.schema.json) ·
  [`schema/1.0/choreo.schema.json`](schema/1.0/choreo.schema.json) (JSON Schema draft-07)
- **Worked examples →** [`examples/`](examples/)

## Why it exists

Classical notation systems — Labanotation, Benesh, Eshkol-Wachman — encode movement as
graphical scores that are precise but hard to compute over. StepJSON encodes movement as
plain data: JSON that validates against a schema, diffs cleanly in version control, and
feeds any renderer.

Two properties are unusual and deliberate:

1. **Provenance is mandatory.** Every document carries `sources`, a rights status, and
   review attribution. Movement heritage deserves citation-grade traceability, and a
   format that makes provenance optional gets used without it.
2. **Deterministic replay.** Identical input and beat position always evaluate to an
   identical pose — no timestamps, randomness, or environment dependence inside the motion
   data. Renderings are reproducible and diffable.

## Quick look

Abridged — required fields are omitted for brevity; see
[`examples/side-step.step.json`](examples/side-step.step.json) for a complete document.

```jsonc
{
  "stepjson": "1.0",
  "id": "example-side-step",
  "name": "Side Step",
  "music": { "defaultBpm": 100, "meter": "4/4", "counts": 4 },
  "loop": { "beats": 4, "seamless": true },
  "keyframes": [
    { "beat": 0, "pose": {}, "ease": "easeInOut" },
    { "beat": 1, "pose": { "leftAnkle": [-0.95, -1.7] }, "ease": "easeInOut" }
  ]
}
```

Poses are expressed on a 19-joint skeleton whose names align with the MediaPipe Pose
landmark subset, so camera-based capture maps directly. Coordinates are in rig units:
origin at the hip midpoint, `+y` up, normalized so `dist(leftHip, rightHip) = 1`. Joints a
keyframe does not mention inherit the rest pose — so a keyframe states only what moves.

## Validating a document

Any JSON Schema draft-07 validator works. For example, with
[ajv-cli](https://github.com/ajv-validator/ajv-cli):

```bash
npx ajv-cli validate -s schema/1.0/step.schema.json -d my-step.json
```

The schemas are kept honest by a test suite in the reference implementation that asserts
they accept every document in a 30-step catalog, accept the examples here, and reject a
crafted invalid set.

**Schema validity is necessary but not sufficient.** A few rules cannot be expressed in
JSON Schema — keyframes sorted and starting at beat 0, `loop.beats` a multiple of
`music.counts`, the `hipWidth = 1` normalization, well-formed source URLs. They are listed
in [specification §9](stepjson-spec.md#9-conformance); a publishing tool should check both.

## Versioning

`"stepjson": "1.0"` is feature-frozen. Additive changes bump the minor version with
backward compatibility; breaking changes bump the major. **Validators should reject unknown
future versions explicitly** rather than ignoring the field.

## Status and scope

Version 1.0 is stable and is the working format of the Shadowstep project, which maintains
a catalog of rights-cleared steps, a choreography studio, and BVH/glTF export built on it.

This repository contains the **specification only**. The Shadowstep step catalog and the
TypeScript reference implementation are separate works and are not published here.

## License

The specification, schemas, and examples are licensed **[CC BY 4.0](LICENSE)** — implement
them freely, including commercially, with attribution to Seolith LLC.

Implementations do not need permission and there is no conformance programme: a document
that satisfies the schemas and the §9 rules, and replays deterministically, is StepJSON.
