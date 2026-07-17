# Interpretable machine learning & metrological uncertainty propagation in V-grooving-assisted sheet metal forming

This repository contains the complete, reproduction-ready Python workflow for the predictive modeling, explainable AI (XAI) analysis, and ISO-GUM quantitative uncertainty propagation framework presented in the manuscript:
> **"Interpretable machine learning for bending angle prediction in V-grooving-assisted sheet metal forming"** Thầy Tuấn

---

##  Experimental and hardware baseline
The underlying dataset consisting of $81$ unique experimental runs ($3^4$ Full Factorial Design) was generated under rigorous laboratory conditions at the Ho Chi Minh City University of Technology (HCMUT):
* **Material:** SS304 Stainless Steel sheets (_t_ $=~1.2$ mm thickness) sourced from a single metallurgical batch.
* **Forming press:** AMADA CB83 hydraulic press brake retrofitted from an open-loop system to a closed-loop configuration utilizing a MOOG D633 servo valve and a PI/AFSMC (Proportional-Integral/Adaptive Fuzzy Sliding Mode Control) algorithm, guaranteeing a positioning error of $≤$ $0.09$ mm.
* **V-grooving:** Machined via a CNC V-grooving system using indexable carbide inserts (CNMG & SNMG series) to produce precise depths, tool nose radii, and angles.
* **Metrology:** Terminal bending angles measured using a Mitutoyo Beyond-A504 Coordinate Measuring Machine (CMM) under strictly controlled environmental conditions ($20 ± 2$°C), in compliance with DIN EN ISO 10360-2.

---

##  Computational architecture & pipeline logic

The Python source code in this repository is architected into $9$ sequential execution blocks, establishing a transparent, reproducible pipeline from raw experimental data ingestion to non-linear explainable modeling and metrological uncertainty propagation.

### 1. Preprocessing and statistical benchmarking (Blocks $1–4$)
* **Block 1: Environment setup & journal-style visualization:** Initializes output directories and enforces strict international journal publication aesthetics. It locks global `matplotlib` parameters, including `Times New Roman` typography, `STIX` math fonts, high-DPI rendering ($600$ DPI), and explicit black axes/tick coloration, to eliminate default software styling.
* **Block 2: Data ingestion and column normalization:** Loads the raw experimental sheet and applies custom regex normalizers to mitigate symbol mutations (_e.g._, matching variations like `°`, `º`, `[o]`, and `[O]`). It extracts the standardized feature space ($X$: Grooving depth $-$ A, Grooving angle $-$ B, Nose radius $-$ C, Rolling direction $-$ D) and the target vector ($y$: Mean angle) before serializing them to disk (`X.joblib`, `y.joblib`).
* **Block 3 & 4: SVR pipeline construction & grid search optimization:** Wraps data transformation (`StandardScaler`) and Support Vector Regression within a unified `TransformedTargetRegressor` pipeline to prevent data leakage during validation. It executes an intensive hyperparameter grid search optimized via leave-one-out cross-validation (LOO-CV). The cross-validated iterations select the optimal non-linear configuration for the final Radial Basis Function (RBF) surrogate model ($C=10, \epsilon=0.01, \gamma=0.1$).

### 2. Fair model comparison & non-parametric validation (Blocks $5–6$)
* **Block 5: Paired RSM _vs._ SVR-RBF LOO-CV benchmarking:** Evaluates the optimized SVR-RBF surrogate against a conventional reduced-order polynomial Response Surface Methodology (RSM) baseline over identical cross-validation folds. It computes paired absolute prediction errors ($AE$) and executes a one-sided Wilcoxon signed-rank test. This statistically evaluates whether the error reduction achieved by the machine learning model ($R^2_{\text{pred}} = 97.44\%$, $RMSE_{\text{pred}} = 0.7801^\circ$) over the quadratic RSM baseline ($R^2_{\text{pred}} = 96.16\%$) is statistically meaningful ($p < 0.001$), generating a standalone error-distribution boxplot.
* **Block 6: Performance graphics generation:** Automatically generates and exports publication-ready parity plots for the Linear, Polynomial, and RBF SVR kernels alongside side-by-side performance bar subplots comparing baseline configurations against grid-searched variants.

### 3. Explainable AI and sensitivity profiling (Block $7$)
* **Block 7: Game-theoretic deconstruction (SHAP framework):** Employs `shap.KernelExplainer` using a stratified background dataset to deconstruct the non-linear SVR-RBF decision boundary into additive feature attributions:
  * *Global importance matrix:* Aggregates mean absolute Shapley values to rank parameters (A $>$ C $>$ B $>$ D), validating the hierarchy against experimental ANOVA metrics.
  * *SHAP summary & dependence plots:* Maps the distribution of parameter impacts across the $81$ experimental trials and extracts the isolated non-linear dependency profile of the dominant factor (grooving depth $-$ A).
  * *Robustness perturbation test:* Evaluates the structural stability of the explainer by injecting $2$% multiplicative Gaussian noise into the data stream, verifying that the generated feature attributions remain stable under operational measurement variance.

### 4. Localized boundary testing & metrological propagation (Blocks $8–9$)
* **Block 8: Boundary case study discrepancy analysis:** Tests both the analytical RSM polynomial equation and the SVR-RBF model at representative boundary parameters (low, intermediate, and severe operational limits) to isolate local prediction discrepancies in critical forming regions.
* **Block 9: ISO-GUM law of uncertainty propagation:** Connects physical hardware tolerances with data-driven predictions in compliance with international ISO-GUM guidelines:
  1. *Numerical local sensitivity:* Because the non-linear SVR-RBF mapping function lacks a direct analytical derivative, the pipeline uses a central finite difference numerical approximation method ($h = 10^{-5}$) to evaluate local sensitivity coefficients (partial derivatives *∂f/∂xᵢ*) at any given design point.
  2. *Metrological budgeting:* Propagates standard physical measurement errors (_u_(A) $=~1.25$ %_t_, _u_(B) $=~0.15$°, _u_(C) $=~0.01$ mm, _u_(D) $=~0.0$) through the local gradients and combines them with the surrogate's intrinsic approximation variance ($u_{\text{model}} = RMSE_{\text{pred}} = 0.7801°$) to compute the overall combined standard uncertainty.
  3. *Expanded confidence limits:* Evaluates the final expanded uncertainty at a $95$% confidence level using a coverage factor of $k = 2$, outputting comprehensive prediction intervals and generating a journal-style parity plot equipped with explicit metrological error bars for the entire dataset.
