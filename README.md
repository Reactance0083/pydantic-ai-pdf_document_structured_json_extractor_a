> **Commercial status:** Preview/unverified starter. This repository is not the current flagship and is not currently a commercially verified paid product.


## Current Status

This repository is a preview/unverified starter. It is not the active flagship and is not currently commercially verified. Do not treat it as a ready-to-buy production package until this notice is removed after a fresh commercial-readiness check.

# PDF/Document → Structured JSON Extractor API

A preview FastAPI service that converts PDFs, images, and documents into typed, validated JSON using [pydantic-ai](https://ai.pydantic.dev/). Sketches three example schemas (invoice, resume, contract), per-field confidence scoring, automatic retries on validation failure, and switchable OpenAI/Anthropic backends.

## Overview

Stop writing brittle regex and one-off LLM prompts every time you need to extract data from a PDF. This template gives you a single HTTP endpoint that accepts a document, runs it through a Pydantic schema you define, and returns clean structured JSON with confidence scores per field. Built on pydantic-ai's structured-output retry loop, so malformed model responses get re-prompted automatically.

## What it does

- Accepts PDF, PNG, JPG, TIFF, and DOCX uploads via multipart form
- Extracts text + layout using `pypdf` for PDFs and `pytesseract` for images
- Runs a pydantic-ai `Agent` with your chosen Pydantic schema as the output type
- Returns validated JSON conforming to that schema, plus per-field confidence (0.0–1.0)
- Ships three production schemas: `invoice`, `resume`, `contract`
- Lets you POST your own ad-hoc JSON Schema for one-off extraction jobs
- Switches between OpenAI (`gpt-4o`, `gpt-4o-mini`) and Anthropic (`claude-sonnet-4`, `claude-haiku`) via env var
- Automatic retry (default 3 attempts) on schema validation failure

## Prerequisites

- Python 3.11+
- `tesseract` binary installed (`brew install tesseract` on macOS, `apt-get install tesseract-ocr` on Debian/Ubuntu)
- An OpenAI **or** Anthropic API key
- `uv` or `pip` for dependency management

## Setup

1. **Clone and enter the repo**
   ```bash
   git clone https://github.com/yourname/pdf-extractor-api.git
   cd pdf-extractor-api
   ```

2. **Create a virtualenv and install dependencies**
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

3. **Verify tesseract is on PATH**
   ```bash
   tesseract --version
   ```

4. **Copy the env template and add your API key**
   ```bash
   cp .env.example .env
   # then edit .env and set OPENAI_API_KEY or ANTHROPIC_API_KEY
   ```

5. **Run the server**
   ```bash
   uvicorn app.main:app --reload --port 8000
   ```

6. **Open the interactive docs**
   ```
   http://localhost:8000/docs
   ```

Total time: ~5 minutes assuming tesseract is already installed.

## Usage

Extract an invoice with the built-in schema:

```bash
curl -X POST http://localhost:8000/extract/invoice \
  -F "file=@samples/acme_invoice.pdf"
```

Sample response:

```json
{
  "data": {
    "invoice_number": "INV-2024-0142",
    "issue_date": "2024-03-14",
    "due_date": "2024-04-13",
    "vendor": {"name": "Acme Corp", "tax_id": "12-3456789"},
    "line_items": [
      {"description": "Consulting (Q1)", "quantity": 40, "unit_price": 150.0, "total": 6000.0}
    ],
    "subtotal": 6000.0,
    "tax": 540.0,
    "total": 6540.0,
    "currency": "USD"
  },
  "confidence": {
    "invoice_number": 0.99,
    "issue_date": 0.97,
    "due_date": 0.95,
    "vendor.name": 0.98,
    "total": 0.99
  },
  "model": "gpt-4o-mini",
  "tokens_used": 1843,
  "elapsed_ms": 2104
}
```

Same call via Python:

```python
import httpx

with open("samples/acme_invoice.pdf", "rb") as f:
    r = httpx.post(
        "http://localhost:8000/extract/invoice",
        files={"file": f},
        timeout=60,
    )
print(r.json())
```

## API Endpoints

### `GET /health`
Liveness check.
```bash
curl http://localhost:8000/health
# {"status":"ok","model":"gpt-4o-mini"}
```

### `POST /extract/invoice`
Extract invoice fields (number, dates, vendor, line items, totals, tax, currency).
```bash
curl -X POST http://localhost:8000/extract/invoice \
  -F "file=@invoice.pdf"
```

### `POST /extract/resume`
Extract contact info, work history, education, skills.
```bash
curl -X POST http://localhost:8000/extract/resume \
  -F "file=@resume.pdf"
```

### `POST /extract/contract`
Extract parties, effective date, term, governing law, key clauses, signatures.
```bash
curl -X POST http://localhost:8000/extract/contract \
  -F "file=@nda.pdf"
```

### `POST /extract/custom`
Bring your own JSON Schema. Pass `schema` as a form field containing a JSON Schema object.
```bash
curl -X POST http://localhost:8000/extract/custom \
  -F "file=@purchase_order.pdf" \
  -F 'schema={"type":"object","properties":{"po_number":{"type":"string"},"buyer":{"type":"string"},"total":{"type":"number"}},"required":["po_number","total"]}'
```

Or via httpx:

```python
import httpx, json

schema = {
    "type": "object",
    "properties": {
        "po_number": {"type": "string"},
        "buyer": {"type": "string"},
        "total": {"type": "number"},
    },
    "required": ["po_number", "total"],
}

with open("po.pdf", "rb") as f:
    r = httpx.post(
        "http://localhost:8000/extract/custom",
        files={"file": f},
        data={"schema": json.dumps(schema)},
        timeout=60,
    )
print(r.json())
```

### `GET /schemas`
List built-in schemas and their JSON Schema representations.
```bash
curl http://localhost:8000/schemas
```

## Configuration

All config lives in `.env`. Defaults shown:

| Variable | Default | Description |
|---|---|---|
| `LLM_PROVIDER` | `openai` | `openai` or `anthropic` |
| `LLM_MODEL` | `gpt-4o-mini` | Any model the provider supports (e.g. `claude-sonnet-4-20250514`) |
| `OPENAI_API_KEY` | — | Required if `LLM_PROVIDER=openai` |
| `ANTHROPIC_API_KEY` | — | Required if `LLM_PROVIDER=anthropic` |
| `MAX_RETRIES` | `3` | Retries on schema validation failure |
| `MAX_FILE_SIZE_MB` | `25` | Upload size limit |
| `OCR_LANGUAGE` | `eng` | Tesseract language code (`eng+fra` for multi) |
| `REQUEST_TIMEOUT_S` | `60` | LLM call timeout |
| `INCLUDE_CONFIDENCE` | `true` | Set `false` to skip confidence scoring (faster, cheaper) |
| `LOG_LEVEL` | `INFO` | `DEBUG`, `INFO`, `WARNING`, `ERROR` |

## Customization

### Add a new schema

1. Create `app/schemas/your_schema.py`:
   ```python
   from pydantic import BaseModel, Field
   from datetime import date

   class PurchaseOrder(BaseModel):
       po_number: str = Field(description="PO identifier")
       buyer: str
       total: float
       delivery_date: date | None = None
   ```

2. Register it in `app/schemas/__init__.py`:
   ```python
   from .your_schema import PurchaseOrder
   REGISTRY["purchase_order"] = PurchaseOrder
   ```

3. The endpoint `POST /extract/purchase_order` is auto-generated by the dynamic router in `app/main.py`.

### Swap the LLM backend at runtime

Set `LLM_PROVIDER` and `LLM_MODEL` in `.env` — no code changes needed. The `Agent` is constructed from `app/llm.py:get_model()`.

### Tune extraction prompts

Per-schema system prompts live in `app/prompts/`. Edit the markdown files directly; they're loaded on startup.

### Disable OCR

If you only handle native-text PDFs, remove `pytesseract` from `requirements.txt` and set `OCR_LANGUAGE=` (empty) — the parser will skip image OCR and fall back to `pypdf` only.

## License

MIT. Use it commercially, fork it, resell it — just keep the license file.
