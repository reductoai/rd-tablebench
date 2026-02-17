# Table Extraction Providers

This directory contains the invocation scripts used to run each provider through the [RD-TableBench](https://reducto.ai/blog/rd-tablebench) evaluation. Each script handles PDF ingestion, API calls, and response storage. Responses are parsed by `parsing.py` and scored by `grading.py` in the root of the repository.

## Reducto

**File:** `reducto.py`
**API:** [Reducto Document Parsing API](https://reducto.ai)
**Concurrency:** Async with 10 concurrent uploads and 10 concurrent parse jobs
**Auth:** `REDUCTO_API_KEY` environment variable

[Reducto](https://reducto.ai) is a hybrid document parsing API that combines OCR and layout analysis to extract structured data from complex documents. Its approach to table extraction handles challenging scenarios — scanned documents, handwritten content, merged cells, and multilingual text — by fusing multiple extraction signals rather than relying on a single method.

The invocation script uses a three-step async workflow: upload the PDF, submit an async parse job with combined OCR mode, and poll for completion.

```bash
export REDUCTO_API_KEY="your-key"
python providers/reducto.py
```

**Dependencies:** `aiohttp`, `tqdm`

To get a Reducto API key, sign up at [reducto.ai](https://reducto.ai).

## Baseline Providers

The following providers are included as baselines for comparison. Each script invokes the provider's API and stores the raw response for parsing and grading.

| Provider | File | Auth Environment Variables |
|---|---|---|
| Azure Document Intelligence | `azure_docintelligence.py` | `AZURE_ENDPOINT`, `AZURE_KEY` |
| AWS Textract | `textract.py` | AWS credentials (boto3 default chain) |
| GPT-4o | `gpt4o.py` | `OPENAI_API_KEY` |
| Google Cloud Document AI | `gcloud.py` | `GCP_PROJECT_ID`, `GCP_PROCESSOR_ID` |
| Unstructured | `unstructured.py` | `UNSTRUCTURED_API_KEY` |
| Chunkr | `chunkr.py` | `CHUNKR_API_KEY` |

Each baseline script follows a similar pattern: read PDFs from the dataset directory, call the provider API with rate-limit handling, and write JSON responses to a provider-specific output directory. See individual files for provider-specific configuration details.

## Adding a New Provider

To add support for a new table extraction service:

1. Create a new Python file in this directory (e.g., `your_provider.py`)
2. Implement a function that processes a PDF and saves the API response as JSON
3. Add a corresponding parser function in `parsing.py` that extracts the HTML table from your provider's response format
4. Run the provider against the benchmark dataset and grade the results using `grading.py`

See any existing provider script for the expected pattern.
