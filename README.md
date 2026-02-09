# Applied-LLMs-for-Business-Cases

# Scoping a Data / ML Problem Using LLMs

---

## Overview: What is this task?

You are acting as a **Product Manager working with a Data Science / ML team**.

You will be given a **vaguely defined real-world problem** where leadership wants to “use AI” or “use ML.”
Your job is **not to build a model**, but to **scope the problem correctly** before any modeling begins.

This includes:
- Clarifying what problem is *actually* being solved
- Identifying the **right class of methods** (including methods you may not yet know)
- Defining realistic data and modeling scope

You are encouraged to use **LLMs (e.g., ChatGPT, Claude)** as a thinking partner to:
- Clarify ambiguity
- Explore unfamiliar ML approaches
- Self-learn concepts you have not formally studied yet

You are responsible for the final reasoning and framing.

---

## Important constraints

- Assume **clean labeled training data does NOT exist initially**
- You may need to:
  - Discover structure before prediction
  - Work with text, images, audio, logs, or time series
  - Justify why supervised learning may not be appropriate (yet)
- You are judged on **clarity of thinking, scope, and justification**

---

## Deliverable: Scoping document structure

For each **selected task** from the provided list, create a scoping document with the sections below.

---

## 1) Problem to solve

### 1.1 Problem clarification
- What is the **actual problem** to solve?
- Who is the **decision-maker or end user**?
- What decision or action will this system inform?

**Deliverable**
- A concise, plain-English problem statement (2–3 sentences)

---

## 2) Problem type classification

Identify the most appropriate framing (choose one primary framing; mention secondary framings if relevant):
- Supervised learning (classification / regression)
- Unsupervised learning (clustering, topic modeling, anomaly detection)
- Time-series forecasting
- Ranking / recommendation
- Representation learning
- Causal analysis / experimentation

**Deliverable**
- Problem type
- Short justification for why this framing fits

---

## 3) Data science scope

Describe:
- Available data (or data that likely exists)
- Data modality (tabular, text, images, audio, time series, logs)
- Granularity (user-level, event-level, session-level, etc.)
- Whether labels exist or could realistically be created
- Key data risks (bias, leakage, sparsity, quality)

**Deliverable**
- A structured list of data sources + risks
- A brief note on label strategy (if any)

---

## 4) Modeling approach (high-level)

Propose:
- 1–3 reasonable model families or approaches
- Why these are appropriate given the data and problem
- What approaches would *not* be appropriate (and why)

> No equations or implementation details required.

**Deliverable**
- 1–3 candidate approaches with reasoning
- 1–2 “not appropriate” approaches with reasoning

---

## 5) Use of LLMs (reflection)

Explain:
- How you used an LLM in this task
- One example where it helped clarify your thinking
- One example where you challenged, corrected, or refined the LLM’s output

**Deliverable**
- A short reflection (5–10 lines)

---

## 6) Unknown methods discovery (required)

You are expected to encounter methods you have **not formally learned yet**.

Include:
- At least one technique or model you did not previously know
- Why it is relevant to this problem
- A short explanation of the concept **in your own words**
- One incorrect assumption you initially had, and how it was corrected

**Deliverable**
- 1 new method + your explanation + the corrected assumption
