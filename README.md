# Pricing Certainty: Short vs Long Debt in US Treasury Issuance

**A Monte Carlo cost-risk study of US Treasury funding**

---
Sohini Mukherjee | PhD Candidate in Mathematics, Stony Brook University

## The question

Every large borrower has to decide how to refinance its debt. Rolling over short-term bills is usually cheaper, because long-term yields include a term premium. But the cost of short debt follows interest rates, so when rates rise the interest bill rises with them. Issuing long-term bonds locks in today's rate: the cost is predictable, but usually higher.

This project asks a simple question with a real trade-off behind it:

> Over the next 10 years, how do 3-month bills, 2-year notes, 10-year bonds and mixes of bills and bonds compare on **expected interest cost** and on the **risk of a very expensive year**?

I study it with a stochastic interest-rate model calibrated to US Treasury yields since 1990, a Monte Carlo simulation of rolling debt portfolios, and a backtest on historical yields.

## Main results

Base case, starting from today's yields (October 2026):

| Strategy | Expected average cost | 95th percentile of average cost | 95th percentile of worst-year cost |
| --- | --- | --- | --- |
| 100% 3-month bills | 3.23% | 4.69% | 5.79% |
| 100% 2-year notes | 3.81% | 4.98% | 5.76% |
| 100% 10-year bonds | 4.75% | 5.13% | 5.30% |
| 50/50 bills and bonds | 3.99% | 4.90% | 5.46% |

- **Bills are the cheapest choice on average**, by about 1.5 percentage points a year compared with 10-year bonds. On \$1 trillion of debt that is roughly \$15 billion a year.
- **Long bonds protect against a bad year.** The worst-year cost in a bad scenario is 5.30% for bonds against 5.79% for bills, and the gap grows when rates are more volatile (5.70% against 7.14% with 50% higher volatility).
- **Holding only long bonds is never the best choice.** A portfolio of about 10% bills and 90% bonds has both a lower expected cost and a lower worst-year cost than 100% bonds, because the worst years for bills and bonds don't always coincide.
- **Long debt only becomes cheaper on average if the long-run interest rate is above about 6.6%**, well above anything the data supports.
- **History agrees:** rolling bills was cheaper than a 10-year bond ladder in 98% of past 10-year windows, although those windows come mostly from a long period of falling rates.

In short, choosing between short and long debt is choosing how much to pay for certainty. In this model, that insurance costs about 1.5 percentage points a year at today's rates.

## How it works

1. **Data.** Daily US Treasury constant-maturity yields from FRED for the 3-month (`DGS3MO`), 2-year (`DGS2`) and 10-year (`DGS10`) maturities, January 1990 to October 2026, converted to month-end values.
2. **Interest-rate model.** I fit the Vasicek model, a mean-reverting model of the short rate, to the 3-month yield. Sampled monthly it is an AR(1) process, so the parameters come from a simple regression. The regression standard errors show that the long-run level and the speed of mean reversion are poorly estimated, which is why the sensitivity analysis matters.
3. **Validation.** Monte Carlo bond prices match the model's exact bond-price formula (0.92481 exact, 0.92481 simulated), and the simulated average short rate matches its exact expected value (3.2203% against 3.2204%).
4. **Simulation.** 20,000 paths of the short rate over 10 years in monthly steps, using the exact model transition. Two-year and 10-year yields are built from the model plus a term premium that starts at today's level and moves back to its historical average.
5. **Debt ladders.** Each strategy is a rolling ladder: for a maturity of M months, 1/M of the debt is refinanced every month at the current yield. For every path I record the 10-year average cost and the most expensive single year.
6. **Sensitivity.** The same random numbers are reused in every scenario, so differences come from the parameters alone. I vary the long-run rate from 2% to 5% and raise volatility by 50%, solve for the break-even long-run rate, and check that a second random seed gives the same answers (within 2 basis points).
7. **Backtest.** The same ladder calculation applied to actual yields over every 10-year window starting between 1999 and 2016.

## Limitations

- A one-factor model drives all yields with the same random shock, so it cannot produce an inverted curve or short and long rates moving in opposite directions.
- About 9% of simulated paths go below zero at some point, a known feature of the Vasicek model.
- The model's parameters are weakly identified and estimated from history, not from market prices.
- Long yields rely on an assumed term premium; the real one moves around considerably.
- Issuing debt does not move yields in the model, and only fixed-rate nominal debt of constant size is considered.
- The historical windows overlap heavily and come mostly from a period of falling rates.

## Repository contents

| File | What it is |
| --- | --- |
| `treasury_debt_issuance.ipynb` | The full analysis, with explanations, code and outputs |
| `fred_yields_pinned.csv` | Saved copy of the FRED data used, so results can be reproduced exactly |
| `requirements.txt` | Python packages needed |
| `README.md` | This file |

## How to run it

You need Python 3.10 or later.

```bash
git clone https://github.com/sohinimk/short-vs-long-treasury-debt.git
cd short-vs-long-treasury-debt
pip install -r requirements.txt
jupyter notebook treasury_debt_issuance.ipynb
```

Then choose **Kernel → Restart & Run All**. The notebook reads `fred_yields_pinned.csv` if it is present, so you get exactly the results shown here. If you delete that file, the notebook downloads fresh data from FRED (newer data will change the numbers slightly).

All random numbers come from fixed seeds (1 for the validation, 42 for the main simulation, 43 for the stability check), so every run gives the same results.
