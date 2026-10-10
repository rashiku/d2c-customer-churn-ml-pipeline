# Retention Strategy
## D2C Customer Churn Capstone — Part 2

**Snapshot date:** `2025-09-30`  
**Segments:** 9 | **Universe:** 2,400 customers | **Overall churn rate:** 47.0%  
**Total revenue at risk (180d):** ₹9,69,751 | **Illustrative campaign budget:** ₹50,000

---

## Segmentation Logic

Segments are assigned via a **priority-ordered rule cascade**. Each customer is placed in the first segment whose conditions they satisfy. Rules combine RFM quintile scores (1–5, percentile-rank based) with six behavioural flags derived from web activity, support tickets, return behaviour, discount usage, loyalty enrolment, and account age.

| Priority | Segment | Core RFM Rule | Behavioural Qualifier |
|---|---|---|---|
| 1 | New Customers | Any | `days_since_signup ≤ 90` |
| 2 | Champions | R≥4, F≥4, M≥4 | `unhappy_flag = 0` |
| 3 | Loyal Customers | F≥4, M≥3, R≥3 | Not Champions |
| 4 | High-Value Unhappy | M≥4 OR lifetime spend in top 25% | `unhappy_flag=1` OR `high_return_flag=1` |
| 5 | At-Risk | R≤2, F≥3 OR M≥3 | — |
| 6 | Discount Hunters | `discount_hunter_flag=1` | M≤3 |
| 7 | Dormant | R≤2, F≤2, M≤2 | — |
| 8 | Promising | R≥3, F≤2 | — |
| 9 | Occasional Buyers | All others | — |

---

## Segment Profiles & Retention Actions

---

### 1. Champions
**140 customers | 5.8% of base | Churn rate: 10.0%**

| Metric | Value |
|---|---|
| Avg recency | 21 days |
| Avg frequency (180d) | 2.9 orders |
| Avg monetary (180d) | ₹2,392 |
| Avg lifetime spend | — (highest tier) |
| Avg discount used | 27.8% |
| Avg sessions (30d) | — (highest engagement) |
| % Loyalty enrolled | high |
| Revenue at risk | ₹33,491 |

**Who they are:** The brand's most valuable active customers — bought recently, buy often, spend the most, and have zero unhappy-flag records. They are not churning at scale (10% rate), but each churned Champion represents the loss of ₹2,392+ in near-term spend.

**Retention action:** Do not send discount campaigns — these customers already buy at full or near-full price. The goal is to deepen the relationship, not subsidise it.
- Send a personalised "thank you" message acknowledging their loyalty, signed by a real team member.
- Offer early access to new launches aligned with their `preferred_category`.
- If not already on Platinum tier, trigger an automatic loyalty upgrade invitation.
- Invite to a VIP feedback panel — Champions who feel heard become advocates.

**Expected business value:** Protecting the 10% churn tail (≈14 customers × ₹2,392 = ₹33,491 at risk) at a cost of ₹5/customer (₹700 total). ROI ratio: **47.8×**.

**Campaign cost estimate:** ₹700 (₹5 × 140 customers)

---

### 2. Loyal Customers
**567 customers | 23.6% of base | Churn rate: 17.1%**

| Metric | Value |
|---|---|
| Avg recency | 39 days |
| Avg frequency (180d) | 2.9 orders |
| Avg monetary (180d) | ₹2,103 |
| Avg discount used | 27.8% |
| % Unhappy flag | — (excluded by rule) |
| % Loyalty enrolled | 40.2% |
| Revenue at risk | ₹2,03,867 |

**Who they are:** The backbone of the business — 23.6% of the customer base, spending ₹2,103 on average in 180 days. Their 17.1% churn rate is low in absolute terms, but because the segment is large, ₹2,03,867 is at risk. Notably, only 40.2% are enrolled in the loyalty programme despite their purchase behaviour qualifying them.

**Retention action:** Two sub-groups require different approaches:
- **Loyalty-enrolled sub-group:** Send a tier-progress update ("You're X points from Gold"). Frame as recognition, not a discount.
- **Not-enrolled sub-group (majority):** This is the highest-leverage action in the entire campaign. Enrol them in the loyalty programme with a "welcome to loyalty" offer (double points on their next order). Converting even 20% of unenrolled Loyal Customers to enrolled status will structurally reduce churn in this cohort.
- Personalise by `preferred_category` — if they buy Skin Care, send Skin Care content, not a generic catalogue.

**Expected business value:** Revenue at risk of ₹2,03,867. Even a 5% churn reduction in this segment recovers ~₹10,000 in spend. Loyalty enrolment drives long-term LTV beyond the 60-day window.

**Campaign cost estimate:** ₹4,536 (₹8 × 567 customers)

---

### 3. At-Risk
**234 customers | 9.8% of base | Churn rate: 72.6%**

| Metric | Value |
|---|---|
| Avg recency | 124 days |
| Avg frequency (180d) | 1.7 orders |
| Avg monetary (180d) | ₹1,492 |
| Avg lifetime spend | ₹2,547 |
| Avg discount used | 26.6% |
| Avg sessions (30d) | 4.4 |
| % Unhappy flag | 19.7% |
| Revenue at risk | ₹2,53,534 |

**Who they are:** Formerly active buyers — their Frequency (F≥3) or Monetary (M≥3) history shows they were once engaged, but Recency has dropped to R≤2 (last order 124 days ago on average). At 72.6% churn rate, this is the highest-urgency retainable segment. They have not yet gone fully dormant (unlike Dormant customers), meaning a well-timed incentive can still pull them back.

**Retention action:** Speed matters here. These customers are already 4 months out and are very likely forming a habit of not buying.
- Send a personalised winback email within 48 hours of campaign launch: "We noticed you haven't ordered in a while — here's 15% off your next order in [preferred_category]."
- Keep discount at 15% — this segment's avg discount history is only 26.6%, meaning they are not conditioned to expect 40%+ deals.
- Follow up with a "last chance" message 7 days later if no conversion.
- For the 19.7% with an unhappy flag, escalate to a human-written apology + offer.
- Prioritise sub-group with M≥4 (₹1,500+ spenders) — they are worth the higher intervention cost.

**Expected business value:** Highest ROI ratio of any segment at **54.2×**. Revenue at risk: ₹2,53,534. Recovering even 15% of this cohort (35 customers) returns ₹52,000+ in spend — more than the entire campaign budget.

**Campaign cost estimate:** ₹4,680 (₹20 × 234 customers)

---

### 4. High-Value Unhappy
**245 customers | 10.2% of base | Churn rate: 62.4%**

| Metric | Value |
|---|---|
| Avg recency | 115 days |
| Avg frequency (180d) | 1.5 orders |
| Avg monetary (180d) | ₹1,281 |
| Avg lifetime spend | ₹4,677 |
| Avg return rate | 10.9% |
| % Unhappy flag | 99.2% |
| % High return flag | 15.9% |
| Revenue at risk | ₹1,95,855 |

**Who they are:** High spenders (avg lifetime spend ₹4,677 — the highest of any segment) who have had a bad experience. 99.2% carry the unhappy flag (negative support sentiment or a product reaction ticket). Their lifetime value is real and large; their dissatisfaction is also real and large. The 62.4% churn rate reflects that most have already decided to leave. The goal is to stop the ones who are still in the decision window.

**Retention action:** Discounts alone will not fix a trust problem. This segment needs acknowledgement before it needs an offer.
- Trigger a personal outreach: "We saw your recent experience and want to make it right." This must feel human — a templated discount email will be ignored or resent.
- For product_reaction cases: offer a free dermatologist consultation or product exchange, depending on the category.
- For refund/resolution cases: expedite any outstanding refunds and communicate proactively.
- Follow the acknowledgement with a meaningful gesture — a loyalty tier upgrade, a free sample pack, or a custom bundle aligned to their category.
- Flag the top 50 by lifetime spend for a direct CRM call rather than an email.

**Expected business value:** Revenue at risk ₹1,95,855. Lifetime spend per customer of ₹4,677 means recovering even 10 customers is worth ₹47,000 in future LTV. Cost per customer (₹25) reflects the higher-touch intervention required.

**Campaign cost estimate:** ₹6,125 (₹25 × 245 customers)

---

### 5. Discount Hunters
**129 customers | 5.4% of base | Churn rate: 65.9%**

| Metric | Value |
|---|---|
| Avg recency | 82 days |
| Avg frequency (180d) | 1.1 orders |
| Avg monetary (180d) | ₹490 |
| Avg discount used | 46.7% |
| % Discount hunter flag | 100% |
| % Unhappy flag | 40.3% |
| Revenue at risk | ₹41,656 |

**Who they are:** Customers who only buy when discounts are deep (avg 46.7%), spend relatively little (₹490 in 180d), and churn at 65.9% — the third-highest rate. Crucially, their monetary value is low. Offering them another large discount to return simply accelerates the pattern: they will buy at 50% off, and leave again when the promotion ends.

**Retention action:** The primary goal is **not** to match their discount expectation — it is to either convert them to value-conscious buyers or let them lapse with minimal spend.
- Test a non-monetary offer: loyalty programme enrolment with bonus points on their next full-price order. This re-frames the relationship around value, not discount.
- For the 40.3% with an unhappy flag, address the service issue first before any offer.
- Do not send a 40%+ discount campaign — it confirms the pattern and trains them further.
- If a discount is used, cap at 20% and pair it with a "members-only" framing to increase perceived exclusivity.
- Segment further: the small sub-group with F≥4 (repeated buyers even at high discount) may be worth a loyalty conversion attempt.

**Expected business value:** Revenue at risk is ₹41,656. Given low avg monetary (₹490), the primary value of this intervention is breaking the discount dependency rather than immediate revenue recovery.

**Campaign cost estimate:** ₹1,032 (₹8 × 129 customers)

---

### 6. Dormant
**498 customers | 20.8% of base | Churn rate: 88.0%**

| Metric | Value |
|---|---|
| Avg recency | 196 days |
| Avg frequency (180d) | 0.5 orders |
| Avg monetary (180d) | ₹244 |
| Avg lifetime spend | ₹1,648 |
| Avg sessions (30d) | 3.2 |
| % Unhappy flag | 31.3% |
| Revenue at risk | ₹1,06,806 |

**Who they are:** The largest single segment (20.8% of base) with the highest churn rate (88.0%). Most have not ordered in the 180-day window (avg 0.46 orders) and last bought 196 days ago. Despite this, their avg lifetime spend of ₹1,648 tells us they were once real customers — this is not a cohort of one-time buyers. A meaningful subset (sessions_30d avg = 3.2) is still visiting the site but not converting.

**Retention action:** Accept that the majority will not be recovered. The strategy is to identify the recoverable minority and prioritise them.
- **Recoverable sub-group (still visiting site):** Target the ~30% who had at least one web session in the last 30 days. Send a deep winback discount (25–30%) framed as "We miss you — come back."
- **High lifetime spend sub-group:** Customers with lifetime spend ≥ ₹5,000 in Dormant (e.g. CUST01360 with ₹12,311) warrant a personal outreach regardless of recent behaviour.
- **Fully lapsed majority:** A low-cost automated email is appropriate. Do not invest ₹18/customer in a manual campaign for customers with ₹244 of recent spend.
- Track response rate carefully — if below 5% after the campaign, re-classify Dormant customers as "lapsed" and stop spending on them in future cycles.

**Expected business value:** Revenue at risk ₹1,06,806 but recovery probability is low. Even a 5% winback (25 customers × ₹244) recovers only ₹6,100 — less than the campaign cost of ₹8,964. The real value is in identifying the recoverable sub-group before spending on the full segment.

**Campaign cost estimate:** ₹8,964 (₹18 × 498 customers) — consider spending only on the recoverable sub-group to reduce this to ~₹3,000.

---

### 7. Promising
**259 customers | 10.8% of base | Churn rate: 34.0%**

| Metric | Value |
|---|---|
| Avg recency | 42 days |
| Avg frequency (180d) | 1.0 orders |
| Avg monetary (180d) | ₹774 |
| Avg sessions (30d) | 6.1 |
| % Loyalty enrolled | 44.0% |
| Revenue at risk | ₹68,148 |

**Who they are:** Recent buyers (avg recency 42 days, R≥3) who have only bought once or twice. They are digitally engaged (avg 6.1 sessions in 30 days) but haven't yet formed a purchase habit. At 34.0% churn rate, roughly one in three will not return — but the other two thirds are winnable with the right second-purchase trigger.

**Retention action:** The second purchase is the most important conversion in a D2C relationship. Once a customer buys twice, repeat-purchase probability climbs significantly.
- Send a "complete your routine" email 21 days after their first order, featuring complementary products in their `preferred_category`.
- Offer a modest incentive (10–15% off, free shipping) for the second order — not enough to create discount dependency, enough to reduce friction.
- For the 56% not yet enrolled in loyalty, include a loyalty enrolment nudge alongside the second-purchase offer.
- Monitor email_opens and campaign_clicks — if a Promising customer clicks but doesn't convert, trigger a cart-abandon recovery flow.

**Expected business value:** Revenue at risk ₹68,148. A 10% churn reduction in this segment (≈26 customers × ₹774) recovers ₹20,000 in 180d spend and sets up long-term repeat-purchase behaviour.

**Campaign cost estimate:** ₹2,590 (₹10 × 259 customers)

---

### 8. New Customers
**298 customers | 12.4% of base | Churn rate: 24.2%**

| Metric | Value |
|---|---|
| Avg recency | 25 days |
| Avg frequency (180d) | 1.1 orders |
| Avg monetary (180d) | ₹817 |
| Avg sessions (30d) | 6.5 |
| % Discount hunter flag | 19.5% |
| % Loyalty enrolled | 35.6% |
| Revenue at risk | ₹58,894 |

**Who they are:** Customers who signed up within the last 90 days. They are the most recently active (avg recency 25 days), show strong digital engagement (avg 6.5 sessions — highest of any segment), and have a 24.2% churn rate. This is the "first 90 days" window: the brand has maximum attention and opportunity to establish a habit before it is lost.

**Retention action:** Onboarding quality is the lever here, not discounts.
- Trigger a structured 4-email onboarding sequence: welcome → category education → first-order follow-up → second-purchase nudge (Day 7, 14, 21, 30).
- 19.5% have a discount_hunter flag already — for these customers, avoid deep discounts in onboarding and instead emphasise loyalty programme benefits.
- For customers who bought once and have not returned in 30 days, send a personalised re-engagement with a skin-type or category-matched product recommendation.
- Only 35.6% are loyalty-enrolled despite being new — launch a "join loyalty at signup" prompt in the post-purchase flow.

**Expected business value:** Revenue at risk ₹58,894. Reducing the 24.2% churn rate to 18% (a 6pp improvement) across 298 customers retains ~18 additional customers × ₹817 = ₹14,700 in near-term spend, plus compound LTV gains from a stronger habit.

**Campaign cost estimate:** ₹3,576 (₹12 × 298 customers)

---

### 9. Occasional Buyers
**30 customers | 1.2% of base | Churn rate: 33.3%**

| Metric | Value |
|---|---|
| Avg recency | 40 days |
| Avg frequency (180d) | 2.0 orders |
| Avg monetary (180d) | ₹751 |
| Avg sessions (30d) | 5.5 |
| % Loyalty enrolled | 50.0% |
| Revenue at risk | ₹7,500 |

**Who they are:** A small residual group (30 customers) that didn't fit cleanly into other segments — moderate recency, moderate frequency, moderate spend. Half are loyalty-enrolled. Their 33.3% churn rate is mid-table and their total revenue at risk is modest at ₹7,500.

**Retention action:** Treat similarly to Promising customers — focus on building purchase frequency rather than retention per se.
- Include in the standard email nurture series.
- No bespoke campaign needed given segment size — bundle into the Promising campaign flow.

**Campaign cost estimate:** ₹450 (₹15 × 30 customers)

---

## Budget Prioritisation

### Illustrative Budget: ₹50,000

The table below ranks segments by **ROI ratio** (revenue at risk ÷ cost to reach all customers in segment). All nine segments fit within the ₹50,000 budget; the full campaign costs ₹32,653.

| Priority | Segment | N | Cost/Customer | Total Cost | Rev @ Risk | ROI Ratio |
|---|---|---|---|---|---|---|
| 1 | At-Risk | 234 | ₹20 | ₹4,680 | ₹2,53,534 | **54.2×** |
| 2 | Champions | 140 | ₹5 | ₹700 | ₹33,491 | 47.8× |
| 3 | Loyal Customers | 567 | ₹8 | ₹4,536 | ₹2,03,867 | 44.9× |
| 4 | Discount Hunters | 129 | ₹8 | ₹1,032 | ₹41,656 | 40.4× |
| 5 | High-Value Unhappy | 245 | ₹25 | ₹6,125 | ₹1,95,855 | 32.0× |
| 6 | Promising | 259 | ₹10 | ₹2,590 | ₹68,148 | 26.3× |
| 7 | Occasional Buyers | 30 | ₹15 | ₹450 | ₹7,500 | 16.7× |
| 8 | New Customers | 298 | ₹12 | ₹3,576 | ₹58,894 | 16.5× |
| 9 | Dormant | 498 | ₹18 | ₹8,964 | ₹1,06,806 | 11.9× |
| | **Total** | **2,400** | | **₹32,653** | **₹9,69,751** | |

**Remaining budget after all segments:** ₹17,347

### Prioritisation Rationale

**Spend first on At-Risk (Priority 1):** Despite not being the largest segment, At-Risk delivers the highest ROI ratio (54.2×) because the combination of high churn rate (72.6%), meaningful spend (₹1,492), and modest intervention cost (₹20) creates the most efficient recovery opportunity. These customers have recent enough history to respond to a well-timed offer. At ₹4,680, it is the most efficient ₹4,680 in the entire budget.

**Protect Champions and Loyal Customers early (Priorities 2–3):** These two segments account for ₹2,37,358 in combined revenue at risk and represent the brand's healthiest customers. Losing a Champion or Loyal Customer is structurally worse than losing a Dormant customer because of their higher LTV trajectory. Their interventions are low-cost (₹5–₹8/customer).

**Treat High-Value Unhappy carefully (Priority 5):** Despite high revenue at risk (₹1,95,855) and high churn rate (62.4%), High-Value Unhappy ranks 5th because cost per customer (₹25) is the highest in the portfolio. The ROI ratio is still strong at 32.0×, but the intervention must be personalised — if a generic email campaign is sent instead of a genuine acknowledgement, the money will be wasted and the customer relationship further damaged.

**Re-evaluate Dormant spend (Priority 9):** Dormant has the highest revenue at risk in absolute terms after At-Risk, but the lowest ROI ratio (11.9×) because avg 180d monetary is only ₹244 against a ₹18/customer cost. Recommendation: do not spend ₹18 on all 498 Dormant customers. Instead, apply a sub-segmentation first (those with sessions > 0 or lifetime spend > ₹3,000) and run a targeted winback on that sub-group at ₹18, leaving the rest to receive a ₹2 automated email only.

---

## Summary: One-Line Action per Segment

| Segment | One-Line Action |
|---|---|
| Champions | VIP recognition + early access to new launches; no discounts |
| Loyal Customers | Loyalty enrolment push for unenrolled half + tier-progress update |
| At-Risk | Immediate personalised winback — 15% off in preferred category |
| High-Value Unhappy | Acknowledge the problem first, then offer a resolution gesture |
| Discount Hunters | Non-monetary loyalty pitch; no deep discounts |
| Dormant | Deep winback for still-active sub-group; automated low-cost email for rest |
| Promising | Second-purchase trigger email + free shipping nudge at Day 21 |
| New Customers | Structured 4-email onboarding sequence; loyalty enrolment at signup |
| Occasional Buyers | Bundle into Promising campaign nurture flow |

---

*All numbers sourced from executed `rfm_segmentation.ipynb` on the full 2,400-customer snapshot.*
