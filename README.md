# lmnas-n8n-benefits

n8n workflow package for a single public webhook that routes benefit assessments to sub-workflows and always returns a standardized JSON envelope.

## Included assets

- `docker/docker-compose.yml` – local n8n runtime.
- `.env.example` – required environment variables.
- `workflows/benefit_router.json` – public webhook router (`POST /webhook/benefit-engine`).
- `workflows/roi_calculator.json`
- `workflows/pipeline_audit.json`
- `workflows/cpq_maturity_scan.json`
- `workflows/sales_cycle_analyzer.json`
- `workflows/tender_complexity_score.json`

## Supported `benefit_type`

- `ROI_Calculator`
- `Pipeline_audit`
- `CPQ_Maturity_scan`
- `Sales_cycle_analyzer`
- `Tender_complexity_score`

## Input contract

```json
{
  "benefit_type": "ROI_Calculator",
  "user_context": {
    "company_name": "Acme",
    "industry": "Manufacturing",
    "annual_revenue": 12000000
  },
  "answers": {
    "question_1": 350000,
    "question_2": 40
  }
}
```

## Standard response contract

Successful responses:

```json
{
  "success": true,
  "benefit_type": "ROI_Calculator",
  "score": 78,
  "summary": "Your ROI potential is high...",
  "recommendations": [],
  "meta": {
    "execution_id": "123",
    "version": "1.0.0",
    "benefit_version": "2026-Q1"
  }
}
```

Error responses:

```json
{
  "success": false,
  "error_code": "INVALID_INPUT",
  "message": "Missing annual_revenue"
}
```

## Run locally

1. Copy env file:
   ```bash
   cp .env.example .env
   ```
2. Start n8n:
   ```bash
   docker compose -f docker/docker-compose.yml --env-file .env up -d
   ```
3. Import all workflow JSON files from `workflows/` in n8n UI.
4. Activate `Benefit_Engine_Router` and sub-workflows.

## Request example

```bash
curl -X POST "http://localhost:5678/webhook/benefit-engine" \
  -H "Content-Type: application/json" \
  -H "x-api-key: supersecretkey" \
  -d '{
    "benefit_type": "ROI_Calculator",
    "user_context": {
      "company_name": "Acme",
      "industry": "Manufacturing",
      "annual_revenue": 12000000
    },
    "answers": {
      "pipeline_value": 350000,
      "manual_effort_hours": 40
    }
  }'
```

## Notes

- API key is validated before routing.
- Router does not respond immediately and waits for the selected sub-workflow completion.
- Sub-workflows return structured validation errors and never surface stack traces.
