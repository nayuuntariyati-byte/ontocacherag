# ArsipDataset

This document describes the **ArsipDataset** — the Indonesian regulatory document corpus used in the OntoCacheRAG paper.

## Overview

The ArsipDataset is a curated collection of **614 Indonesian government regulations** ("Peraturan") spanning the years **2016–2025**. Each document is annotated with provenance metadata (issuing agency, year, regulation number) and revocation relations explicitly extracted from the document `status` field.

## Source

All documents are derived from [**peraturan.go.id**](https://peraturan.go.id), the official portal of the Indonesian Ministry of Law and Human Rights for centralized regulation publication.

## Composition

### Document Count by Status

| Status | Count |
|---|---|
| Active (Berlaku) | 557 |
| Revoked (Tidak Berlaku) | 48 |
| Unspecified | 9 |
| **Total** | **614** |

### Revocation Relations

| Type | Count |
|---|---|
| In-corpus revocations (resolved to corpus document) | **42** |
| Out-of-corpus revocations (revoker outside corpus) | 6 |

The **42 in-corpus revocations** form the verified gold-standard set used for detection metrics in paper Sections 6.1 and 6.2.

### Agency Coverage

The dataset spans **132 unique issuing agencies (`pemrakarsa`)**, including:

- 33 ministries (Kementerian)
- 12 ministerial coordination ministries (Kementerian Koordinator)
- 25+ non-ministerial agencies (ARSIP NASIONAL, BPS, BNPB, etc.)
- 60+ regional governments (Pemerintah Provinsi, Kabupaten, Kota)

### Coordination Cluster Distribution

After mapping each agency to one of five coordination clusters:

| Cluster | Documents | Percentage |
|---|---|---|
| Lainnya (Other) | 417 | 67.9% |
| Polhukam | 68 | 11.1% |
| Marves | 55 | 9.0% |
| Perekonomian | 38 | 6.2% |
| PMK | 36 | 5.9% |

The dominance of "Lainnya" reflects the heavy presence of non-ministerial bodies (especially ARSIP NASIONAL with 203 documents) and regional governments in the source corpus.

## Schema

### Raw Format (19 columns)

The raw CSV from peraturan.go.id has Indonesian column names:

| # | Column | Description |
|---|---|---|
| 1 | `title` | Document title |
| 2 | `jenis_bentuk_peraturan` | Regulation type (e.g., PERATURAN MENTERI) |
| 3 | `pemrakarsa` | Issuing agency (uppercase) |
| 4 | `nomor` | Regulation number |
| 5 | `tahun` | Establishment year |
| 6 | `tentang` | Subject ("regarding") |
| 7 | `tempat_penetapan` | Place of establishment |
| 8 | `ditetapkan_tanggal` | Establishment date |
| 9 | `pejabat_yang_menetapkan` | Establishing officer |
| 10 | `status` | Status text (encodes revocation when present) |
| 11 | `dokumen_peraturan` | Document URL |
| 12–13 | `jumlah_dilihat`, `jumlah_didownload` | View/download counts |
| 14–19 | Various promulgation metadata | |

### Normalized Format (8 columns)

After running `preprocess_arsipdataset.py`:

| Column | Type | Description |
|---|---|---|
| `doc_id` | string | Auto-generated unique identifier (D0001–D0614) |
| `regulation_number` | string | From `nomor` |
| `year` | integer | From `tahun` |
| `agency` | string | Extracted from `pemrakarsa` (slugified) |
| `coordination_cluster` | string | One of: Polhukam, Perekonomian, PMK, Marves, Lainnya |
| `status` | string | Normalized: active / revoked / unspecified |
| `revoked_by` | string | Resolved `doc_id` of revoking document (or empty) |
| `title` | string | From `title` |

## Revocation Extraction

In the raw CSV, revocation information is **embedded in the `status` field** as free text:

> `Tidak Berlaku Dicabut Oleh :Peraturan Menteri Pertahanan Nomor 15 Tahun 2021 Tentang ...`

The preprocessing script:

1. Detects revocation via regex pattern `Tidak Berlaku Dicabut Oleh :(.+)`
2. Extracts `(nomor, tahun, agency_keywords)` from the revoking text
3. Matches against in-corpus documents by `(nomor, tahun, agency_overlap)`
4. Records the matched `doc_id` in the `revoked_by` column

When no in-corpus match exists, the revocation is counted as **out-of-corpus** (the revoking document was issued before the corpus collection window).

## Data Licensing and Availability

### Public Domain Status

Indonesian government regulations are considered **public information** under Indonesian Law No. 14/2008 (Public Information Disclosure Act). The original document texts are freely accessible at peraturan.go.id.

### Compiled Dataset Status

The **compiled CSV with structured metadata** (agency taxonomy, coordination cluster mapping, verified revocation links) represents the authors' intellectual contribution. This compilation is made available **on request** to facilitate reviewer verification and research reproducibility.

### How to Request Access

Contact: **Nimas Ayu Untariyati**
Email: `nayuuntariyati@students.undip.ac.id`
Affiliation: Universitas Diponegoro / BRIN

Please include in your request:
- Your name and institution
- Intended use (research, replication, teaching, etc.)
- A brief description of the work

## Acknowledgment

If you use the ArsipDataset in your work, please:

1. Cite the OntoCacheRAG paper (see [main README](../README.md))
2. Acknowledge **peraturan.go.id** as the original data source
3. Note that the compiled dataset represents the authors' contribution

## Limitations

- **Domain specificity**: All documents are Indonesian regulatory archives. Generalization to other legal systems (EU, US, etc.) is untested.
- **Language**: Document titles and subjects are in Indonesian. The metadata schema is language-agnostic.
- **Time window**: Documents span 2016–2025; older revocation chains may be incomplete.
- **Cluster imbalance**: 67.9% of documents fall into the "Lainnya" cluster. Refined ministerial classification would produce more balanced clusters (see paper Section 7).

## See Also

- [`../data/README.md`](../data/README.md) — Data file specifications
- [`../ontology/README.md`](../ontology/README.md) — Ontology construction
- Paper Section 5.1 — ArsipDataset description
- Paper Section 5.2 — Ontology construction
