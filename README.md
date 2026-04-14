# Thor-AssemblyArt4

Parametric reconstruction of the [Thor Open-Source Robot Arm](https://github.com/AngelLM/Thor) assembly parts using **build123d** — a Python CAD kernel built on OpenCascade.

Each part is reverse-engineered from the original Fusion 360 mesh geometry via coordinate extraction, then rebuilt programmatically and validated against the original STL files.

---

## Project Overview

| Detail | Info |
|---|---|
| **Assigned Repo** | [AngelLM/Thor](https://github.com/AngelLM/Thor) |
| **Submission Repo** | [Thor-AssemblyArt4](https://github.com/avajones081196/Thor-AssemblyArt4) |
| **Completion** | 4 parts done (Art4BearingFix, Art4BodyBot, Art4BodyFan, Art4Optodisk) |
| **Method** | Fusion 360 → CSV coordinates → build123d → STL → Validation |

---

## Parts Progress

| # | Part Name | Completion | Vol. Diff (%) | Sym. Diff (%) | Time |
|---|---|---|---|---|---|
| 1 | Art4BearingFix | ✅ Done | 0.036% | 0.050% | 5 hrs |
| 2 | Art4BodyBot | ✅ Done | 0.030% | 0.037% | 4 hrs |
| 3 | Art4BodyFan | ✅ Done | 0.012% | 0.071% | 2 hrs |
| 4 | Art4Optodisk | ✅ Done | 0.003% | 0.251% | 1.5 hrs |

**Total Time: 12.5 hours**

*Additional parts will be added as they are identified from the Thor assembly.*

---

## Methodology

The workflow follows a 4-stage pipeline for each part:

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
| G1 | Two 3-point circles at Z=0 → annular ring face (inner R≈21, outer R≈30.2) | S1 |
| G2 | Extrude annular profile 15mm in +Z | S1 |
| G3 | Watertight check + export STL + summary (deferred to end) | — |
| G4 | Two 3-point circles at Z=5 → annular groove region (inner R≈21, outer R≈29.2) | S2 |
| G5 | Extrude-cut annular groove 10mm in +Z | S2 |
| G6 | Two 3-point circles at different Z levels, both centered at (0, 24) | S3 |
| G7 | Extrude-cut countersink top (R≈2.95 at Z=5) 3mm in −Z | S3 |
| G8 | Extrude-cut through-hole (R≈1.7 at Z=2) 2mm in −Z | S3 |
| G9 | Circular pattern: replicate G7+G8 cuts — 4 copies at 90° intervals | S3 |
| G10 | 4 corner points → rectangle at X≈30.196, YZ plane | S4 |
| G11 | Extrude-cut rectangle 2mm in −X to make slot (face offset +0.5mm outside body) | S4 |
| G12 | One 3-point circle at Z=5 → circle face (R≈29.2, centered at origin) | S5 |
| G13 | Extrude-cut S5 circle profile 16mm in +Z | S5 |

**Key design decisions (Part 2):**
- 3-point circles use **exact circumscribed-circle fitting** (center + radius from 3 sample points) for smooth CAD geometry.
- Hexagons are drawn from **ordered line-segment chains** snapped into closed loops.
- Boss cut-solids (G12–G14) are assembled from **triangular faces + polygon caps**, sewn, **normal-oriented** (`BRepLib.OrientClosedSolid_s`), then boolean-cut. Without normal orientation, the cut performs a complement subtraction instead of a true cut.
- Tilted-plane circle cuts (G15–G16) use **3-D circumcircle computation** (center, radius, plane normal) with `BRepPrimAPI_MakePrism` along the outward normal.
- Bottom-face circle cuts (G17–G18) use the same OCP prism approach on the flat XY plane at Z=0.

**Key design decisions (Part 3):**
- S1 profile is a **22-vertex polygon** (discretized arc + straight edges) chained into a closed loop using `order_line_segments()`.
- S2 rectangle on the YZ plane uses build123d's `Plane.YZ` with `Mode.SUBTRACT` for the cut.
- S3 arc-slot profile uses **OCP `GC_MakeArcOfCircle`** for 3-point arcs combined with line edges, wired together and prism-extruded via `BRepPrimAPI_MakePrism` along +X.
- S3 screw holes use **OCP circular prism** extrusion on the YZ plane at X=11.

**Key design decisions (Part 4):**
- All circles use **exact circumscribed-circle fitting** from 3 sample points for smooth CAD geometry.
- Annular ring (G1–G2) and annular groove cut (G4–G5) both use `Mode.SUBTRACT` on the inner circle within `BuildSketch` to create clean ring profiles.
- Countersink screw holes (G6–G9) use a two-depth strategy: wider countersink top (R≈2.95) cut 3mm in −Z first, then narrower through-hole (R≈1.7) cut 2mm deeper — replicated 4× via manual 90° rotation of hole centers.
- Rectangle slot (G10–G11) uses **OCP `BRepPrimAPI_MakePrism`** directly with a +0.5mm overhang offset so the tool face starts outside the curved wall, eliminating semicircular residual artifacts from flush-face boolean operations.
- S5 circle cut (G12–G13) uses build123d's `BuildSketch` + `extrude(..., mode=Mode.SUBTRACT)` in +Z, consistent with the annular groove approach in G4–G5.

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

---

## Repository Structure

```
Thor-AssemblyArt4/
├── README.md
├── requirements.txt
│
├── 1_Art4BearingFix/
│   ├── csv_data_1_Art4BearingFix/       # Raw CSVs from Fusion 360
│   │   ├── Fusion_Coordinates_S1.csv
│   │   ├── Fusion_Coordinates_S2_1.csv
│   │   ├── Fusion_Coordinates_S2_2.csv
│   │   ├── Fusion_Coordinates_S2_3.csv
│   │   ├── Fusion_Coordinates_S3.csv
│   │   ├── Fusion_Coordinates_S4.csv
│   │   ├── Fusion_Coordinates_S5.csv
│   │   ├── Fusion_Coordinates_S6.csv
│   │   └── Fusion_Coordinates_S7.csv
│   │
│   ├── csv_merged/                      # Preprocessed CSVs
│   │   ├── Fusion_Coordinates_S1.csv
│   │   ├── Fusion_Coordinates_S2.csv    # Merged from S2_1, S2_2, S2_3
│   │   ├── Fusion_Coordinates_S3.csv
│   │   ├── Fusion_Coordinates_S4.csv
│   │   ├── Fusion_Coordinates_S5.csv
│   │   ├── Fusion_Coordinates_S6.csv
│   │   └── Fusion_Coordinates_S7.csv
│   │
│   ├── 0_preprocess_csvs.py             # Stage 2: CSV preprocessing
│   ├── 1_1_Art4BearingFix_build123d.py  # Stage 3: build123d reconstruction
│   ├── 1_2_compare_stl_files.py         # Stage 4: STL validation
│   │
│   ├── 1_Art4BearingFix_original.stl    # Downloaded from Thor repo
│   ├── 1_Art4BearingFix_build123d_G_1_9.stl  # Our reconstruction
│   │
│   ├── 0_preprocess_csvs_summary.txt
│   ├── 1_Art4BearingFix_build123d_summary_G_1_9.txt
│   └── 1_Art4BearingFix_build123d_vs_original_G_1_9.txt
│
├── 2_Art4BodyBot/
│   ├── csv_data_2_Art4BodyBot/          # Raw CSVs from Fusion 360
│   │   ├── Fusion_Coordinates_S1.csv
│   │   ├── Fusion_Coordinates_S2.csv
│   │   ├── Fusion_Coordinates_S3.csv
│   │   ├── Fusion_Coordinates_S4.csv
│   │   ├── Fusion_Coordinates_S5.csv
│   │   ├── Fusion_Coordinates_S6.csv
│   │   ├── Fusion_Coordinates_S7.csv
│   │   ├── Fusion_Coordinates_S8.csv
│   │   └── Fusion_Coordinates_S9.csv
│   │
│   ├── csv_merged/                      # Preprocessed CSVs
│   │   ├── Fusion_Coordinates_S1.csv
│   │   ├── Fusion_Coordinates_S2.csv
│   │   ├── Fusion_Coordinates_S3.csv
│   │   ├── Fusion_Coordinates_S4.csv
│   │   ├── Fusion_Coordinates_S5.csv
│   │   ├── Fusion_Coordinates_S6.csv
│   │   ├── Fusion_Coordinates_S7.csv
│   │   ├── Fusion_Coordinates_S8.csv
│   │   └── Fusion_Coordinates_S9.csv
│   │
│   ├── 0_preprocess_csvs.py             # Stage 2: CSV preprocessing
│   ├── 2_1_Art4BodyBot_build123d.py     # Stage 3: build123d reconstruction (G1–G18)
│   ├── 2_2_compare_stl_files.py         # Stage 4: STL validation
│   │
│   ├── 2_Art4BodyBot_original.stl       # Downloaded from Thor repo
│   ├── 2_Art4BodyBot_G_1_18.stl         # Our reconstruction
│   │
│   ├── 0_preprocess_csvs_summary.txt
│   ├── 2_Art4BodyBot_summary_G_1_18.txt
│   └── 2_Art4BodyBot_original_build123d_vs_original_G_1_18.txt
│
├── 3_Art4BodyFan/
│   ├── csv_data_3_Art4BodyFan/          # Raw CSVs from Fusion 360
│   │   ├── Fusion_Coordinates_S1.csv
│   │   ├── Fusion_Coordinates_S2.csv
│   │   └── Fusion_Coordinates_S3.csv
│   │
│   ├── csv_merged/                      # Preprocessed CSVs
│   │   ├── Fusion_Coordinates_S1.csv
│   │   ├── Fusion_Coordinates_S2.csv
│   │   └── Fusion_Coordinates_S3.csv
│   │
│   ├── 0_preprocess_csvs.py             # Stage 2: CSV preprocessing
│   ├── 3_1_Art4BodyFan_build123d.py     # Stage 3: build123d reconstruction (G1–G7)
│   ├── 3_2_compare_stl_files.py         # Stage 4: STL validation
│   │
│   ├── 3_Art4BodyFan_original.stl       # Downloaded from Thor repo
│   ├── 3_Art4BodyFan_G_1_7.stl          # Our reconstruction
│   │
│   ├── 0_preprocess_csvs_summary.txt
│   ├── 3_Art4BodyFan_summary_G_1_7.txt
│   └── 3_Art4BodyFan_original_build123d_vs_original_G_1_7.txt
│
├── 4_Art4Optodisk/
│   ├── csv_data_4_Art4Optodisk/         # Raw CSVs from Fusion 360
│   │   ├── Fusion_Coordinates_S1.csv
│   │   ├── Fusion_Coordinates_S2.csv
│   │   ├── Fusion_Coordinates_S3.csv
│   │   ├── Fusion_Coordinates_S4.csv
│   │   └── Fusion_Coordinates_S5.csv
│   │
│   ├── csv_merged/                      # Preprocessed CSVs
│   │   ├── Fusion_Coordinates_S1.csv
│   │   ├── Fusion_Coordinates_S2.csv
│   │   ├── Fusion_Coordinates_S3.csv
│   │   ├── Fusion_Coordinates_S4.csv
│   │   └── Fusion_Coordinates_S5.csv
│   │
│   ├── 0_preprocess_csvs.py             # Stage 2: CSV preprocessing
│   ├── 4_1_Art4Optodisk_build123d.py    # Stage 3: build123d reconstruction (G1–G13)
│   ├── 4_2_compare_stl_files.py         # Stage 4: STL validation
│   │
│   ├── 4_Art4Optodisk_original.stl      # Downloaded from Thor repo
│   ├── 4_Art4Optodisk_G_1_13.stl        # Our reconstruction
│   │
│   ├── 0_preprocess_csvs_summary.txt
│   ├── 4_Art4Optodisk_summary_G_1_13.txt
│   └── 4_Art4Optodisk_original_build123d_vs_original_G_1_13.txt
│
└── ...                                  # Additional parts as identified
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
python 4_1_Art4Optodisk_build123d.py

# 3. Compare against original
python 4_2_compare_stl_files.py
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
