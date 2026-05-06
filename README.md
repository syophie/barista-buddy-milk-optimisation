# Barista Buddy — Ticket Triage & Milk Optimisation

A decision model that selectively intervenes in a coffee queue to reduce milk waste while preserving drink quality, triggered by real‑time demand conditions. Most coffee-shop optimisation tools focus on throughput and speed, not explicitly on waste reduction and quality such as the proposed, co-primary objectives of Barista Buddy.

The model defines two optimisation objectives — minimise milk waste (by milk type, by site) and maximise milk quality (freshness, correct volume, no over-stretching) — subject to order integrity and acceptable delivery time constraints. 
- Speed is treated as a constraint or trigger, not the objective itself.
- The decision loop proceeds: order enters → logged → triaged (yes/no) → if yes, adjust milk allocation, re-route order, or remove from backlog; if no, serve via standard process.
- The output is a suggested ounces figure and a drink list to fill.

---

## How it came to be?

While waiting for coffee, I noticed two near‑identical orders arrive seconds apart — yet one took noticeably longer to be called out.  The only meaningful difference was the milk type: **oat vs cow**.

This raised a simple question:

> How do baristas implicitly triage tickets when milk is the binding constraint?

The purpose of this project is to formalise a triage algorithm designed to decide *when* and *how* to intervene in a coffee queue to reduce milk waste without harming drink quality or service time. 

To begin, I plan to model a simple, testable algorithm. Not every order warrants intervention; the system should act only when conditions justify it.
- Translate an operational bottleneck into a decision model
- Define demand‑based intervention triggers (when *not* to intervene)
- Frame a multi‑objective optimisation problem (waste vs quality)
- Build reproducible modelling artefacts from first principles

---

## Problem statement

During peak periods, baristas handle a stream of drink *tickets* that compete for constrained milk handling capacity (pouring, steaming, freshness windows). 

Coffee shops routinely waste 6–25% of their milk through over-steaming, pitcher leftovers, and imprecise portioning. For a moderately busy shop using 10 litres per day, even a 10% waste rate equates to roughly £436.80 lost annually; at higher waste rates (15–20%), the true cost is significantly greater. In a high-volume Sydney café consuming 105 litres of milk daily, measured waste reached 6.3 litres per day — AU$5,733 per year.

The goal is to design a **selective triage system** that reduces milk waste and retains coffee quality **without disrupting standard service**.

---

## Core idea: selective triage (not blanket optimisation)

### Demand trigger: only intervene when it matters

Let `ticket_t` be the timestamp of order `t`.  
Define the average inter‑arrival time over the last `k` tickets:

\[
\text{AvgGap}_t = \text{Avg}(ticket_t - ticket_{t-1})
\]

Compare this to an acceptable service threshold (`delivery_time`):

- If `AvgGap_t < delivery_time` → demand is high → **triage**
- If `AvgGap_t ≥ delivery_time` → demand is stable → **do nothing**

`delivery_time` can be:
- fixed (simple baseline), or
- updated dynamically using recent ticket history.

The intention is explicitly *not* to optimise continuously.

---

## What “triage” means here

Triage is **not reordering the queue indiscriminately**.  
It triggers a constrained optimisation decision focused on milk usage.

### Optimisation objectives

This model does **not** optimise for speed directly.  
Speed is treated as a **trigger and constraint**, not the objective.

**Minimise**
- Milk waste (by milk type / by barista)

**Maximise**
- Milk quality (by freshness, avoiding over‑stretching or under‑filling)

**Subject to**
- Order integrity (each drink still receives what it requires)
- Acceptable service expectations (`delivery_time`)

---

## Milk requirement mapping

Each drink type maps to an expected milk requirement (oz and Type):

## Drink to milk-volume mapping (waste contraints)

| Drink order   | Milk required (oz) | Notes |
|--------------|-------------------|-------|
| Flat white   | 4 oz              | Steamed milk with minimal foam; smaller volume, tighter quality constraint |
| Latte        | 8 oz              | Larger milk volume; higher waste risk during busy periods |
| Cappuccino   | 6 oz              | Moderate milk volume with foam component |
| Long black   | 0 oz              | No milk required; excluded from milk optimisation |

## Drink to milk‑type mapping (quality constraints)

| Milk type        | Quality considerations | Implications for optimisation |
|------------------|------------------------|--------------------------------|
| Full‑fat cow     | Most forgiving texture and stretch; stable foam; tolerates small timing variation | Lowest quality risk; suitable baseline for batching heuristics |
| Semi‑skimmed cow | Less elastic foam; narrower temperature sweet spot | Tighter timing constraint than full‑fat; avoid delayed pours |
| Skimmed cow      | Weak foam structure; prone to overheating | High quality sensitivity; batching discouraged |
| Oat              | Highly temperature‑sensitive; foam collapses quickly; separates if over‑heated | Very narrow window between steaming and pour; no re‑use; strong candidate for end‑of‑bottle optimisation |
| Almond           | Poor foam stability; scorches easily | Avoid re‑steaming and long holds; minimise partial pours |
| Soy              | Can curdle at high temperature or with acidic espresso | Strict upper temperature bound; sequencing must respect shot timing |
| Coconut          | Low protein → unstable foam; flavour dominance | Quality degradation dominates waste reduction; batching rarely justified |
| Lactose‑free cow | Slightly sweeter; foams faster than standard dairy | Faster steaming requires tighter coordination with shot timing |

---

## Decision loop

Ticket‑based flow:

1. **Order arrives** (ticket)
2. **Log order** into dataset
3. **Evaluate demand trigger**
4. **Decision**
   - **No triage** → process as normal
   - **Triage** →
     - suggest milk volume to prepare
     - identify which orders can be fulfilled with that milk
     - update backlog by removing filled items

---

## Outputs

The model returns:

- Suggested ounces of milk to prepare (by milk type)
- List of coffee orders that fit that allocation
- Updated backlog with processed items removed

Formally:
\[
y = \left( \text{orders\_filled},\ \text{milk\_oz\_suggested} \right)
\]

---

## Repository structure
Repo: barista-buddy-milk-optimisation/
### Table.Folders and usage

| Path / File | What it is | Typical usage |
|------------|------------|---------------|
| `barista-buddy-milk-optimisation/` | Project root | Top-level folder for the whole project. Organises code, data, outputs, and documentation. |
| `README.md` | Project overview | Explains what the project does, how to run it, assumptions, inputs/outputs, and setup instructions. |
| `requirements.txt` | Dependencies list | Lists required Python packages. Installed via `pip install -r requirements.txt`. |
| `src/` | Source code | Contains the core Python logic for the project. |
| `src/trigger.py` | Entry-point / runner | Main script to run the full pipeline end-to-end (load data → optimise → save results). |
| `src/triage.py` | Pre-processing & rules | Handles validations, filtering, and decision logic before optimisation runs. |
| `src/optimiser.py` | Optimisation logic | Core algorithm that performs milk optimisation (e.g. minimising waste, meeting demand). |
| `src/utils.py` | Helper functions | Shared utility functions (data loading, formatting, common calculations). |
| `data/` | Input data | Stores raw or sample input datasets. Not modified by the code. |
| `data/sample_orders.csv` | Sample input data | Example dataset used for testing, demos, or development. |
| `notebooks/` | Exploration & analysis | Jupyter notebooks for exploratory analysis and prototyping. |
| `notebooks/exploration.ipynb` | EDA notebook | Used to understand data patterns and test ideas before finalising logic. |
| `outputs/` | Generated outputs | Stores results produced by the optimiser. |
| `outputs/example_results.csv` | Example output | Sample output showing what the optimiser produces. |

---

## Assumptions & limitations

- Ticket data is synthetic / simplified
- Milk‑to‑drink mappings are stylised
- Barista‑specific workflow constraints are not explicitly modelled
- Optimisation begins as heuristic‑based before formal solvers

---

# Future extensions

1. **Learning delivery-time distributions from data**  ### Conceptual extensions currently absent or underspecified
   Dynamically infer `delivery_time` from historical queue behaviour rather than treating it as a fixed parameter.  
   *Approach:* unsupervised or weakly supervised modelling over order-level timestamps (e.g. clustering or density estimation over observed preparation times).

2. **Hard physical constraints on milk preparation (freshness & re‑steaming)**  
   Explicitly encode the physical constraints of milk handling:
   - Milk allocated to one drink and not used **cannot** be rolled into the next.
   - The algorithm must operate within a very narrow window between *order placed* and *milk steamed*.
   - Re‑steaming milk is disallowed, as it degrades texture and introduces hygiene risks. Re‑using steamed milk “ruins the texture of the coffee” and risks losing customers.  
   
   These constraints are necessary to uphold the *quality maximisation* objective and should be treated as hard bounds, not soft penalties.

3. **Additional process constraints (e.g. espresso shot timing)**  
   Extend the constraint set to include espresso-specific timing rules (shot ageing, synchronisation with milk steaming, machine availability), ensuring optimisation does not violate established quality thresholds.

4. **Multi‑station and multi‑milk setups**  
   Generalise the model to environments with:
   - Multiple espresso machines or milk stations
   - Parallel baristas with heterogeneous skill levels
   - Multiple milk fridges or dispensers  
   
   This introduces routing and assignment decisions in addition to simple queue triage.

5. **Alternative‑milk economics**  
   Incorporate milk‑type–specific cost structures. Non‑dairy milks (oat, almond, soy) often cost ~2× dairy per litre, making waste disproportionately expensive.  
   
   The model’s value is therefore highest in shops with high alternative‑milk penetration, yet the current framing treats all milk symmetrically.  
   *Illustrative benchmark:* in Starbucks’ AI‑enabled inventory system, oat milk priced at USD 4–5 per carton saw 25,000–30,000 fewer cartons wasted annually, yielding USD 100,000–150,000 in chain‑wide savings on that ingredient alone.

6. **Demand forecasting as a complementary layer**  
   Add a short‑horizon demand forecaster (15–30 minutes) using time‑of‑day, day‑of‑week, and weather signals to enable proactive decisions, such as:
   - Avoiding opening a new bottle of oat milk near shift end
   - Adjusting triage aggressiveness ahead of predicted lulls or surges  
   
   *Existing concepts:*  
   - BaristaIQ includes a demand‑forecasting module at ~15‑minute horizons.  
   - Starbucks’ inventory AI similarly uses predictive analytics to minimise over‑ordering and expiry‑driven waste.

---

## Barriers to scaling and productionisation

1. **Staff turnover and training burden**  
   Hospitality has extreme churn, implying that any system requiring non‑trivial training will face constant re‑onboarding.
   - US hospitality turnover exceeds 70% (Bureau of Labor Statistics).
   - In the UK, ~6% of hospitality workers quit every month.
   - Nearly 3 million US hospitality workers left their jobs in early 2024 (≈204% above the national average quit rate).  
   
   *Implication:* the interface must be near‑zero‑learning‑curve and function as decision support rather than prescriptive automation.

2. **Regulatory and food‑safety constraints**  
   Milk handling is subject to strict regulations (temperature control, shelf life, contamination prevention). Any recommendation to hold, delay, or reroute milk must respect these rules.  
   The current approach would benefit from a configurable regulatory layer that can be adapted by region or operator.

3. **POS system integration**  
   The model assumes access to a live order stream. In practice, this requires integration with Point‑of‑Sale systems (e.g. Square, Lightspeed, Toast, or bespoke enterprise stacks).  
   Each integration introduces engineering cost, maintenance overhead, and potential partnership dependency.

4. **Uncertain marginal benefit vs. existing solutions**  
   Software‑only queue optimisation must be demonstrably additive given that:
   - Hardware solutions (e.g. Übermilk) can reduce milk waste from ~6.3 L/day to ~0.2 L/day.
   - Low‑tech practices (training, pre‑measured jugs) can already cut waste by ~5–20% at minimal cost.  
   
   The value proposition must clearly exceed these baselines or complement them.

5. **Need for rigorous value measurement**  
   The largest uncertainty is *how often* triage‑eligible conditions actually arise in real order streams.
   - If same‑milk‑type clustering occurs in only ~5% of tickets, absolute savings may be modest regardless of algorithm sophistication.  
   
   *Required next step:*  
   - Run data‑driven pilots using real order logs.
   - Simulate intervention frequency and theoretical waste reduction.
   - Validate results in live trials.  
   
   If validated, the most viable commercial paths are likely POS‑level integration or B2B licensing rather than direct‑to‑café sales. Sustainability and ESG alignment provide a strong narrative, but adoption will ultimately depend on hard, quantified savings.

   6. **Barista barriers to adoption**
### Barriers to Adoption| Barrier | Severity | Mitigation |
|---|---|---|
| POS integration complexity | High | Partner with 1–2 dominant POS providers (e.g. Square, Toast) for initial plug‑in development. |
| Barista trust and UX | High | Design as a *suggestion engine* with clear, explainable rationale (e.g. “same milk type as previous order — combine?”), not a mandatory override; user‑test with baristas before launch. |
| Proving ROI to operators | Medium–High | Run controlled pilots measuring milk waste before/after; publish results openly to build credibility. |
| Low‑tech alternatives suffice | Medium | Position as complementary to training and portion control, not a substitute; target operators who have implemented basics but still see residual waste. |
| Regulatory food‑safety review | Medium | Ensure the system never recommends holding milk beyond safe temperature windows; document compliance and constraints. |
| “Good enough” hardware competition | Medium | Target segments where hardware is impractical (small independents, pop‑ups, workplace hospitality) or position as a software layer that augments existing hardware solutions. |


---

# Glossary
## Model & language glossary

| Term | What it means in this project |
|---|---|
| **Decision model** | A set of rules that decides *whether* to intervene in the queue and *what to do* if intervention happens. |
| **Selective intervention** | The system often does nothing; it only steps in when demand is high enough to justify it. |
| **Triage** | Deciding whether a given order (ticket) should receive special handling under current conditions. |
| **Ticket** | A single coffee order treated as a unit of work in a queue. |
| **Operational bottleneck** | Milk handling (type, volume, freshness), not espresso shots, is the limiting factor. |
| **Binding constraint** | Milk is the resource constraining throughput at busy times. |
| **Demand conditions** | How quickly orders are arriving at the bar. |
| **Demand trigger** | A rule that switches triage on or off based on recent order arrival rates. |
| **Average inter‑arrival time** | The average time gap between recent orders; used as a simple demand signal. |
| **Delivery time threshold** | A tolerance for how slow service can get before intervention is triggered. |
| **Baseline / do nothing** | Standard queue processing with no optimisation applied. |
| **Selective triage system** | A system designed to intervene sparingly rather than continuously optimising. |
| **Constrained optimisation** | Improving outcomes while respecting hard limits (quality, service expectations). |
| **Multi‑objective** | Balancing more than one goal — here, waste reduction and drink quality. |
| **Minimise milk waste** | Reduce leftover or unusable milk across orders. |
| **Maximise milk quality** | Preserve freshness and correct volume; avoid over‑stretching or under‑filling. |
| **Speed as a constraint** | Speed matters, but the model does not directly optimise for it. |
| **Order integrity** | Every drink still receives the correct ingredients and quantities. |
| **Milk requirement mapping** | A lookup from drink type to expected milk volume (oz). |
| **Milk demand profile** | Total milk required by a set of orders at a given point in the queue. |
| **Decision loop** | The repeated process applied each time an order arrives. |
| **Backlog** | Orders waiting to be processed. |
| **Heuristic‑based optimisation** | Starting with simple, interpretable rules rather than complex solvers. |
| **Formal solvers** | More mathematically intensive optimisation methods that could be added later. |
| **Synthetic data** | Generated but realistic order data used for testing the model. |
| **Reproducible artefacts** | Code and data that allow someone else to run and inspect the model. |
| **Future extensions** | Ideas intentionally left out to keep the initial model focused. |


## Real-World Working Examples:
Several existing products and projects address parts of your problem space. None replicates the exact concept of *queue-level milk triage*, but each demonstrates relevant capabilities.
