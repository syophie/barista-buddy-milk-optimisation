# Barista Buddy — Ticket Triage & Milk Optimisation

A decision model that **selectively intervenes** in a coffee queue to reduce **milk waste** while preserving **drink quality**, triggered by real‑time demand conditions.

The purpose of this projects was to formalises triage intuition into a simple, testable algorithm: 

- Translating an observed operational bottleneck into a decision model
- Designing demand‑based intervention triggers
- Framing a multi‑objective optimisation problem (waste vs quality)
- Building reproducible modelling artefacts from first principles

---

## Background

While waiting for coffee, I noticed two near‑identical orders arrive seconds apart — yet one took substantially longer to be called out.  
The only meaningful difference was **milk type: oat vs cows**.

That observation raised a question:

> How do baristas implicitly triage tickets when milk is the binding constraint?


---

## Problem statement

In peak periods, baristas manage a stream of drink *tickets* that compete for constrained milk handling capacity (pouring, steaming, freshness windows):

- Orders arrive continuously
- System conditions change dynamically
- Only a subset of tickets benefit from intervention

The goal is to design a **selective triage system** that improves outcomes **without disrupting normal service**.

---

## Core idea: selective triage (not blanket optimisation)

### Demand trigger: only triage when busy

Let `ticket_t` be the timestamp of order `t`.  
Define average inter‑arrival time over the last `k` tickets:

\[
\text{AvgGap}_t = \text{Avg}(ticket_t - ticket_{t-1})
\]

Compare this to an acceptable service threshold (`delivery_time`):

- If `AvgGap_t < delivery_time` → demand is high → **triage**
- If `AvgGap_t ≥ delivery_time` → demand is stable → **no triage**

`delivery_time` can be:
- a fixed threshold, or
- updated dynamically using historical ticket data.

---

## What “triage” means here

Triage is **not reordering the queue indiscriminately**.  
It triggers a constrained optimisation decision around milk usage.

### Optimisation objectives (multi‑objective)

This model does **not** directly optimise for speed.  
Speed acts as a **trigger and constraint**, not the objective.

**Minimise**
- Milk waste (by milk type / by person)

**Maximise**
- Milk quality
  - freshness
  - avoiding over‑stretching / under‑filling

**Subject to**
- Order integrity (each drink gets what it requires)
- Acceptable service expectations (“delivery time”)

---

## Milk requirement mapping

Each drink maps to an expected milk requirement (oz):

| Drink type   | Milk required |
|-------------|---------------|
| Flat white  | X oz          |
| Latte       | X oz          |
| Cappuccino  | X oz          |
| Long black  | 0 oz          |

This translates a set of orders into a **milk demand profile** by milk type at a given point in the queue.

---

## Decision loop

Ticket‑based decision flow:

1. **Order arrives** (ticket)
2. **Log order** into dataset
3. **Evaluate demand trigger**
4. **Decision**
   - **No triage** → standard queue processing
   - **Triage** →
     - recommend milk volume to prep/use
     - identify which orders can be fulfilled with that milk
     - update backlog by removing processed items

---

## Outputs

The model produces:

- Suggested ounces of milk to prepare (by milk type)
- List of coffee orders that fit that allocation
- Updated backlog with triaged items removed

Formally:

\[
y = \{ \text{orders filled},\ \text{milk oz suggested} \}
\]

---

## Repository structure (recommended)
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

- Uses synthetic or simplified ticket data
- Milk to drink mappings are stylised
- Does not model barista‑specific workflow constraints (yet)
- Optimisation logic starts heuristic‑based before formal solvers

---

## Future extensions

- Learn `delivery_time` dynamically from queue history
- Explicit upper bounds on milk prep to preserve freshness
- Add additional constraints (e.g. espresso shot timing)
- Extend to multi‑station or multi‑milk scenarios

