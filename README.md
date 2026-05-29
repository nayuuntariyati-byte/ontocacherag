# OntoCacheRAG

**Ontology-Driven Selective Cache Invalidation for Knowledge-Graph-Augmented Retrieval Systems**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)

> **Paper submitted to:** Knowledge and Information Systems (KAIS), Springer
> **Authors:** Nimas Ayu Untariyati et al.
> **Extends:** [SKV-Cache (IC3INA 2025)](https://doi.org/10.1109/IC3INA68387.2025.11325643)

## Overview

OntoCacheRAG is a three-component framework for ontology-aware selective cache invalidation in KG-augmented RAG systems. When the underlying ontology evolves, existing KV cache systems either flush the entire cache or serve stale results. OntoCacheRAG resolves this trade-off through subsumption-aware reasoning.

| Component | Function | Overhead |
|-----------|----------|----------|
| **OCD** — Ontology Change Detector | Monitors SPARQL update events, classifies via OCIC taxonomy | < 0.1% |
| **SIM** — Semantic Impact Mapper | Computes transitive impact set via rdfs:subClassOf* | 99.0% |
| **SCI** — Selective Cache Invalidator | Invalidates affected entries via reverse index | < 0.1% |

### Key results

- **ABox revocation:** String-Match recall 0.55 vs OntoCacheRAG 1.00
- **TBox restructuring:** String-Match recall 0.00, ABox-1hop 0.50 vs OntoCacheRAG 1.00
- **Statistical significance:** Wilcoxon signed-rank, p < 0.001 (20 seeds)
- **Cache efficiency:** Only 6-9% of cache invalidated (vs 100% for Full-Flush)
- **Scalability:** Sub-linear growth O(|C|^0.31), under 26 ms at 50K classes

## Repository structure

```
ontocacherag/
├── data/
│   ├── ArsipDataset-final.csv         # Raw regulatory corpus (614 documents)
│   └── arsip_clean.csv                # Cleaned & structured version
├── ontology/
│   └── arsip_real.ttl                 # Resolved OWL ontology (1,901 triples)
├── experiments/
│   ├── 01_clean_arsip.py              # CSV cleaning & relation extraction
│   ├── 02_build_ontology.py           # OWL ontology construction
│   ├── 03_run_experiment_abox.py      # ABox revocation experiment (Table 4)
│   ├── 04_run_experiment_tbox.py      # TBox restructuring experiment (Table 5)
│   ├── 05_run_final.py                # Full 9-system comparison + Wilcoxon
│   ├── 06_measure_latency.py          # Latency breakdown (Table 6)
│   ├── 07_analysis_comprehensive.py   # All analyses (Tables 4-8)
│   └── 08_scalability_experiment.py   # Scalability (Table 9) - STANDALONE
├── figures/
│   ├── Fig1_architecture.png
│   └── Fig2_scalability_measured.png
└── results/
```

## Quick start

```bash
pip install rdflib scipy matplotlib

cd experiments
python 01_clean_arsip.py
python 02_build_ontology.py
python 03_run_experiment_abox.py
python 04_run_experiment_tbox.py
python 05_run_final.py
python 06_measure_latency.py
python 07_analysis_comprehensive.py
python 08_scalability_experiment.py   # standalone, no data needed
```

## Dataset: ArsipDataset

| Property | Value |
|----------|-------|
| Source | Indonesian regulatory records (peraturan.go.id) |
| Documents | 614 (557 active, 48 revoked, 9 unspecified) |
| Revocation relations | 42 verified |
| Ontology hierarchy | 5 levels, 131 classes, 2 properties |
| Triples | 1,901 |

## Citation

```bibtex
@article{untariyati2026ontocacherag,
  title   = {OntoCacheRAG: Ontology-Driven Selective Cache Invalidation
             for Knowledge-Graph-Augmented Retrieval Systems},
  author  = {Untariyati, Nimas Ayu},
  journal = {Knowledge and Information Systems},
  year    = {2026},
  note    = {Under review}
}
```

## License

MIT License - see [LICENSE](LICENSE) for details.
