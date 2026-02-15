# lmnas-n8n-benefits

n8n workflow package for a single public webhook that routes benefit assessments to sub-workflows and always returns a strict JSON envelope for benefit engine responses.

## Included assets

- `docker/docker-compose.yml` – local n8n runtime.
- `workflows/benefit_router.json` – public webhook router (`POST /webhook/benefit-engine`).
- Benefit workflows in `workflows/*.json` (ROI, Pipeline, CPQ, Sales Cycle, Tender Complexity).

## Input contract

```json
{
  "session": {
    "sessionId": "af0dd49d-a480-475a-8081-ccae63cad870",
    "anonymousId": "5e8739fc-acfd-4e1c-9190-0e703dc10200"
  },
  "input": {
    "benefitSlug": "roi-calculator",
    "sessionId": "af0dd49d-a480-475a-8081-ccae63cad870",
    "answers": [
      { "questionId": "annualRevenue", "value": "10000" },
      { "questionId": "quoteTurnaroundDays", "value": "10" },
      { "questionId": "winRate", "value": "10" }
    ],
    "stage": "standard_completed"
  },
  "prompts": {
    "system": "You are LMNAs benefit intelligence assistant.",
    "user": "{...}"
  }
}
```

## Strict response contract (all flows)

```json
{
  "followup_required": false,
  "followupQuestions": [
    { "id": "q1", "prompt": "Optional follow-up" }
  ],
  "result": {
    "score": 78,
    "summary": "You can reduce quote cycle by 15%.",
    "recommendation": "Automate pricing approvals."
  }
}
```

- `followup_required` is `true` when `input.stage !== "standard_completed"`.
- Follow-up mode returns a placeholder result with score `0` and follow-up question(s).
- Each benefit workflow now includes a `Mock LLM RAG` node that produces prompt-aware mocked recommendations.

## Supported `benefitSlug`

- `roi-calculator`
- `pipeline-audit`
- `cpq-maturity-scan`
- `sales-cycle-analyzer`
- `tender-complexity-score`
