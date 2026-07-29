# Co-occurring occupational hazard exposure profiles — 7th Korean Working Conditions Survey

Analysis code and derived results for the study:

> **Co-occurring Occupational Hazard Exposure Profiles and Their Associations With Presenteeism, Sickness Absence, and Work–Life Balance: An Unsupervised Clustering Analysis of the 7th Korean Working Conditions Survey**

*Manuscript under review. Citation details will be added on acceptance.*

---

## What this study does

Occupational-health surveillance usually studies one hazard at a time, but workers meet
hazards in bundles. Using the 7th Korean Working Conditions Survey (2023, N = 44,125
working-age respondents in employment — unweighted 62.7% wage employees, 23.4% employers, 6.8% own-account self-employed, 7.1% unpaid family workers; survey-weighted 78.4%, 13.5%, 5.3%, 2.8%), we z-standardized **18 self-reported hazard-exposure items** across
physical, ergonomic, and psychosocial domains and partitioned workers with **k-means
clustering**. Four outcomes were deliberately **withheld** from clustering and only then
compared across the resulting profiles.

**Four profiles emerged** (bootstrap adjusted Rand index = 0.92 over 500 replicates):

| Profile | n | Weighted % | Physical (z) | Ergonomic (z) | Psychosocial (z) |
|---|---:|---:|---:|---:|---:|
| Low-exposure/office-sedentary | 19,924 | 46.0 | −0.45 | −0.35 | −0.39 |
| Interpersonal/service | 10,214 | 20.5 | −0.35 | +0.32 | **+0.71** |
| Physical-environmental | 12,034 | 28.2 | **+0.62** | +0.16 | −0.08 |
| High-intensity multi-hazard | 1,953 | 5.2 | **+2.58** | +0.86 | +0.79 |

The key structural result is that the workforce varies along **two** axes, not one: overall
exposure *intensity* (PC1, 36.5% of variance) **and** a physical-versus-interpersonal
*composition* axis (PC2, 12.6%). The Interpersonal/service profile reports low physical
exposure and would be scored "low-risk" by any intensity-based screen, yet carries one of the
highest presenteeism burdens — which is the finding with the most direct prevention relevance.

---

## What the notebook produces (revised 2026-07-28)

Running the notebook top to bottom regenerates **every figure, table and reported statistic**
in the article and its Supporting Information, including:

| Cell | Output |
|---|---|
| 2c | Table SIII — the 18-item battery, response scale, distributions, Cronbach's α by domain |
| 4c | Table SV — domain-mean profile of **every class under all four** GMM covariance structures |
| 4d | Table SVI — Ward ARI across 8 subsamples; domain-balanced and domain-mean re-clustering; k = 5 |
| 6b | Table III — unadjusted and adjusted models with the **survey sandwich** variance estimator |
| 6c | Table SIV — presenteeism denominator diagnostics and sensitivity; Table SIX — general-health indicators by profile |
| 6d | Table SVII — finer control for job structure (3-digit occupation, industry, weekly hours) |

Two points a reader checking the code should know:

- **Survey weights are not frequency weights.** Cell 6b builds the pseudo-maximum-likelihood
  sandwich `B · Σ wᵢ²(yᵢ−p̂ᵢ)²xᵢxᵢ′ · B` explicitly. Passing the weights to `freq_weights=`
  with `cov_type="HC1"` — the natural-looking shortcut — understates every standard error by
  26–40% in these data. Earlier versions of this notebook did exactly that.
- **The mixture BIC is not usable here.** With discrete 7-point inputs, the diagonal and
  unconstrained components approach variance degeneracy (minimum eigenvalue pinned at
  `reg_covar` = 1e-6), so the log-likelihood is a function of the regulariser. Cell 4c prints
  the minimum eigenvalue alongside each BIC so this is visible rather than implicit.

`B_BOOT = 500` in Cell 4b took about 190 s for k = 2–8 on a 10-core Apple-silicon machine (2026-07-28); set it to `0` to skip the bootstrap sweep.

---

## Reproducing the analysis

### 1. Obtain the data (not included here)

The raw microdata is **not** in this repository and must not be redistributed. The 7th Korean
Working Conditions Survey is de-identified public-use microdata released by the **Korea
Occupational Safety and Health Agency (KOSHA)** to researchers on application through the
KWCS data portal. Apply to KOSHA directly.

Place the survey CSV where the notebook's first cell expects it and set `BASE` accordingly.
Absolute paths in the committed notebook have been replaced with `<PROJECT_ROOT>`.

### 2. Environment

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Python 3.10. Random seeds are fixed and linear-algebra threading is pinned, so a full run is
deterministic and regenerates every figure, table, and reported statistic from the raw survey
file in one pass.

### 3. Run

Open `KWCS7_exposure_profiles.ipynb` and run all cells.

---

## Repository layout

```
KWCS7_exposure_profiles.ipynb   Full analysis: cleaning -> clustering -> stability -> models
results/                        Derived aggregate results (all profile-level; no individual records)
  ├── table1_profile_z.csv              Standardized item means by profile (Table I / Figure 3)
  ├── table1_summary.csv                Profile sizes, weighted shares, demographics (Table I)
  ├── table2_validation.csv             Outcome prevalences, unweighted and survey-weighted (Table II)
  ├── table3_adjusted.csv               Unadjusted and adjusted models, survey sandwich SEs (Table III)
  ├── tableS2_bootstrap500.csv          Bootstrap reproducibility, k = 2-8, 500 replicates (Table SII)
  ├── tableS3_items.csv                 The 18-item battery: labels, distributions (Table SIII)
  ├── tableS4_presenteeism_sensitivity.csvPresenteeism denominator sensitivity (Table SIV)
  ├── tableS5_gmm_profiles.csv          GMM cross-check, every class under 4 covariances (Table SV)
  ├── tableS7_finer_adjustment.csv      Finer control for job structure (Table SVII)
  ├── tableS8_wage_employees_only.csv   Wage-employee-only sensitivity (Table SVIII)
  ├── tableS_gmm_crosscheck.csv         GMM ARI / label agreement / entropy summary
  ├── tableS_selection.csv              Cluster-number selection indices (Table SI)
  ├── table_effect_sizes.csv            Cramér's V, eta-squared, design-adjusted Wald tests
  └── table_occ_share.csv               Occupational composition by profile (Figure 4)
figures/                        Manuscript figures (PNG + PDF)
tables/                         Manuscript tables (CSV)
```

Everything in `results/` and `tables/` is aggregated to the profile level (4–27 rows per file).
No individual-level records are published in this repository.

---

## Methods notes worth knowing before reusing this code

- **Clustering is unweighted; description is weighted.** Profiles are defined by the geometry
  of the standardized exposure space, so survey weights are not used to form clusters. Weights
  are reintroduced when reporting how prevalent each profile is in the working population.
- **k = 4 was not chosen by any single index.** Because all 18 hazards are positively
  intercorrelated, the silhouette and the Calinski–Harabasz index both favour the trivial
  k = 2 intensity split, and the Davies–Bouldin index is non-monotonic. k = 4 was selected as the smallest solution separating exposure
  composition from intensity, with the inertia elbow concurring; bootstrap stability was NOT
  used to select k.
  `results/tableS_selection.csv` has the full index table.
- **Bootstrap stability is high at every k** (mean ARI 0.92–0.99). It is therefore evidence
  that the chosen partition reproduces, *not* a criterion for choosing k.
- **The Gaussian-mixture cross-check depends on the covariance assumption.** Diagonal and
  unconstrained mixtures absorb composition into within-class variance and grade purely along
  the intensity axis, but the tied (homoscedastic, canonical-LPA) specification independently
  recovers both the interpersonal-composition class (physical z = −0.25, psychosocial +0.51;
  35.2%) and the extreme class (physical +2.62). The low overall ARI vs k-means (0.14–0.35)
  reflects different boundary placement (35.2% vs 20.5%), not absence of the compositional
  axis — see `results/tableS5_gmm_profiles.csv`.
- **Design-adjusted inference is approximate.** Primary-sampling-unit identifiers are not
  released in the public-use file, so inference uses weight-normalized pseudo-maximum-likelihood
  models with the survey sandwich variance estimator built explicitly in Cell 6b, rather than
  a full survey design object.
- **Odds ratios are on the survey-weighted scale** and reproduce from the weighted, not the
  unweighted, prevalences.

---

## Licence

Code and derived results: MIT (see `LICENSE`).
The KWCS microdata is not covered by this licence and is not distributed here.

## Funding

This research was supported by the ANCHOR program through the Gyeongbuk ANCHOR Center, funded
by the Ministry of Education (MOE) and Gyeongsangbuk-do, Republic of Korea (2026-ANCHOR-15-102).
