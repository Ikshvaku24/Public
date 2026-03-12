# Meridian Mathematics Guide

This guide explains the mathematics behind Meridian from the perspective of the demo notebook `meridian/demo/Meridian_Getting_Started.ipynb`.

The goal is to answer four questions:

1. What data does Meridian consume?
2. What mathematical model does it fit?
3. What parameters does it learn or sample?
4. How does it turn those parameters into diagnostics, contribution, ROI, and budget optimization?

This document is based on the implementation in:

- `meridian/meridian/data/input_data.py`
- `meridian/meridian/data/data_frame_input_data_builder.py`
- `meridian/meridian/model/context.py`
- `meridian/meridian/model/transformers.py`
- `meridian/meridian/model/media.py`
- `meridian/meridian/model/adstock_hill.py`
- `meridian/meridian/model/equations.py`
- `meridian/meridian/model/spec.py`
- `meridian/meridian/model/prior_distribution.py`
- `meridian/meridian/model/posterior_sampler.py`
- `meridian/meridian/model/model.py`
- `meridian/meridian/analysis/analyzer.py`
- `meridian/meridian/analysis/optimizer.py`

---

## 1. Big-picture view

At a high level, Meridian builds a Bayesian hierarchical media mix model over a panel of data indexed by:

- geo `g`
- time `t`
- media channel `m`
- reach-frequency channel `r`
- control variable `c`
- non-media treatment `n`

It tries to explain the observed KPI as:

\[
\text{KPI} = \text{baseline} + \text{time trend} + \text{media effects} + \text{controls} + \text{non-media effects} + \text{noise}
\]

The model is not fit by ordinary least squares. It is fit by Bayesian posterior inference using NUTS, which means it learns a distribution over parameters rather than a single point estimate.

The end result is a probabilistic description of:

- expected KPI by geo and time
- media contribution
- ROI and marginal ROI
- uncertainty in those quantities
- predicted impact of changing budget allocations

---

## 2. End-to-end system flow

The complete flow of Meridian is:

1. Raw tables are converted into structured `InputData`.
2. `InputData` is converted into model tensors.
3. KPI, media, controls, and treatments are transformed and normalized.
4. Media is pushed through carryover and saturation functions.
5. A hierarchical Bayesian generative model is constructed.
6. Priors are applied either on coefficients or on business quantities such as ROI and contribution.
7. NUTS samples the posterior.
8. Posterior draws are turned into expected outcome, incremental outcome, ROI, response curves, and optimized budgets.

In symbols:

\[
\text{Raw Data}
\rightarrow
\text{InputData}
\rightarrow
\text{Scaled Tensors}
\rightarrow
\text{Adstock/Hill Features}
\rightarrow
\text{Bayesian Hierarchical Model}
\rightarrow
\text{Posterior Samples}
\rightarrow
\text{Attribution / Forecasting / Optimization}
\]

---

## 3. Input data and notation

### 3.1 Core observed data

Meridian organizes the data into the following mathematical objects.

- KPI:
\[
y_{g,t}
\]

- Population:
\[
\text{pop}_g
\]

- Controls:
\[
z_{g,t,c}
\]

- Revenue per KPI:
\[
v_{g,t}
\]

- Paid media units:
\[
x_{g,\tau,m}
\]

- Paid media spend:
\[
s_{g,t,m}
\]

- Reach-frequency data:
\[
\text{reach}_{g,\tau,r}, \quad \text{freq}_{g,\tau,r}
\]

- Non-media treatments:
\[
n_{g,t,n}
\]

- Organic media and organic RF are analogous to paid media and RF, but without spend denominators.

Here `t` indexes KPI time and `\tau` indexes media time. Meridian allows the media time dimension to be longer than the KPI time dimension so lagged effects can be modeled.

### 3.2 Why media time can be longer than KPI time

If media has carryover, then KPI this week can depend on media from previous weeks. So Meridian allows:

\[
n_{\text{media times}} \ge n_{\text{times}}
\]

and typically recommends including up to `max_lag` additional historical periods.

---

## 4. The pre-model transformations

Meridian does not fit raw KPI and raw media directly. It transforms them before they enter the statistical model.

### 4.1 KPI transformation

First, KPI is divided by population:

\[
\tilde{y}_{g,t} = \frac{y_{g,t}}{\text{pop}_g}
\]

Then it is globally standardized:

\[
y^{\text{scaled}}_{g,t} =
\frac{\tilde{y}_{g,t} - \mu_y}{\sigma_y}
\]

where:

\[
\mu_y = \text{mean}_{g,t}(\tilde{y}_{g,t}), \qquad
\sigma_y = \text{sd}_{g,t}(\tilde{y}_{g,t})
\]

Interpretation:

- dividing by population puts geos on a comparable per-capita scale
- standardization improves numerical stability and makes priors easier to share across data sets

### 4.2 Paid media transformation before adstock/Hill

For paid media units:

\[
\tilde{x}_{g,\tau,m} = \frac{x_{g,\tau,m}}{\text{pop}_g}
\]

Then Meridian computes the non-zero median of the population-scaled media for each channel:

\[
\text{med}_m = \text{median of non-zero values of } \tilde{x}_{g,\tau,m}
\]

Then it scales:

\[
x^{\text{scaled}}_{g,\tau,m}
=
\frac{x_{g,\tau,m}}{\text{pop}_g \cdot \text{med}_m}
\]

This makes media channels more comparable and helps the response curves behave consistently across channels.

### 4.3 Reach transformation

Reach is transformed the same way as media units:

\[
\text{reach}^{\text{scaled}}_{g,\tau,r}
=
\frac{\text{reach}_{g,\tau,r}}{\text{pop}_g \cdot \text{med}^{\text{reach}}_r}
\]

Frequency is not population-scaled in the same way. It is used directly inside the RF response function.

### 4.4 Controls and non-media treatment transformation

Controls and non-media treatments are optionally population-scaled, then standardized.

For a generic variable `u_{g,t,k}`:

\[
u^*_{g,t,k}
=
\frac{u_{g,t,k} / a_{g,k} - \mu_k}{\sigma_k}
\]

where:

- `a_{g,k} = pop_g` if that variable is marked for population scaling
- otherwise `a_{g,k} = 1`

Interpretation:

- controls become zero-centered, unit-scale regressors
- non-media treatments are handled similarly, but can also be compared to a user-defined baseline for attribution

---

## 5. Media response math

The central idea in Meridian is that media does not affect KPI linearly in raw form. It goes through:

1. carryover over time
2. diminishing returns

These are modeled with Adstock and Hill functions.

### 5.1 Adstock

Adstock captures lagged carryover. The intuitive form is:

\[
\text{Adstock}_{g,t,m}
\approx
\sum_{\ell=0}^{L}
w_{\ell,m} \, x_{g,t-\ell,m}
\]

where:

- `L = max_lag`
- `w_{\ell,m}` are decay weights

In the geometric case, Meridian builds weights proportional to:

\[
w_{\ell,m} \propto \alpha_m^\ell
\]

and then normalizes them so the weights sum to one over the lag window.

Interpretation of `alpha_m`:

- `alpha_m` near `0` means little carryover
- `alpha_m` near `1` means long carryover

Meridian also supports a binomial decay form. In that case the unnormalized weights are:

\[
w_{\ell,m} \propto \left(1 - \frac{\ell}{W}\right)^{(1/\alpha_m)-1}
\]

where `W` is the lag window size.

### 5.2 Hill saturation

Hill captures diminishing returns:

\[
\text{Hill}(x; ec, s)
=
\frac{x^s}{x^s + ec^s}
\]

Interpretation:

- `ec` is the half-saturation level
- `s` is the curvature or steepness

When `x = ec`, the Hill output is `1/2`.

### 5.3 Order of transformations for standard media

For regular paid media or organic media, Meridian supports two orders:

- default:
\[
\text{Hill}(\text{Adstock}(x))
\]

- optional when `hill_before_adstock=True`:
\[
\text{Adstock}(\text{Hill}(x))
\]

This matters because "saturate then carry over" is not the same as "carry over then saturate."

### 5.4 Reach-frequency channels

Reach-frequency channels use a different structure. Meridian applies Hill to frequency, multiplies by reach, then adstocks the result:

\[
\text{RFResponse}_{g,t,r}
=
\text{Adstock}\left(
\text{reach}_{g,t,r}^{\text{scaled}}
\cdot
\text{Hill}(\text{freq}_{g,t,r}; ec_r, s_r)
\right)
\]

Interpretation:

- reach measures how many unique people are touched
- Hill on frequency captures that repeating exposures help at first but saturate
- the product approximates effective impressions

---

## 6. Time effects and intercept structure

Meridian uses a baseline made of:

1. geo-specific intercept
2. time-varying intercept

### 6.1 Geo intercept

For each geo:

\[
\tau_g
\]

One geo is treated as the baseline geo. Its intercept is fixed at zero, and the remaining geos are modeled relative to it.

### 6.2 Time-varying intercept

Meridian also uses a shared time-varying baseline:

\[
\mu_t
\]

This is built from knot values and knot weights:

\[
\mu_t = \sum_{k=1}^{K} \text{knot\_value}_k \, w_{k,t}
\]

where:

- `K` is the number of knots
- `w_{k,t}` are interpolation weights derived from knot locations

Interpretation:

- this is a flexible latent time trend
- it absorbs common changes over time not explained by media or controls

### 6.3 Combined baseline

The baseline part of the model is:

\[
\tau_g + \mu_t
\]

So yes, Meridian does use a time-varying intercept component.

### 6.4 Knot selection

If knots are manually provided, Meridian uses those locations.

If `knots=None`:

- geo model: default can be very flexible
- national model: default is much simpler

If `enable_aks=True`, Meridian runs Automatic Knot Selection to choose the time-trend flexibility from the data.

---

## 7. The full generative model

After all transformations, Meridian models the scaled KPI as Gaussian:

\[
y^{\text{scaled}}_{g,t}
\sim
\mathcal{N}( \hat{y}_{g,t}, \sigma_g )
\]

or, if residual variance is shared across geos:

\[
y^{\text{scaled}}_{g,t}
\sim
\mathcal{N}( \hat{y}_{g,t}, \sigma )
\]

The linear predictor is:

\[
\hat{y}_{g,t}
=
\tau_g + \mu_t
+ \sum_m f^{(m)}_{g,t,m}\beta_{g,m}
+ \sum_r f^{(rf)}_{g,t,r}\beta_{g,r}^{(rf)}
+ \sum_o f^{(om)}_{g,t,o}\beta_{g,o}^{(om)}
+ \sum_q f^{(orf)}_{g,t,q}\beta_{g,q}^{(orf)}
+ \sum_c z^*_{g,t,c}\gamma_{g,c}
+ \sum_n n^*_{g,t,n}\gamma_{g,n}^{(nm)}
\]

where:

- `f` terms are transformed media features after adstock/Hill
- `z^*` are normalized controls
- `n^*` are normalized non-media treatment variables

This is the central regression equation of Meridian.

---

## 8. Hierarchical parameterization across geos

The coefficients are hierarchical, which means every geo can have its own coefficient, but all geos share information through common hyperparameters.

### 8.1 Media coefficients

For media channels, Meridian supports two distributions across geos.

#### Normal random effects

\[
\beta_{g,m} = \beta_m + \eta_m z_{g,m}, \qquad z_{g,m}\sim \mathcal{N}(0,1)
\]

#### Log-normal random effects

\[
\beta_{g,m} = \exp(\beta_m + \eta_m z_{g,m}), \qquad z_{g,m}\sim \mathcal{N}(0,1)
\]

Interpretation:

- `beta_m` is the channel-level average effect parameter
- `eta_m` controls heterogeneity across geos
- log-normal effects enforce positivity

The same structure is used for:

- `beta_rf`
- `beta_om`
- `beta_orf`

### 8.2 Control coefficients

Controls use normal hierarchical effects:

\[
\gamma_{g,c} = \gamma_c + \xi_c u_{g,c}, \qquad u_{g,c}\sim \mathcal{N}(0,1)
\]

### 8.3 Non-media treatment coefficients

Non-media treatments use:

\[
\gamma_{g,n}^{(nm)} = \gamma_n + \xi_n u_{g,n}, \qquad u_{g,n}\sim \mathcal{N}(0,1)
\]

---

## 9. Priors: the most important Meridian idea

Meridian can place priors on:

- raw coefficients
- ROI
- marginal ROI
- contribution share

This is the key reason Meridian feels more business-native than a standard regression.

### 9.1 Coefficient prior

If prior type is `coefficient`, the model samples `beta_m` directly:

\[
\beta_m \sim p(\beta_m)
\]

### 9.2 ROI prior

If prior type is `roi`, the user specifies:

\[
\text{ROI}_m \sim p(\text{ROI}_m)
\]

and Meridian turns that into a coefficient.

Conceptually:

\[
\text{ROI}_m
=
\frac{\text{incremental outcome from channel } m}{\text{spend}_m}
\]

Therefore:

\[
\text{incremental outcome}_m
=
\text{ROI}_m \cdot \text{spend}_m
\]

Meridian then solves for the latent coefficient `beta_m` so that the transformed media feature implies that incremental outcome.

### 9.3 Marginal ROI prior

If prior type is `mroi`, Meridian compares the actual media level to a slightly increased media level:

\[
x' = x \cdot \text{MROI\_FACTOR}
\]

Then:

\[
\text{mROI}_m
=
\frac{\Delta \text{incremental outcome}_m}{\Delta \text{spend}_m}
\]

So:

\[
\Delta \text{incremental outcome}_m
=
\text{mROI}_m \cdot \Delta \text{spend}_m
\]

Again, it backs out the coefficient that would make the transformed media response consistent with that prior.

### 9.4 Contribution prior

If prior type is `contribution`, Meridian uses:

\[
\text{ContributionShare}_m \sim p(\text{ContributionShare}_m)
\]

and defines:

\[
\text{incremental outcome}_m
=
\text{ContributionShare}_m \cdot \text{TotalOutcome}
\]

Then it solves for the latent coefficient that makes this true on the transformed scale.

---

## 10. How Meridian converts business priors into coefficients

This conversion is one of the most important internal calculations.

Let:

- `\Delta LP_{g,t,x}` be the counterfactual difference in transformed feature space for channel `x`
- `\Delta Y_x` be the desired incremental outcome implied by ROI, mROI, or contribution priors

Then Meridian computes a coefficient parameter so that:

\[
\Delta Y_x
\approx
\sum_{g,t}
\Delta LP_{g,t,x}
\cdot
\text{coefficient}_{g,x}
\cdot
v_{g,t}
\cdot
\text{pop}_g
\cdot
\sigma_y
\]

That last factor appears because the model is fit on the scaled KPI, so Meridian must map business units back to model scale.

The exact algebra depends on whether geo-level random effects are normal or log-normal.

### 10.1 Normal random effects case

If coefficients are normal across geos, the structure is:

\[
\beta_{g,x} = \beta_x + \eta_x z_{g,x}
\]

so Meridian can solve approximately:

\[
\beta_x
=
\frac{
\Delta Y_x - \text{random-effect correction term}
}{
\text{sum of transformed exposure terms}
}
\]

### 10.2 Log-normal random effects case

If coefficients are log-normal:

\[
\beta_{g,x} = \exp(\beta_x + \eta_x z_{g,x})
\]

then Meridian solves for `beta_x` on the log scale:

\[
\beta_x
=
\log(\Delta Y_x) - \log(\text{weighted transformed exposure sum})
\]

This is how Meridian makes priors on ROI or contribution mathematically consistent with the latent model coefficients.

---

## 11. Counterfactuals used by priors

Priors on business quantities rely on counterfactual media states.

### 11.1 ROI prior counterfactual

If a calibration period is specified:

- actual scenario: historical media
- counterfactual scenario: media set to zero only during the calibration period

So the prior asks:

\[
\text{What incremental outcome was generated by media during the calibration window?}
\]

### 11.2 mROI prior counterfactual

Meridian scales media by a small multiplicative factor:

\[
x' = x \cdot \text{MROI\_FACTOR}
\]

Then it asks:

\[
\text{What extra outcome would come from that small extra spend?}
\]

### 11.3 Contribution prior counterfactual

Contribution priors effectively compare:

- actual media
- zero media

and express the channel effect as a share of total observed outcome.

---

## 12. Non-media treatment baseline math

Non-media treatments are special because attribution is defined relative to a baseline value.

For each non-media channel `n`, the baseline can be:

- minimum observed value
- maximum observed value
- a fixed numeric value

Let the baseline be:

\[
b_n
\]

Then the model uses the normalized difference:

\[
\Delta n_{g,t,n} = n^*_{g,t,n} - b_n^*
\]

for contribution-style calculations.

Interpretation:

- media is often compared to zero
- non-media treatments are often compared to a business baseline such as "no promo" or "minimum promo intensity"

---

## 13. Default prior families

Meridian ships with default priors. These are not arbitrary; they encode weak but structured assumptions.

### 13.1 Time and geo priors

- `knot_values ~ Normal(0, 5)`
- `tau_g_excl_baseline ~ Normal(0, 5)`

### 13.2 Media hierarchy priors

- `beta_* ~ HalfNormal(5)` when coefficient priors are used
- `eta_* ~ HalfNormal(1)`

### 13.3 Controls and non-media priors

- `gamma_c ~ Normal(0, 5)`
- `gamma_n ~ Normal(0, 5)`
- `xi_c ~ HalfNormal(5)`
- `xi_n ~ HalfNormal(5)`

### 13.4 Adstock and Hill priors

- `alpha_* ~ Uniform(0, 1)`
- `ec_m ~ TruncatedNormal(0.8, 0.8, low=0.1, high=10)`
- `ec_rf ~ Shift(0.1) + LogNormal(0.7, 0.4)`
- `slope_m = 1` by default
- `slope_rf ~ LogNormal(0.7, 0.4)`

### 13.5 Noise prior

- `sigma ~ HalfNormal(5)`

### 13.6 Business priors

- `roi_m ~ LogNormal(0.2, 0.9)`
- `roi_rf ~ LogNormal(0.2, 0.9)`
- `mroi_m ~ LogNormal(0.0, 0.5)`
- `mroi_rf ~ LogNormal(0.0, 0.5)`
- `contribution_m ~ Beta(1, 99)`
- `contribution_rf ~ Beta(1, 99)`
- `contribution_om ~ Beta(1, 99)`
- `contribution_orf ~ Beta(1, 99)`
- `contribution_n ~ TruncatedNormal(0.0, 0.1, low=-1, high=1)`

---

## 14. What exactly gets sampled?

Meridian samples latent variables from the prior and posterior.

The main sampled blocks are:

- `knot_values`
- `sigma`
- `tau_g_excl_baseline`
- media transform parameters: `alpha_*`, `ec_*`, `slope_*`
- hierarchical scale parameters: `eta_*`, `xi_*`
- latent geo deviations:
  - `beta_gm_dev`
  - `beta_grf_dev`
  - `beta_gom_dev`
  - `beta_gorf_dev`
  - `gamma_gc_dev`
  - `gamma_gn_dev`
- plus either:
  - coefficient means such as `beta_m`, `beta_rf`, `gamma_n`
  - or business prior variables such as `roi_m`, `mroi_m`, `contribution_m`

Then deterministic variables are reconstructed, such as:

- `tau_g`
- `mu_t`
- `beta_gm`
- `beta_grf`
- `gamma_gc`
- `gamma_gn`
- `y_pred`

---

## 15. Posterior inference

Meridian uses TensorFlow Probability's adaptive NUTS sampler.

Mathematically:

\[
p(\theta \mid y)
\propto
p(y \mid \theta) p(\theta)
\]

where:

- `\theta` is the set of all latent parameters
- `p(y | \theta)` is the likelihood from the Gaussian observation model
- `p(\theta)` is the product of all priors

NUTS then samples from the posterior.

### 15.1 Meaning of notebook training arguments

- `n_chains`: number of MCMC chains
- `n_adapt`: warmup/adaptation iterations
- `n_burnin`: additional discarded draws after adaptation
- `n_keep`: saved draws per chain
- `seed`: randomness control

This is what "training" means in Meridian:

- it is sampling-based Bayesian inference
- not gradient descent on a point estimate

---

## 16. Holdout logic

If a holdout set is supplied, Meridian excludes holdout KPI values from the fitting likelihood while still keeping media history available for lag computations.

Conceptually:

- training likelihood is removed on held-out KPI observations
- adstock can still use media from those periods where relevant

This means holdout affects fit evaluation, but not the structural media history used by the model.

---

## 17. Expected outcome and incremental outcome

Once posterior draws are available, Meridian can compute:

- expected outcome under actual data
- expected outcome under a counterfactual data set

Incremental outcome is the difference:

\[
\text{IncrementalOutcome}
=
\mathbb{E}[Y \mid \text{actual data}]
-
\mathbb{E}[Y \mid \text{counterfactual data}]
\]

For a single channel, the counterfactual often means reducing that channel while keeping others fixed.

Important Meridian assumptions during this step:

1. media effects are additive, with no interactions
2. additive changes on the transformed KPI scale are treated as additive changes on the original KPI scale after inversion

These assumptions are central to all attribution and optimization outputs.

---

## 18. ROI, mROI, and contribution after fitting

### 18.1 ROI

For a channel:

\[
\text{ROI}
=
\frac{\text{IncrementalOutcome}}{\text{Spend}}
\]

If `revenue_per_kpi` is present, this is usually revenue ROI.

If not, this can be KPI-per-spend.

### 18.2 Marginal ROI

\[
\text{mROI}
=
\frac{
\text{IncrementalOutcome}(x + \Delta x) - \text{IncrementalOutcome}(x)
}{
\text{Spend}(x + \Delta x) - \text{Spend}(x)
}
\]

This measures the return of the next extra unit of spend, not the average return.

### 18.3 Contribution

Contribution share is:

\[
\text{ContributionShare}_m
=
\frac{\text{IncrementalOutcome}_m}{\text{TotalOutcome}}
\]

This is often reported over a selected time window and set of geos.

---

## 19. Optimization math

Budget optimization is not a new model fit. It is a decision problem solved using the fitted model.

### 19.1 Core optimization problem

For a fixed budget:

\[
\max_{b_1, \dots, b_M}
\sum_{m=1}^{M}
\text{ExpectedIncrementalOutcome}_m(b_m)
\]

subject to:

\[
\sum_{m=1}^{M} b_m = B
\]

and channel-level box constraints:

\[
(1-l_m) B p_m \le b_m \le (1+u_m) B p_m
\]

where:

- `B` is total budget
- `p_m` is baseline percentage allocation
- `l_m` and `u_m` are lower and upper spend-constraint values

### 19.2 Historical flighting assumption

If historical spend for a channel is `H_m` and optimized spend is `b_m`, Meridian assumes media units scale proportionally:

\[
x'_{g,t,m}
=
x_{g,t,m} \cdot \frac{b_m}{H_m}
\]

Interpretation:

- same flighting pattern
- same cost per unit
- only total spend changes

This is one of the most important assumptions in the optimizer.

### 19.3 Flexible-budget optimization

If budget is flexible, Meridian solves until one of these conditions is met:

\[
\text{TotalROI} = \text{target\_roi}
\]

or

\[
\text{TotalMarginalROI} = \text{target\_mroi}
\]

### 19.4 RF optimization

For RF channels, Meridian can optionally search over frequency and use model-implied optimal frequency.

So for RF channels, the optimization is over budget and potentially effective frequency, subject to an upper frequency bound.

### 19.5 Grid search nature

Optimization is done over a spend grid, not continuous closed-form calculus.

`gtol` controls the spend granularity. Smaller `gtol` means:

- finer grid
- slower runtime
- more accurate budget matching

---

## 20. Parameter cheat sheet

### 20.1 Data and transformation parameters

| Parameter | Meaning |
|---|---|
| `population` | Used to convert KPI and media to a per-capita scale |
| `revenue_per_kpi` | Converts KPI-based effects into revenue effects |
| `max_lag` | Maximum number of lag periods used in carryover |
| `adstock_decay_spec` | Chooses geometric or binomial carryover for each channel |
| `hill_before_adstock` | Chooses transformation order for standard media |

### 20.2 Time and baseline parameters

| Parameter | Meaning |
|---|---|
| `tau_g` | Geo baseline effect |
| `tau_g_excl_baseline` | Non-baseline geo intercept parameters |
| `mu_t` | Time-varying intercept/trend |
| `knot_values` | Free parameters underlying `mu_t` |
| `knots` | Controls time-trend flexibility |
| `enable_aks` | Lets Meridian choose knot complexity automatically |
| `baseline_geo` | Reference geo with intercept fixed to zero |

### 20.3 Media response parameters

| Parameter | Meaning |
|---|---|
| `alpha_*` | Carryover strength |
| `ec_*` | Half-saturation point |
| `slope_*` | Saturation curvature |
| `beta_*` | Average treatment-effect parameter across geos |
| `eta_*` | Cross-geo heterogeneity in media effects |

### 20.4 Controls and non-media parameters

| Parameter | Meaning |
|---|---|
| `gamma_c` | Average control effect |
| `gamma_n` | Average non-media treatment effect |
| `xi_c` | Cross-geo control heterogeneity |
| `xi_n` | Cross-geo non-media heterogeneity |
| `non_media_baseline_values` | Baseline used for non-media attribution |

### 20.5 Prior-type parameters

| Parameter | Meaning |
|---|---|
| `media_prior_type` | Whether media is anchored by ROI, mROI, contribution, or coefficient priors |
| `rf_prior_type` | Same for reach-frequency channels |
| `organic_media_prior_type` | Same for organic media |
| `organic_rf_prior_type` | Same for organic RF |
| `non_media_treatments_prior_type` | Same for non-media treatments |
| `roi_calibration_period` | Time window to which the media ROI prior applies |
| `rf_roi_calibration_period` | Same for RF ROI priors |

### 20.6 Observation model parameters

| Parameter | Meaning |
|---|---|
| `sigma` | Residual noise scale |
| `unique_sigma_for_each_geo` | One sigma per geo or one global sigma |
| `holdout_id` | Excludes selected KPI observations from fitting |

### 20.7 Optimization parameters

| Parameter | Meaning |
|---|---|
| `budget` | Total spend to allocate |
| `pct_of_spend` | Baseline spend allocation vector |
| `spend_constraint_lower` | Lower box constraint around channel baseline spend |
| `spend_constraint_upper` | Upper box constraint around channel baseline spend |
| `target_roi` | Flexible-budget stopping target on average ROI |
| `target_mroi` | Flexible-budget stopping target on marginal ROI |
| `gtol` | Budget-grid resolution tolerance |
| `use_optimal_frequency` | Whether to optimize RF frequency |
| `max_frequency` | Upper bound for RF frequency search |
| `use_kpi` | Optimize KPI instead of revenue |

---

## 21. How to read the notebook after reading this guide

Now the notebook can be interpreted in the following way.

### Step 1: Build `InputData`

This is where raw business data is mapped into:

\[
(y, \text{pop}, z, v, x, s, \text{reach}, \text{freq}, n, \dots)
\]

### Step 2: Configure priors and model spec

This chooses:

- the structural form of the model
- the regularization assumptions
- how much prior business knowledge is injected

### Step 3: Sample prior and posterior

This computes:

\[
p(\theta)
\quad \text{and} \quad
p(\theta \mid y)
\]

### Step 4: Diagnostics

This checks whether posterior sampling is trustworthy and whether the implied model fit is reasonable.

### Step 5: Summaries

This turns posterior draws into business quantities:

- contribution
- ROI
- response curves
- baseline decomposition

### Step 6: Optimization

This solves a constrained decision problem on top of the fitted response curves.

---

## 22. Final mental model

The best compact mental model of Meridian is:

\[
\text{Observed KPI}
\xleftarrow{\text{Gaussian likelihood}}
\text{Baseline + Time Trend + Media Response + Controls + Treatments}
\]

where:

- baseline is geo-specific plus time-varying
- media response is transformed by carryover and saturation
- geo effects are hierarchical
- priors can be placed in business units
- inference is Bayesian and uncertainty-aware
- optimization uses the fitted response curves rather than re-fitting the model

If you remember only one expanded formula, remember this one:

\[
y^{\text{scaled}}_{g,t}
\sim
\mathcal{N}
\left(
\tau_g + \mu_t
+ \sum_m f^{(m)}_{g,t,m}\beta_{g,m}
+ \sum_r f^{(rf)}_{g,t,r}\beta^{(rf)}_{g,r}
+ \sum_c z^*_{g,t,c}\gamma_{g,c}
+ \sum_n n^*_{g,t,n}\gamma^{(nm)}_{g,n},
\sigma_g
\right)
\]

Everything else in Meridian exists to:

- build the transformed features `f`
- regularize the coefficients
- infer the posterior distribution
- convert the fitted model into business decisions

