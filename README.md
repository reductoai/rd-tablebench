# RD-TableBench: An Open Benchmark for PDF Table Extraction

RD-TableBench is an open-source evaluation framework for benchmarking table extraction from PDFs and document images. Built by [Reducto](https://reducto.ai) for the document AI community, it provides a standardized way to measure and compare how accurately different document parsing services extract complex tables.

If your team processes documents with tables — financial reports, invoices, scientific papers, government filings — and you need to evaluate which extraction provider handles your use cases best, this benchmark gives you the methodology and tooling to do exactly that.

## Why RD-TableBench?

Enterprises working with tables at scale often lack reliable ways to evaluate extraction quality. Existing metrics tend to penalize minor formatting differences as hard failures, making it difficult to distinguish genuinely poor extraction from cosmetic variation. Off-the-shelf benchmarks are scarce, and building internal evaluation pipelines is costly.

RD-TableBench addresses this with:

- **1,000 human-annotated tables** from diverse, publicly available documents — scanned pages, handwritten content, merged cells, multilingual text, and other real-world complexity
- **PhD-level annotation quality** ensuring ground truth accuracy across challenging table structures
- **A flexible similarity metric** based on sequence alignment that handles structural and content variation gracefully
- **Support for 7 major extraction providers** out of the box, with straightforward patterns for adding your own

## Supported Providers

The benchmark includes invocation and parsing code for the following document AI and table extraction services:

| Provider | Description |
|---|---|
| [Reducto](https://reducto.ai) | Hybrid document parsing API with OCR and layout analysis |
| [Azure Document Intelligence](https://azure.microsoft.com/en-us/products/ai-services/ai-document-intelligence) | Microsoft's prebuilt layout model for document analysis |
| [AWS Textract](https://aws.amazon.com/textract/) | Amazon's ML-based document text and table extraction |
| [GPT-4o](https://openai.com/index/hello-gpt-4o/) | OpenAI's multimodal model with vision-based table extraction |
| [Google Cloud Document AI](https://cloud.google.com/document-ai) | Google's document processing and understanding platform |
| [Unstructured](https://unstructured.io/) | Open-source document parsing with high-resolution strategy |
| [Chunkr](https://chunkr.ai/) | Document chunking and extraction API |

See the [providers directory](./providers/) for implementation details and configuration for each.

## Evaluation Methodology

### The Problem with Exact Matching

A naive approach to table comparison — checking cell-by-cell equality — penalizes minor deviations like whitespace differences, OCR artifacts, or slightly different text normalization. This makes scores unreliable and hard to interpret.

### Hierarchical Sequence Alignment

RD-TableBench treats table comparison as a **hierarchical alignment problem**, adapting the [Needleman-Wunsch algorithm](https://en.wikipedia.org/wiki/Needleman%E2%80%93Wunsch_algorithm) (originally designed for DNA sequence alignment) to work at two levels:

**Cell-Level Alignment**
Individual cells within each row are aligned using a modified Needleman-Wunsch algorithm. Instead of binary match/mismatch, cell similarity is computed using [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance), allowing partial credit for cells where extraction captured most of the content but introduced minor errors.

- Cell match score: `+1`
- Cell mismatch penalty: `-1`
- Column gap penalty: `-1`

**Row-Level Alignment**
After cell-level scores are computed, entire rows are aligned using a second Needleman-Wunsch pass. Each row's alignment score is derived from its cell-level alignment, capturing both structural fidelity (correct number and order of rows) and content accuracy.

- Row match score: `+5`
- Row gap penalty: `-3`
- Free-end gap penalties for subtable alignment

### Normalization

Before comparison, all cell content is normalized by stripping whitespace, newlines, and hyphens. The final similarity score is a value between `0` and `1`, where `1` represents a perfect structural and content match.

### Table Representation

Tables are represented as 2D string arrays. Merged cells (via HTML `rowspan`/`colspan`) are expanded by repeating their values across every cell they occupy, ensuring consistent dimensionality for alignment.

## Repository Structure

```
rd-tablebench/
├── README.md           # This file
├── convert.py          # HTML table to NumPy array conversion
├── grading.py          # Similarity scoring using Needleman-Wunsch alignment
├── parsing.py          # Provider response parsing (extract HTML tables)
└── providers/          # Provider invocation scripts
    ├── azure_docintelligence.py
    ├── chunkr.py
    ├── gcloud.py
    ├── gpt4o.py
    ├── reducto.py
    ├── textract.py
    └── unstructured.py
```

## How It Works

```
PDF Documents
     │
     ▼
┌─────────────────────┐
│  Provider Invocation │  ← providers/*.py
│  (7 services)       │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Response Parsing    │  ← parsing.py
│  (extract HTML)     │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  HTML → NumPy Array  │  ← convert.py
│  (normalize tables) │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Similarity Scoring  │  ← grading.py
│  (Needleman-Wunsch) │
└─────────┬───────────┘
          │
          ▼
   Scores (0–1 scale)
```

## Getting Started

### Dependencies

The core evaluation code requires:

```
numpy
lxml
python-Levenshtein
```

Each provider script has its own dependencies (e.g., `boto3` for Textract, `azure-ai-documentintelligence` for Azure, `openai` for GPT-4o). See the [providers README](./providers/README.md) for per-provider requirements.

### Using the Grading Pipeline

```python
from convert import html_to_numpy
from grading import table_similarity

# Convert HTML tables to arrays
ground_truth = html_to_numpy(ground_truth_html)
predicted = html_to_numpy(predicted_html)

# Score the extraction (returns 0–1)
score = table_similarity(ground_truth, predicted)
```

### Parsing Provider Responses

```python
from parsing import parse_reducto_response, parse_textract_response

# Each parser extracts the HTML table from a provider's response format
html_table = parse_reducto_response(response_json)
html_table = parse_textract_response(response_json)
```

## Results and Dataset

Full benchmark results, provider comparisons, and dataset details are available in the accompanying blog post:

**[RD-TableBench: Accurately Evaluating Table Extraction](https://reducto.ai/blog/rd-tablebench)**

The evaluation dataset is available on HuggingFace: [reducto/rd-tablebench](https://huggingface.co/datasets/reducto/rd-tablebench).

## Integrity

To maintain scoring integrity and prevent benchmark contamination through training data leakage, only a subset of the evaluation framework is publicly released. The full evaluation dataset is held separately.

## Contributing

Contributions are welcome — whether adding support for new providers, improving the scoring methodology, or extending the evaluation dataset. Please open an issue or pull request.

## About Reducto

[Reducto](https://reducto.ai) builds document parsing infrastructure for enterprises. RD-TableBench is part of our commitment to transparent, reproducible evaluation in the document AI space.

## Citation

If you use RD-TableBench in your research or evaluation, please reference:

```
RD-TableBench: An Open Benchmark for PDF Table Extraction
https://reducto.ai/blog/rd-tablebench
Reducto, 2024
```
