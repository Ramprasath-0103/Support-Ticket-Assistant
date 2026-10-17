# Support Ticket Assistant

A service that classifies customer support tickets and suggests the right team, priority, and sentiment. Its LangGraph pipeline adds structured-output validation, PII redaction, prompt injection checks, retry/fallback handling, prompt versioning, and token-cost estimates.

## Features

- Classifies issue category, assigned team, priority, and customer sentiment.
- Returns a confidence score, short reasoning, and human-review flag.
- Redacts detected email addresses, phone numbers, and credit-card-like numbers before classification.
- Blocks tickets identified as prompt-injection attempts.
- Validates model output and falls back to a conservative classification when needed.
- Reports prompt version and estimated token cost.
- Includes a browser demo and automatically generated API documentation.

## Requirements

- Python 3.11+
- An OpenAI API key (Appropriate code changes to be done incase of other API keys)

## Quick start

From the project root:

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
python -m pip install -r requirements.txt
cp .env.example .env               # Windows: copy .env.example .env
```

Set `OPENAI_API_KEY` in `.env`, then start the API:

```bash
python -m uvicorn main:app --reload --port 8000
```

The service is available at `http://localhost:8000`. Open `demo_ui/index.html` in a browser to use the demo, or visit `http://localhost:8000/docs` for interactive API documentation.

## Configuration

| Variable | Default | Description |
| --- | --- | --- |
| `OPENAI_API_KEY` | Required | API key used for model requests. |
| `DEFAULT_MODEL` | `gpt-4o-mini` | Model used for classification and injection checks. |
| `PROMPT_VERSION` | `v2` | Registered prompt version (`v1` or `v2`). |
| `LOG_COSTS` | `true` | Log estimated token use and cost. |

## API

### `POST /classify`

Request fields: `ticket_text` (5–4000 characters) and optional `channel` (`web_form` or `email`; defaults to `web_form`).

```bash
curl -X POST http://localhost:8000/classify \
  -H "Content-Type: application/json" \
  -d '{"ticket_text":"I was charged twice for my order. Please refund me.","channel":"web_form"}'
```

The JSON response includes `issue_category`, `assigned_team`, `priority`, `user_sentiment`, `confidence_score`, `reasoning`, `requires_human_review`, `pii_detected`, `prompt_version`, `cost_info`, and `injection_blocked`.

Other endpoints:

- `GET /health` — service health check.
- `GET /prompts` — available prompt versions and the active version.
- `GET /docs` and `GET /redoc` — interactive and reference API docs.

## Classification values

- **Categories:** `order_issue`, `payment_issue`, `delivery_issue`, `product_issue`, `account_issue`, `refund_request`, `other`
- **Teams:** `fulfillment_team`, `payments_team`, `logistics_team`, `customer_support`, `tech_team`
- **Priorities:** `low`, `medium`, `high`, `critical`
- **Sentiments:** `positive`, `neutral`, `negative`, `angry`

## Pipeline

`PII redaction → injection check → structured classification → validation → cost logging`

If validation fails, the pipeline uses its retry/fallback path before returning a result. Tickets flagged as injection attempts are blocked and marked for human review.

## Tests

```bash
pytest tests/test_classifier.py -v
```

## Project layout

```text
main.py                       FastAPI application and endpoints
graph.py                      LangGraph classification pipeline
schema.py                     Pydantic response models and enums
production_modules/           Redaction, validation, prompts, retries, costs
demo_ui/index.html            Standalone browser demo
tests/test_classifier.py      Test suite
documentation.md              Detailed implementation and extension guide
requirements.txt              Python dependencies
```


