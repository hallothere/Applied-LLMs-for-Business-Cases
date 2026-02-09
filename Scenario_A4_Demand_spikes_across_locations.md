# Scoping Document — A4: Demand spikes across locations

---

## 1) Problem to solve

### 1.1 Problem clarification
- **Actual problem:** Retail stores face unpredictable demand spikes, causing stockouts or overstock. The goal is to anticipate spikes early enough to take operational action.
- **Decision-maker / end user:** Store operations, supply chain planners, inventory management team.
- **Decision / action informed by the system:** Replenishment decisions (how much to ship), staffing adjustments, and potentially dynamic safety stock levels.

**Deliverable — Problem statement**  
Retail locations experience unpredictable demand spikes that lead to stockouts, lost revenue, and operational stress. We want a system that predicts short-term demand spikes per store (and/or flags upcoming anomalies) early enough to trigger actions such as replenishment, staffing, or inventory rebalancing. The system must work across many stores where demand patterns differ significantly.

---

## 2) Problem type classification

**Primary framing:** Time-series forecasting + anomaly/spike detection  
**Secondary framing (optional):** Hierarchical forecasting*, representation learning for store similarity**

**Justification (short):**  
We have historical sales time series per store, and the business goal is to predict near-future demand changes. Since the main pain point is *spikes* (rare but impactful), forecasting alone may miss the real objective, so anomaly/spike detection should be included as a parallel framing.

*Hierarchical forecasting ensures that store-level predictions and regional/company-level predictions are consistent, so different teams can plan from the same numbers.

**Representation learning helps the model automatically learn which stores behave similarly, so it can share patterns across stores and improve predictions—especially for stores with limited data.

---

## 3) Data science scope

### 3.1 Available data (or likely to exist)
- Historical sales per store (daily/hourly)
- Product-level sales (optional, but likely exists)
- Store metadata (location, store type, size, neighborhood, opening hours)
- Calendar features (weekday, month, holidays, school breaks)
- Promotions/discounts and marketing campaigns
- Stock levels and stockouts (very important if available)
- Weather data (often relevant depending on retail type)
- Local events (sports, concerts, etc. — maybe not available initially)

### 3.2 Data modality
- Primarily **time-series** (structured numeric)
- Potentially **tabular** metadata (store attributes)
- Potentially **logs** (events / promotion schedules)

### 3.3 Granularity
- Store-level time series (daily or hourly)
- Possibly SKU-level within store (higher complexity)
- Forecast horizon: likely next 1–14 days (depends on supply chain lead time, we can only act if we have time.)

### 3.4 Labels
- No clean labels initially for “spike events”
- Labels (target) can be created using business rules, e.g.:
  - Spike = sales > rolling mean + k * rolling std
  - Spike = top X% of daily demand for that store
  - Spike = demand jump above threshold vs previous week
- Important: “spike” definition must match business impact (stockouts, revenue loss). A spike should represent an event worth acting on.

### 3.5 Key data risks
- Bias: stores with more traffic dominate learning; new/small stores have sparse data
- Leakage: using future promo info incorrectly; using post-stockout sales as true demand (Stockouts “hide” the true demand signal)
- Sparsity: rare spikes → class imbalance
- Quality: missing days, POS errors (the cash register system), store closures, stockouts masking true demand
- Other: demand differs per store → global model may fail without store-specific adaptation

**Deliverable — data + risks + label strategy**  
We likely have sales time series per store plus calendar and promotions. Labels for spikes do not exist and should be derived via store-specific statistical thresholds aligned with business impact (statistical impact, operational impact, staffing impact). Biggest risks are leakage (future promo signals), stockouts hiding true demand, and high store heterogeneity.

---

## 4) Modeling approach (high-level)

Because stores behave differently and spikes are rare but high-impact, we propose a staged approach.

### 4.1 Stage 1 — Per-store forecasting baselines (benchmark)
Build simple forecasting baselines per store (e.g., seasonal naïve, ARIMA, Prophet).

**Why**
- Creates a strong reference point (“minimum viable performance”)
- Captures store-specific seasonality
- Highly interpretable and easy to validate

---

### 4.2 Stage 2 — Global forecasting model (shared learning across stores)
Train a single forecasting model across all stores, while allowing store-specific adaptation using store metadata or store embeddings.

**Why**
- Learns shared patterns across locations (weekday effects, holiday seasonality, promo sensitivity)
- Improves performance for stores with limited history
- More scalable than maintaining one model per store

Possible model families here include:
- Gradient-boosted trees (e.g., LightGBM) with lag/rolling features
- Neural forecasting models (optional, if needed later)

---

### 4.3 Stage 3 — Spike detection layer (anomaly detection)
Add a spike detection mechanism on top of the forecasting system.

Two practical options:
- **Residual-based spike detection:** flag days where actual demand is far above forecast (irst forecast demand, then flag a spike if actual − forecast is unusually large.)
- **Dedicated anomaly model:** directly predicts whether a spike event is likely, a separate model whose direct job is to predict “spike / no spike” (without relying only on forecast errors).

**Why**
- Forecasting models optimize average error and may miss rare spikes
- A dedicated spike layer aligns directly with the business pain (stockouts, staffing issues)

---

### 4.4 Not appropriate approaches (initially)
- Pure supervised spike classification from day 1 without a strong, business-aligned spike definition
- A single “generic” forecasting model that ignores store differences (store-level heterogeneity), which will likely average behaviors and underperform

---

**Deliverable — approach selection**
Start by defining spike labels and validating per-store forecasting baselines. Then train a global forecasting model that shares learning across stores while adapting per store. Finally, add a spike detection layer (based on forecast residuals or anomaly detection) to specifically catch rare but operationally critical demand spikes.


---

## 5) Use of LLMs (reflection)

- How I used an LLM: to explore how “demand spikes” is typically framed in ML and what alternative approaches exist beyond basic forecasting.
- Where it helped: suggested combining forecasting + anomaly detection rather than treating this as only forecasting.
- Where I challenged/refined it: I rejected generic “classification” framing because labels do not exist and spike definitions must be aligned with operational decisions and stockout risk.

I used an LLM as a thinking partner to clarify the correct ML framing for this scenario and to explore solution approaches beyond standard supervised learning. It helped me identify that this problem is not only a forecasting task, but also a spike/anomaly detection problem, because rare demand spikes are the main operational pain point.

The LLM was especially helpful in structuring the scoping document (problem clarification → problem type → data scope → modeling approach) and in introducing concepts I had not used before, such as hierarchical forecasting and representation learning for store similarity. It also helped me reason about time-series data modality, which I have not worked with deeply so far in a predictive setting (I previously encountered time-related analysis mainly in EDA, but not as a full forecasting pipeline).

I also challenged and refined several parts of the LLM output. For example that spike labels must be defined in a way that matches real business impact (e.g., stockouts or staffing overload), not only statistical outliers. I also corrected ambiguity around promotions and leakage by distinguishing between promo information that is known at prediction time (planned campaigns, which are valid features) versus promo indicators that are only known after the fact (which would cause leakage). Finally, I improved the modeling section by rewriting it into a clear staged approach, separating baselines, the global forecasting model, and the spike detection layer.

---

## 6) Unknown methods discovery

### New methods discovered

**1) Hierarchical forecasting**
- **Why relevant:** Retail demand often needs to be forecasted at multiple levels (store, region, total company), and those forecasts must be consistent for planning.
- **Explanation in my own words:** Hierarchical forecasting ensures that predictions at different aggregation levels do not contradict each other (e.g., store-level forecasts add up to the regional forecast).

**2) Representation learning / store embeddings**
- **Why relevant:** Stores differ widely, but many stores behave similarly. Learning store representations allows a global model to share patterns across stores while still adapting per store.
- **Explanation in my own words:** Store embeddings are like learned “store fingerprints” that capture hidden similarities between stores, so the model can generalize better, especially for stores with limited history.

**3) Rolling statistics (rolling mean / rolling standard deviation)**
- **Why relevant:** Rolling statistics are a practical way to define and detect spikes relative to a store’s recent baseline and to create time-series features without losing the time structure.
- **Explanation in my own words:** Rolling means a moving window over recent days (e.g., last 7 days) that updates over time, so we can compare today’s demand to what has been normal recently.

**4) Forecast horizon (linked to supply chain lead time)**
- **Why relevant:** The forecast is only useful if it predicts far enough ahead for the business to take action (e.g., replenishment).
- **Explanation in my own words:** Forecast horizon is how far into the future we predict (e.g., 1–14 days), and it must match how long it takes the business to react.

---

### Incorrect assumptions I initially had (and how they were corrected)

**Assumption 1: “Spike detection is just a separate third approach.”**  
- **Correction:** Spike detection is best treated as a layer that can sit on top of forecasting (either per-store baselines or a global forecasting model). It is not necessarily a completely separate standalone pipeline.

**Assumption 2: “The global model will learn from the per-store baseline models.”**  
- **Correction:** Per-store baselines are mainly for benchmarking, interpretability, and understanding store-level behavior. The global model is trained directly on the data (with engineered features), not on the baseline model outputs.

**Assumption 3: “If we define spike labels, the problem is automatically supervised from day one.”**  
- **Correction:** While defining a spike label enables supervised learning, the spike definition itself is non-trivial and must be validated against real operational impact (stockouts, staffing overload). Without that validation, a supervised classifier may optimize the wrong target.

**Assumption 4: “Granularity is just the unit of time.”**  
- **Correction:** Granularity also includes the level of aggregation (store-level vs SKU-level), which changes the complexity of modeling and the ability to prevent stockouts.
