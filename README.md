# Barista Buddy — Ticket Triage & Milk Optimisation

A decision model that selectively intervenes in a coffee queue to reduce milk waste while preserving drink quality, triggered by real‑time demand conditions.

The purpose of this project is to formalise an observed triage intuition into a simple, testable algorithm:

- Translate an operational bottleneck into a decision model
- Define demand‑based intervention triggers (when *not* to intervene)
- Frame a multi‑objective optimisation problem (waste vs quality)
- Build reproducible modelling artefacts from first principles

---

## Background

While waiting for coffee, I noticed two near‑identical orders arrive seconds apart — yet one took noticeably longer to be called out.  
The only meaningful difference was the milk type: **oat vs cows**.

This raised a simple question:

> How do baristas implicitly triage tickets when milk is the binding constraint?

---

## Problem statement

During peak periods, baristas handle a stream of drink *tickets* that compete for constrained milk handling capacity (pouring, steaming, freshness windows).

Key characteristics:

- Orders arrive continuously
- System conditions shift quickly
- Only some tickets benefit from intervention

The goal is to design a **selective triage system** that improves outcomes **without disrupting standard service**.

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
- Milk quality
  - freshness
  - avoiding over‑stretching or under‑filling

**Subject to**
- Order integrity (each drink still receives what it requires)
- Acceptable service expectations (`delivery_time`)

---

## Milk requirement mapping

Each drink type maps to an expected milk requirement (oz):

| Drink type   | Milk required |
|-------------|---------------|
| Flat white  | X oz          |
| Latte       | X oz          |
| Cappuccino | X oz          |
| Long black | 0 oz          |

This converts a list of orders into a **milk demand profile** at a given point in the queue.

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
y = \{ \text{orders filled},\ \text{milk oz suggested} \}
\]

---

## Repository structure
barista-buddy-milk-optimisation/
├── README.md
├── src/
│   ├── trigger.py
│   ├── triage.py
│   ├── optimiser.py
│   └── utils.py
├── data/
│   └── sample_orders.csv
├── notebooks/
│   └── exploration.ipynb
├── outputs/
│   └── example_results.csv
└── requirements.txt

---

## Assumptions & limitations

- Ticket data is synthetic / simplified
- Milk‑to‑drink mappings are stylised
- Barista‑specific workflow constraints are not explicitly modelled
- Optimisation begins as heuristic‑based before formal solvers

---

## Future extensions

- Learn `delivery_time` dynamically from queue history
- Explicit bounds on milk prep to preserve freshness
- Additional constraints (e.g. espresso shot timing)
- Extension to multi‑station or multi‑milk setups

