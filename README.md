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
└── ...                                  # Additional parts as identified
