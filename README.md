# Document to Structured JSON API Design

**[Portfolio](https://github.com/Reactance0083)** · **Status: interface scaffold**

This repository contains a small FastAPI interface and typed Pydantic models. It is useful for inspecting an API contract and proposed workflow shape.

## What is in the source

The application is `main.py`, with models for **DocumentInput, ExtractedField, and DocumentOutput**.

| Route | Current behavior |
| --- | --- |
| GET /health | Returns service health. |
| POST /extract | Upload interface; handler raises NotImplementedError. |
| POST /batch-extract | Metadata-list interface; handler raises NotImplementedError. |

The business handlers deliberately raise `NotImplementedError`. Without an exception handler, calls to those routes fail; they are not functioning demonstrations.

## What remains to build

The public source does not implement OCR, text extraction, LLM inference, batch processing, persistence, or validated extraction accuracy. Descriptive comments about a fuller version are not evidence that those features exist in this repository.

## Inspect locally

```bash
python -m venv .venv
# Activate the environment for your shell.
python -m pip install -r requirements.txt
python -m uvicorn main:app --host 127.0.0.1 --port 8000
```

If dependencies import successfully, inspect `/docs` and `/health`. Do not supply real customer documents or tickets to placeholder endpoints.

This scaffold is listed among earlier design experiments rather than the portfolio's completed implementations. No paid package or separate deployment was verified in this review.

Developed with AI assistance. Documentation reviewed against source on November 10, 2026.
