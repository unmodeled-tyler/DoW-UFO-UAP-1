---
license: cc0-1.0
pretty_name: Department of War UFO/UAP Release 01 OCR + Metadata
task_categories:
- text-classification
- token-classification
- question-answering
- summarization
- feature-extraction
language:
- en
tags:
- public-records
- ocr
- document-ai
- uap
- ufo
- fbi
- archival-documents
- pursue-release-01
configs:
- config_name: pages
  data_files:
  - split: train
    path: data/pages/*.parquet
- config_name: packets
  data_files:
  - split: train
    path: data/packets/*.parquet
- config_name: sources
  data_files:
  - split: train
    path: metadata/sources.parquet
- config_name: classification_markings
  data_files:
  - split: train
    path: data/classification_markings/*.parquet
- config_name: triage
  data_files:
  - split: train
    path: data/triage/*.parquet
- config_name: media_assets
  data_files:
  - split: train
    path: metadata/media_assets.parquet
---

![Department of War UFO/UAP Release 01 OCR archive thumbnail](assets/thumbnail.png)

# Department of War UFO/UAP Release 01 OCR + Metadata

This repository is intended as the canonical machine-readable Hugging Face dataset for public Department of War / PURSUE UFO-UAP Release 01 records.

It uses **one dataset repo with internal sharding**, not one repo per source file. Users can load only the table they need via named configs: `pages`, `packets`, `sources`, `classification_markings`, `triage`, or `media_assets`.

Current build status:

- Source records indexed in this export: 41
- OCR page rows: 3580
- Packet rows: 796
- Detected-marking rows: 470
- Human-triage rows: 3
- Physical shard target: 4

> Early builds may contain only the PDFs processed so far. The goal is to expand this repo to cover all 161 Release 01 records: PDFs with OCR/packet rows, and non-PDF assets represented in source metadata until image/video-specific processing is added.

## Loading

```python
from datasets import load_dataset

pages = load_dataset("unmodeled-tyler/DoW-UFO-UAP-1", "pages")
packets = load_dataset("unmodeled-tyler/DoW-UFO-UAP-1", "packets")
sources = load_dataset("unmodeled-tyler/DoW-UFO-UAP-1", "sources")
markings = load_dataset("unmodeled-tyler/DoW-UFO-UAP-1", "classification_markings")
triage = load_dataset("unmodeled-tyler/DoW-UFO-UAP-1", "triage")
media_assets = load_dataset("unmodeled-tyler/DoW-UFO-UAP-1", "media_assets")
```

## Tables

### `sources`

One row per source file/record represented in the dataset. Includes official source URL, agency, release metadata, source file name, hashes when available, page count, batch id, processing status, and parser version.

### `pages`

One row per OCRed PDF page. Includes raw OCR, conservative normalized OCR, summary fields, quality flags, document labels, detected classification/security markings, source provenance, and relative image/text paths where applicable.

Raw OCR is preserved in `ocr_text`. Conservative cleanup is provided in `ocr_text_normalized` and `summary_normalized`; it only normalizes whitespace/control characters and does not rewrite content.

### `packets`

One row per inferred or manually overridden document packet. Packets group adjacent pages into higher-level archival/document units.

### `classification_markings`

Subset of page rows where machine OCR detected possible classification/security markings. These are **not authoritative** unless `classification_marking_validated` says otherwise.

### `triage`

Human-reviewed packet labels such as `KEEP_INVESTIGATE` or `DEPRIORITIZED`, when available.

### `media_assets`

One row per non-PDF media asset mirrored into the dataset. Video rows preserve the raw released file under `media/videos/`, with derived provenance metadata such as SHA256, file size, duration, dimensions, codec information, and optional derived keyframe paths. This table is for provenance and discovery; media interpretation should be framed as "the archive contains/shows..." rather than treating footage as standalone proof of a claim.

- `pages` and `packets` preserve array-like values as JSON-encoded strings in the corpus export for stable multi-shard Hugging Face loading. Parse with `json.loads` when you want Python lists.

## OCR and visual-rescue methodology

The current pipeline is intentionally staged rather than fully LLM-generated:

- Page images are rendered locally from the released PDFs.
- Tesseract provides the first-pass OCR text stored in `ocr_text`.
- Conservative text heuristics classify page/document type and detect common security/classification markings.
- A local Ollama vision model, `gemma4:e4b`, is used as a targeted visual-rescue pass for high-risk pages: low-OCR pages, first/cover pages, and pages where visible stamps or degraded scans may hide markings from Tesseract.
- Visual rescue is used primarily for marking detection and triage context, not for rewriting the OCR transcript.

Raw OCR is preserved. Model-assisted fields are dataset aids, not authoritative readings of the original records.

## Important caveats

- OCR text, summaries, categories, packet boundaries, and detected markings are machine-generated unless explicitly human-validated.
- Original source PDFs remain authoritative.
- Detected markings are not confirmed classification status.
- Recorded claims are not verified facts and should not be treated as government endorsement.
- Some source records may contain third-party material such as newspaper clippings; downstream users are responsible for rights assessment.

## Intended uses

- Public-record search/retrieval
- OCR correction and evaluation
- Document classification
- Entity extraction
- Page/packet triage
- Long-context summarization experiments
- Civic/archive research tooling

## Not intended uses

- Treating machine-generated labels as authoritative
- Treating recorded claims as verified facts
- Inferring government endorsement of claims
- Using unvalidated OCR-detected markings as confirmed classification status

## License note

Derived annotations and metadata generated by this pipeline are released under CC0-1.0 unless otherwise noted. Source records were publicly released by the U.S. government, but some pages may reproduce third-party materials.
