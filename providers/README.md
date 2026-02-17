# Table Extraction Providers

This directory contains invocation scripts for each document AI provider evaluated in the [RD-TableBench](https://reducto.ai/blog/rd-tablebench) benchmark. Each script handles PDF ingestion, API calls, and response storage for a specific table extraction service.

All scripts process PDFs from a local dataset directory and write JSON responses to provider-specific output directories. These responses are then parsed by `parsing.py` and scored by `grading.py` in the root of the repository.

## Provider Overview

### Reducto

**File:** `reducto.py`
**API:** [Reducto Document Parsing API](https://reducto.ai)
**Concurrency:** Async with 10 concurrent uploads and 10 concurrent parse jobs
**Auth:** `REDUCTO_API_KEY` environment variable

Reducto's hybrid parsing API combines OCR and layout analysis. The script uses a three-step async workflow: upload the PDF, submit an async parse job with combined OCR mode, and poll for completion.

```bash
export REDUCTO_API_KEY="your-key"
python providers/reducto.py
```

**Dependencies:** `aiohttp`, `tqdm`

---

### Azure Document Intelligence

**File:** `azure_docintelligence.py`
**API:** [Azure AI Document Intelligence](https://azure.microsoft.com/en-us/products/ai-services/ai-document-intelligence)
**Concurrency:** ThreadPoolExecutor with 200 workers
**Auth:** `AZURE_ENDPOINT` and `AZURE_KEY` environment variables

Uses Microsoft's prebuilt layout model (`prebuilt-layout`) for document analysis. Includes exponential backoff for HTTP 429 rate limit responses.

```bash
export AZURE_ENDPOINT="your-endpoint"
export AZURE_KEY="your-key"
python providers/azure_docintelligence.py
```

**Dependencies:** `azure-ai-documentintelligence`, `azure-core`, `backoff`, `tqdm`

---

### AWS Textract

**File:** `textract.py`
**API:** [Amazon Textract](https://aws.amazon.com/textract/)
**Concurrency:** ThreadPoolExecutor with 20 workers
**Auth:** AWS credentials (via `boto3` default credential chain)

Converts each PDF's first page to an image, then uses the Textract `AnalyzeDocument` API with the `TABLES` feature. Caption tags are stripped from the HTML output. Includes exponential backoff for rate limiting.

```bash
# AWS credentials configured via aws configure, env vars, or IAM role
python providers/textract.py
```

**Dependencies:** `textractor`, `pdf2image`, `backoff`, `tqdm`

---

### GPT-4o

**File:** `gpt4o.py`
**API:** [OpenAI GPT-4o](https://openai.com/index/hello-gpt-4o/)
**Concurrency:** ThreadPoolExecutor with 5 workers
**Auth:** `OPENAI_API_KEY` environment variable (via `openai` SDK defaults)

A vision-based approach: each PDF's first page is converted to a base64-encoded PNG and sent to GPT-4o with a structured prompt requesting HTML table output with `rowspan`/`colspan` attributes. Includes exponential backoff for rate limiting.

```bash
export OPENAI_API_KEY="your-key"
python providers/gpt4o.py
```

**Dependencies:** `openai`, `pdf2image`, `backoff`, `tqdm`

---

### Google Cloud Document AI

**File:** `gcloud.py`
**API:** [Google Cloud Document AI](https://cloud.google.com/document-ai)
**Concurrency:** ThreadPoolExecutor with 5 workers
**Auth:** `GCP_PROJECT_ID` and `GCP_PROCESSOR_ID` environment variables, plus GCP application default credentials

Uses a configured Document AI processor to analyze documents, then converts extracted tables to HTML via the Document AI Toolbox library's DataFrame-to-HTML pipeline. Includes exponential backoff for quota errors.

```bash
export GCP_PROJECT_ID="your-project"
export GCP_PROCESSOR_ID="your-processor"
python providers/gcloud.py
```

**Dependencies:** `google-cloud-documentai`, `google-cloud-documentai-toolbox`, `backoff`, `tqdm`

---

### Unstructured

**File:** `unstructured.py`
**API:** [Unstructured.io](https://unstructured.io/)
**Concurrency:** ThreadPoolExecutor with 10 workers
**Auth:** `UNSTRUCTURED_API_KEY` environment variable

Uses the Unstructured SDK with `HI_RES` (high-resolution) processing strategy for maximum extraction quality. Includes exponential backoff with up to 3 retries.

```bash
export UNSTRUCTURED_API_KEY="your-key"
python providers/unstructured.py
```

**Dependencies:** `unstructured-client`, `backoff`, `tqdm`

---

### Chunkr

**File:** `chunkr.py`
**API:** [Chunkr](https://chunkr.ai/)
**Concurrency:** Async with 100 concurrent requests
**Auth:** `CHUNKR_API_KEY` environment variable

Async upload and poll workflow using the Chunkr task API. Configured with `HighQuality` model, 512 target chunk length, and automatic OCR strategy.

```bash
export CHUNKR_API_KEY="your-key"
python providers/chunkr.py
```

**Dependencies:** `aiohttp`, `tqdm`

## Adding a New Provider

To add support for a new table extraction service:

1. Create a new Python file in this directory (e.g., `your_provider.py`)
2. Implement a function that processes a PDF and saves the API response as JSON
3. Add a corresponding parser function in `parsing.py` that extracts the HTML table from your provider's response format
4. Run the provider against the benchmark dataset and grade the results using `grading.py`

See any existing provider script for the expected pattern.
