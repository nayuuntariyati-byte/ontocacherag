# Results

This directory holds reproducible results of paper experiments.

## Structure

```
results/
├── tables/          # Numerical results (CSV format)
│   ├── table3_ontology_stats.csv
│   ├── table4_abox_detection.csv
│   ├── table5_tbox_detection.csv
│   ├── table6_latency_breakdown.csv
│   ├── table7_cache_efficiency.csv
│   ├── table8_ablation.csv
│   └── table9_scalability.csv
│
└── figures/         # Generated plots (PDF + PNG)
    ├── fig1_architecture.pdf
    ├── fig2_scalability.pdf
    ├── fig3_detection.pdf
    └── fig4_cache_state.pdf
```

## Provenance

Each result file documents in its first line:

- Source: which script generated it
- Date: timestamp of generation
- Seed: random seed used (where applicable)
- Hash: SHA-256 of the input data (for verification)

## Reproducing These Results

See [`docs/REPRODUCIBILITY.md`](../docs/REPRODUCIBILITY.md) for complete instructions.

Quick reference:

| Paper Element | How to Reproduce |
|---|---|
| Table 3 (Section 5.2) | `python scripts/build_ontology.py` |
| Table 9 (Section 6.6) | `python scripts/synthetic_generator.py --paper-scales` |
| Figure 2 (Section 6.6) | Plot Table 9 values on log-log scale |

The detection accuracy and cache efficiency experiments (Tables 4–8, Figures 3–4) require the full evaluation pipeline including cache workload simulation, which is documented separately upon publication.
