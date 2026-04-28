# Customer Support Ticket Analysis — Project 1

**Operational insight, built on data.**
Crystal Olisa · Operations Generalist · [LinkedIn](https://linkedin.com/in/crystalolisa) · [Portfolio](https://github.com/crystalolisa)

---

## The business problem

A support queue generates data every day. Most teams read it by volume — the busiest category gets the attention, the slowest month gets the review. Volume is an indicator. Resolution time is the cost. Reading only one gives you half the picture.

This project analyses a customer support dataset to demonstrate what becomes visible when you read both — and what remains invisible when your classification system was built around how tickets were filed rather than what actually happened to the customer.

---

## Dataset

**Source:** [Kaggle — Customer Support Ticket Dataset by suraj520](https://www.kaggle.com/datasets/suraj520/customer-support-ticket-dataset)

8,469 tickets. Five categories: Refund request, Technical issue, Billing inquiry, Product inquiry, Cancellation request. Synthetic but structurally realistic. Analysis runs on 2,769 closed tickets — open and pending tickets excluded from resolution time calculations.

Note on the synthetic structure: the near-uniform volume distribution across all five categories is itself an analytical signal. Real support queues cluster. Everyday frictions dominate. The flat distribution means the volume chart is not where the insight lives.

---

## Findings

### 1. Volume is not cost — Rank Shift reveals the gap

Cancellation requests had the lowest volume in the closed subset (516 of 2,769) but the most disproportionate cost relative to that volume. Rank Shift measures the distance between where a category sits in the volume ranking and where it sits in the resolution time ranking. Positive means more expensive than volume predicts. Negative means more cost efficient.

![Total Resolution Hours by Rank Shift](charts/chart_1_rank_shift.png)

| Ticket Type | Count | Total Hours | Rank Shift |
|---|---|---|---|
| **Cancellation request** | **516** | **3,970** | **+3** |
| Product inquiry | 533 | 4,091 | +1 |
| Refund request | 596 | 4,838 | 0 |
| Technical issue | 580 | 4,272 | -2 |
| Billing inquiry | 544 | 3,814 | -2 |

Billing inquiry and Technical issue both sit at -2 — more cost efficient than volume predicts. That is not an accident. These are the categories where the cost of a bad resolution is immediately visible. A billing error costs money. A technical failure stops the product working. Process investment follows visible pain. The gap in Cancellation requests is the natural result of triage, not negligence. A team acting on volume alone would deprioritise the most expensive unresolved process gap in the queue.

Translating to headcount: at 1,800 productive hours per FTE per year, the combined Cancellation and Refund categories consume the equivalent of approximately 4.9 full-time employees. At a loaded support cost of $60k–$80k per FTE, that is a $290k–$390k annual line item — before accounting for the acquisition cost of replacing the customers being lost. We are paying more to lose a customer than to support a paying one.

*Assumption: 1,800 productive hours per FTE accounts for 52 weeks at 40 hours, minus approximately 10% for leave, meetings, and overhead. $60k–$80k reflects a fully loaded support FTE including salary, benefits, and tooling. Adjust both figures to your own cost structure.*

---

### 2. The deflection finding — not all queue volume belongs in the queue

Reading the description field rather than the category label reveals how much inbound volume could have been resolved before it reached the team. A customer who cannot find where to change their payment method files a Billing inquiry ticket — technically correct, operationally avoidable. A persistent account menu link would have deflected a significant proportion of that category's volume. That is a product navigation decision surfaced by reading support data. One function reading its own data created a decision in another function — not by overstepping, but by being legible enough to pass the finding across.

Deflection rate — the proportion of inbound tickets resolvable without team intervention — cannot be calculated until the taxonomy exists to classify deflectable ticket types separately.

---

### 3. The taxonomy trap — two categories may be measuring the same customer exit twice

Cancellation requests (516 tickets, 3,970 hours) and Refund requests (596 tickets, 4,838 hours) sit as separate categories in the dataset. In a subscription model, cancellation stops future billing and refund recovers past billing — sequential stages of the same customer exit event. If 30–50% of cancellation tickets generate a subsequent refund request, a reasonable estimate for subscription businesses, neither metric reflects the true cost of customer exit.

The taxonomy was built around what agents filed, not what the customer journey looks like. The handoff between cancellation and refund has no owner in the current classification system because the taxonomy never named that moment as something that needs to be managed. Every tool configured, every routing rule written, every satisfaction metric built on top of this taxonomy is measuring a fragment of a larger event. The true exit journey time — from first contact to final resolution — does not exist as a metric in the current system.

Three implications follow directly from this finding. First, the CRM taxonomy needs to move from ticket-based classification to event-based tagging — where a customer exit is a single named event carrying both a cancellation tag and a refund tag, with a shared identifier that allows the full journey cost to be measured as one thing. Second, the cancellation workflow is the immediate audit target: 3,970 hours on a process with no documented resolution path is a workflow automation candidate before it is a headcount problem. Third, support QA should pivot from speed to first response toward resolution accuracy — the satisfaction data shows that speed is not what is moving the satisfaction score, and investing in it further is a low-return move until the taxonomy is fixed.

---

### 4. Resolution time and satisfaction are functionally uncorrelated

Correlation between resolution time and customer satisfaction: −0.0035. Functionally zero. In this dataset, the data does not support the assumption that faster resolution produces higher satisfaction.

| Resolution bucket | Count | Avg satisfaction |
|---|---|---|
| 0–3 hrs | 352 | 2.997 |
| 3–6 hrs | 316 | 2.997 |
| **6–12 hrs** | **414** | **3.126** |
| 12–24 hrs | 320 | 2.975 |

![Satisfaction correlation scatter plot](charts/chart_2_satisfaction_scatter.png)

The 6–12 hour bucket scored highest. The fastest bucket scored lowest. Speed did not produce better outcomes. Satisfaction scores across all five categories span less than 0.1 points — resolution time is not moving that number.

If speed and satisfaction are uncorrelated, every investment made in reducing handle time has had no measurable effect on customer experience in this dataset. That changes where you invest next. The operational cost problem and the customer experience problem are not the same problem. They require separate investigations.

Refund requests had the lowest satisfaction score (2.935). This is consistent with the taxonomy trap finding — those customers may not be dissatisfied because the refund was slow. They may be dissatisfied because they already went through the cancellation process the data treats as a separate event. The satisfaction score is measuring the end of a journey that the taxonomy treats as a beginning.

---

## Analytical decisions

Two decisions shaped the analysis and are documented here because the headline numbers depend on them.

**Negative resolution hours excluded (1,365 records).** 1,365 of 2,769 closed tickets produced negative resolution hours — a timestamp anomaly in the synthetic dataset where Time to Resolution was recorded before First Response Time. These records were excluded to ensure the cost of churn calculation was based on valid operational timestamps, preventing artificial inflation of the margin figures. The clean dataset (df_clean, 1,404 records) is used for all resolution time and Rank Shift calculations. This is a judgement call about data quality, not an outlier removal. In a real dataset, negative resolution times would require investigation into the ticketing system's timestamp logic before any exclusion decision.

**Mixed population analysis.** Volume counts use the full closed dataset (df_closed, 2,769 records). Resolution time calculations use the clean dataset (df_clean, 1,404 records). Satisfaction scores use the full closed dataset — all closed tickets carry a rating regardless of timestamp quality. The two populations are merged on Ticket Type for the Rank Shift and combined summary tables. This decision is documented in the notebook and the merge logic is validated there.

---

## Methodology

**Rank Shift** is a calculated column: volume rank minus resolution time rank per category. Positive value means more expensive than volume predicts. Negative means more cost efficient. Calculated on closed tickets only.

**FTE conversion** assumes 1,800 productive hours per FTE per year and a loaded cost of $60k–$80k per support FTE. Both figures should be adjusted to the actual cost structure of the business being analysed.

**Resolution time buckets:** 0–3 hrs, 3–6 hrs, 6–12 hrs, 12–24 hrs. Satisfaction correlation calculated across all closed tickets using Pearson r.

**Deflection rate** is defined as the proportion of inbound tickets resolvable without team intervention. Not directly calculable from this dataset without agreed deflectable category definitions — named here as the metric that would exist if the taxonomy were built to capture it.

**Event-based tagging** is proposed as the taxonomy fix for the customer exit blind spot. Rather than filing cancellation and refund as separate ticket types, an event-based system tags both actions as stages of a single named event — customer exit — with a shared identifier that allows the full journey cost to be measured, staffed for, and improved as one thing.

Full methodology, code, and charts in the notebook linked below.

---

## Metric glossary

**Rank Shift** — `Volume Rank − Resolution Time Rank` per category. A positive value indicates a category is more expensive than its volume predicts. A negative value indicates it is more cost efficient. Zero means cost is proportionate to volume.

**Taxonomy trap** — a classification system that is internally consistent but structurally blind to the journey it is supposed to be measuring. Occurs when a single customer journey is fragmented across multiple ticket types, masking the true total cost of that journey.

**Deflection rate** — the proportion of inbound support tickets that could have been resolved before reaching the team, through self-serve infrastructure, better documentation, or product design. Requires a taxonomy that classifies deflectable ticket types separately before it can be calculated.

**Event-based tagging** — a classification approach where a customer action (such as a customer exit) is named as a single event and all associated ticket types carry a shared event tag. Allows total journey cost to be measured across ticket types rather than within them.

---

## Repo structure

```
project_1/
├── README.md
├── notebooks/
│   └── ticket_analysis_final.ipynb
├── data/
│   └── customer_support_tickets.csv
└── charts/
    ├── chart_1_rank_shift.png
    ├── chart_2_satisfaction_scatter.png
    └── chart_3_satisfaction_by_bucket.png
```

---

## Key outputs

- Total resolution hours by category coloured by Rank Shift (Chart 1 — cited in Post 6)
- Satisfaction correlation scatter plot (r = −0.0035)
- Satisfaction by resolution time bucket
- FTE cost translation of combined exit-category hours

---

## Notebook

[Project 1 Notebook →](notebooks/ticket_analysis_final.ipynb)

---

## Related posts

- [Post 4 — What the volume tells you](link)
- [Post 5 — What the description field is telling you](link)
- [Post 6 — The Cancellation finding](link)
- [Post 6b — The taxonomy trap](link)
- [Post 7 — Resolution time does not predict satisfaction](link)

---

*The sprint does not fix the operation. It makes fixing possible.*
