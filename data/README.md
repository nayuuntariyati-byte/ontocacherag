# Data Directory

This directory contains the data files for OntoCacheRAG.

## Files

| File | Description | Status |
|---|---|---|
| `sample_metadata.csv` | 10-row format demonstration | Public |
| `ArsipDataset.csv` | Full 614-document corpus | **On request** |

## Full ArsipDataset

The full **ArsipDataset** (614 Indonesian regulatory documents) is **available on request** due to data provenance considerations. To obtain access:

**Contact:** Nimas Ayu Untariyati
**Email:** `nayuuntariyati@students.undip.ac.id`
**Affiliation:** Universitas Diponegoro / BRIN

The dataset is derived from [peraturan.go.id](https://peraturan.go.id), the official Indonesian government regulations portal. The compiled metadata (document IDs, agency taxonomy, coordination cluster mapping, and verified revocation links) represents the authors' intellectual contribution.

## CSV Format

### Raw ArsipDataset Format (Indonesian columns)

The raw CSV contains 19 columns. Key columns:

| Column | Description |
|---|---|
| `title` | Document title in Indonesian |
| `jenis_bentuk_peraturan` | Regulation type |
| `pemrakarsa` | Issuing agency (uppercase) |
| `nomor` | Regulation number |
| `tahun` | Year |
| `status` | Status text (encodes revocation when present) |

Process this format with `scripts/preprocess_arsipdataset.py` first.

### Normalized Format (Required by `build_ontology.py`)

After preprocessing, the CSV has the following columns:

| Column | Type | Required | Example |
|---|---|---|---|
| `doc_id` | string | Yes | `D0136` |
| `regulation_number` | string | Yes | `24` |
| `year` | integer | Yes | `2019` |
| `agency` | string | Yes | `Pertahanan` |
| `coordination_cluster` | string | Yes | `Polhukam` |
| `status` | string | Yes | `active` / `revoked` / `unspecified` |
| `revoked_by` | string | Optional | `D0085` (or empty) |
| `title` | string | Optional | Full title text |

See `sample_metadata.csv` for a working example.

## Reproducing the Paper Ontology

Once you have the full ArsipDataset CSV:

```bash
# Step 1: Preprocess raw CSV to normalized format
python ../scripts/preprocess_arsipdataset.py \
    --input ArsipDataset.csv \
    --output arsipdataset_normalized.csv

# Step 2: Build the OWL ontology
python ../scripts/build_ontology.py \
    --input arsipdataset_normalized.csv \
    --output-dir ../ontology/ \
    --verbose
```

### Expected Output Statistics (Section 5.2 of paper)

| Metric | Expected Value |
|---|---|
| Total classes | 131 |
| Properties | 2 |
| Instances | 614 |
| In-corpus revocation links | 42 |
| Out-of-corpus revocations | 6 |
| Total triples | 1,901 |

If your output does not match these numbers, please check:
1. CSV is the full 614-document version
2. Column names match the normalized schema
3. No documents were duplicated or dropped during preprocessing
