# Manual Review Cases
## D2C Customer Churn Capstone — Part 2

These are customers where the automated segment assignment is technically correct
but the full signal profile creates genuine ambiguity about the right retention
action. Each case is drawn directly from `segments.csv` and the executed notebook.

---

## Case 1 — CUST00004 | Segment: At-Risk

| Signal | Value |
|---|---|
| R / F / M scores | 2 / 2 / 4 |
| Recency | 131 days |
| Frequency (180d) | 1 order |
| Monetary (180d) | ₹1,604 |
| Lifetime spend | ₹1,604 (same — only ever one order) |
| Return rate | 0% |
| Avg discount | 16% |
| Sessions (30d) | 1 |
| Last visit | 14 days ago |
| Tickets | 0 |
| Loyalty tier | Not enrolled |
| Last campaign | Free shipping |
| Manual priority | High |
| Churn label | 1 (will churn) |

**Why this is ambiguous:** The M=4 score signals a high-value buyer (₹1,604 in a single order), and the 131-day recency earns At-Risk correctly. But the lifetime spend is identical to the 180d spend — this customer has made exactly one purchase, ever. That makes the ₹1,604 a single high-value transaction from a new-ish customer (168 days since signup), not evidence of a recurring relationship. The At-Risk label implies a formerly active customer going quiet, but this is actually a customer who bought once and never came back.

**Decision:** Do not treat this as a standard At-Risk winback. The playbook for At-Risk assumes historical loyalty that needs to be reactivated — that relationship doesn't exist here. Instead, treat as a lapsed first-time buyer: send a "complete your [preferred_category] routine" email with a moderate 15% off to encourage a second purchase. The low discount history (16%) suggests they are not a discount hunter. One visit 14 days ago is a positive signal — they are still aware of the brand.

**Action:** Second-purchase email campaign (Promising playbook, not At-Risk winback).

---

## Case 2 — CUST00057 | Segment: At-Risk

| Signal | Value |
|---|---|
| R / F / M scores | 2 / 2 / 4 |
| Recency | 136 days |
| Frequency (180d) | 1 order |
| Monetary (180d) | ₹1,713 |
| Lifetime spend | ₹2,511 |
| Return rate | 0% |
| Avg discount | 14% |
| Sessions (30d) | 7 |
| Last visit | 29 days ago |
| Tickets | 0 |
| Campaign clicks (30d) | 1 |
| Last campaign | None |
| Manual priority | High |
| Churn label | 1 (will churn) |

**Why this is ambiguous:** Unlike CUST00004, this customer has a meaningful lifetime spend of ₹2,511 (vs ₹1,713 in the window), suggesting they have bought more than once in their history — they are genuinely lapsing, not simply a one-time buyer. More importantly: they clicked a campaign link in the last 30 days despite having received no formal campaign (`last_campaign_received = none`). That click is likely an organic marketing link. It means the customer is not disengaged — they are still interacting with brand communications.

**Decision:** This customer is genuinely At-Risk and the standard winback is appropriate. However, the fact that they already clicked something in the last 30 days suggests the bar for conversion is lower than for a fully disengaged At-Risk customer. Prioritise this customer within the At-Risk batch — send the winback offer first, possibly with a slightly smaller incentive (10% rather than 15%) given they are already showing re-engagement signals.

**Action:** Standard At-Risk winback, but flag as high-conversion-probability within the segment. Reduce discount depth to 10%.

---

## Case 3 — CUST00072 | Segment: At-Risk

| Signal | Value |
|---|---|
| R / F / M scores | 2 / 4 / 4 |
| Recency | 90 days |
| Frequency (180d) | 2 orders |
| Monetary (180d) | ₹1,698 |
| Lifetime spend | ₹2,587 |
| Return rate | 0% |
| Avg discount | 21% |
| Sessions (30d) | 7 |
| Last visit | 26 days ago |
| Tickets | 0 |
| Campaign clicks (30d) | 1 |
| Days since signup | 343 |
| Churn label | 1 (will churn) |

**Why this is ambiguous:** This is the strongest At-Risk case in the manual review — F=4 and M=4 confirm genuine historical engagement, and recency at exactly 90 days is the boundary between "at risk" and "still active." The 90-day gap is long but not catastrophic; 7 sessions in the last 30 days shows the customer is still browsing. Yet the churn label confirms they will not buy in the next 60 days without intervention.

**Decision:** This is a textbook At-Risk case that the standard winback is designed for. The high sessions (7) mean the email is likely to be seen. The low discount history (21%) means a 15% winback offer is attractive without being expected. Zero return rate and zero tickets remove any service-quality concern. This customer should be among the first batch actioned in the At-Risk campaign.

**Action:** Standard At-Risk winback (15% off). High priority within segment. Consider a personalised subject line referencing their preferred category.

---

## Case 4 — CUST00005 | Segment: Loyal Customers

| Signal | Value |
|---|---|
| R / F / M scores | 4 / 5 / 4 |
| Recency | 38 days |
| Frequency (180d) | 3 orders |
| Monetary (180d) | ₹1,782 |
| Avg discount | 48% — **discount_hunter_flag = 1** |
| Sessions (30d) | 18 |
| Tickets | 1 |
| Avg sentiment | **−1.0** (maximum negative) |
| Unhappy flag | 1 |
| Loyalty tier | Gold |
| Campaign clicks (30d) | 1 |
| Churn label | 0 (retained) |

**Why this is ambiguous:** The RFM scores (R=4, F=5, M=4) are excellent — this customer buys frequently and recently. They landed in Loyal Customers because F≥4, M≥3, R≥3 and not qualifying for Champions (unhappy_flag=1 blocked that). But two flags are alarming: a sentiment score of −1.0 (worst possible) and an avg discount of 48% (discount_hunter territory). This customer is very active but deeply unhappy and buying only at large discounts. The retained label (churn=0) may reflect inertia rather than satisfaction.

**Decision:** Do not treat this as a standard Loyal Customer. The −1.0 sentiment demands an acknowledgement before any retention offer is made. Sending a loyalty tier update or a product launch email to this customer without addressing the underlying complaint will feel tone-deaf and may tip them from "retained but unhappy" to "churned." Escalate to the High-Value Unhappy playbook: personal outreach addressing the ticket issue, then a relationship-rebuild offer. Given their Gold tier and 18 sessions in 30 days, they are likely salvageable — but only if the complaint is resolved.

**Action:** Override to High-Value Unhappy playbook. Prioritise ticket resolution, then retention offer. Do not send a discount — this would deepen the discount dependency.

---

## Case 5 — CUST00021 | Segment: Loyal Customers

| Signal | Value |
|---|---|
| R / F / M scores | 3 / 4 / 3 |
| Recency | 77 days |
| Frequency (180d) | 2 orders |
| Monetary (180d) | ₹1,059 |
| Lifetime spend | ₹1,059 (identical — all spend is within 180d) |
| Avg discount | 43.5% — **discount_hunter_flag = 1** |
| Tickets | 1 |
| Avg sentiment | −0.85 |
| Unhappy flag | 1 |
| Loyalty tier | Silver |
| Churn label | **1 (will churn)** |

**Why this is ambiguous:** This customer qualifies for Loyal Customers by RFM rules (F=4, M=3, R=3), but has two compounding negatives: an unhappy flag (sentiment −0.85) and a discount hunter flag (43.5% avg discount). They are a 173-day-old customer whose entire spend history sits within the 180-day window — in other words, they are a relatively new customer who quickly established a high-discount, low-satisfaction pattern. The churn label confirms the model's intuition: this customer will leave.

**Decision:** The Loyal Customers campaign (loyalty tier update, second-purchase nudge) is inappropriate here — it rewards a pattern that is not healthy. The risk of sending a discount campaign is that it further trains the discount dependency without addressing the service complaint. Given their Silver tier, short tenure, and relatively low spend (₹1,059), this customer is not high enough value to justify the High-Value Unhappy playbook either. Recommend: address the support ticket with a brief resolution email, then let natural behaviour determine if they re-engage. Do not invest a full campaign.

**Action:** Service resolution only. Low investment. Flag for re-evaluation in 30 days.

---

## Case 6 — CUST00033 | Segment: Loyal Customers

| Signal | Value |
|---|---|
| R / F / M scores | 5 / 5 / 4 |
| Recency | 12 days |
| Frequency (180d) | 3 orders |
| Monetary (180d) | ₹1,878 |
| Return rate | **33.3%** — high_return_flag = 1 |
| Avg discount | 40.3% — discount_hunter_flag = 1 |
| Tickets | 2 |
| Avg sentiment | −0.94 |
| Unhappy flag | 1 |
| Loyalty tier | Gold |
| Churn label | 0 (retained) |

**Why this is ambiguous:** Near-perfect RFM (R=5, F=5, M=4) with a 12-day recency. This customer is highly active. Yet they carry three compounding risk flags: high return rate (33%), extreme discount dependency (40.3%), and near-maximum negative sentiment (−0.94) across 2 tickets. They were excluded from Champions by the unhappy_flag rule, which was the correct call. The retained label (churn=0) likely reflects very high recent activity — but it does not mean the customer is satisfied.

**Decision:** This customer is a "Champion in behaviour, not in satisfaction." The Gold loyalty tier and recent purchases suggest they value the product range, but the return rate and sentiment suggest product-fit or quality issues that have not been resolved. The correct action is to acknowledge the negative experiences proactively — not to reward the purchase frequency with a generic loyalty message. Recommend: a personal CRM outreach asking for feedback on the returned items, offering a product-fit consultation (e.g. skin type quiz) and a small compensation gesture. This converts a behavioural loyal customer into an emotionally loyal one.

**Action:** Personalised feedback outreach + product-fit consultation offer. Resolve root cause of returns.

---

## Case 7 — CUST00030 | Segment: Loyal Customers

| Signal | Value |
|---|---|
| R / F / M scores | 5 / 5 / 5 |
| Recency | 5 days |
| Frequency (180d) | 5 orders |
| Monetary (180d) | ₹2,820 |
| Lifetime spend | ₹3,436 |
| Return rate | 0% |
| Avg discount | 36.2% |
| Tickets | 2 |
| Avg sentiment | −0.415 |
| Unhappy flag | 1 |
| Loyalty tier | Gold |
| Churn label | 0 (retained) |

**Why this is ambiguous:** This customer would have been the highest-scoring Champion in the dataset (R=5, F=5, M=5, recency = 5 days) but was excluded by the unhappy_flag rule due to two tickets with avg sentiment −0.415. The churn label is 0 — they will be retained. But the concern is not about the next 60 days; it is about the next 6–12 months. A customer with a perfect RFM profile who is registering negative support sentiment is a churner-in-waiting if the issues are not resolved.

**Decision:** Despite being retained, this customer should be treated with the same care as High-Value Unhappy. The goal is to preserve a perfect-RFM customer before frustration escalates. Send a proactive "how was your recent experience" message. Given Gold tier and 5-day recency, they are actively engaged and receptive. Offer a direct line to a senior CRM contact if they want to discuss any issues. Do not send a discount — this customer clearly buys at full frequency regardless.

**Action:** Proactive experience check-in + priority support escalation. No discount needed.

---

## Case 8 — CUST00408 | Segment: Promising

| Signal | Value |
|---|---|
| R / F / M scores | 4 / 2 / 2 |
| Recency | 40 days |
| Frequency (180d) | 1 order |
| Monetary (180d) | ₹729 |
| Lifetime spend | **₹4,156** |
| Sessions (30d) | **0** |
| Last visit | 0 days ago — but sessions = 0 |
| Tickets | 4 (highest in manual review set) |
| Avg sentiment | +0.49 (positive) |
| Discount hunter flag | 0 |
| Loyalty tier | Not enrolled |
| Manual priority | High |
| Churn label | 0 (retained) |

**Why this is ambiguous:** Two things are contradictory here. First, the lifetime spend of ₹4,156 against a 180d monetary of ₹729 means this customer was a very active buyer historically but has dramatically pulled back — they should arguably be in At-Risk, not Promising (they have R=4 which kept them out of At-Risk). Second, they have 4 support tickets — the most in this review set — yet avg sentiment is positive (+0.49). This is unusual: many tickets, but happy with the resolutions. The 0 sessions in 30 days despite a "last visit 0 days ago" may reflect a data quirk (one very brief, unlogged visit).

**Decision:** The Promising playbook (second-purchase trigger) is appropriate in form but needs recalibration in tone. Given the high lifetime spend, this is not a new-ish buyer who needs a first-repeat nudge — it is a lapsed heavy buyer who needs a reactivation offer sized to their historical value. The positive ticket sentiment suggests no unresolved grievance. Treat more like a moderate At-Risk case than a first-repeat Promising case. Send a reactivation email referencing their purchase history ("You've bought X before — back in stock") rather than a generic category email.

**Action:** Reactivation email referencing historical purchases. Offer free shipping (matches last campaign type they responded to). No deep discount needed.

---

## Case 9 — CUST01283 | Segment: Loyal Customers

| Signal | Value |
|---|---|
| R / F / M scores | 4 / 5 / 5 |
| Recency | 40 days |
| Frequency (180d) | — (high) |
| Monetary (180d) | ₹5,651 |
| Lifetime spend | ₹6,697 |
| Sessions (30d) | 7 |
| Last visit | 0 days ago |
| Tickets | 3 |
| Avg sentiment | −0.67 |
| Unhappy flag | 1 |
| Loyalty tier | Not enrolled |
| Churn label | **1 (will churn)** |

**Why this is ambiguous:** ₹5,651 in 180d spend is one of the highest in the dataset. This customer is in Loyal Customers but has three tickets with negative sentiment (−0.67) and is confirmed to churn. The segment assignment is correct (F=5, M=5, R=4 satisfies Loyal Customers rules), but the action must not be the standard Loyal Customers playbook. Sending a tier-progress loyalty email to a customer with ₹6,697 in lifetime spend who is about to leave — and who is not even enrolled in the loyalty programme — is a critical missed opportunity.

**Decision:** Escalate immediately to the highest-priority intervention. This is a ₹6,697 lifetime customer with 7 sessions in the last 30 days (still engaged), three support grievances, and no loyalty programme relationship. The gap between their behaviour and their loyalty status is the core of the problem. Action: direct CRM call (not email), acknowledgement of the three tickets, immediate Platinum tier fast-track enrolment offer with bonus points, and a personalised category offer. Every ₹100 spent here has outsized expected return.

**Action:** Escalate to personal CRM outreach. Fast-track Platinum enrolment. Priority 1 within the campaign.

---

## Case 10 — CUST01360 | Segment: Dormant

| Signal | Value |
|---|---|
| R / F / M scores | 1 / 2 / 1 |
| Recency | 160 days |
| Frequency (180d) | 1 order (₹307) |
| Monetary (180d) | ₹307 |
| Lifetime spend | **₹12,311** — highest in Dormant segment |
| Sessions (30d) | 6 |
| Campaign clicks (30d) | 2 |
| Last campaign | Bundle discount |
| Loyalty tier | Silver |
| Churn label | 1 (will churn) |

**Why this is ambiguous:** This customer is classified as Dormant (R=1, F=2, M=1) because their recent activity is minimal. But ₹12,311 in lifetime spend is extraordinary — it is roughly 7.5× the average Dormant customer's lifetime value. More importantly, they had 6 sessions in the last 30 days and clicked a campaign link twice. This is not a dead customer; they are browsing and engaging. The 160-day purchase gap combined with active web engagement strongly suggests they are in a consideration phase — possibly comparing alternatives.

**Decision:** Do not treat this as a standard Dormant customer. The standard Dormant playbook (low-cost automated email) is wrong here — it risks leaving ₹12,311 worth of LTV on the table. The bundle_discount campaign they last received is the right format (they engaged with it), but needs to be more personalised. Send a premium winback: acknowledge their Silver status, highlight new products since their last purchase, offer a personalised bundle with 20% off based on their category history. The 2 campaign clicks signal they are ready to be converted.

**Action:** Premium winback campaign — personalised bundle offer at 20% off. Upgrade to Silver+ or Gold enrolment as part of the offer. Manual review before send.

---

## Case 11 — CUST00841 | Segment: High-Value Unhappy

| Signal | Value |
|---|---|
| R / F / M scores | 2 / 5 / 5 |
| Recency | 99 days |
| Frequency (180d) | — (F=5, high) |
| Monetary (180d) | ₹5,236 |
| Lifetime spend | ₹6,838 |
| Return rate | 16.7% |
| Avg discount | 19% |
| Sessions (30d) | 2 |
| Tickets | 2 |
| Avg sentiment | −0.715 |
| Loyalty tier | Not enrolled |
| Churn label | **0 (retained)** |

**Why this is ambiguous:** High-Value Unhappy with F=5, M=5 and ₹6,838 lifetime spend — but churn label is 0. This is a customer who despite negative support sentiment (−0.715), a 99-day purchase gap, and two complaints, will make a purchase in the next 60 days. The segment assignment is correct, but the playbook needs adjustment: this customer does not need a winback, they need a relationship repair before they buy again. If they buy without the issue being addressed, another bad experience could push sentiment to the point of no return.

**Decision:** Get ahead of the next purchase. Before this customer converts naturally, reach out to resolve the outstanding sentiment issues. A proactive "we've improved X" message, referencing their category, combined with a genuine acknowledgement of past issues, turns the next purchase from a grudging transaction into a restored relationship. Do not wait for them to buy — intervene now.

**Action:** Proactive relationship-repair message before next purchase. Offer a product guarantee on their next order. Not a standard High-Value Unhappy winback — this is a retention maintenance case.

---

*All customer data sourced from executed `segments.csv`. Churn labels are used here only to evaluate the quality of the segment assignment decision, not as model inputs.*
