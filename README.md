


This repository contains an end-to-end A/B testing and hypothesis testing workflow on a mobile game dataset (Cookie Cats).

The analysis focuses on how changing a game progression gate (from level 30 to level 40) affects player engagement and retention.



---



## 1. Project Overview



**Goal:**

Evaluate the impact of moving the first progression gate in a mobile puzzle game from **level 30** to **level 40** on:



* Total game rounds played in the first week

* Day-1 retention (did players return the next day?)

* Day-7 retention (did players return within a week?)



**What this project demonstrates:**



* Practical A/B testing on real product data

* Full statistical testing pipeline:



&nbsp; * Normality checks (Shapiro–Wilk)

&nbsp; * Variance homogeneity checks (Levene)

&nbsp; * Parametric tests (t-test / Welch)

&nbsp; * Non-parametric test (Mann–Whitney U)

* Handling outliers and understanding player behavior

* Translating statistical results into product recommendations



The core analysis lives in the notebook:



* `notebooks/a-b-testing-step-by-step-hypothesis-testing.ipynb`



---



## 2. Business Problem



The game uses **gates** that slow player progression and can encourage in-app purchases.

An experiment was run to test whether moving the **first gate**:



* from **level 30** (control: `gate_30`)

* to **level 40** (treatment: `gate_40`)



would improve player engagement and retention.



Key questions:



1. Does changing the gate level alter total game rounds played?

2. Does the change improve Day-1 and Day-7 retention?

3. Is any observed difference statistically significant or just noise?



---



## 3. Dataset



The dataset contains **90,189 players** who installed the game while the A/B test was running.



| Column           | Type   | Description                                                       |

| ---------------- | ------ | ----------------------------------------------------------------- |

| `userid`         | int    | Unique identifier for each player                                 |

| `version`        | object | Experimental group: `gate_30` (control) or `gate_40` (treatment)  |

| `sum_gamerounds` | int    | Number of game rounds played in the first week after installation |

| `retention_1`    | bool   | `True` if the player returned 1 day after install, else `False`   |

| `retention_7`    | bool   | `True` if the player returned 7 days after install, else `False`  |



Players are randomly assigned to one of the two gate versions, ensuring a valid A/B test setup.



---



## 4. Analysis Workflow (Notebook Structure)



The notebook follows a clear, step-by-step structure:



1. **Import Libraries**



&nbsp;  * `numpy`, `pandas`, `seaborn`, `matplotlib`

&nbsp;  * `scipy.stats` (Shapiro–Wilk, Levene, t-test, Mann–Whitney U)

&nbsp;  * Display and warning configuration



2. **Load Data**



&nbsp;  * Custom `load(path, info=True)` function:



&nbsp;    * Imports `.csv` or `.xlsx`

&nbsp;    * Prints dimensions, dtypes, missing values, and memory usage

&nbsp;  * Verifies there are **no missing values** and **90,189 unique users**



3. **Summary Statistics**



&nbsp;  * Descriptive statistics for `sum_gamerounds`:



&nbsp;    * Heavy right skew, with a very large maximum value

&nbsp;  * Group-level stats by `version` (gate_30 vs gate_40)

&nbsp;  * Visualizations:



&nbsp;    * Histograms by group

&nbsp;    * Boxplots comparing distributions



4. **Outlier Handling**



&nbsp;  * Removes the single extreme outlier (`sum_gamerounds` at the max)

&nbsp;  * Recomputes summary statistics and visualizations

&nbsp;  * Updated distribution remains right-skewed but more stable



5. **Behavioral Insights ("Some Details")**



&nbsp;  * A substantial number of users install but never play:



&nbsp;    * 3,994 users with `sum_gamerounds = 0`

&nbsp;  * Most players churn early, with counts decreasing sharply as rounds increase

&nbsp;  * Explores:



&nbsp;    * Why players might churn quickly

&nbsp;    * How difficulty, competing games, and in-game rewards might affect retention

&nbsp;  * Visualizations:



&nbsp;    * Number of users by `sum_gamerounds` (full range and first 200 rounds)



6. **A/B Testing & Hypothesis Testing**



&nbsp;  * Re-labels groups:



&nbsp;    * `gate_30 → A`

&nbsp;    * `gate_40 → B`

&nbsp;  * Derived features:



&nbsp;    * `Retention` (1 if retained at both Day 1 and Day 7, else 0)

&nbsp;    * `NewRetention` (string combination of `retention_1` and `retention_7`, e.g. `"True-False"`)

&nbsp;  * Grouped statistics:



&nbsp;    * Summary metrics by `version`, `retention_1`, `retention_7`, `Retention`, and `NewRetention`

&nbsp;  * **AB_Test function** encapsulates the full statistical pipeline:



&nbsp;    * Split data into A and B

&nbsp;    * Check normality (Shapiro–Wilk)

&nbsp;    * If both groups are normal:



&nbsp;      * Check variance homogeneity (Levene)

&nbsp;      * Use t-test (equal or unequal variance)

&nbsp;    * Otherwise:



&nbsp;      * Use Mann–Whitney U test

&nbsp;    * Return:



&nbsp;      * Test type (Parametric / Non-parametric)

&nbsp;      * p-value

&nbsp;      * Decision on H0 (A = B)

&nbsp;      * Human-readable comment ("A/B groups are similar" / "not similar")



7. **Conclusion**



&nbsp;  * Summarizes business context, methods, and statistical outcome

&nbsp;  * Provides a product recommendation based on the results



---



## 5. Key Results



### Data and Retention



* No missing values in the dataset.

* Many players never meaningfully engage (e.g., 3,994 users with 0 rounds).

* Churn is high:



&nbsp; * ~55% do not return on **Day 1**

&nbsp; * ~81% do not return by **Day 7**

* Roughly **14%** of users play both on Day 1 and Day 7, indicating potential long-term players.



### A/B Test Outcome



* The distribution of `sum_gamerounds` is **not normal**, so a **non-parametric Mann–Whitney U test** is appropriate.

* The test yields:



&nbsp; * p-value ≈ **0.025**

&nbsp; * Decision: **Reject H0** → A/B groups differ statistically.

* Retention comparison:



&nbsp; * Day-1 retention is slightly higher for the gate at **level 30**.

&nbsp; * Day-7 retention is also slightly higher for the gate at **level 30**.



**Interpretation:**

Within this experiment, keeping the gate at **level 30** appears **better or at least safer** for player retention and engagement than moving it to level 40.

However, retentions are close, and additional data or follow-up experiments would help confirm long-term effects.



---



## 6. A/B Testing Methodology



The project uses a standard, reusable A/B testing framework:



1. **Define hypotheses**



&nbsp;  * H0: Metric for group A = Metric for group B

&nbsp;  * H1: Metric for group A ≠ Metric for group B



2. **Check assumptions**



&nbsp;  * Normality (Shapiro–Wilk) for each group

&nbsp;  * If normal:



&nbsp;    * Variance homogeneity (Levene)

&nbsp;    * Choose between t-test (equal variances) or Welch t-test (unequal variances)

&nbsp;  * If non-normal:



&nbsp;    * Use Mann–Whitney U test directly



3. **Compute p-value**



&nbsp;  * Compare p-value to α (0.05)

&nbsp;  * Reject or fail to reject H0



4. **Translate to business decision**



&nbsp;  * Move from “p = 0.025” to:



&nbsp;    * “Gate at level 30 performs better in terms of engagement and retention under current data.”



The `AB_Test` helper function makes this process reusable for other metrics and experiments.



---



## 7. Project Structure



Recommended minimal structure:



```text

.

├─ data/

│  └─ cookie_cats.csv

│

├─ notebooks/

│  └─ a-b-testing-step-by-step-hypothesis-testing.ipynb

│

├─ README.md

└─ requirements.txt

```



---



## 8. How to Run the Analysis



1. **Clone the repository**



```bash

git clone <your-repo-url>.git

cd <your-repo-folder>

```



2. **Create and activate a virtual environment (optional but recommended)**



```bash

python -m venv venv

venvScriptsactivate  # On Windows

# source venv/bin/activate  # On macOS/Linux

```



3. **Install dependencies**



```bash

pip install -r requirements.txt

```



Minimal `requirements.txt` example:



```text

numpy

pandas

scipy

seaborn

matplotlib

jupyter

```



4. **Launch Jupyter Notebook**



```bash

jupyter notebook

```



Then open:



* `notebooks/a-b-testing-step-by-step-hypothesis-testing.ipynb`



and run all cells.



---



## 9. Learning Resources



The project also serves as a learning companion for A/B testing and statistics.

Helpful external resources include:



* **DataCamp**



&nbsp; * Mobile Games A/B Testing with Cookie Cats

&nbsp; * A/B Testing in R

&nbsp; * Customer Analytics and A/B Testing in Python



* **Udacity**



&nbsp; * A/B Testing by Google – Online Experiment Design and Analysis

&nbsp; * Intro to Inferential Statistics

&nbsp; * Statistics – The Science of Decisions



These courses reinforce topics such as hypothesis testing, bootstrap analysis, experimental design, and interpreting p-values in a business context.



---



## 10. Acknowledgements



* Original dataset and inspiration from **Aurelia Sui (@yufengsui)** and the **DataCamp** Cookie Cats A/B testing project.

* Cookie Cats is developed by **Tactile Entertainment**; all game-related assets and branding belong to their respective owners.



This repository focuses purely on analytical methods and educational purp



