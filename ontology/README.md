# Ontology Directory

This directory contains the **ArsipDataset OWL ontology** used in the OntoCacheRAG paper.

## Files

| File | Format | Size | Use Case |
|---|---|---|---|
| `arsipdataset_ontology.ttl` | Turtle | ~150 KB | Human inspection, text-based tools |
| `arsipdataset_ontology.owl` | OWL/XML | ~250 KB | Protégé, OWL reasoners (HermiT, Pellet) |
| `ontology_stats.json` | JSON | <1 KB | Quick statistical verification |

## Ontology Specification

### TBox (Class Hierarchy)

The ontology uses a **5-level subsumption hierarchy** mirroring the Indonesian ministerial structure:

```
Level 1: Regulasi (root)
   └── Level 2: PeraturanKementerian
          └── Level 3: Coordination clusters (5)
                 ├── Permen_Polhukam
                 ├── Permen_Perekonomian
                 ├── Permen_PMK
                 ├── Permen_Marves
                 └── Permen_Lainnya
                        └── Level 4: Per-agency classes (~124)
                               └── Level 5: Document instances (ABox)
```

**Total classes:** 131 (1 + 1 + 5 + 124)

### ABox (Instances)

- **614 document individuals** — one per regulatory document in ArsipDataset
- **42 verified revocation relations** — extracted from the original CSV `status` field
- Each instance is typed at the most specific (Level 4) agency class

### Object Properties

| Property | Domain | Range | Description |
|---|---|---|---|
| `dicabutOleh` | Regulasi | Regulasi | "revoked by" — directed from revoked document to revoking document |
| `mencabut` | Regulasi | Regulasi | Inverse of `dicabutOleh` |

### Total Triples

**1,901** (as reported in paper Section 5.2)

## Loading the Ontology

### Python (rdflib)

```python
import rdflib

g = rdflib.Graph()
g.parse('arsipdataset_ontology.ttl', format='turtle')
print(f"Loaded {len(g)} triples")
```

### Protégé

Open `arsipdataset_ontology.owl` directly in Protégé 5.x or later.

### SPARQL Query Examples

**Query 1: All documents in the Polhukam coordination cluster**

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ex: <http://ontocacherag.org/arsipdataset#>

SELECT ?doc WHERE {
    ?doc rdf:type/rdfs:subClassOf* ex:Permen_Polhukam .
}
```

**Query 2: All documents revoking other documents (the SIM impact pattern)**

```sparql
PREFIX ex: <http://ontocacherag.org/arsipdataset#>

SELECT ?revoker ?revoked WHERE {
    ?revoker ex:mencabut ?revoked .
}
```

**Query 3: Transitive subclass traversal (SIM core query)**

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ex: <http://ontocacherag.org/arsipdataset#>

SELECT ?instance WHERE {
    ?instance rdf:type/rdfs:subClassOf* ex:Permen_Polhukam_Pertahanan .
}
```

## Regenerating This Ontology

These files are committed for reviewer convenience but can be regenerated from source:

```bash
# Requires the full ArsipDataset CSV (see data/README.md)
python ../scripts/preprocess_arsipdataset.py \
    --input ../data/ArsipDataset.csv \
    --output ../data/arsipdataset_normalized.csv

python ../scripts/build_ontology.py \
    --input ../data/arsipdataset_normalized.csv \
    --output-dir .
```

Verify the output matches `ontology_stats.json`.
