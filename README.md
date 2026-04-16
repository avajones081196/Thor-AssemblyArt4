# Thor-AssemblyArt4

Parametric reconstruction of the [Thor Open-Source Robot Arm](https://github.com/AngelLM/Thor) assembly parts using **build123d** — a Python CAD kernel built on OpenCascade.

Each part is reverse-engineered from the original Fusion 360 mesh geometry via coordinate extraction, then rebuilt programmatically and validated against the original STL files.

---

## Project Overview

| Detail | Info |
|---|---|
| **Assigned Repo** | [AngelLM/Thor](https://github.com/AngelLM/Thor) |
| **Submission Repo** | [Thor-AssemblyArt4](https://github.com/avajones081196/Thor-AssemblyArt4) |
| **Completion** | 7 parts done |
| **Method** | Fusion 360 → CSV coordinates → build123d → STL → Validation |

---

## Parts Progress

| # | Part Name | Completion | Vol. Diff (%) | Sym. Diff (%) | Time |
|---|---|---|---|---|---|
| 1 | Art4BearingFix | ✅ Done | 0.036% | 0.050% | 5 hrs |
| 2 | Art4BodyBot | ✅ Done | 0.030% | 0.037% | 4 hrs |
| 3 | Art4BodyFan | ✅ Done | 0.012% | 0.071% | 2 hrs |
| 4 | Art4Optodisk | ✅ Done | 0.003% | 0.251% | 1.5 hrs |
| 5 | Art4TransmissionColumn | ✅ Done | 0.428% | N/A* | 6 hrs |
| 6 | Art4BearingPlug | ✅ Done | 0.784% | 0.913% | 3 hrs |
| 7 | Art4BearingRing | ✅ Done | 0.095% | 0.391% | 2 hrs |

**Total Time: 23.5 hours**

*\*Symmetric difference could not be computed for Part 5 because the original downloaded STL mesh is not watertight (`Mesh B watertight: False`). This is a property of the source mesh, not the reconstruction.*

*Note: Small volume differences (especially on circular features) are expected because the original STL uses faceted polygon approximations for circles, while build123d uses true mathematical circle geometry — making the reconstruction more geometrically accurate than the source mesh.*

---

## Methodology

The workflow follows a 4-stage pipeline for each part:

Fusion 360 Mesh  →  CSV Extraction  →  build123d Reconstruction  →  STL Validation
(1)                (2)                    (3)                       (4)


```
Fusion 360 Mesh  →  CSV Extraction  →  build123d Reconstruction  →  STL Validation
     (1)                (2)                    (3)                       (4)
```

### Stage 1 — Coordinate Extraction (Fusion 360)

A Fusion 360 Python script traverses the mesh body and exports vertex coordinates to CSV files. Each CSV captures a specific geometric feature:

- **Flat profiles** — Line segments forming closed outlines (base faces)
- **Triangular faces** — Vertex triplets defining mesh surfaces (top, sides)
- **Hole boundaries** — Line loops defining hole edges on curved surfaces
- **Circle profiles** — 3-point circle definitions or polygonal approximations
- **Hexagonal profiles** — Line-segment loops for bolt/nut pocket cuts
- **Arc profiles** — 3-point arc definitions for curved slot openings
- **Tooth profiles** — Closed polygon outlines of gear teeth at different Z planes
- **Revolution profiles** — Line + arc chains on YZ plane for revolve operations
- **D-shaped profiles** — Line + arc closed loops for loft-cut and tapered extrude operations

### Stage 2 — CSV Preprocessing (`0_preprocess_csvs.py`)

- Detects and merges split CSV files (e.g., `S2_1.csv`, `S2_2.csv` → `S2.csv`)
- Removes geometry-aware duplicates (direction-independent lines, rotation-independent triangles)
- Re-numbers steps sequentially
- Incremental mode — safe to re-run without reprocessing existing shapes

### Stage 3 — build123d Reconstruction

The reconstruction follows numbered guidelines, each building on the previous. Each part has its own guideline set tailored to the geometry.

**Part 1 — Art4BearingFix** (`1_1_Art4BearingFix_build123d.py`, G1–G9):

| Guideline | Operation | Data Source |
|---|---|---|
| G1 | Build bottom face from line profile | S1 |
| G2 | Build top surface triangles + fill holes | S2, S3, S4 |
| G3 | Build side wall triangles | S5 |
| G4 | Sew all faces → watertight shell → solid | All above |
| G5 | Export STL (always last) | — |
| G6 | Draw circle profiles at Z=0 | S6 |
| G7 | Extrude-cut S6 circles +4mm in +Z | S6 |
| G8 | Draw circle profiles at Z=1 | S7 |
| G9 | Extrude-cut S7 circles +4mm in +Z | S7 |

**Part 2 — Art4BodyBot** (`2_1_Art4BodyBot_build123d.py`, G1–G18):

| Guideline | Operation | Data Source |
|---|---|---|
| G1 | Two 3-point circles → annular ring profile | S1 |
| G2 | Extrude annular disc 4.5mm in −Z | S1 |
| G3 | Checkpoint | — |
| G4 | Four hexagon profiles from line segments | S2 |
| G5 | Extrude-cut hexagons 2mm in −Z | S2 |
| G6 | Four 3-point circle profiles at Z=2.5 | S3 |
| G7 | Extrude-cut S3 circles 2.5mm in −Z | S3 |
| G8 | Four 3-point circle profiles at Z=4.5 | S4 |
| G9 | Extrude-cut S4 circles 4.5mm in −Z | S4 |
| G10 | Four enclosed line-segment profiles | S5 |
| G11 | Extrude (join) S5 profiles 10mm in +Z | S5 |
| G12 | Triangular side faces + top quad lines (4 bosses) | S6 |
| G13 | Non-planar pentagonal cap loops (fan-triangulated) | S7 |
| G14 | Sew + orient + boolean-cut 4 boss solids | S6, S7 |
| G15 | Four 3-point circles on tilted planes | S8 |
| G16 | Extrude-cut S8 circles 8mm outward-normally | S8 |
| G17 | Four 3-point circles at Z=0 (flat XY plane) | S9 |
| G18 | Extrude-cut S9 circles 3mm in +Z | S9 |

**Part 3 — Art4BodyFan** (`3_1_Art4BodyFan_build123d.py`, G1–G7):

| Guideline | Operation | Data Source |
|---|---|---|
| G1 | Read S1 line segments → enclosed half-box profile at Z=50 | S1 |
| G2 | Extrude profile 50mm in −Z | S1 |
| G3 | Export STL + summary (deferred to end) | — |
| G4 | Read S2 → rectangular profile on YZ plane (X=0) | S2 |
| G5 | Extrude-cut rectangle 11mm in +X | S2 |
| G6 | Read S3 → 1 arc-slot + 4 circle profiles at X=11 | S3 |
| G7 | Extrude-cut S3 profiles 10mm in +X | S3 |

**Part 4 — Art4Optodisk** (`4_1_Art4Optodisk_build123d.py`, G1–G13):

| Guideline | Operation | Data Source |
|---|---|---|
| G1 | Two 3-point circles at Z=0 → annular ring face | S1 |
| G2 | Extrude annular profile 15mm in +Z | S1 |
| G3 | Watertight check + export STL + summary (deferred to end) | — |
| G4 | Two 3-point circles at Z=5 → annular groove region | S2 |
| G5 | Extrude-cut annular groove 10mm in +Z | S2 |
| G6 | Two 3-point circles at different Z levels | S3 |
| G7 | Extrude-cut countersink top 3mm in −Z | S3 |
| G8 | Extrude-cut through-hole 2mm in −Z | S3 |
| G9 | Circular pattern: 4 copies at 90° intervals | S3 |
| G10 | 4 corner points → rectangle at X≈30.196 | S4 |
| G11 | Extrude-cut rectangle 2mm in −X | S4 |
| G12 | One 3-point circle at Z=5 | S5 |
| G13 | Extrude-cut S5 circle 16mm in +Z | S5 |

**Part 5 — Art4TransmissionColumn** (`5_1_Art4TransmissionColumn_build123d.py`, G1–G16):

| Guideline | Operation | Data Source |
|---|---|---|
| G1 | Read S1 — 11 lines + 2 arcs on YZ plane (report only, arcs are bad) | S1 |
| G12 | Read S5 corrected arcs, combine with S1 lines → closed revolution profile | S1, S5 |
| G2 | Revolve corrected profile 360° about Z axis | S1, S5 |
| G3 | Watertight check + export STL + summary (deferred to end) | — |
| G4 | One 3-point circle at Z=76 (top face) | S2 |
| G5 | Extrude-cut S2 circle 13mm in −Z | S2 |
| G6 | Circular pattern of G5 — 4 copies at 90° around Z | S2 |
| G7 | Four hexagonal line-loop profiles at Z=76 | S3 |
| G8 | Extrude-cut 4 hexagons 7mm in −Z | S3 |
| G9 | Two 3-point circles at Z=64 (countersink) | S4 |
| G10 | Extrude-cut inner circle 13mm +Z, outer circle 3mm +Z | S4 |
| G11 | Circular pattern of G10 — 4 copies at 90° around Z | S4 |
| G13 | Read S6 — front tooth profile at Z=0 (33-segment closed loop) | S6 |
| G14 | Read S7 — back tooth profile at Z=17 (36-segment closed loop) | S7 |
| G15 | Loft S6 → S7 via ruled ThruSections → one gear tooth, boolean-join | S6, S7 |
| G16 | Circular pattern — 20 teeth at 18° intervals around Z axis | S6, S7 |

**Part 6 — Art4BearingPlug** (`6_1_Art4BearingPlug_build123d.py`, G1–G9):

| Guideline | Operation | Data Source |
|---|---|---|
| G1 | Read S1 — circle at X=6.95 (R≈3.6) on YZ plane | S1 |
| G2 | Extrude circle 3mm in −X → cylinder (X=6.95 to X=3.95) | S1 |
| G3 | Watertight check + export STL + summary (deferred to end) | — |
| G4 | Read S1 — circle at X=0 (R≈3.0) on YZ plane | S1 |
| G5 | Loft X=0 (R=3.0) → X=3.95 (R=3.6), join to cylinder | S1 |
| G6 | Read S6–S8: three D-shaped profiles at Y=0, Y=−1.978, Y=+1.978 | S6, S7, S8 |
| G7 | 3-section loft-cut S8→S6→S7 (top→mid→bottom), subtract from body | S6, S7, S8 |
| G8 | Tapered extrude-cut S8 in +Y, taper=−1.361°, dist=2.3 (top) | S8 |
| G9 | Tapered extrude-cut S7 in −Y, taper=−1.361°, dist=3.9 (bottom) | S7 |

**Key design decisions (Part 5):**
- **First revolve-based part**: S1 revolution profile revolved 360° about Z using `BRepPrimAPI_MakeRevol`.
- S1 arcs were degenerate — **G12 reads corrected arcs from S5**.
- **Helical gear teeth** (G13–G16): ruled loft + circular pattern of 20 teeth.
- Volume difference (0.43%) from ruled loft approximation + mesh-vs-circle geometry.

**Key design decisions (Part 6):**
- **Cylinder + lofted cone** body: S1 provides two circles at different X levels. Cylinder extruded in −X, then lofted to smaller circle at X=0.
- **D-shaped cut profiles**: S6 (Y=0, 9 lines), S7 (Y=−1.978, line+arc), S8 (Y=+1.978, line+arc). Lines and 3-point arcs handled by `build_wire_from_rows()` with `GC_MakeArcOfCircle`.
- **3-section loft-cut** (G7): `BRepOffsetAPI_ThruSections` through S8→S6→S7 creates the volume between the three D-shaped profiles, then boolean-subtracted.
- **Tapered extrude** (G8–G9): simulated via loft between original wire and translated+scaled copy. Scale factor = `1 + dist × tan(taper_angle)` where taper = −1.361°. This removes the remaining corner material on top and bottom.
- Volume difference (0.78%) from tapered extrude approximation via loft scaling.

### Stage 4 — Validation (`*_compare_stl_files.py`)

Compares the build123d STL against the original downloaded from the Thor repo:

- **Volume comparison** — absolute difference + % error
- **Symmetric difference** — boolean intersection to measure spatial overlap
- **Bounding box check** — per-axis tolerance ±0.1mm
- **Summary scorecard** — automatic grading (Excellent / Good / Acceptable / Poor)

---

## Part 1: Art4BearingFix — Results

```
🟢 EXCELLENT across all metrics

Volume % error:          0.036%
Symmetric diff % error:  0.050%
Overlap coverage:        99.99%
Bounding box:            ✅ PASS (all axes within ±0.1mm)

Watertight mesh:         ✅ 0 free edges
Solid volume:            880.307 mm³  (original: 879.991 mm³)
```

## Part 2: Art4BodyBot — Results

```
🟢 EXCELLENT across all metrics

Volume % error:          0.030%
Symmetric diff % error:  0.037%
Overlap coverage:        100.00%
Bounding box:            ✅ PASS (all axes within ±0.1mm)

Watertight mesh:         ✅ 0 free edges
Solid volume:            43,401.296 mm³  (original: 43,384.240 mm³)
```

## Part 3: Art4BodyFan — Results

```
🟢 EXCELLENT across all metrics

Volume % error:          0.012%
Symmetric diff % error:  0.071%
Overlap coverage:        99.97%
Bounding box:            ✅ PASS (all axes within ±0.1mm)

Watertight mesh:         ✅ 0 free edges
Solid volume:            12,563.366 mm³  (original: 12,561.910 mm³)
```

## Part 4: Art4Optodisk — Results

```
🟢 EXCELLENT across all metrics

Volume % error:          0.003%
Symmetric diff % error:  0.251%
Overlap coverage:        99.88%
Bounding box:            ✅ PASS (all axes within ±0.1mm)

Watertight mesh:         ✅ 0 free edges
Solid volume:            8,853.517 mm³  (original: 8,853.255 mm³)
```

## Part 5: Art4TransmissionColumn — Results

```
🟢 EXCELLENT (volume)  |  Sym diff: N/A (original mesh not watertight)

Volume % error:          0.428%
Symmetric diff % error:  N/A (original STL not watertight — cannot compute)
Bounding box:            ✅ PASS (all axes within ±0.1mm)

Watertight mesh:         ✅ 0 free edges (1398 faces)
Solid volume:            78,084.016 mm³  (original: 78,419.464 mm³)
```

## Part 6: Art4BearingPlug — Results

```
🟢 EXCELLENT across all metrics

Volume % error:          0.784%
Symmetric diff % error:  0.913%
Overlap coverage:        99.94%
Bounding box:            ✅ PASS (all axes within ±0.1mm)

Watertight mesh:         ✅ 0 free edges (15 faces)
Solid volume:            239.415 mm³  (original: 237.553 mm³)

Note: Volume difference primarily from tapered extrude approximation
(loft between original and scaled wire) and mesh-vs-circle geometry.
```

---

## Repository Structure

```
Thor-AssemblyArt4/
├── README.md
├── requirements.txt
│
├── 1_Art4BearingFix/
│   ├── csv_data_1_Art4BearingFix/       # Raw CSVs from Fusion 360
│   ├── csv_merged/                      # Preprocessed CSVs
│   ├── 0_preprocess_csvs.py
│   ├── 1_1_Art4BearingFix_build123d.py
│   ├── 1_2_compare_stl_files.py
│   ├── 1_Art4BearingFix_original.stl
│   └── 1_Art4BearingFix_build123d_G_1_9.stl
│
├── 2_Art4BodyBot/
│   ├── csv_data_2_Art4BodyBot/
│   ├── csv_merged/
│   ├── 0_preprocess_csvs.py
│   ├── 2_1_Art4BodyBot_build123d.py
│   ├── 2_2_compare_stl_files.py
│   ├── 2_Art4BodyBot_original.stl
│   └── 2_Art4BodyBot_G_1_18.stl
│
├── 3_Art4BodyFan/
│   ├── csv_data_3_Art4BodyFan/
│   ├── csv_merged/
│   ├── 0_preprocess_csvs.py
│   ├── 3_1_Art4BodyFan_build123d.py
│   ├── 3_2_compare_stl_files.py
│   ├── 3_Art4BodyFan_original.stl
│   └── 3_Art4BodyFan_G_1_7.stl
│
├── 4_Art4Optodisk/
│   ├── csv_data_4_Art4Optodisk/
│   ├── csv_merged/
│   ├── 0_preprocess_csvs.py
│   ├── 4_1_Art4Optodisk_build123d.py
│   ├── 4_2_compare_stl_files.py
│   ├── 4_Art4Optodisk_original.stl
│   └── 4_Art4Optodisk_G_1_13.stl
│
├── 5_Art4TransmissionColumn/
│   ├── csv_data_5_Art4TransmissionColumn/
│   ├── csv_merged/
│   ├── 0_preprocess_csvs.py
│   ├── 5_1_Art4TransmissionColumn_build123d.py
│   ├── 5_2_compare_stl_files.py
│   ├── 5_Art4TransmissionColumn_original.stl
│   └── 5_Art4TransmissionColumn_G_1_16.stl
│
├── 6_Art4BearingPlug/
│   ├── csv_data_6_Art4BearingPlug/         # Raw CSVs from Fusion 360
│   │   ├── Fusion_Coordinates_S1.csv       # Two circles (X=6.95, X=0)
│   │   ├── Fusion_Coordinates_S6.csv       # Middle D-shape (Y=0, 9 lines)
│   │   ├── Fusion_Coordinates_S7.csv       # Bottom D-shape (Y=−1.978, line+arc)
│   │   └── Fusion_Coordinates_S8.csv       # Top D-shape (Y=+1.978, line+arc)
│   │
│   ├── csv_merged/                         # Preprocessed CSVs
│   ├── 0_preprocess_csvs.py
│   ├── 6_1_Art4BearingPlug_build123d.py    # G1–G9
│   ├── 6_2_compare_stl_files.py
│   ├── 6_Art4BearingPlug_original.stl
│   └── 6_Art4BearingPlug_G_1_9.stl
│
└── ...
```

---

## Setup & Usage

### Prerequisites

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### requirements.txt

```
build123d
ocp_vscode
numpy-stl
trimesh
manifold3d
```

### Running (per part)

```bash
# 1. Preprocess CSVs (merge splits, remove duplicates)
python 0_preprocess_csvs.py

# 2. Build the part
python 6_1_Art4BearingPlug_build123d.py

# 3. Compare against original
python 6_2_compare_stl_files.py
```

---

## CSV Format

All coordinate files follow the same schema:

| Column | Description |
|---|---|
| Steps | Sequential row number |
| Draw Type | `Line`, `triangular_face_N`, `3_point_arc_N`, `3_point_circle_N`, `Point` |
| X1, Y1, Z1 | First point coordinates |
| X2, Y2, Z2 | Second point (or `NA`) |
| X3, Y3, Z3 | Third point (or `NA`) |

---

## Technologies

- **Fusion 360** — Source mesh geometry + coordinate extraction
- **build123d** — Python CAD kernel (OpenCascade-based)
- **OCP CAD Viewer** — VS Code extension for 3D visualization
- **trimesh + manifold3d** — STL comparison and boolean operations
- **numpy-stl** — Mesh property calculations

---

## License

This project reconstructs parts from the [Thor robot arm](https://github.com/AngelLM/Thor) by AngelLM, which is shared under open-source terms. This reconstruction is for educational purposes.
