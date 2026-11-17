# OLTS 0.5 delta from OLTS 0.4

Date: 17 November 2026. Status: approved for the project library by the user on 17 November 2026.

This document changes the supplied OLTS 0.4 standard only where stated below. The existing standard and printed sheets retain their original meaning. The five scoring geometries, point-score rules, integer coordinate precision, marker dictionary and ID derivation, compression format, and QR capacity limits remain in force.

## 1. Shared A4 and US Letter layout

Replace the fixed A4 sheet coordinate system with a **common frame 210 × 279.4 mm**. The frame origin is its top-left corner, with x right and y down. All payload coordinates refer to this frame, not to a paper edge.

The frame MUST be centred, without scaling or rotation, on either supported portrait sheet:

| Paper | Page size, mm | Frame origin on page, mm |
|---|---|---|
| A4 | 210 × 297 | (0, 8.8) |
| US Letter | 215.9 × 279.4 | (2.95, 0) |

The common print-safe area is **x = 6..204 mm, y = 6..273.4 mm**. All artwork, identifying symbols, protected regions, labels, and calibration ink MUST fit inside it. Target bounds and every scoring zone MUST also fit inside it. A target MAY extend behind protected regions under Section 4 below.

A4 and Letter copies of the same design use identical payloads and physical dimensions. Only page placement changes. Print at actual size / 100%, centred on the selected paper. Do not use fit-to-page scaling. A centred A4 PDF printed on Letter at 100% loses only blank page margins when this layout is followed. A printer unable to reproduce the common print-safe area cannot produce a conforming print without another printer or paper configuration.

## 2. Explicit marker and QR placement

Add these required JSON fields:

| Key | Value | Meaning |
|---|---|---|
| `m` | `[ax, ay, bx, by, cx, cy]` | Three black-marker top-left positions, in A, B, C ID order |
| `q` | `[qx, qy]` | Top-left of the entire QR area, including its quiet zone |

Every array element MUST be an integer in 0.01 mm units. There are no implicit position defaults. A decoder MUST use these fields for version 0.5; it MUST NOT substitute an earlier version's fixed positions.

Markers remain **24 × 24 mm**, upright, with four-by-four data cells and a one-cell black border. Each retains a white surround **4 mm wide on all sides**. The protected square is therefore exactly **32 × 32 mm**. The QR area remains **50 × 50 mm**, with at least four quiet modules per side and a module pitch of at least **0.34 mm**.

Marker A MUST be above marker C and to the left of marker B. For this draft's rectangular arrangement, `ay = by`, `ax = cx`, `bx > ax`, and `cy > ay`. The protected rectangles MUST have disjoint interiors; touching boundaries are permitted. This gives at least 32 mm separation between the corresponding marker top-left positions. All four protected rectangles MUST fit inside the common print-safe area. The QR MAY be placed separately from the compact marker arrangement.

There is no requirement for markers to sit near the paper corners. Designers MAY bring the three markers closer to a small target without changing target size, angular width, marker size, or scoring geometry. Marker identity A/B/C comes from the ordered hash-derived triple, not from an assumed paper-corner location.

Two valid example layouts, expressed in millimetres for readability:

| Layout | Marker A | Marker B | Marker C | QR area |
|---|---|---|---|---|
| Full-frame example | (10, 10) | (176, 10) | (10, 245.4) | (150, 219.4) |
| Compact rifle example | (55, 80) | (131, 80) | (55, 156) | (127, 134) |

These are examples, not hard-coded parser defaults. The compact example has a 108 × 108 mm bounding rectangle around the three protected marker squares, or 126 × 108 mm including its adjacent QR area. QR scanning can take place before using a tighter camera view; the target definition remains self-contained.

## 3. Payload, units, and version dispatch

The version 0.5 URL prefix is `https://<service-host>/t#v0.5.`. JSON remains UTF-8, compressed with zlib-wrapped DEFLATE, then base64url encoded without padding.

Required keys are **`n`, `b`, `a`, `z`, `m`, `q`**. Optional keys are **`r`, `u`, `du`**. No other keys are permitted.

- `n`, `b`, `a`, `z`, `r`, and `u` retain their version 0.4 meanings.
- `m` and `q` have the meanings in Section 2.
- `du` is an optional design-distance presentation unit: `"m"` or `"yd"`, default `"m"`. It does not change geometry or angular calculations. It is independent of `u`, which specifies the reference-distance unit.

Yard-based design distances were already possible in 0.4. The existing `r` plus `u = "yd"` already represents competition/reference distances such as 25 or 50 yards. A design made for 5 yards MUST use 4572 mm in the angular calculation, not the value for 5 metres. **1 yard = 914.4 mm; 1 inch = 25.4 mm**, exactly. Source dimensions in inches are converted before coordinate rounding.

The exact angular relation remains:

```text
width_mm = b[2] / 100
angle_rad = (a / 100) * pi / 10800
design_distance_mm = width_mm / (2 * tan(angle_rad / 2))
```

Generators MUST label the design distance with its unit. When a reference distance is printed, its unit MUST also be explicit. Labels MAY round the derived design distance for readability, but MUST agree with the encoded angle and width at the displayed precision. `du` records the designer's presentation unit; a reader MAY additionally show an equivalent distance in a user-selected unit.

Version 0.5 retains **QR Model 2, correction M, smallest fitting version, maximum version 30, maximum complete byte-mode URL 1370 bytes**, and the 32768-byte decompressed JSON limit. Position metadata consumes part of that same capacity. A generator MUST fail if the complete definition does not fit; it MUST NOT discard or silently simplify geometry.

Within one published version of a target library, variants with different scoring geometry, target bounds, angular width, or symbol layout MUST have distinct names and distinct ordered marker triples. A4 and Letter copies of the same definition MAY share the same identity. Marker triples are still not globally unique; readers must retain the version and complete definition identity and must not treat a triple alone as proof of a particular definition.

## 4. Protected regions and scoring

The payload encodes symbol positions. Protected rectangles are **derived**, avoiding a second, potentially contradictory list of mask coordinates.

For a marker top-left position `(x, y)`, in millimetres:

```text
marker protection = [x - 4, y - 4, x + 28, y + 28]
QR protection     = [qx, qy, qx + 50, qy + 50]
```

These are closed rectangles in common-frame coordinates. Their boundaries and interiors have score **zero**, taking precedence over all scoring zones. Artwork is drawn first, followed by opaque white protected rectangles and then their symbols. No artwork, labels, or calibration marks may intrude into surrounds or quiet zones.

All other point-score rules remain unchanged. Changing marker or QR placement can change which part of an underlying zone is protected; generators MUST show the same masks on the scoring companion and document material masking of target artwork or scoring areas.

## 5. Print calibration

Each target sheet and scoring companion MUST include one horizontal and one vertical labelled calibration segment, with clear endpoint ticks. Both segments MUST use the same chosen length: **50 mm** or **2 in (50.8 mm)**. They MUST lie inside the common print-safe area and outside target bounds and protected regions. Measure tick-centre to tick-centre. Line thickness MUST NOT change the declared endpoint separation.

The selected unit MUST be stated on both segments. A dual-unit label MAY show the equivalent length. A 50 mm segment MUST NOT be labelled 2 inches. Imperial editions SHOULD use the 2-inch option; metric editions SHOULD use the 50 mm option. Segment placement is a sheet-generation choice and is not included in the scoring payload.

## 6. Compatibility and validation

A reader claiming version 0.5 support MUST dispatch by the QR version and retain the original rules for valid versions 0.1 through 0.4. Earlier versions do not acquire movable markers or a common-frame origin retroactively. A 0.5 definition missing `m` or `q` is invalid. A 0.4 definition containing these new keys remains invalid under 0.4.

In addition to retained geometry and encoding tests, check:

1. Required field presence, array lengths, integer coordinates, permitted units and unknown-key rejection.
2. Marker size, orientation, rectangular arrangement, non-overlap of protection interiors, and containment of all protection rectangles in the common print-safe area.
3. The exact A4/Letter frame translation, identical physical geometry, and absence of automatic scaling.
4. Point-score precedence at the derived protection rectangles, including their boundaries.
5. Metric and yard design examples, independently selected reference units, and correct calibration lengths.
6. Distinct variant identities within the library, preserving the separate 5 m Ordonnanz identity.
7. Complete QR capacity after adding layout metadata, QR decoding, and detection of the three expected marker IDs in the rendered sheets.

## 7. Library deliverables and implementation sequence

The following is a project workflow requirement, not additional QR schema:

- For each target variant: a two-page PDF, target first and scoring companion second, plus both SVG originals and the decoded definition.
- For distribution: one PDF containing all target pages in catalogue order, with no companion pages.
- Keep a catalogue, source/dimension references, versioned specification/delta, previews, and validation results with the library.
- Keep rebuilds manually triggerable and repeatable. A single build entry point must rebuild and verify the complete catalogue before replacing the distribution package.

The first rebuild under this delta covers fifteen variants: thirteen from the original eight Swiss families, including the distinct 5 m Ordonnanz variant, plus NRA B-6 and B-8 editions for 5-yard practice. Rifle sheets use the compact placement example; larger pistol sheets use the full-frame example unless documented otherwise. The separate bonus 50 m ISSF drawing is not part of these thirteen variants.

App support is separate work. The updated library carries the new version prefix and requires a reader that implements this draft; it is not compatible with a reader that only implements the version 0.4 layout. Prior library files remain available for the existing app.
