# Machine Learning-Based Reservoir Drive Mechanism Classifier

## 📌 Project Overview

Identifying a reservoir’s primary energy source (drive mechanism) is critical for optimal field development, reserve estimation, and secondary recovery planning. Traditional diagnostic methods—such as manual Material Balance plotting or decline curve analysis—are often slow, subjective, and easily skewed by noisy field data.

This repository delivers an automated, data-driven **4-Class Supervised Machine Learning Classifier** utilizing tree-based ensemble modeling (**Random Forest**). Rather than evaluating static daily production snapshots out of context, this framework features a custom **Asset-Level Aggregation Wrapper**. 

By evaluating the entire 200-month physical timeline of an individual asset, the pipeline converts raw surface production streams into physics-informed engineering trends, achieving robust, publication-grade asset diagnostics even in the presence of severe operational multi-phase measurement noise.

---

## 🔬 Core Engineering Concepts & Drive Signatures

The pipeline maps production data histories into one of four operational categories based on distinct thermodynamic and fluid-proportionality signatures:

* **Label 0: Rock & Fluid Expansion Drive**
  * *Condition:* Operates strictly in under-saturated conditions ($P > P_b$).
  * *Signature:* Rapid, continuous power-law pressure drop due to low fluid/rock compressibilities. GOR ($R$) stays flat and equal to initial solution gas ($R_{si}$). Water cut is negligible.

* **Label 1: Solution Gas (Depletion) Drive**
  * *Condition:* Operates below the bubble point ($P < P_b$) with no initial free gas cap.
  * *Signature:* Steep pressure decline. As dissolved gas liberates, it forms a classic "mountain-shaped" peak in the producing GOR, followed by an aggressive decline as reservoir energy is exhausted.

* **Label 2: Gas Cap Drive**
  * *Condition:* Supported by a pre-existing free gas column above the oil zone.
  * *Signature:* Slow, buffered pressure decline. As the gas cap expands downward and breaches well perforations (gas coning), the GOR exhibits a permanent, aggressive upward migration.

* **Label 3: Water Drive**
  * *Condition:* Powered by an active, encroaching aquifer connected to the reservoir boundary.
  * *Signature:* Highly maintained, near-flat reservoir pressure over long periods. The primary diagnostic footprint is an exponential upward kick in water production ($W_p$) signaling water front breakthrough.

---

## 🛠️ Data Strategy & Feature Engineering

To evaluate the reservoir purely on thermodynamic variations—independent of asset or field size—the training pipeline avoids absolute raw volumes and structures **8 dynamic physical features** calculated via Pandas:

1. `Current_Reservoir_Pressure_psi`: Tracks structural grid energy depletion.
2. `Cumulative_Oil_Np_stb`: Used as the core physical variable representing true underground volumetric voidage instead of operational time.
3. `Producing_GOR_R_scf_stb`: Captures gas evolution and coning tendencies.
4. `Cumulative_Gas_Gp_scf`: Total produced gas phase volume.
5. `Cumulative_Water_Wp_bbl`: Normalized diagnostic tool for active aquifer breakthroughs.
6. `Oil_Production_Rate_qo_bbl_day`: Tracks structural drawdown decay.
7. `Water_Production_Rate_qw_bbl_day`: Captures onset and speed of water cut.
8. `Current_Solution_Gas_Ratio_Rs_scf_stb`: Direct tracker of fluid phase thermodynamic behavior.

### High-Fidelity Data Synthesizer
The dataset comprises **10,400 rows** structured as **13 distinct oil fields per drive mechanism** (200 months of history per field). To simulate real-world operations, the underlying script injects up to **14%–16% multi-phase allocation scatter noise**, mimicking back-pressure variations, separator dump valve fluctuations, and gauge calibration drifts.

---

## 💻 Machine Learning Pipeline Workflow

### 1. Dynamically Randomized Field Split (Zero Leakage)
To prevent target leakage, data splitting is performed **at the asset level rather than the row level**. From each drive category, 3 fields are randomly isolated into the testing pool while the remaining 10 form the training matrix.
* **Training Set:** 40 random, complete fields (8,000 rows)
* **Testing Set:** 12 completely unseen fields (2,400 rows)

### 2. Random Forest Classifier Implementation
Tree-based ensemble models excel at capturing the sharp, non-linear step-changes characteristic of reservoir transitions (such as water breakthroughs or crossing $P_b$). Because decision trees isolate data via threshold splits (e.g., `Water_Cut > 0.45`), the algorithm completely bypasses the need for feature scaling, preserving the raw physical units.

### 3. Asset-Level Aggregation Wrapper (The Noise Killer)
Instead of accepting independent row-by-row classifications which can fluctuate on noisy operational days, the custom wrapper loops through each unseen test field:

$$\text{Field Probability Profile} = \frac{1}{N} \sum_{i=1}^{N} P_i(\text{Row Snapshot})$$

Taking the mean of the probability matrix over the 200-month timeline filters out unstructured noise, leaving the true dominant drive mechanism to rise to the top.

---

## 📊 Results & Evaluation

When evaluated across the completely unseen blind field trials, the model achieves highly robust, realistic performance metrics:

### Field-Level Classification Report
```text
=============================================================
             FIELD-LEVEL CLASSIFICATION REPORT             
=============================================================
               precision    recall  f1-score   support

Expansion (0)       1.00      1.00      1.00       3.0
Depletion (1)       1.00      1.00      1.00       3.0
  Gas Cap (2)       1.00      1.00      1.00       3.0
Water Drive (3)     1.00      1.00      1.00       3.0

     accuracy                           1.00      12.0
    macro avg       1.00      1.00      1.00      12.0
 weighted avg       1.00      1.00      1.00      12.0
-------------------------------------------------------------
