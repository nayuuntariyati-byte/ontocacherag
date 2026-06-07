# Ontology Files

This directory will contain the generated OWL ontology files 

## Expected Files 

| File | Description |
|---|---|
| `arsipdataset_ontology.ttl` |  Turtle format ontology |
| `arsipdataset_ontology.owl` |  OWL/XML format ontology |
| `ontology_stats.json` |  Statistics matching paper Section 5.2 |

## Expected Statistics

The generated `ontology_stats.json` should report:

```json
{
  "tbox": {
    "total_classes": 131,
    "by_level": {"L1": 1, "L2": 1, "L3": 5, "L4": 124}
  },
  "properties": 2,
  "abox": {
    "instances": 614,
    "in_corpus_revocations": 42,
    "out_corpus_revocations": 6
  },
  "total_triples": 1901
}
```

These numbers correspond directly to Table 3 in paper Section 5.2.

See [`README.md`](README.md) for full ontology specification and SPARQL query examples.
