# Open Laser Target Standard (OLTS)
## Target format specification — Version 0.4

| Item | Value |
|---|---|
| Specification version | **0.4** |
| Release date | **2026-11-10** |
| Status | **Fourth draft; proposed, not ratified** |
| Scope | Printed target, identifying symbols, QR payload, and scoring geometry |
| Companion | `open-laser-target-implementation-notes-v0.4.md` (non-normative) |

**Language note:** This document uses short sentences and a controlled set of terms. It follows the style of ASD-STE100 Simplified Technical English. It does not claim formal ASD-STE100 compliance.

---

### 1. Scope

1.1. OLTS defines the physical layout and machine-readable definition of a paper target for laser dry-fire training.

1.2. A conforming sheet contains three ArUco markers and one QR code. The markers identify and locate the sheet. The QR code contains the complete scoring definition.

1.3. This standard defines the printed format. It does not prescribe a camera, software product, website implementation, training exercise, or method of detecting a laser pulse.

1.4. A decoder can recover the target definition from a conforming QR code without a network connection or a target-specific server record.

### 2. Terms and requirement words

- **Sheet:** One A4 portrait page that follows this standard.
- **Artwork:** The visible target image, excluding the identifying symbols and their protected regions.
- **Target bounds:** The rectangle declared by `b`. It bounds the artwork and scoring zones. Its width is the angular reference width.
- **Zone:** A geometric region with one integer score.
- **Marker:** One black-and-white ArUco symbol.
- **Marker triple:** The ordered IDs of the top-left, top-right, and bottom-left markers.
- **Protected region:** An area reserved for a symbol and its required white surround. It is not a scoring region.
- **Definition:** The target metadata and scoring zones carried in the QR payload.
- **Angular width:** The angle subtended by the width of the target bounds at the target's design distance.
- **MOA:** Minute of angle; one sixtieth of one degree. It is independent of metres, feet, and yards.

**MUST** states a requirement. **SHOULD** states a recommendation. **MAY** states a choice.

### 3. Sheet and coordinates

3.1. The sheet MUST be **A4 portrait**, 210 × 297 mm.

3.2. The origin `(0, 0)` is the top-left corner. The x-axis points right. The y-axis points down.

3.3. The artwork and all scoring geometry MUST fit inside the **printable design area**: `x = 6..204 mm`, `y = 6..291 mm`. This area is not a reserved, unobstructed rectangle. Protected regions MAY cover parts of the artwork and scoring geometry.

3.4. All coordinates and lengths in the definition MUST be integers in units of **0.01 mm**. For example, `10500` means `105.00 mm`.

3.5. Printed dimensions MUST match the defined dimensions. Do not apply automatic page scaling. A changed physical scale requires a new target definition and QR payload.

3.6. The target bounds `b` MUST fit inside the printable design area. The target designer MUST place all target artwork and scoring zones inside these bounds. The bounds MAY intersect protected regions.

### 4. ArUco and QR positions

4.1. The following positions and dimensions are in millimetres. ArUco positions refer to the outer edge of the black marker, **not** its white surround. The QR area includes the QR symbol and its quiet zone.

| Symbol | Top-left x | Top-left y | Width | Height |
|---|---:|---:|---:|---:|
| Marker A, top left | 10 | 10 | **24** | **24** |
| Marker B, top right | 176 | 10 | **24** | **24** |
| Marker C, bottom left | 10 | 263 | **24** | **24** |
| QR area, bottom right | 150 | 237 | **50** | **50** |

4.2. Every ArUco marker MUST be printed upright in sheet coordinates. Use a 4 × 4 data grid and a one-cell black border. A 24 mm marker has six cells along each side; each cell is **4 mm** wide.

4.3. Each ArUco marker MUST have an opaque white surround at least **4 mm** wide on every side. The entire symbol and surround form one protected region.

4.4. The QR code MUST have a white quiet zone at least **four QR modules** wide on every side. The QR symbol and quiet zone MUST fit inside the 50 × 50 mm QR area. All QR modules MUST be square and have the same side length. The printed module side length MUST be at least **0.34 mm**. The full QR area is a protected region.

4.5. The protected regions have these exact closed rectangular bounds:

| Region | Left x | Top y | Right x | Bottom y |
|---|---:|---:|---:|---:|
| Marker A and surround | 6 | 6 | 38 | 38 |
| Marker B and surround | 172 | 6 | 204 | 38 |
| Marker C and surround | 6 | 259 | 38 | 291 |
| QR area | 150 | 237 | 200 | 287 |

4.6. The artwork MUST be rendered **before** the protected regions. Each protected region MUST have an opaque white background. The identifying symbol is then rendered on that background. The surround and quiet zones MUST contain no artwork, text, or other marks.

4.7. The target bounds and scoring geometry MAY pass behind a protected region. A protected region takes precedence over all scoring zones. A point in a protected region has score **0**. This rule also applies to the region boundary.

### 5. Marker IDs and target name

5.1. Use OpenCV ArUco dictionary **`DICT_4X4_50`**. It contains marker IDs `0` through `49`.

5.2. The three markers MUST have different IDs.

5.3. The target name `n` MUST contain one to six characters. Only uppercase ASCII `A`–`Z` and digits `0`–`9` are permitted. The name has no required semantic meaning.

5.4. Derive the ordered marker triple from the target name as follows:

1. Encode the name as ASCII bytes.
2. Compute SHA-256 over those bytes.
3. Read the first eight digest bytes as one unsigned, big-endian 64-bit integer `h`.
4. Compute `k = h mod 117600`.
5. Convert `k` to three different IDs by the algorithm below.

There are `50 × 49 × 48 = 117600` ordered triples with distinct IDs.

```text
A = floor(k / (49 * 48))
t = k mod (49 * 48)
b_rank = floor(t / 48)
c_rank = t mod 48
B = b_rank                if b_rank < A
B = b_rank + 1            otherwise
L = [0, 1, ..., 49] with A and B removed
C = L[c_rank]              # zero-based index
```

5.5. Place ID `A` at the top left, `B` at the top right, and `C` at the bottom left.

5.6. The marker triple is a short identifier, not a globally unique or cryptographically authenticated identifier. Different names can have the same triple. The target name cannot be reconstructed from the triple.

**Marker reference values:**

| Name | First 8 SHA-256 bytes (hex) | `k` | A | B | C |
|---|---|---:|---:|---:|---:|
| `CHF50M` | `948721a47b11c5bd` | 89341 | 37 | 49 | 13 |
| `CHA30` | `23f21b230c9490e8` | 95240 | 40 | 24 | 8 |
| `CHB30` | `e45d27b204f4e501` | 15041 | 6 | 20 | 18 |
| `CHA300` | `0c3aee86e7d8e6d7` | 67799 | 28 | 41 | 23 |
| `DEMO01` | `8633dfcaa308be51` | 102737 | 43 | 33 | 17 |

### 6. QR code and URL encoding

6.1. Use **QR Code Model 2**, error-correction level **M**. Use the smallest QR version that holds the complete URL. The maximum permitted version is **30**.

6.2. Version 30 at level M holds at most **1370 bytes in byte mode**. The complete URL MUST fit. A generator MUST report an error if it does not fit. It MUST NOT discard scoring data to make it fit. Version 30 has 137 × 137 symbol modules. With four-module quiet zones, it occupies 145 × 145 modules. The physical module size MUST satisfy Section 4.4.

6.3. The QR code MUST contain one complete HTTPS URL in this form:

```text
https://<service-host>/t#v0.4.<encoded>
```

The service host is chosen by the publisher. OLTS does not prescribe a provider, product, or domain. The URL fragment MUST contain the complete target definition. The definition MUST NOT depend on a target-specific web record or a redirect service.

6.4. Produce `<encoded>` by applying these operations in order:

1. Make a JSON object that follows Section 7.
2. Encode the JSON text as UTF-8, without a byte-order mark.
3. Compress the UTF-8 bytes with **zlib-wrapped DEFLATE**, as specified in RFC 1950.
4. Encode the compressed bytes with **base64url**, as specified in RFC 4648.
5. Remove all `=` padding from the base64url text.

6.5. Use the exact fragment prefix `v0.4.`. The version prefix identifies the layout and schema rules. A decoder MUST NOT apply version 0.4 rules to a sheet marked `v0.1.`, `v0.2.`, or `v0.3.`.

6.6. The QR URL is self-contained. It can be decoded offline. The HTTPS URL is also usable as an ordinary camera-readable web link when a network is available.

### 7. Target definition and angular width

7.1. The decoded JSON MUST be an object. It MUST contain `n`, `b`, `a`, and `z`. It MAY contain `r` and `u`. No other keys are permitted in version 0.4.

| Key | Type | Required | Meaning |
|---|---|---|---|
| `n` | string | Yes | Target name from Section 5 |
| `b` | four-integer array | Yes | Target bounds `[left, top, width, height]`, in 0.01 mm |
| `a` | positive integer | Yes | Angular width of `b`, in **0.01 MOA** |
| `z` | array | Yes | One or more scoring zones |
| `r` | positive integer | No | Original competition/reference distance (metadata only) |
| `u` | string | No | Unit of `r`: `"m"` or `"yd"`; default `"m"` |

7.2. The target bounds MUST have positive width and height. The entire bounds rectangle MUST fit inside Section 3's printable design area. All scoring zones MUST fit inside the bounds rectangle.

7.3. `a` MUST be an integer from `1` through `1079999`. One unit is **0.01 MOA**. The value describes the horizontal angle subtended by the physical width of `b`, viewed on-axis from the centre at the target's design distance. The design distance is **chosen by the designer**. The standard does not set or limit it.

7.4. The angular relation is exact:

```text
width_mm = b[2] / 100
angle_rad = (a / 100) * pi / 10800
design_distance_mm = width_mm / (2 * tan(angle_rad / 2))
```

The angle is recorded to the nearest 0.01 MOA. A derived distance can therefore differ slightly from the distance used during design. No unit for distance is imposed: for example, `1 yd = 914.4 mm` exactly.

7.5. The original competition/reference distance `r`, if supplied, is **not** the design distance and does not affect scoring. If `u` is omitted, `r` is in metres. `u = "yd"` means yards. `u` MUST NOT occur without `r`.

7.6. There are no permitted-distance lists or fixed training distances in OLTS. The angular width `a` and the physical width of `b` together define the target's intended angular scale.

### 8. Scoring geometries

8.1. All positions and lengths in this section use 0.01 mm units. Scores MUST be integers from `0` through `100`.

8.2. A **circle** zone has this form:

```json
["c", score, center_x, center_y, radius]
```

`radius` MUST be greater than zero.

8.3. A **polygon** zone has this form:

```json
["p", score, [x1, y1, x2, y2, x3, y3]]
```

The coordinate array MAY contain more vertices. The polygon MUST have at least three vertices. Do not repeat the first vertex at the end. The boundary closes from the last vertex to the first. The polygon MUST NOT cross itself.

8.4. An **ellipse** zone has this form:

```json
["e", score, center_x, center_y, radius_x, radius_y, angle]
```

Both radii MUST be greater than zero. One angle unit is **0.01 degree**. `angle` MUST be from `0` through `35999`. Positive angles rotate clockwise in sheet coordinates. At angle `0`, `radius_x` is horizontal and `radius_y` is vertical. An ellipse MUST NOT be used as an approximation to a stadium boundary.

8.5. A **stadium** zone has this form:

```json
["s", score, center_x, center_y, radius, straight_length, angle]
```

`radius` MUST be greater than zero. `straight_length` MUST be zero or greater. The angle uses Section 8.4's units and range. At angle `0`, the straight sides are vertical; the width is `2 × radius` and the height is `straight_length + 2 × radius`. The ends are semicircles. The stadium is the set of points no more than `radius` from the line segment joining its two end-circle centres. If `straight_length` is zero, the stadium is a circle.

**Geometry example:** The Swiss 25 m Ordonnanz inner ten-zone has an official-size stadium with radius 50 mm and straight length 50 mm. Its width is 100 mm and its height is 150 mm. Those are source-target dimensions, not fixed OLTS print dimensions.

8.6. An **SVG path** zone has this form:

```json
["d", score, "M 9500 14000 L 11500 14000 L 11500 16000 L 9500 16000 Z"]
```

The third element MUST use only absolute SVG path commands `M`, `L`, `Q`, `C`, and `Z`. Each command MUST contain exactly one parameter set: `M` and `L` take two coordinates; `Q` takes four; `C` takes six; `Z` takes none. Coordinates MUST be signed decimal integers in 0.01 mm units. Separate every command and number with one ASCII space. Commas, relative commands, and implicit commands are not permitted.

The path MUST contain one or more closed subpaths. Each subpath MUST start with `M` and end with `Z`. Use the SVG **even-odd fill rule** for all subpaths. Holes are outside the path zone. Ignore strokes. Do not use styles, transforms, scripts, or external resources. All endpoints and control points MUST be inside `b`.

8.7. Every scoring region, including rotated shapes and curved paths, MUST fit fully inside target bounds `b`. A shape MAY intersect a protected region, subject to Section 4.7.

### 9. Point-score semantics

9.1. A scoring query is a point in printed-sheet coordinates.

9.2. If the point lies inside a protected region, including its boundary, its score is **0**.

9.3. Otherwise, if the point lies in one or more scoring zones, its score is the **highest** score of those zones. This permits nested scoring regions.

9.4. A point exactly on a zone boundary counts as inside that zone. A point outside all zones has score **0**.

9.5. OLTS defines a point score. It does not define a detected laser spot, projectile diameter, line-cut rules, shot timing, or hit-detection hardware.

### 10. Version compatibility and validity

10.1. A version 0.4 definition MUST use the version 0.4 QR prefix, schema, printed symbol sizes, and protected regions.

10.2. A decoder that claims OLTS 0.4 support MUST also decode valid OLTS 0.1, 0.2, and 0.3 definitions according to **their own versions**. Versions 0.1 and 0.2 have 16 mm markers and different layout and target-distance rules. Their scoring zones and QR data MUST NOT be reinterpreted as version 0.4 data.

10.3. Version 0.1 supports `c` and `p` zones. Version 0.2 adds `e`, `s`, and `d` zones. Version 0.3 adds target bounds, angular width, protected-region precedence, larger markers, and optional yard metadata. Version 0.4 retains the version 0.3 layout and scoring schema. It increases the maximum QR version from 15 to 30 and adds the minimum QR module size in Section 4.4.

10.4. A version 0.4 definition is invalid if it has an unknown key or zone type; invalid geometry; missing required data; a target name inconsistent with its marker triple; or a QR URL that exceeds the specified capacity. A printed sheet is invalid if its QR version, module size, or quiet zone violates Sections 4 and 6.

10.5. Reject a decoded payload if its decompressed JSON exceeds **32768 bytes**, or if its base64url, zlib, UTF-8, or JSON encoding is invalid. QR error correction and the zlib checksum do **not** authenticate the publisher. OLTS 0.4 has no digital signature.

### 11. Conformance test data

11.1. The marker triples in Section 5 MUST match the reference table.

11.2. The following definition is **test data**, not an official 50 m target. Its target bounds are 120 × 120 mm. Its angular width is **82.50 MOA**, which corresponds to a design distance of approximately **5.0001 m**.

```json
{"n":"DEMO01","a":8250,"b":[4500,8500,12000,12000],"r":50,"z":[["c",1,10500,14500,6000],["c",5,10500,14500,3000],["c",10,10500,14500,1000]]}
```

11.3. For this definition, a query at `(115.00, 145.00) mm` has score `10`; `(135.00, 145.00) mm` has score `5`; `(165.00, 145.00) mm` has score `1`; `(170.00, 145.00) mm` has score `0`.

11.4. A test QR URL for the exact JSON in Section 11.2 follows. The domain is a reserved example and is not a live service. The URL is encoded with zlib compression level 9. This is a version 0.4 URL; the JSON and scoring geometry remain unchanged.

```text
https://t.example.org/t#v0.4.eNpVyyEOgDAQRNG7jF4xWyhpVoMkHKCpADwCCeHu0E0IwYx5f05sMPTDOFEhmGEpRAoWWG4jKamOBr5bBDusJseTZKwQFaVXfug8cog_aD5Q_kSrlOsGVBgeEA
```

A conforming decoder MUST recover the exact JSON in Section 11.2 from this URL. Other valid JSON serializations or zlib compression levels MAY produce different URLs for the same target definition.

11.5. For an ellipse zone `["e",8,10500,14500,3000,1500,9000]`, a query at `(105.00, 175.00) mm` is inside; a query at `(105.00, 175.01) mm` is outside.

11.6. For a stadium zone `["s",10,10500,14500,1500,1500,0]`, the width is 30 mm and the height is 45 mm. A query at `(105.00, 167.50) mm` is inside; `(105.00, 167.51) mm` is outside. The point `(120.00, 152.50) mm` is inside this stadium but outside an ellipse of the same centre, with radii 15 mm and 22.5 mm.

11.7. For the path `M 9500 14000 L 11500 14000 L 11500 16000 L 9500 16000 Z M 10000 14500 L 11000 14500 L 11000 15500 L 10000 15500 Z`, use the even-odd rule. `(97.00, 150.00) mm` is inside. `(105.00, 150.00) mm` is outside.

11.8. To test the symbol overlap rule, use target bounds `[600,600,19800,28500]` and a polygon zone `["p",9,[14900,23500,20100,23500,20100,28900,14900,28900]]`. A point at `(160.00, 260.00) mm` is in the polygon but also in the QR protected region, so its score is `0`. A point at `(149.50, 245.00) mm` is in the polygon and outside all protected regions, so its score is `9`.

11.9. Conformance tests MUST also reject an open path, an unsupported path command, an unknown zone type, malformed JSON, an invalid angular width, and a missing `b`. Version 0.4 decoders MUST continue to accept valid version 0.1, 0.2, and 0.3 vectors with their original rules.

11.10. A version 0.4 generator MUST support an ASCII URL of **1264 bytes** in a **Version 29, level M** QR code. It MUST also support a URL of up to **1370 bytes** in a **Version 30, level M** QR code. It MUST reject a longer URL and a QR symbol above Version 30. The QR quiet zone and minimum module size MUST satisfy Section 4.4.

11.11. The `CHF50M` regression fixture in the companion test data has target bounds `b = [7125,5850,6750,15000]`, angular width `a = 3094`, reference distance `r = 50`, and five SVG-path zones. Its minified JSON has SHA-256 `9df5939b4a6d4f4ff9d55f3e5916b11d705f39f80fcecd5e7e500ffe9ba3f84c`. The complete QR URL in `olts-v0.4-CHF50M-regression-url.txt` is **1264 ASCII bytes** and requires **Version 29, level M** when encoded in byte mode. This test verifies capacity and decoding. Physical conformance also requires the correct version 0.4 print layout.

### 12. Exclusions and change control

12.1. This standard does not specify software operation, calibration workflow, shot-detection technique, reader positioning, training distance recommendations, caching, online registries, or user-interface controls. Those topics may appear in non-normative implementation notes.

12.2. Version 0.4 does not embed the full visible artwork SVG in the QR code. It embeds the scoring definition. Future extensions may add the visible artwork or other metadata; such extensions require a new version.

12.3. After publication, do not change the meaning of version 0.4 without issuing a new format version.

### 13. References

- OpenCV, *Detection of ArUco Markers*: <https://docs.opencv.org/4.x/d5/dae/tutorial_aruco_detection.html>
- DENSO WAVE, *QR Code capacity*: <https://www.qrcode.com/en/about/version.html>
- RFC 1950, *ZLIB Compressed Data Format*: <https://www.rfc-editor.org/rfc/rfc1950>
- RFC 4648, *Base64 URL alphabet*: <https://www.rfc-editor.org/rfc/rfc4648>
- RFC 3986, *URI fragments*: <https://www.rfc-editor.org/rfc/rfc3986>
- W3C, *SVG 2 — Paths*: <https://www.w3.org/TR/SVG2/paths.html>
- ASD-STE100, *Simplified Technical English*: <https://www.asd-ste100.org/about.html>

---

**End of OLTS format specification 0.4 — 2026-11-10.**
