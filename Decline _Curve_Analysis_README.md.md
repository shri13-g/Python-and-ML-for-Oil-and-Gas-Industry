# Production Decline Curve Analysis (DCA)

A Python-based reservoir engineering tool to analyze production decline data, identify diagnostic trends, evaluate model parameters, and forecast cumulative asset recovery ($N_p$). This project applies classic empirical decline analysis principles to determine key production characteristics from a 24-month historical gas field dataset.

## Project Overview

Decline Curve Analysis (DCA) is an essential empirical method used in petroleum engineering to evaluate production history and forecast future performance of oil and gas wells. This project automates diagnostic workflow checks and computes production invariants by:
1. Evaluating the continuous loss ratio ($D$) step-by-step to diagnose the decline model archetype.
2. Formulating regression constants to solve for initial production rate ($q_i$) and nominal decline factor ($b$).
3. Integrating historical tracking data over time to output cumulative volumetric production ($N_p$).

---

## Technical Methodology & Equations

### 1. Diagnostic Loss Ratio ($D$)
To avoid assumptions, the project computes the nominal relative loss rate across active intervals to determine the model type:
$$D = -\frac{\Delta q / \Delta t}{q}$$

Because the calculated $D$ vs. $q$ plot yields a perfectly flat horizontal profile independent of production rates ($D \approx 0.1000 \text{ month}^{-1}$), the asset is verified as an **Exponential Decline Model** ($b = 0$).

### 2. Parameter Invariants
For an exponential asset, performance constants are computed across known tracking thresholds:
* **Decline Factor ($b$):** Logarithmic variance over elapsed time.
* **Initial Production ($q_i$):** Hindcast theoretical interception back to $t=0$.

### 3. Cumulative Material Balance
Volumetric integration tracks the total gas recovered over 24 months, using an average daily-to-monthly scaling coefficient ($30.42 \text{ days/month}$):
$$N_p = \frac{(q_i - q_t) \times 30.42}{D}$$

---

## Performance Results

Executing the code outputs the following reservoir parameters based on the 24-month dataset:

| Reservoir Parameter | Evaluated Result | Unit |
| :--- | :--- | :--- |
| **Identified Decline Model** | Exponential Decline ($b=0$) | Archetype |
| **Nominal Decline Rate ($b$)** | `0.1000` | $\text{month}^{-1}$ |
| **Initial Production Rate ($q_i$)** | `999.99` | Mscf/day |
| **Cumulative Gas Production ($N_p$)**| `276,602.88` | Mscf |

---

## Repository Structure

* `dca_analysis.py` - The core execution script containing data arrays, differentiation layers, matplotlib plotting, and integration loops.
* `README.md` - Technical project documentation and summary.

## Prerequisites & Installation

Ensure you have a standard Python environment with the following scientific computing libraries installed:

```bash
pip install numpy matplotlib pandas
