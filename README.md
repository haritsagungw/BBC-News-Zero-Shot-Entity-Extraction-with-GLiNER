# BBC-News-Zero-Shot-Entity-Extraction-with-GLiNER
A high-performance Zero-Shot Named Entity Recognition (NER) pipeline tailored for Indonesian news processing. Built on **GLiNER (`urchade/gliner_multi-v2.1`)**, this project extracts domain-specific policy and regulatory entities from long-form text without requiring fine-tuning or training data.
## Features
* Zero-Shot Flexibility: Dynamically extracts complex entity schemas (e.g., policy names, regulator agencies, target demographics, and budget allocations) using natural language prompts.
* Sliding-Window Chunking: Handles long text articles by splitting input into 384-token windows with 64-token overlap, preserving contextual boundaries across chunk cuts.
* GPU-Accelerated Inference: Leverages `batch_predict_entities` for efficient GPU parallelization during large-scale extraction.
* Overlap Deduplication: Uses confidence-score sorting and span-overlap matching to resolve duplicate entities caused by overlapping chunk boundaries.
* Resilient Data Ingestion: Safely parses line-delimited JSON (`.jsonl`) streams with built-in error handling for malformed data.
## Pipeline Workflow
1. Ingestion & Validation: Reads JSONL datasets, validates string lengths, and sanitizes input data.
2. Text Segmentation: Tokens are chunked using DeBERTa/GLiNER tokenizers while preserving character offset maps.
3. Zero-Shot Extraction: GLiNER predicts targeted entity labels using configurable threshold scores.
4. Offset Alignment & Deduplication: Local chunk offsets are re-mapped to original global text positions, followed by Non-Maximum Suppression (NMS) deduplication.
5. Reporting & Export: Generates an analytical extraction report and exports structured entities to CSV.
## Quick Start
### 1. Installation
Clone the repository and install the dependencies:
```bash
git clone [https://github.com/your-username/bbc-gliner-entity-extraction.git](https://github.com/your-username/bbc-gliner-entity-extraction.git)
cd bbc-gliner-entity-extraction
pip install -r requirements.txt
```
## 2. Basic Usage
``` python
from gliner import GLiNER

# Load model
model = GLiNER.from_pretrained("urchade/gliner_multi-v2.1")

# Define text & target labels
text = "Kemenpora memberikan tanggapan terkait sanksi PSSI dari FIFA."
labels = ["instansi_regulator", "kelompok_terdampak", "nama_kebijakan"]

# Extract entities
entities = model.predict_entities(text, labels, threshold=0.60)

for entity in entities:
    print(f"Text: {entity['text']} | Label: {entity['label']} | Score: {entity['score']:.4f}")
```
## Target Entity Schema
The pipeline is pre-configured to extract the following domain labels:
| Label Name | Description | Example Match |
| :--- | :--- | :--- |
| `instansi_regulator` | Governing bodies & regulatory agencies | PSSI, FIFA, Kemenpora |
| `kelompok_terdampak` | Target groups or affected parties | Masyarakat, Karyawan, Warga |
| `wilayah_penerapan` | Geographic regions or jurisdictions | Filipina selatan, Eropa |
| `nama_kebijakan` | Official names of policies/regulations | Black Friday |
| `alokasi_anggaran` | Monetary and budget allocations | Financial figures |
## Extraction Sample Output
| Article ID | Detected Entity | Entity Label | Confidence Score |
| :--- | :--- | :--- | :--- |
| `1` | `pemerintah suriah` | `instansi_regulator` | `0.6848` |
| `17` | `ADB` | `instansi_regulator` | `0.8651` |
| `21` | `Filipina selatan` | `wilayah_penerapan` | `0.6205` |
| `24` | `Warga` | `kelompok_terdampak` | `0.6386` |
| `26` | `Kemenpora` | `instansi_regulator` | `0.9096` |
