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
2. UI calls the **Lindy Orchestrator** webhook.  
3. Orchestrator reads campaign, segment, and competitor data from Supabase.  
4. Orchestrator sends this data to three sub-agents:  
   - **Analyst** → underperforming segments  
   - **Gap-Finder** → missed opportunities vs competitors  
   - **Copywriter** → 3 ready-to-run campaigns  
5. Each sub-agent writes its output to Supabase `insights`.  
6. UI pulls the latest `insights` for the most recent run and displays them in panels.

## Mermaid Diagram
```mermaid
flowchart LR
  UI[Lovable UI] --> ORCH([Lindy Orchestrator])
  ORCH --> DB[(Supabase: campaigns, segments, competitor_offers)]
  ORCH --> ANALYST[Analyst]
  ORCH --> GAP[Gap-Finder]
  ORCH --> COPY[Copywriter]
  ANALYST --> INS[(Supabase: insights)]
  GAP --> INS
  COPY --> INS
  INS --> UI
