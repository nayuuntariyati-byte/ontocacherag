# Results

This directory holds results of paper experiments.

## Structure

```
results/
├── tables/          # All tables in 1 Excel file and 9 CSV files
└── figures/         # Image file (PDF + PNG)
    ├── fig1_architecture.png
    ├── fig2_scalability_measured.png
    ├── fig3_detection.pdf
    └── fig4_cache_state.pdf
```

## Provenance

Each result file documents in its first line:
- Date: timestamp of generation
- Seed: random seed used (where applicable)
- Hash: SHA-256 of the input data (for verification)
