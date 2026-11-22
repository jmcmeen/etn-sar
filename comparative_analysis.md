# Eastern Tennessee Herpetofauna: Updated Comparative Analysis Using NLS vs. Log-Linear SAR Fitting

A companion to [`analysis_updates.ipynb`](https://github.com/jmcmeen/etn-sar/blob/main/analysis_updates.ipynb), comparing the nonlinear least squares (NLS) results from the [`sars`](https://pypi.org/project/sars/) Python library against the original log-linear OLS calculations published in Jessee, Stout & McMeen (2022).

---

## Fitting Method Differences

The original calculations used log₁₀-transformed OLS regression:

> log₁₀(S) = z · log₁₀(A) + log₁₀(c)

This minimizes squared residuals in log-space and implicitly assumes multiplicative error — i.e., error proportional to the predicted value. The c and z parameters are recovered as c = 10^(intercept) and z = slope.

The `sars` library fits the power model directly in arithmetic space via nonlinear least squares (`scipy.optimize.least_squares`):

> S = cA^z

This minimizes squared residuals in the original species-count scale and assumes additive error. The NLS approach is generally preferred in modern SAR literature because it does not impose the variance structure that log-transformation assumes (Tjørve & Tjørve 2021).

## Parameter Estimates

| Group              | Method                | c     | z      | R²          |
| ------------------ | --------------------- | ----- | ------ | ----------- |
| Total Herpetofauna | Original (log-linear) | 29.54 | 0.1137 | 0.86 (adj.) |
| Total Herpetofauna | `sars` NLS            | 22.49 | 0.1478 | 0.9162      |
| Amphibians         | Original (log-linear) | 14.09 | 0.1449 | 0.98 (adj.) |
| Amphibians         | `sars` NLS            | 12.28 | 0.1616 | 0.9849      |
| Reptiles           | Original (log-linear) | 15.45 | 0.0747 | 0.33 (adj.) |
| Reptiles           | `sars` NLS            | 10.96 | 0.1205 | 0.6619      |

> **R² values are not directly comparable between rows.** The original R² is adjusted R² computed in log-space (measuring fit of log S vs. log A), while the `sars` R² is unadjusted and computed in arithmetic space (measuring fit of S vs. A). Both metrics are retained here to match their respective source methods, but side-by-side numerical comparison is not meaningful.

The c parameter (baseline richness) is systematically lower under NLS, while z (the scaling exponent) is higher. This shift is expected: log-linear regression overweights small-area sites relative to their absolute residuals, pulling c upward and z downward. The discrepancy is most dramatic for reptiles, where z increases by 61% (0.0747 → 0.1205).

The original R² values are adjusted R² in log-space, while the `sars` values are unadjusted R² in arithmetic space. These are not directly comparable, but both tell the same story: herpetofauna and amphibians fit the power model well; reptiles do not.

## z-Value Interpretation

All three NLS z-values fall within the canonical mainland/nested range of 0.10–0.20 (Rosenzweig 1995, Drakare et al. 2006), consistent with the study's nested sampling design (Steele Creek Park ⊂ Sullivan County ⊂ NE Tennessee ⊂ Eastern Tennessee).

| Group              | z (Original) | z (NLS) | Range Context                              |
| ------------------ | ------------ | ------- | ------------------------------------------ |
| Total Herpetofauna | 0.1137       | 0.1478  | Mainland/nested (0.10–0.20)                |
| Amphibians         | 0.1449       | 0.1616  | Mainland/nested                            |
| Reptiles           | 0.0747       | 0.1205  | Below range (original); within range (NLS) |

The original reptile z of 0.0747 fell below the expected mainland range, suggesting almost no area effect. Under NLS the estimate shifts to 0.12, placing it within the expected range — though the poor overall fit (R² = 0.66) means neither estimate is reliable.

## The Reptile Anomaly

The most analytically significant finding is the reptile SAR's departure from the power model. Examining the raw data reveals why:

| Site              | Area (km²) | Reptile Species |
| ----------------- | ---------- | --------------- |
| Steele Creek Park | 9.3        | 21              |
| Sullivan County   | 1,114      | 21              |
| NE Tennessee      | 4,137      | 24              |
| Eastern Tennessee | 37,438     | 44              |

Reptile richness is flat from 9.3 km² to 1,114 km² (21 species at both scales), increases modestly to 24 at 4,137 km², then jumps to 44 at 37,438 km². This step-function pattern is fundamentally non-power-law. The residuals confirm the systematic misfit:

| Site              | Power Residual | Linear Residual |
| ----------------- | -------------- | --------------- |
| Steele Creek Park | +6.66          | +0.09           |
| Sullivan County   | −4.52          | −0.59           |
| NE Tennessee      | −5.89          | +0.54           |
| Eastern Tennessee | +5.02          | −0.04           |

The linear model (S = 20.90 + 0.000618·A) produces dramatically smaller residuals and achieves R² = 0.998 vs. 0.662 for the power model. The multi-model comparison confirms this: the linear model has the lowest AIC of all 20 models for reptiles.

This suggests that reptile richness in this region may be driven by a threshold effect at the largest spatial scale rather than by smooth area-dependent scaling — potentially reflecting the geographic range boundaries of southern Appalachian reptile species, which may require the full eastern Tennessee extent to capture meaningful turnover.

## Multi-Model Rankings

### The AICc Limitation

With n = 4 observations and k ≥ 3 estimated parameters (the `sars` library counts σ), AICc is undefined for every model:

> AICc = AIC + 2k(k+1) / (n − k − 1)

When n ≤ k + 1 the correction is undefined (`sars` reports AICc as ∞), so AICc can neither rank the models nor weight them.

This is not a limitation of the library but a fundamental constraint of the sample size. AIC and BIC remain finite, and the notebook ranks the models by AIC: at n = 4 it penalises each extra parameter more than BIC does (2 against ln 4 ≈ 1.39), the safer choice when extra parameters fit four points so easily. `sars` can also weight and average models by AIC or BIC (`sar_average(ic="AIC")`), but on four points the notebook doesn't: a three-parameter model leaves one residual degree of freedom, so any weights would rest on fits that can't be told apart, and suggest a precision the data don't have.

The same holds for `sars.bootstrap_ci()`, which resamples the four rows with replacement and averages the models fitted to each resample. Nine in ten resamples repeat a point, leaving three distinct areas or fewer; a three-parameter model then passes through every distinct point exactly, with no residual variance, so its information criterion, and the weights built on it, are meaningless (and by default the bootstrap weights by AICc, which is undefined here anyway). A fifth data point would make AICc finite for the two-parameter models; several more would be needed before model averaging and bootstrap intervals mean much.

### Rankings by AIC

A three-parameter model leaves one residual degree of freedom on four points, so it fits almost perfectly whatever its shape: three-parameter models with R² > 0.99 crowd the top of the rankings, and their order says nothing about which curve is right. The comparison among the two-parameter models, with two residual degrees of freedom, is the more meaningful one, and even it can't reliably tell the forms apart.

**Herpetofauna** — The top models by AIC are three-parameter (epm1, powerR, epm2; AIC 19.6–27.0). Among the two-parameter models, the linear and power models are effectively tied (AIC 33.58 and 33.66, R² 0.918 and 0.916), well ahead of koba and loga (AIC 37.3).

**Amphibians** — The same three-parameter models lead (epm1 > powerR > epm2), and the power model is the best two-parameter model by a wide margin (AIC 23.49, R² = 0.985; next koba, 30.84): it already fits so well that a third parameter adds little.

**Reptiles** — The linear model has the lowest AIC of all 20 models (10.09), far ahead of the power model (31.11). Three three-parameter models follow within 0.3 (gompertz, asymp and ratio; AIC 10.18–10.35, R² ≈ 0.999), and they lead by BIC, but only because BIC's per-parameter penalty is the smaller one at n = 4. Over these four points they stay close to the straight line, although `sars` classifies their shape as convex: the asymptotic model, for one, converges to d = 78.5 with z = 1.4 × 10⁻⁵, an asymptote far above the data approached at a tiny rate.

### Curve Shape

For herpetofauna and amphibians, all well-fitting models produce **convex** shapes (upward curvature without asymptote), consistent with the original finding and with typical nested SAR data (Tjørve & Tjørve 2021).

For reptiles, the linear model ranks first, and the three-parameter models just behind it are classified as convex but have approach rates so low that they stay close to a straight line across the observed range. The data do not support distinguishing between convex, asymptotic, and linear forms for this group.

## Predictive Validation

Testing both fitting methods against independent sites from the original publication:

| Site               | Area (km²) | Amp. Reported | Amp. NLS | Amp. Original | Rep. Reported | Rep. NLS | Rep. Original |
| ------------------ | ---------- | ------------- | -------- | ------------- | ------------- | -------- | ------------- |
| GSMNP              | 2,114      | 43            | 42.3     | 42.7          | 38            | 27.6     | 27.4          |
| Cumberland Gap NHP | 82.74      | 28            | 25.1     | 26.7          | 20            | 18.7     | 21.5          |
| Radford AAP        | 32.7       | 19            | 21.6     | 23.3          | 14            | 16.7     | 20.1          |
| Henderson Wetland  | 0.101      | 12            | 8.5      | 10.1          | —*            | 8.3      | 13.0          |

\* Henderson Wetland reptile count is unavailable because the site was not surveyed for reptiles in the original study. Reptile predictions are shown for completeness but cannot be validated.

For amphibians, both methods produce similar predictions. The NLS model gives slightly lower estimates that happen to be closer to reported values at GSMNP (42.3 vs. 43) and Radford AAP, but slightly farther from Cumberland Gap.

For reptiles, both methods substantially underpredict GSMNP (reported 38 vs. predicted ~27–28), confirming the weak predictive power of the reptile SAR regardless of fitting method. This underprediction was noted in the original publication and attributed to the non-significant regression relationship.

At small areas (Henderson Wetland, 0.101 km²), the methods diverge more noticeably: the NLS model predicts 8.5 amphibians vs. 10.1 from the original. The reported count of 12 suggests that both models underpredict at very small, hyper-diverse sites — consistent with the original discussion.

## Threshold Analysis

`sars.sar_threshold()` tests for breakpoints (a small-island effect) by comparing piecewise models with a single straight line, by AICc. With four points AICc is undefined even for the straight line (two coefficients plus σ need n > 4), so `sars` refuses the analysis: a breakpoint can't be tested at this sample size, and the data neither support nor rule out a small-island effect. Earlier versions of `sars` returned the straight-line baseline alone, since no piecewise model could be fitted to four points; that was never a choice among models.

## Limitations

All conclusions in this analysis are based on n = 4 data points, which is extremely small for any regression methodology. With only 4 observations:

- **Degrees of freedom are minimal.** Two-parameter models have only 2 residual degrees of freedom; 3-parameter models have just 1. Any model with 3+ parameters can fit the data nearly perfectly regardless of its ecological plausibility.
- **AICc is undefined** for all models (see "The AICc Limitation" above), so models are ranked by AIC, not averaged, and no threshold model can be tested.
- **Statistical power is negligible.** Confidence intervals on parameters are wide, and distinguishing between competing model forms (power vs. logarithmic vs. linear) is not statistically meaningful at this sample size.
- **The NLS vs. OLS comparison is illustrative, not definitive.** Both methods fit the amphibian data well and the reptile data poorly. The parameter shifts between methods, while consistent with theoretical expectations, should not be over-interpreted given the sample size.

These limitations apply equally to the original publication's log-linear analysis and to the NLS reanalysis presented here. The results are best viewed as a baseline that motivates expanded sampling at intermediate spatial scales.

## Recommendations for Further Work

**Expand the dataset.** The single most impactful improvement would be adding intermediate spatial scales (individual watersheds, additional well-surveyed parks, or state-level data). A single additional data point would make AICc finite for the two-parameter models; several would make model averaging, bootstrap confidence intervals and a test for a small-island effect worthwhile.

**Taxonomic decomposition.** Disaggregating reptiles into lizards, snakes, and turtles could reveal whether the weak SAR is driven by a specific subgroup — particularly turtles, whose richness may be more habitat-specific (aquatic habitat availability) than area-dependent.

**Integration with iNaturalist.** The `sars` library supports `sars.from_pyinaturalist()` for direct ingestion of citizen-science observation data, which could update species counts since the 2022 publication.

**GIS integration.** If park boundary and county shapefiles are available, `sars.from_geodataframe()` can ingest GeoDataFrames directly with automatic area calculation, streamlining the addition of new spatial scales.

## Summary of Key Findings

1. The NLS fitting method in `sars` produces systematically lower c and higher z values than the original log-linear regression, as expected from the different error assumptions.

2. The qualitative conclusions of the original publication are fully supported: amphibians show a strong power-law SAR (R² ≈ 0.98), total herpetofauna are moderately well-fit (R² ≈ 0.92), and reptiles are poorly fit by the power model (R² ≈ 0.66).

3. Multi-model comparison suggests that reptile richness in this region follows a straight line rather than a curvilinear SAR: the linear model has the lowest AIC of all 20 models, and the three-parameter models just behind it stay close to a straight line over these points. This is driven by the identical species count at 9.3 km² and 1,114 km² (both 21 species).

4. AICc-based model selection, model averaging, bootstrap confidence intervals and threshold analysis are fundamentally inapplicable at n = 4. AIC ranks the models cautiously, and only the comparison among two-parameter models carries much information.

5. All NLS z-values fall within the expected mainland/nested range (0.10–0.20), consistent with the study's spatial design; the original reptile z (0.0747) falls below it.

6. Predictive accuracy at independent test sites is similar between the two methods for amphibians. Both methods fail for reptiles, confirming the original conclusion that reptile richness in eastern Tennessee is not reliably predicted by a simple area-based model.

## References

- Drakare, S., Lennon, J.J. & Hillebrand, H. (2006). The imprint of the geographical, evolutionary and ecological context on species–area relationships. *Ecology Letters* 9:215–227.
- Jessee, L.D., Stout, J.B. & McMeen, J.N. (2022). Herpetofauna of Steele Creek Park (Sullivan County, TN), with Comments on Species–Area Relationships of Amphibians and Reptiles in Eastern Tennessee. *Southeastern Naturalist* 21(1):63–73.
- Rosenzweig, M.L. (1995). *Species Diversity in Space and Time*. Cambridge University Press.
- Tjørve, E. & Tjørve, K.M.C. (2021). Mathematical expressions for the species–area relationship and the assumptions behind the models. In: Matthews, T.J., Triantis, K.A. & Whittaker, R.J. (Eds.), *The Species–Area Relationship: Theory and Application*. Cambridge University Press, pp. 157–184.
