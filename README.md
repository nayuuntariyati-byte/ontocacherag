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
│   ├── arsipdataset_ontology.ttl
│   ├── arsipdataset_ontology.owl      # Resolved OWL ontology (1,901 triples)
│   └── ontology_stats.json                 
├── figures/
│   ├── Fig1_architecture.png
│   └── Fig2_scalability_measured.png
└── results/
```

## Dataset: ArsipDataset

| Property | Value |
|----------|-------|
| Source | Indonesian regulatory records (peraturan.go.id) |
| Documents | 614 (557 active, 48 revoked, 9 unspecified) |
| Revocation relations | 42 verified |
| Ontology hierarchy | 5 levels, 131 classes, 2 properties |
| Triples | 1,901 |


## License

MIT License - see [LICENSE](LICENSE) for details.
