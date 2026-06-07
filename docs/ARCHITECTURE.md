# Architecture

This document provides a detailed description of the OntoCacheRAG pipeline. For the high-level overview, see the [main README](../README.md).

## Three-Component Pipeline

```
┌──────────────┐    ChangeEvent     ┌──────────────┐    Impact set     ┌──────────────┐
│              │  ────────────────► │              │ ────────────────► │              │
│     OCD      │                    │     SIM      │                   │     SCI      │
│              │  ────────────────► │              │ ────────────────► │              │
└──────────────┘                    └──────────────┘                   └──────────────┘
  Ontology                            Semantic                            Selective
  Change                              Impact                              Cache
  Detector                            Mapper                              Invalidator
   (<0.1%)                            (~99%)                              (<0.1%)
                          ── latency percentage of total pipeline ──
```

## OCD — Ontology Change Detector

**Input:** Raw SPARQL Update statement, OWL diff, or change notification
**Output:** Structured ChangeEvent with semantic classification

### OCIC Taxonomy

The Ontology Change Impact Classifier (OCIC) classifies changes along two dimensions:

| Dimension | Values |
|---|---|
| **Level** | TBox (terminological) / ABox (assertional) |
| **Severity** | Low / Medium / High |

### 8 Change Types

| Type | Level | Severity | Example |
|---|---|---|---|
| Class addition | TBox | Low | Add new agency class |
| Class deletion | TBox | High | Remove agency class |
| Property addition | TBox | Low | Add new relation |
| Subclass restructuring | TBox | High | Move class under different parent |
| Instance creation | ABox | Low | New regulatory document published |
| Instance reclassification | ABox | Medium | Document moves to different agency |
| Property assertion | ABox | Medium | Add `dicabutOleh` link |
| Instance deletion | ABox | High | Remove document (revocation) |

### OCD Output Format

```python
ChangeEvent {
    "type": "ABox-revocation",
    "level": "ABox",
    "severity": "High",
    "axioms": [
        ("D0136", "dicabutOleh", "D0085")
    ],
    "timestamp": "2025-04-12T08:34:00Z"
}
```

## SIM — Semantic Impact Mapper

**Input:** ChangeEvent
**Output:** Impact set — instances whose cache entries are semantically affected

### Core Algorithm

The SIM computes the transitive impact set via SPARQL property-path traversal:

```sparql
SELECT ?instance WHERE {
    ?instance rdf:type/rdfs:subClassOf* ex:AffectedClass .
}
```

This single query computes the **reflexive transitive closure** of the subclass hierarchy, returning all instances of the affected class or any of its subclasses.

For ABox-revocation changes, the SIM additionally follows the resolved property link:

```sparql
SELECT ?cached WHERE {
    ?cached ex:dicabutOleh ex:D0085 .
}
```

### Completeness Guarantee

**Proposition 1** (Paper Section 4.2.3): For any ontology change of type ABox-revocation or TBox-restructuring within a single-property subsumption hierarchy expressed in the OWL 2 RL/RDFS fragment, the SIM returns a set `S` such that:

```
C_inv ⊆ {k ∈ C | Anchor(k) ∩ S ≠ ∅}
```

That is, every truly stale cache entry is covered by the structural impact set. The proof relies on RDFS entailment semantics — see paper Section 4.2.3.

### Scalability

| |C| (classes) | SIM latency (ms) | Impact set size |
|---|---|---|---|
| 193 | 4.69 | 69 |
| 991 | 6.87 | 162 |
| 9,976 | 13.19 | 522 |
| 49,924 | 25.77 | 1,158 |

Empirical scaling exponent: **O(|C|^0.31)** — sub-linear.

## SCI — Selective Cache Invalidator

**Input:** Impact set from SIM
**Output:** Updated cache state

### Three Invalidation Modes

| Mode | Trigger | Use Case |
|---|---|---|
| **Lazy** | On next access | Bulk batch processing, low-priority changes |
| **Eager** | Immediately on event | High-severity changes (revocations, class deletions) |
| **Adaptive** | Routed by OCIC severity | Production deployment with mixed change types |

### Adaptive Mode Logic

```python
def route_invalidation(change_event):
    if change_event.severity == "High":
        return "Eager"
    elif change_event.severity == "Medium":
        return "Eager" if cache_size > threshold else "Lazy"
    else:  # Low
        return "Lazy"
```

## Reverse Index

The SCI maintains a **reverse index** mapping each anchor entity to its dependent cache entries:

```
Anchor(e) → {k_1, k_2, ..., k_n}
```

When the SIM returns impact set `S = {e_1, e_2, ..., e_m}`, the SCI computes the invalidation set as the union of dependent caches:

```
C_inv = ⋃_{e ∈ S} Anchor⁻¹(e)
```

This O(1) lookup per affected entity is why SCI contributes negligible latency (<0.1%) despite being the action-taking component.

## Pipeline Latency Profile

From paper Table 6:

| Component | Latency | % of total |
|---|---|---|
| OCD | <0.05 ms | <0.1% |
| **SIM** | **2.6 ms** | **99.0%** |
| SCI | <0.05 ms | <0.1% |
| **Total** | **~2.65 ms** | **100%** |

This bottleneck concentration is intentional: by making SIM the single dominant cost, optimization efforts (e.g., incremental ELK reasoning) can be focused on one component without redesigning the pipeline.

## See Also

- [`REPRODUCIBILITY.md`](REPRODUCIBILITY.md) — How to reproduce paper experiments
- [`../ontology/README.md`](../ontology/README.md) — Ontology specification
- Paper Section 4 — Formal architecture description
- Paper Section 6 — Empirical evaluation
