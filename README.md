# Thor-AssemblyArt4

Parametric reconstruction of the [Thor Open-Source Robot Arm](https://github.com/AngelLM/Thor) assembly parts using **build123d** — a Python CAD kernel built on OpenCascade.

Each part is reverse-engineered from the original Fusion 360 mesh geometry via coordinate extraction, then rebuilt programmatically and validated against the original STL files.

---

## Project Overview

| Detail | Info |
|---|---|
| **Assigned Repo** | [AngelLM/Thor](https://github.com/AngelLM/Thor) |
| **Submission Repo** | [Thor-AssemblyArt4](https://github.com/avajones081196/Thor-AssemblyArt4) |
| **Completion** | 1 part done (Art4BearingFix) |
| **Method** | Fusion 360 → CSV coordinates → build123d → STL → Validation |

---

## Parts Progress

| # | Part Name | Completion | Vol. Diff (%) | Sym. Diff (%) | Time |
|---|---|---|---|---|---|
| 1 | Art4BearingFix | ✅ Done | 0.036% | 0.050% | 5 hrs |

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
- **Circle profiles** — Polygonal approximations of circular cut features

### Stage 2 — CSV Preprocessing (`0_preprocess_csvs.py`)

- Detects and merges split CSV files (e.g., `S2_1.csv`, `S2_2.csv` → `S2.csv`)
- Removes geometry-aware duplicates (direction-independent lines, rotation-independent triangles)
- Re-numbers steps sequentially
- Incremental mode — safe to re-run without reprocessing existing shapes

### Stage 3 — build123d Reconstruction (`1_1_*_build123d.py`)

The reconstruction follows numbered guidelines, each building on the previous:

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

**Key design decisions:**
- Circles are kept as **line-segment polygons** from the CSV data — no center/radius fitting. This preserves exact vertex positions for mesh comparison.
- Holes in the top surface are filled using **fan triangulation** from the centroid of the boundary loop.
- The solid is built by **face assembly + sewing** (no boolean extrude-then-cut for the main body), ensuring a watertight mesh with 0 free edges.

### Stage 4 — Validation (`1_2_compare_stl_files.py`)

Compares the build123d STL against the original downloaded from [Thor/stl/Art4BearingFix.stl](https://github.com/AngelLM/Thor/blob/main/stl/Art4BearingFix.stl):

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
├── 2_Art4BodyBot/                       # (Next part — TBD)
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
python 1_1_Art4BearingFix_build123d.py

# 3. Compare against original
python 1_2_compare_stl_files.py
```

---

## CSV Format

All coordinate files follow the same schema:

| Column | Description |
|---|---|
| Steps | Sequential row number |
| Draw Type | `Line`, `triangular_face_N`, `3_point_arc`, `3_point_circle`, `Point` |
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
