1. Your baseline requirements ( distilled )

Requirement	Why it matters for your ISV scenario
Massive throughput – millions of POs/invoices/month	Batch OCR/validation queues must peak-scale without throttling.
Per-customer code & data boundaries	Some customers need bespoke parsing logic while keeping their data, compute and keys isolated.
Full audit & observability	Finance workflows demand end-to-end trace-ability, tamper-proof logs and clear SLAs.
Long-term platform stability	A multi-year product can’t afford to rebuild every time an SDK flips a verb.
Security & network control	Ability to pin workloads to specific VNets / regions and to run completely inside the ISV’s tenant.


⸻

2. Option A – Dapr + low-level Azure SDK (+ Semantic Kernel where needed)

Strengths	Weak spots
Runs anywhere. Dapr sidecars can sit in AKS, customer-hosted K8s, edge devices or dedicated clusters. Full control over region, VNet and network policies.	You run the cluster life-cycle, upgrades and patching.
Mature & backward-compatible. Dapr’s 1.x line guarantees non-breaking upgrades; v 1.15 (Feb 2025) marked the Workflow API “stable”.  ￼	Each Dapr component (bindings, secret-stores, state-stores) still needs hardening for finance workloads.
Adoption proof. 72 % of surveyed teams already run Dapr in mission-critical production in 2025.  ￼	You build your own evaluator/guard-rails and LLM safety net.
First-class OpenTelemetry. Sidecars emit traces/metrics you can pipe straight into Azure Monitor or Grafana.	Content-safety, prompt-shields and red-teaming are DIY integrations.
Semantic Kernel embeddable. The process-framework library lets you compose multi-agent workflows inside your own pods; roadmap shows continued investment in 2025 H1.  ￼	SK APIs are still settling; pin a minor version and run contract tests to avoid accidental breaks.

Verdict when it shines

Deep control plane work (OCR, classification, reconciliation, PDF generation) where you want per-customer logic in separate micro-services, need to self-host in a regulated region, or must guarantee zero-egress architectures.

⸻

3. Option B – Azure AI Foundry Agent Service

Strengths	Weak spots / open questions
Fully managed & GA – announced GA at Build 2025.	Compute runs in Microsoft-managed clusters; no on-prem or air-gapped option today.
Threads, runs, built-in evals & tracing – out-of-box prompt-shield, content filters, red-team evaluators, Application Insights hooks.	Per-tenant throughput quotas; need benchmarks for sustained doc bursts.
Multi-agent orchestration (Process Framework) and rich tool connectors (Azure AI Search, Databricks, Logic Apps, OpenAPI actions).	Custom actions run as HTTPS calls – not as in-process code; heavy CPU OCR still lives outside the service.
Inter-op ready – supports Google A2A, Anthropic MCP, Copilot Studio “connected agents”. Future proofing for mixed clouds.	Identity & RBAC via Entra “coming soon” – not GA yet, which may delay fine-grained customer isolation.
Managed content safety & audit – Microsoft keeps guard-rails current.	No SLA numbers yet for long-running (multi-minute) jobs; validate against SLA once published.

Verdict when it shines

Conversation-centric or workflow-orchestration layers (supplier portal chat, exception triage, status Q&A) where development speed, built-in guard-rails and integration with Microsoft 365 / Logic Apps are critical and you are comfortable running inside Azure’s managed boundary.

⸻

4. Hybrid pattern that many large ISVs are landing on

Dapr Micro-services  (OCR, validation, tax rules)  <-- AKS / customer-scoped clusters
        │
        │  REST / gRPC  (OpenAPI exported)
        ▼
Azure AI Foundry Agent Service
        – Supplier FAQ agent
        – Exception-triage agent
        – Human-in-loop escalation

Heavy document crunching stays in Dapr pods you fully control; knowledge and conversational surfaces live in Foundry Agent Service, calling back into your APIs through secured OpenAPI tools.

⸻

5. Targeted research questions before you commit

Area to validate	Why it matters
Throughput & cost model – load-test Foundry’s runs/sec limits with a month-worth of invoices; compare to AKS + Dapr node pricing.	Ensures the managed service won’t choke or blow the budget at quarter-end.
VNet / Private Link availability & latency in Agent Service.	You promised customers compute-boundary clarity; measure egress paths.
Entra integration GA timeline & feature depth (role-assignments, managed-identity impersonation).	Defines whether you can offer per-customer RBAC from day 1.
Document Intelligence pipeline options – benchmark DI v4 container vs DI cloud and measure hand-off cost when called from Agent Service.	May influence where OCR lives.
Upgrade and deprecation cadence for Semantic Kernel and Agent Service.	Establish contract-tests and pin versions to absorb API shifts.
Observability parity – compare OTEL traces from Dapr with Foundry’s built-in run logs and Insights export format.	You need a single audit lake for regulators.
Protocol roadmap – track MCP and A2A specs; test that your custom tools expose compliant OpenAPI (MCP) so you can later swap orchestrators with minimal change.	Future-proof multi-cloud agent usage.


⸻

6. Pragmatic recommendation
	1.	Spin up a spike:
Deploy Dapr v1.15 on AKS with one customer’s invoice pipeline; instrument with OTEL and SK Process Framework for orchestration.
	2.	Parallel POC on Agent Service:
Build a supplier self-service conversational agent that answers PO status, escalates issues and calls your Dapr APIs via OpenAPI tools.
	3.	Run the research items (throughput, cost, network, identity) for a month’s synthetic traffic; push all telemetry into the same Log Analytics workspace so results are comparable.
	4.	Decide by capability, not platform:
	•	Core document handling → keep in Dapr until Agent Service offers BYO-container or edge deployment.
	•	User-facing automation & multi-agent reasoning → leverage Agent Service to avoid re-implementing guard-rails and evaluation.

This staged approach gives you immediate control where auditors demand it, while you de-risk the newer managed stack and stay ready to migrate more workloads as Azure AI Foundry matures over the next 6-12 months.