# Scoping Document — B4: Feature success evaluation

---

## 1) Problem to solve

### 1.1 Problem clarification
- **Actual problem:** Leadership wants to know whether a feature launched last quarter created measurable business value and whether it should be expanded, improved, or rolled back.
- **Decision-maker / end user:** Leadership, Product team, Growth/Analytics team.
- **Decision / action informed by the system:** Continue investment, iterate, roll back, scale rollout, or adjust targeting.

**Deliverable — Problem statement**  
A major product feature was launched last quarter, and leadership wants to evaluate whether it was successful. The goal is to measure the feature’s causal impact on key business outcomes (e.g., retention, conversion, revenue), while accounting for confounding factors such as seasonality and user mix changes. The result should inform whether to scale, iterate, or discontinue the feature.

---

## 2) Problem type classification

**Primary framing:** Causal analysis / experimentation  
**Secondary framing:** A/B testing (if possible), observational causal inference, time-series intervention analysis

**Justification:**  
This is not primarily a prediction problem. Leadership is asking “did the feature cause improvement?”, which requires estimating causal impact rather than training a model to predict outcomes.

---

## 3) Data science scope

### 3.1 Available data (or likely to exist)
- Product analytics event logs (feature usage events)
- User-level outcomes (retention, churn, conversion, purchases)
- User metadata (country, device, acquisition channel, account age)
- Feature rollout information (launch date, rollout % by region/platform, eligibility rules- who could access the feature)
- Marketing calendar (campaigns, pricing changes)
- System reliability data (outages, latency: technical performance like the app was slow after launch, API latency increased, error rates spiked etc.) if relevant

### 3.2 Data modality
- **Event logs / clickstream** (user actions with timestamps)
- **Tabular user-level aggregates** (weekly retention, sessions/week, revenue/user)
- Potentially **time-series** at product level (overall KPIs over time)

### 3.3 Granularity
- User-level (per user, per week)
- Event-level (each interaction with the feature)
- Cohort-level (users grouped by signup date, region, platform)
- Time period: before vs after launch, and post-launch weeks

### 3.4 Labels (targets)
No single “success” label exists by default. Success must be defined through business KPIs and guardrail metrics.

**Primary KPI (example)**
- Retention (D7 or D30) OR Conversion rate (depending on feature goal)

**Secondary KPIs**
- Revenue per user (ARPU)
- Engagement (sessions per user per week)
- Feature adoption rate (% of eligible users who used the feature)

**Guardrail metrics**
- Refund rate / cancellation rate
- Support tickets / complaints
- Latency / error rate
- Churn increase in key segments

**Note:** Success may be multi-dimensional. A feature can improve one metric while harming another (e.g., increased engagement but reduced revenue, or feature adoption gains that cannibalize other high-value flows).


### 3.5 Key data risks
- Bias / confounding: feature may have been rolled out to certain user segments first (power users, specific countries)
- Leakage: To avoid bias, “treatment” should be defined as feature exposure/eligibility (who had access), not post-launch usage, since adopters are typically more engaged and would otherwise inflate the estimated impact.
- Seasonality: quarter-to-quarter effects, holidays, pricing cycles
- Selection effects: users who adopt the feature may already be more engaged
- Missing data: tracking gaps, inconsistent event instrumentation
- Misaligned metrics: feature improves engagement but harms revenue (or vice versa)

**Deliverable — data + risks + label strategy**  
We likely have event logs and user-level KPIs before and after the launch. “Success” must be defined through business KPIs (retention, conversion, revenue), not a single label. The biggest risks are confounding and selection bias (who received/used the feature), seasonality, and instrumentation quality.

---

## 4) Modeling approach (high-level)

This is primarily a **causal evaluation** task (feature impact analysis), not a prediction task. The goal is to estimate whether the feature *caused* improvements in global product KPIs.

### 4.1 Step 1 — Define success metrics (global KPIs) and guardrails
First, define what “success” means at product level.

**Global KPI (primary)**
- Retention (D7/D30) OR Conversion rate OR Revenue per user (depending on feature goal)

**Secondary KPIs**
- Engagement (sessions per user per week)
- Feature adoption rate

**Guardrail metrics**
- Churn increase
- Refund/cancellation rate
- Support tickets / complaints
- Latency / error rate

---

### 4.2 Step 2 — Define exposure (“treatment”) and comparison groups
To estimate causal impact, we need to compare users who had access to the feature with users who did not.

- **Treatment group:** users who were eligible/exposed to the feature (not defined by usage)
- **Comparison group:** users not exposed, or exposed later (depending on rollout)

This step depends strongly on how the feature was launched:
- Randomized A/B test
- Gradual rollout (by region/platform/time)
- Full release to everyone

---

### 4.3 Step 3 — Choose the causal evaluation method (based on rollout)

**Option A: A/B test analysis (best case)**
Use this if a randomized experiment or holdout group exists.
- Compare KPI outcomes between treatment vs control groups.

**Option B: Difference-in-Differences (DiD)**
Use this if rollout was staged (e.g., some regions/platforms got it earlier).
- Compare KPI changes before vs after launch between early-exposed vs later-exposed groups.

**Option C: Interrupted time-series / causal impact**
Use this if no clean comparison group exists and the feature was released to everyone at once.
- Model the expected KPI trend from historical data (including seasonality) and estimate whether the KPI shifted after launch beyond normal variation.

---

### 4.4 Not appropriate approaches (initially)
- Training a supervised ML model to “predict success” without causal framing (prediction ≠ causation. A model can predict changes perfectly and still not tell you why.) 
- Comparing feature users vs non-users without correcting for selection bias (will overestimate impact)

---

**Deliverable — approach selection**  
Prefer randomized evidence (A/B test or holdout) when available. If not, leverage staged rollout timing for Difference-in-Differences. If the feature launched globally with no control group, use interrupted time-series / causal impact on aggregated KPI time series, while documenting assumptions and limitations.

---

## 5) Use of LLMs (reflection)

I used an LLM as a thinking partner to correctly frame this task. My initial intuition was to treat “feature success” as an ML prediction problem, but the LLM helped clarify that leadership is asking a causal question (“did the feature cause improvement?”), which requires experimentation or causal inference rather than supervised learning.

The LLM was helpful in structuring the evaluation into a clear workflow: defining global KPIs and guardrails, identifying treatment and comparison groups based on rollout design, and then selecting an appropriate causal method (A/B testing, Difference-in-Differences, or interrupted time-series). It also helped me distinguish between product-level time series (overall KPI trends) and user-level event logs, which are the typical data modalities for feature evaluation.

I also challenged and refined the LLM output. For example, I clarified that comparing “feature users vs non-users” is only valid when groups are defined by exposure/eligibility (randomized or rollout-based), not by post-launch adoption behavior, which creates selection bias. I also improved the modeling section by rewriting it so that the causal methods are presented as options chosen based on rollout conditions, rather than as unrelated techniques.

---

## 6) Unknown methods discovery (required)

### New methods discovered

**1) Difference-in-Differences (DiD)**
- **Why relevant:** Feature rollouts are often staged by region/platform/time rather than randomized, so DiD provides a practical way to estimate impact.
- **Explanation in my own words:** DiD estimates the feature effect by comparing how outcomes changed before vs after launch in the treated group, and subtracting the change observed in a comparison group that was not yet exposed. This helps remove background trends like seasonality or marketing effects.

**2) Interrupted time-series / causal impact (Bayesian structural time series)**
- **Why relevant:** If the feature launched globally and there is no clean control group, we can still estimate impact using historical KPI trends.
- **Explanation in my own words:** This method builds a baseline forecast of what the KPI would have looked like without the feature (based on past patterns and seasonality), then checks whether the post-launch KPI deviates beyond normal variation.

**3) Guardrail metrics**
- **Why relevant:** Feature-level success can be misleading if the feature improves one KPI while harming others (cannibalization or unintended side effects).
- **Explanation in my own words:** Guardrails are metrics tracked alongside the main KPI to ensure the feature does not “win” by breaking something else (e.g., higher engagement but lower revenue, or higher conversion but higher refund rate).

---

### Incorrect assumptions I initially had (and how they were corrected)

**Assumption 1: “This is a prediction problem because leadership said they want ML.”**  
- **Correction:** The core question is causal (“did the feature cause success?”). The correct framing is experimentation/causal inference, not training a supervised model.

**Assumption 2: “Comparing feature users vs non-users is always valid.”**  
- **Correction:** Comparing adopters vs non-adopters is biased because adoption is self-selected. Treatment should be defined by exposure/eligibility (who had access), not by post-launch usage.

**Assumption 3: “DiD is basically the same as A/B testing.”**  
- **Correction:** Both compare treated vs untreated groups, but DiD is not randomized and relies on assumptions such as parallel trends, while A/B testing is the strongest evidence when randomization exists.

**Assumption 4: “Success can be captured by one label.”**  
- **Correction:** Feature success is multi-dimensional and should be evaluated using a primary global KPI plus secondary KPIs and guardrails to detect cannibalization and unintended harm.
