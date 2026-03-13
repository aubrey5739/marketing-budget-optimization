# marketing-budget-optimization

Integer Programming-based Marketing Budget Allocation  
RM 294 – Optimization I | MSBA, University of Texas | Fall 2025

**Team:** Mikyung Oh · Carter St.Geme · Riju Hariharan · Vyshnavi Maringanti

---

## Overview

This project uses **linear programming (LP)** and **mixed-integer programming (MIP)** to optimally allocate a \$10M marketing budget across 10 advertising platforms (AdWords, Email, Facebook, Instagram, LinkedIn, Print, SEO, Snapchat, TV, Twitter).

ROI for each platform is modeled as a **piecewise constant function** over spending tiers, making the total return a piecewise linear function. Depending on whether ROI is non-increasing (concave) or non-monotone (non-concave), the problem is solved as either an LP or MIP using **Gurobi**.

---

## Repository Structure

```
marketing-budget-optimization/
│
├── OptimizationProject2.ipynb     # Main Python notebook (all parts 1–8)
├── Project2_Report.pdf            # Final written report submitted to boss
│
├── data/
│   ├── roi_company1.csv           # Concave ROI data (Consulting Firm 1)
│   ├── roi_company2.csv           # Non-concave ROI data (Consulting Firm 2)
│   ├── min_amount.csv             # Minimum effective spend per platform
│   └── roi_monthly.csv            # Monthly ROI tiers for 12-month planning
│
└── README.md
```

---

## Problem Parts

| Part | Description | Model Type |
|------|-------------|------------|
| 1–2 | Problem setup, data overview, constraints | — |
| 3 | Optimize budget with concave ROI (Firm 1) | LP |
| 4 | Optimize budget with non-concave ROI (Firm 2) | MIP |
| 5 | Cross-evaluate allocations; robustness analysis | Analysis |
| 6 | Add minimum effective spend constraints | MIP |
| 7 | 12-month rolling optimization with 50% reinvestment | LP (sequential) |
| 8 | Stability check; model design for stable plans | Analysis + MIP |

---

## Constraints

All models respect these managerial rules:

1. **Budget cap:** Total spend ≤ \$10M (grows monthly in Part 7 via reinvestment)
2. **Platform cap:** Each platform ≤ \$3M
3. **Traditional vs. digital:** Print + TV ≤ Facebook + Email
4. **Social vs. Search:** Facebook + LinkedIn + Instagram + Snapchat + Twitter ≥ 2 × (SEO + AdWords)
5. **Minimum spend (Part 6+):** Each active platform must meet its threshold from `min_amount.csv`

---

## Key Results

| Part | Model | Total Return | Overall ROI |
|------|-------|-------------|-------------|
| 3 | LP (ROI1) | \$0.54M | 5.44% |
| 4 | MIP (ROI2) | \$0.45M | 4.53% |
| 6 | MIP + min spend (ROI2) | \$0.45M | 4.53% |
| 7 | Monthly LP (ROI monthly) | ~\$0.45–0.76M/mo | Varies |

**Cross-evaluation (Part 5):**
- Applying LP allocation (Firm 1) to ROI2 → **−38.67%** vs. ROI2 optimum
- Applying MIP allocation (Firm 2) to ROI1 → **−49.43%** vs. ROI1 optimum
- → LP (Firm 1) plan is more robust on a minimax-regret basis

**Stability (Part 8):**  
The unconstrained monthly plan violates the ±\$1M/month stability rule on multiple platforms. A stable model can be enforced by adding auxiliary increase/decrease variables with a capped absolute change per platform per month.

---

## How to Run

### Requirements

```bash
pip install gurobipy pandas numpy matplotlib
```

> A valid **Gurobi license** is required. Free academic licenses are available at [gurobi.com](https://www.gurobi.com/academia/academic-program-and-licenses/).

### Running the Notebook

1. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/marketing-budget-optimization.git
   cd marketing-budget-optimization
   ```

2. Place all CSV files inside the `data/` folder (or update the `pd.read_csv` paths at the top of the notebook).

3. Open and run the notebook:
   ```bash
   jupyter notebook OptimizationProject2.ipynb
   ```

> ⚠️ **Grading note:** The first code chunk reads all CSV files via `pd.read_csv`. To test with new data, simply replace the CSV files — all downstream analysis is fully generalized with no hardcoded values.

---

## CSV File Format

### `roi_company1.csv` / `roi_company2.csv`
| Column | Description |
|--------|-------------|
| `platform` | Advertising platform name |
| `tier` | Tier index (1, 2, 3, …) |
| `lowerbound` | Lower bound of investment range (\$M) |
| `upperbound` | Upper bound of investment range (\$M) |
| `roi` | ROI for this tier (e.g., 0.05 = 5%) |

### `min_amount.csv`
| Column | Description |
|--------|-------------|
| `platform` | Advertising platform name |
| `min_amount` | Minimum investment required to activate (\$M) |

### `roi_monthly.csv`
| Column | Description |
|--------|-------------|
| `month` | Month name (Jan–Dec) |
| `platform` | Advertising platform name |
| `tier` | Tier index |
| `lowerbound` | Lower bound of investment range (\$M) |
| `upperbound` | Upper bound of investment range (\$M) |
| `roi` | ROI for this tier |

---

## Methods

### LP (Part 3)
- Decision variable: `x[p, t]` = dollars invested in tier `t` for platform `p`
- Concavity of ROI guarantees LP optimality (no need for binary variables)

### MIP (Parts 4, 6)
- Binary variable `z[p, t]` = 1 if tier `t` is active for platform `p`
- Lambda variables for convex combination representation of piecewise linear function
- Ensures tier ordering: a higher tier is only active if all previous tiers are fully funded

### Monthly LP with Reinvestment (Part 7)
- Solved sequentially month by month
- Budget for month `m+1` = \$10M + 0.5 × (return from month `m`)

---

## Report

See [`Project2_Report.pdf`](./Project2_Report.pdf) for the full written analysis including figures, formulations, and managerial recommendations.

---

## License

This project was completed as coursework for RM 294 – Optimization I at the University of Texas MSBA program. For academic use only.
