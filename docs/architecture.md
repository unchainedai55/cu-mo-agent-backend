# CU Marketing Opportunity Agent — Architecture

## Stack Overview
- **UI:** Lovable (exec-friendly web app with “Run Analysis”, panels, PDF export)
- **Orchestration:** Lindy.ai agents  
  - Orchestrator (webhook)  
  - Analyst (underperforming segments)  
  - Gap-Finder (missed opportunities vs competitors)  
  - Copywriter (ready-to-run campaigns)
- **Data:** Supabase (Postgres + REST), tables: `campaigns`, `segments`, `competitor_offers`, `insights`
- **Auth/Secrets:**  
  - Supabase: anon (read) + service_role (write) keys  
  - Lindy: workspace/agent secrets (Supabase keys, webhook)  
  - GitHub: repo Actions Secrets (backup of env values)

## High-Level Flow
1. Exec clicks **Run Analysis** in the Lovable UI.  
2. UI calls the **Lindy Orchestrator** via webhook.  
3. Orchestrator fetches data from Supabase (`campaigns`, `segments`, `competitor_offers`) using REST.  
4. Orchestrator invokes sub-agents:  
   - **Analyst** → finds underperforming segments  
   - **Gap-Finder** → detects missed opportunities vs. competitors  
   - **Copywriter** → produces 3 campaign cards  
5. Orchestrator writes results to Supabase `insights` with a `run_id`.  
6. UI queries the **latest `run_id`** and renders three panels; user can **Export PDF**.

## Mermaid Diagram
```mermaid
flowchart LR
  A[Exec clicks 'Run Analysis' (Lovable UI)] --> B{{Lindy Orchestrator (Webhook)}}
  B --> C[(Supabase: campaigns)]
  B --> D[(Supabase: segments)]
  B --> E[(Supabase: competitor_offers)]
  B --> F[Analyst Agent]
  B --> G[Gap-Finder Agent]
  B --> H[Copywriter Agent]
  F --> I[(Supabase: insights)]
  G --> I
  H --> I
  I --> A
