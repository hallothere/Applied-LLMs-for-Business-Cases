# Scoping Document — B5: Executive dashboard automation

---

## 1) Problem to solve

### 1.1 Problem clarification
- **Actual problem:** Executives review weekly KPI dashboards but struggle to quickly understand *why* metrics changed and what actions to take. Leadership wants an AI-powered system that automatically explains KPI movements in plain language.
- **Decision-maker / end user:** Executives, senior leadership, product leadership, business owners.
- **Decision / action informed by the system:** Identify drivers behind KPI changes, prioritize investigations, and decide business actions (marketing spend, product focus, operational response).

**Deliverable — Problem statement (2–3 sentences)**  
Executives want an AI-powered dashboard that automatically explains weekly KPI changes in plain language. The system should detect significant KPI movements, identify likely drivers (e.g., campaigns, product changes, seasonality, outages), and generate short, trustworthy explanations. The output should support faster decision-making while avoiding misleading causal claims.

---

## 2) Problem type classification

**Primary framing:** Analytics automation + LLM-based summarization (with structured inputs)  
**Secondary framing (optional):** Anomaly detection, time-series change detection, causal analysis (limited), retrieval-augmented generation (RAG)

**Justification (short):**  
The goal is not to predict outcomes, but to detect KPI shifts and explain them using available structured data and business context. This is best framed as change detection + automated reporting, where LLMs generate natural-language summaries grounded in data.

*The difference between the 2 scenarions:
1. Demand spike task (A4): predict future demand spikes early enough to take operational action (inventory/staffing).
2. Executive dashboard task (B5): explain past KPI changes so leadership understands what happened and what to do next.

So:
➡️ A4 = forecasting for prevention
➡️ B5 = explanation for decision-making

---

## 3) Data science scope

### 3.1 Available data (or likely to exist)
- Weekly KPI time series (retention, conversion, revenue, DAU/WAU, churn, etc.)
- Breakdown dimensions (country, device, channel, product area)
- Experiment results (A/B tests)
- Release notes / feature launches
- Marketing campaign calendar
- Incident logs (outages, latency spikes)
- Support tickets / complaint volume (optional)

### 3.2 Data modality
- **Time-series KPI tables** (weekly aggregates)
- **Tabular breakdowns** (KPI by segment)
- **Text sources** (release notes, incident summaries, marketing notes)
- **Logs** (experiments, system reliability)

### 3.3 Granularity
- Weekly KPI level (executive cadence)
- Segment-level comparisons (week-over-week by country/device/channel)
- Optional daily view for debugging

### 3.4 Labels (targets)
- No clean labeled dataset for “correct explanation”
- Possible weak supervision:
  - past human-written weekly business reviews
  - analyst summaries
  - incident postmortems linked to KPI drops
- The system output must be evaluated via human review and trustworthiness metrics.


### 3.5 Key data risks
- Hallucination risk: LLM generates plausible but incorrect explanations
- Causality risk: correlational explanations presented as causal
- Data freshness: delays in KPI pipelines create incorrect narratives
- Missing context: campaigns/releases not recorded consistently
- Trust: executives may over-trust automated explanations
- Privacy/security: sensitive business information must be handled carefully

**Deliverable — data + risks + label strategy**  
The system will rely on weekly KPI time series, segment breakdowns, and contextual business logs (campaigns, releases, incidents). Labels for “correct explanations” likely do not exist and may require weak supervision from past reports. The biggest risks are hallucinations, incorrect causal claims, and missing context leading to misleading executive decisions.

---

## 4) Modeling approach (high-level)

This is best designed as a pipeline, not a single model.

### 4.1 Stage 1 — KPI change detection and prioritization
- Detect statistically significant week-over-week changes
- Identify which KPIs require explanation (avoid noise)
- Rank changes by business impact

Possible approaches:
- time-series change point detection
- anomaly detection
- rule-based thresholds with confidence scoring

---

### 4.2 Stage 2 — Driver analysis (structured analytics)
- Break down changes by key dimensions:
  - country
  - device
  - acquisition channel
  - product area
- Identify which segments contributed most to the KPI movement
- Attach relevant known events:
  - campaigns
  - launches
  - outages
  - experiments

---

### 4.3 Stage 3 — LLM explanation generation (grounded)
Use an LLM to generate a short executive summary based on:
- detected KPI changes
- segment drivers
- retrieved relevant context (RAG)

Key requirement:
- explanations must be grounded in retrieved facts and clearly separate correlation from causation.

---

### 4.4 Not appropriate approaches (initially)
- End-to-end supervised ML to “generate explanations” without grounding data
- LLM-only approach without structured driver analysis (high hallucination risk)

---

**Deliverable — approach selection**  
Build a three-stage system: (1) detect meaningful KPI changes, (2) compute structured driver breakdowns, and (3) generate executive summaries using an LLM grounded in retrieved business context (campaigns, releases, incidents). Emphasize trust, transparency, and guardrails against hallucination and false causal claims.

---

## 5) Use of LLMs (reflection)
[Write after completing the scoping work]

---

## 6) Unknown methods discovery (required)
[Write after completing the scoping work]
