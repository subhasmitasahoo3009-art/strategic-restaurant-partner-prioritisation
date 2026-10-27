# Bengaluru Dine-Out Partner Prioritization

A data analysis project that scores and ranks restaurants in a single Bengaluru territory (**Indiranagar**) to build a prioritized **Top 75 onboarding list** for dine-out partnerships, using Zomato restaurant data.

The notebook takes ~8.9K raw listings through cleaning, a business funnel, exploratory analysis, a weighted scoring model, a sensitivity test, and a simulated GMV projection against an ₹80 Lakh target.

---

## Highlights

- **Dataset:** 8,923 Bengaluru restaurant listings across 169 areas (8,921 after de-duplication)
- **Funnel:** 8,921 → 3,950 with indoor seating → 3,315 with a dinner rating → **2,833** with ≥10 dinner reviews
- **Territory:** Indiranagar, with **142** qualified candidates (median dinner reviews 449, highest of the top areas)
- **Output:** 75 ranked accounts split into Tier A / B / C (25 each), each with a recommended action
- **Model robustness:** the Top 75 overlaps **94.7%–96%** with the base list under three alternative weight scenarios
- **Simulated GMV:** ₹77.43 Lakh vs. ₹80 Lakh target (gap of ₹2.57 Lakh)

---

## Project Structure

```
.
├── notebook.ipynb                            # Full analysis
├── BangaloreZomatoData.csv.zip               # Raw dataset (add this yourself)
├── indiranagar_prioritized_targets_75.csv    # Final ranked target list
├── top_15_scores.png                         # Chart: top 15 by priority score
├── reviews_vs_ratings.png                    # Chart: demand vs. quality by tier
├── tier_cost_distribution.png                # Chart: targets by price band and tier
└── sensitivity_overlap.png                   # Chart: weight-scenario overlap
```

---

## Methodology

### 1. Data cleaning
- Kept 14 relevant columns and renamed them to snake_case
- Stripped `, Bangalore` from the `area` column
- Converted `dinner_rating` and `delivery_rating` to numeric (`-` becomes `NaN`)
- Dropped duplicate outlets on `name` + `address` (2 removed)

### 2. Dine-out funnel
A restaurant is a candidate only if it has indoor seating, a dinner rating, and at least `MIN_REVIEWS = 10` dinner reviews, so ratings are backed by real volume.

### 3. Chain detection
Any brand name appearing **5 or more times** in the cleaned data is flagged `is_chain` (294 brands). Chains are routed differently in the action playbook.

### 4. Scoring model
Each pillar is scaled to 0–100 and combined with configurable weights:

| Pillar | Weight | How it's measured |
|---|---|---|
| Demand | 40% | `log1p(dinner_reviews)`, min-max scaled |
| Quality | 25% | Dinner rating, min-max scaled |
| Dine-in orientation | 20% | `dinner_reviews / (dinner_reviews + delivery_reviews)` |
| Price fit | 15% | 100 within ₹400–₹1,200 for two; linear decay to 0 at ₹150 and ₹2,500 |

Restaurants are ranked by final score: ranks 1–25 are **Tier A**, 26–50 **Tier B**, and 51–75 **Tier C**.

### 5. Sensitivity analysis
The Top 75 is recomputed under three alternative weightings and compared with the base case:

| Scenario | Demand / Quality / Dine-in / Price | Overlap |
|---|---|---|
| Demand heavy | 60 / 15 / 15 / 10 | 96.00% |
| Quality heavy | 20 / 50 / 15 / 15 | 96.00% |
| Equal weights | 25 / 25 / 25 / 25 | 94.67% |

### 6. Action playbook and GMV simulation

| Segment | Recommended action |
|---|---|
| Tier A, independent | Onboard first: pitch premium offers and event participation |
| Tier A, chain | Route via regional/HQ partnerships team |
| Tier B | Second wave: pitch footfall growth and lunch/dinner offers |
| Tier C | Nurture: low priority, revisit next quarter |

Simulated monthly GMV = assumed bills per month × average cost for two, with **150 / 90 / 50** bills for Tier A / B / C. These volumes are **assumptions, not observed data**.

- Simulated total: **₹77,43,000** vs. target **₹80,00,000**
- Hitting the target with a flat distribution would need about **109 bills per partner per month** (average cost for two across targets: ₹974.67)

---

## Top 5 Priority Accounts (Indiranagar)

| Rank | Restaurant | Score | Chain? |
|---|---|---|---|
| 1 | Barbeque Nation | 90.27 | Yes |
| 2 | Vapour Pub & Brewery | 88.82 | No |
| 3 | Phobidden Fruit | 87.35 | No |
| 4 | Onesta | 86.23 | Yes |
| 5 | Chinita Real Mexican Food | 85.27 | No |

---

## Getting Started

### Requirements
- Python 3.9+
- `pandas`, `numpy`, `matplotlib`, `seaborn`

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

### Run
1. Clone the repo and place `BangaloreZomatoData.csv.zip` in the project root.
2. Open the notebook:
   ```bash
   jupyter notebook notebook.ipynb
   ```
3. Run all cells. The notebook was written in Google Colab, so the upload cell (`from google.colab import files`) should be skipped or replaced with a local `pd.read_csv('BangaloreZomatoData.csv.zip')` when running locally.

### Configuration
Key parameters are defined as variables near the top of their sections:

```python
MIN_REVIEWS = 10
TERRITORY   = 'Indiranagar'
WEIGHTS     = {'demand': 0.40, 'quality': 0.25, 'dine_in_orientation': 0.20, 'price_fit': 0.15}
PRICE_CONFIG = {'ideal_low': 400, 'ideal_high': 1200, 'abs_min': 150, 'abs_max': 2500}
BILL_VOLUMES = {'Tier A': 150, 'Tier B': 90, 'Tier C': 50}
GMV_TARGET_INR = 8_000_000
```

Change `TERRITORY` to any area in the dataset (for example `HSR`)
