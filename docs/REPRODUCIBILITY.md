# Reproducibility Guide

This document explains how to reproduce the results reported in the OntoCacheRAG paper.

## Prerequisites

### Hardware

**No GPU required.** All ontology construction and synthetic generation runs on CPU. The scripts use `rdflib` for in-memory graph manipulation, which is CPU-bound.

### Software

```bash
Python >= 3.9
rdflib >= 7.0.0
pandas >= 2.0.0
```

Install via:

```bash
pip install -r requirements.txt
```

### Data

You need either:

- **The full ArsipDataset CSV** (614 documents) — request from `nayuuntariyati@students.undip.ac.id`, OR
- **Use the included sample** (`data/sample_metadata.csv`, 10 rows) — for format demonstration only; statistics will not match the paper

## Reproduction Workflow

### Stage 1 — Preprocess Raw CSV

The raw ArsipDataset has Indonesian column names and embedded revocation information in the `status` field. The preprocessing script normalizes this format:

```bash
python scripts/preprocess_arsipdataset.py \
    --input data/ArsipDataset.csv \
    --output data/arsipdataset_normalized.csv
```

**Expected output:**

```
[1/4] Loaded 614 records from data/ArsipDataset.csv
[2/4] Resolving revocation links against corpus ...
      In-corpus revocations resolved: 42
      Out-of-corpus revocations:     6
[3/4] Computing cluster distribution ...
[4/4] Writing normalized CSV to data/arsipdataset_normalized.csv ...

============================================================
PREPROCESSING COMPLETE
============================================================
  Input records:       614
  Output records:      614
  Active:              557
  Revoked:             48
  Unspecified:         9
  In-corpus revocations resolved:  42
============================================================
```

The key number to verify is **42 in-corpus revocations** (matches paper Section 5.2).

### Stage 2 — Build the OWL Ontology

```bash
python scripts/build_ontology.py \
    --input data/arsipdataset_normalized.csv \
    --output-dir ontology/ \
    --verbose
```

**Expected output:**

```
============================================================
ONTOLOGY CONSTRUCTION COMPLETE
============================================================
  Total classes:     131
  Properties:        2 (dicabutOleh, mencabut)
  Instances:         614
  Revocation links:  42 verified in-corpus
  Total triples:     1901
============================================================
```

These numbers correspond directly to **Table 3** of the paper (Section 5.2).

### Stage 3 — Generate Synthetic Ontologies

For the scalability experiments (Section 6.6 and Table 9):

```bash
python scripts/synthetic_generator.py \
    --paper-scales \
    --output-dir synthetic/ \
    --seed 42
```

This generates **8 synthetic ontologies** at the scales used in Table 9:

| Target |C| | Actual |C| | Instances | Triples |
|---|---|---|---|
| 193 | 193 | 552 | 783 |
| 493 | 493 | 1,440 | 2,033 |
| 991 | 991 | 2,916 | 4,111 |
| 1,976 | 1,976 | 5,850 | 8,235 |
| 4,961 | 4,961 | 14,760 | 20,754 |
| 9,976 | 9,976 | 29,754 | 41,812 |
| 19,927 | 19,927 | 59,535 | 83,629 |
| 49,924 | 49,924 | 149,382 | 209,762 |

**Total runtime:** ~2 minutes on Colab CPU. **Total output size:** ~50 MB.

## Verifying Reproducibility

### Quick Statistical Check

```python
import json

# After running Stage 2
with open('ontology/ontology_stats.json') as f:
    stats = json.load(f)

assert stats['tbox']['total_classes'] == 131, f"Expected 131 classes, got {stats['tbox']['total_classes']}"
assert stats['abox']['instances'] == 614, f"Expected 614 instances, got {stats['abox']['instances']}"
assert stats['abox']['in_corpus_revocations'] == 42, f"Expected 42 revocations, got {stats['abox']['in_corpus_revocations']}"
assert stats['total_triples'] == 1901, f"Expected 1901 triples, got {stats['total_triples']}"

print("All paper statistics reproduced ✓")
```

### Verifying SPARQL Queries

Once the ontology is built, you can verify it answers the core SIM queries correctly:

```python
import rdflib

g = rdflib.Graph()
g.parse('ontology/arsipdataset_ontology.ttl', format='turtle')

# Test query: all instances of Polhukam coordination cluster
query = """
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ex: <http://ontocacherag.org/arsipdataset#>

SELECT (COUNT(?d) AS ?count) WHERE {
    ?d rdf:type/rdfs:subClassOf* ex:Permen_Polhukam .
}
"""

result = list(g.query(query))
print(f"Polhukam cluster instances: {result[0][0]}")
```

## Reproducing Paper Figures and Tables

| Paper Element | How to Reproduce |
|---|---|
| Table 3 (Ontology stats) | Run Stage 2; see `ontology/ontology_stats.json` |
| Table 9 (Scalability) | Run Stage 3; measure SIM latency on each synthetic ontology |
| Figure 2 (Scalability plot) | Plot Table 9 values on log-log scale |

The detection accuracy and cache efficiency experiments (Tables 4–8, Figures 3–4) require the full evaluation pipeline including cache workload simulation, which is documented separately.

## Determinism

All scripts use **deterministic seeds**:

- `preprocess_arsipdataset.py` — no randomness (regex-based)
- `build_ontology.py` — no randomness (deterministic graph construction)
- `synthetic_generator.py` — uses `--seed 42` by default

Running the same command twice on the same input produces **byte-identical output** for `.ttl` files (modulo timestamp metadata in OWL headers).

## Troubleshooting

| Symptom | Likely Cause | Solution |
|---|---|---|
| `ModuleNotFoundError: rdflib` | Dependencies not installed | `pip install -r requirements.txt` |
| `KeyError: 'coordination_cluster'` | Using raw CSV before preprocessing | Run Stage 1 first |
| `numpy.dtype size changed` (Colab) | Stale numpy/pandas in runtime | Restart runtime via `Runtime → Restart session` |
| Statistics don't match paper | Wrong CSV (sample vs full) or modified `CLUSTER_PATTERNS` | Use full 614-record CSV with default mappings |
| Cluster counts slightly off | Subjective mapping of agencies to clusters | Tune `CLUSTER_PATTERNS` in `preprocess_arsipdataset.py` |

## Contact

For reproducibility questions:
- **Nimas Ayu Untariyati** — `nayuuntariyati@students.undip.ac.id`
