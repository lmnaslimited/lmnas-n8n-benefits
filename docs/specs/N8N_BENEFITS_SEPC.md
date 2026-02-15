Objective
Create an n8n setup that:
Exposes one public webhook
Routes request based on payload.benefit_type
Executes the corresponding workflow
Returns standardized JSON response

Supported benefit flows:
ROI_Calculator
Pipeline_audit
CPQ_Maturity_scan
Sales_cycle_analyzer
Tender_complexity_score

🧠 Overall Architecture
LMNAs Website
     ↓
POST /webhook/benefit-engine
     ↓
Router Workflow
     ↓
Execute Sub-Workflow
     ↓
Return JSON Response
📘 CODEx SPECIFICATION

🔷 1️⃣ Project Structure
Create repository structure:
lmnas-n8n-benefits/
│
├── docker/
│   └── docker-compose.yml
│
├── workflows/
│   ├── benefit_router.json
│   ├── roi_calculator.json
│   ├── pipeline_audit.json
│   ├── cpq_maturity_scan.json
│   ├── sales_cycle_analyzer.json
│   └── tender_complexity_score.json
│
├── README.md
└── .env.example

🔷 2️⃣ Environment Variables
Create .env.example
N8N_PORT=5678
N8N_WEBHOOK_URL=http://localhost:5678/
N8N_API_KEY=supersecretkey

🔷 3️⃣ Docker Setup
Create docker/docker-compose.yml
version: "3.8"

services:
  n8n:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    environment:
      - N8N_PORT=5678
      - WEBHOOK_URL=${N8N_WEBHOOK_URL}
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=admin
    volumes:
      - ./n8n_data:/home/node/.n8n

🔷 4️⃣ Standard Payload Contract
Codex must enforce this input format:
{
  "benefit_type": "ROI_Calculator",
  "user_context": {
    "company_name": "",
    "industry": "",
    "annual_revenue": 0
  },
  "answers": {
    "question_1": "",
    "question_2": ""
  }
}

🔷 5️⃣ Standard Response Contract
All workflows MUST return:
{
  "success": true,
  "benefit_type": "ROI_Calculator",
  "score": 78,
  "summary": "Your ROI potential is high...",
  "recommendations": [],
  "meta": {}
}
If error:
{
  "success": false,
  "error_code": "INVALID_INPUT",
  "message": "Missing annual_revenue"
}

🔷 6️⃣ Master Router Workflow
Workflow name:
Benefit_Engine_Router
Nodes:
Webhook Node
Path: /benefit-engine
Method: POST
Authentication:
Validate header x-api-key
Respond immediately: OFF
IF Node – Validate API Key
Check:
$headers["x-api-key"] === $env.N8N_API_KEY
If false → Return error
Switch Node – Route by benefit_type
Switch on:
{{$json["benefit_type"]}}
Cases:
ROI_Calculator
Pipeline_audit
CPQ_Maturity_scan
Sales_cycle_analyzer
Tender_complexity_score
Each case:
→ Execute Workflow node
Execute Workflow Node
Pass full input JSON
Wait for completion
Return response
Respond to Webhook Node
Return sub-workflow output

🔷 7️⃣ Individual Workflow Spec
Each benefit workflow must:
Structure:
Start Node (Execute Trigger)
Function Node – Validate Input
Function Node – Compute Score
Function Node – Generate Summary
Return JSON

🔷 ROI_Calculator Logic Example
Score Calculation
Use formula:
roi_score = 
   (annual_revenue * 0.02) 
   + (pipeline_value * 0.01)
   - (manual_effort_hours * 10)
Normalize to 0–100 scale.
Return:
{
  success: true,
  benefit_type: "ROI_Calculator",
  score,
  summary,
  recommendations
}

🔷 Pipeline_audit
Evaluate:
Lead leakage
Stage conversion gaps
Deal aging
Return:
pipeline_health_score (0–100)
bottleneck_stage
recommendation_list

🔷 CPQ_Maturity_scan
Evaluate:
Configuration automation
Pricing governance
Engineering collaboration
Approval workflows
Return:
maturity_level (1–5)
improvement_priority

🔷 Sales_cycle_analyzer
Evaluate:
Average cycle length
Engineering delay
Approval turnaround
Return:
cycle_efficiency_score
delay_source

🔷 Tender_complexity_score
Evaluate:
Customization ratio
Number of variants
Technical review cycles
Bid submission time
Return:
complexity_index (0–100)
risk_level (Low/Medium/High)

🔷 8️⃣ Validation Rules
All workflows must:
Validate required fields
Return structured error
Never return raw stack traces

🔷 9️⃣ Observability
Add:
Execution ID in response:
meta: {
   execution_id: $execution.id
}

🔷 🔐 Security Requirements
API key validation
No open webhook
Rate limit ready (future)

🔷 🔄 Versioning Strategy
Each workflow must include metadata:
version: "1.0.0"
benefit_version: "2026-Q1"

🔷 📘 README Must Include
How to run
How to import workflows
How to test via curl
Expected request format
Environment setup

🔷 Example curl test
curl -X POST http://localhost:5678/webhook/benefit-engine \
  -H "Content-Type: application/json" \
  -H "x-api-key: supersecretkey" \
  -d '{
    "benefit_type": "ROI_Calculator",
    "user_context": {
      "company_name": "ABC Transformers",
      "industry": "Manufacturing",
      "annual_revenue": 10000000
    },
    "answers": {
      "pipeline_value": 5000000,
      "manual_effort_hours": 40
    }
  }'