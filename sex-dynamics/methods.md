# The master post of sexual and relationship dynamics: looks, money, height, paying for sex, marriage, porn and wanting more

<p class="meta">Methodology for a chart from <a href="https://performativebafflement.substack.com/">Performative Bafflement</a>. Updated 2026-11-14.</p>

![chart 1](chart.png)

![chart 2](chart2.png)

![chart 3](chart3.png)

![chart 4](chart4.png)

![chart 5](chart5.png)

![chart 6](chart6.png)

<p class="ai">For AI fact-checking: the plain-text version of this page is at <a href="methods.md">https://pbmethods.github.io/sex-dynamics/methods.md</a>. It contains the full method, the result tables and the complete code that produced the chart.</p>

## Method

### `sexfreq_data.py`

```text
Sex frequency extracts from the GSS (US) and Natsal-3 (Britain), for the married-vs-single and years-married charts.

Run:  uv run --with pandas,pyarrow python notebooks/sexfreq/sexfreq_data.py
Reads: notebooks/us_public_data/GSS_stata/gss7224_r3a.dta (GSS 1972-2024 cumulative, R3a)
       notebooks/us_public_data/NATSAL/UKDA-7799-stata/stata/stata11/eul_natsal_2010_for_archive.dta (Natsal-3, 2010-12)
Writes: gss_sexfreq.parquet, natsal_sexfreq.parquet (next to this script)

GSS
  sexfreq  past-year frequency in 7 bands (asked 1989-2024, self-completion). Mapped to times/year at the usual
           midpoints 0, 2, 12, 30, 52, 130, 208: banded, so means are only as good as the midpoints, and the top
           band ("more than 3 times a week") is open-ended.
  partners numwomen + nummen, partners since age 18 (asked 1989-2022, not 2024). 989 = "989 or more"; codes above
           are missing. "Body count" here = all partners since 18, both sexes.
  dur      years in the current marriage = age - agewed, kept only for the married who were never divorced/separated
           (divorce == 2) or widowed (widowed == 2), so the first marriage is the current one. agewed was asked only
           in 1988-94 and 2006, so dur exists for those years only; it is whole years and can be off by one.
  conrinc  respondent's own income, constant 2000 $ (rescaled to 2024 $). Used as the income control because family
           income (coninc) mechanically counts a spouse's pay, which would soak up part of the married effect.
  w        wtssps (person post-stratification weight) scaled to mean 1 within each year.
Natsal-3
  sex4wks  occasions of heterosexual sex in the past 4 weeks: a count, not a band (995+ = missing).
  dur      years married to the current spouse = interview year - year married, for "married & living with spouse"
           whose most recent live-in partner is the spouse: year cohabitation began if they were married at the start
           (pp1ms == 1), otherwise the year they later married (pp1mar == 1, pp1ymar). Whole years, off by up to one.
  frequent "would you like to have sex more often, less often, or is it about right": 1-2 = wants more.
  sffsatis "I feel satisfied with my sex life": 1-2 agree, 4-5 disagree.
  totlife  lifetime partners, both sexes (9995+ missing). income: household, 8 bands, top band 50,000+.
  w        total_wt (core + boost samples).
```

### `sexfreq_model.py`

```text
Regression helpers for the sex-frequency charts: weighted quintiles and regression-adjusted cell means.

adjusted_means fits one weighted least-squares model per call,
    y ~ cell dummies (group x quintile, no intercept) + controls
with HC1 sandwich SEs, and reports each cell's mean standardized to the controls of the whole sample passed in
(every person's controls, with the cell switched on), so all cells are compared at the same education, income, age
and survey-year mix. Being linear, that standardized mean is b_cell + mean(controls) @ b_controls, and its SE comes
straight from the coefficient covariance.
```

### `master_post_data.py`

```text
Data for "The master post of sexual and relationship dynamics": every number behind the post's reworked charts.

    uv run --with pandas,pyarrow,numpy,scipy,statsmodels python notebooks/master_post/master_post_data.py

Writes small CSVs next to this script; `master_post_charts.py` draws them. Run from the repo root.

**Sources computed here.**

* **GSS 1972-2024 cumulative file** (`gss7224_r2.dta`), survey-weighted with `wtssps` (missing weight = 1).
  - *Looks* (`rlooks`, asked 2016, 2018 and 2022): the interviewer rates the respondent's physical attractiveness 1-5 at the end of the
    interview. Men aged 25-60, lifetime female partners since 18 (`numwomen`, 0-989), winsorised at 100.
    Ratings 1 and 2 are pooled ("1-2, below average") because rating 1 has too few men to stand alone.
  - *SES rungs*: own income (`conrinc`) and occupational prestige (`prestg10`), each ranked within survey year x
    sex and cut into thirds. "Bottom both" = bottom third on both; "bottom either" = bottom third on one; "high
    both" = top third on both; "high either" = top third on one (and not bottom on the other); the rest are
    "middle". Split by bachelor's degree or more (`degree` >= 3). Men 25-60, partners winsorised at 100.
  - *Paying for sex* (`evpaidsx`, men 25-60, 1991-2022): share who have ever paid, by own-income decile ranked
    within survey year; and standardised differences (buyers minus non-buyers, divided by the pooled SD) on
    degree, income and prestige rank, interviewer looks (raw and SES-adjusted: residual after regressing on age,
    age^2, income rank, prestige rank, degree and year), height (`rheight`, 2022 only), lifetime partners
    (capped 100) and marital status.
  - Intervals are percentile bootstraps of the weighted mean (800 draws; 600 for the standardised differences).

* **Natsal-3** (Britain 2010-12, `eul_natsal_2010_for_archive.dta`), weighted with `total_wt`.
  - *Duration*: respondents living with a partner (`partnerhh` = 1); duration = age - age at first sex with
    the current partner (`rafsmr`). Occasions of sex in the last 4 weeks (`sex4wks`, a count) are fitted with a
    Poisson GLM, and "would like sex more often" (`frequent` = much or a bit more) with a logistic GLM. Both use
    age (centred at 35, quadratic) and duration (log1p, a linear term to 5 years, and a slope after 5) as
    separate terms, and are drawn at age 35, so the curves show years together rather than ageing.
  - *Wanting more*: partnered (`partnerhh` = 1) men and women aged 25-40. Share wanting more, by fifths of
    achieved frequency within sex. Desired monthly frequency is fitted per sex as an interval-censored lognormal
    (achieved count plus direction of the wish; "a bit more" is capped at twice the achieved count), and the
    matching table gives, for a stated desired frequency, the share of each sex whose desired frequency is at
    least that high.

* **NSFG 2011-13, 2013-15, 2015-17 male files** (CDC public-use, `nsfg_2011_2017/`), pooled with each cycle's
  final weight divided by 3. Heterosexual (`ORIENT`/`ORIENT_A`/`ORIENT_B` = 1) non-Hispanic white (`HISPRACE2` = 2)
  men aged 18-29 with a self-reported height (`INCHES`): share with no opposite-sex partner in the past 12 months
  (`PARTS1YR` = 0, which includes virgins) and share who never had intercourse (`HADSEX` = 2), by height in
  inches (65 and under, 76 and over pooled). Replicates NuancePill, "The Bumble height filter graph and height's
  effect on male celibacy" (https://nuancepill.substack.com/p/the-bumble-height-filter-graph-and-heights-effect-on-male-celibacy),
  whose idea and sample definition this follows; our n is 2,191 against their 2,623 (they likely used a looser
  orientation filter), and the pattern is the same: flat across heights.

**Sources digitised, not recomputed** (we do not have the microdata; values read off the author's published
charts by pixel measurement against their axis gridlines, accurate to about +/-0.2 points):

* **Sex and porn by relationship length, and sex-life satisfaction**: Aella, "Monogamous men in long-term
  relationships aren't doin too hot" (Knowingless, 14 Mar 2023,
  https://aella.substack.com/p/monogamous-men-in-long-term-relationships). Her relationships survey (~25,000
  respondents, mostly her own followers). Monthly sex and porn occasions by relationship length in years; and
  agreement with "I'm satisfied with my sex life" (-3 to +3) for straight, non-poly men in relationships of
  12+ years (n = 3,720).
```

### `master_post_charts.py`

```text
Charts for "The master post of sexual and relationship dynamics" (Performative Bafflement).

    uv run --with matplotlib,pillow,fonttools,pandas,numpy python notebooks/master_post/master_post_charts.py

Draws the post's five reworked figures from the CSVs that `master_post_data.py` writes (run that first), saves
the unwatermarked PNGs to ~/Desktop/Recent Projects/Substack Notes/Unwatermarked graphs/Sex/ and the watermarked
`_wm.png` copies to ~/Desktop/Recent Projects/Substack Notes/Sex/ (house watermark, style 7; this also rebuilds
the methods page pbmethods.github.io/sex-dynamics, unless PB_NO_PUSH=1).

1. `motivation_looks_ses_height` -- men's lifetime partners by interviewer-rated looks and by income x prestige
   rung (GSS), and the share of young men sexless in the past year by height (NSFG 2011-17).
2. `paid_sex_profile` -- how men who have paid for sex differ from those who have not (standardised
   differences), and the share who have paid by income decile (GSS).
3. `married_sex_porn` -- monthly sex and wanting more by years together at a fixed age of 35 (Natsal-3), and sex
   versus porn by relationship length (Aella's relationships survey, digitised).
4. `wants_more` -- who wants more sex by how much they already have, and how many people of the other sex want
   at least a given frequency (Natsal-3).
5. `sex_satisfaction` -- long-partnered men's agreement with "I'm satisfied with my sex life", in full and
   grouped (Aella's relationships survey, digitised).

Every number and method is in `master_post_data.py`'s docstring; caveats are in CAVEATS.md.
```

## Results

### `looks_partners.csv`

| looks | code | n | mean_partners | lo | hi |
|---|---|---|---|---|---|
| 1 (lowest) | 1 | 19 | 9.985 | 5.316 | 18.398 |
| 2 | 2 | 60 | 13.213 | 8.287 | 20.331 |
| 3 | 3 | 520 | 11.431 | 9.822 | 13.086 |
| 4 | 4 | 294 | 14.499 | 10.689 | 19.172 |
| 5 (highest) | 5 | 66 | 16.112 | 10.348 | 23.359 |

### `ses_partners.csv`

| college | rung | n | mean_partners | lo | hi |
|---|---|---|---|---|---|
| True | bottom both | 117 | 7.832 | 5.758 | 10.386 |
| True | bottom either | 538 | 11.505 | 9.814 | 13.204 |
| True | middle | 246 | 13.567 | 9.555 | 18.003 |
| True | high either | 897 | 10.826 | 9.580 | 12.134 |
| True | high both | 1127 | 10.796 | 9.717 | 11.845 |
| False | bottom both | 1166 | 11.699 | 10.546 | 12.887 |
| False | bottom either | 2113 | 14.743 | 13.717 | 16.000 |
| False | middle | 848 | 13.823 | 12.364 | 15.460 |
| False | high either | 960 | 14.147 | 12.668 | 15.842 |
| False | high both | 468 | 14.685 | 12.670 | 16.906 |

### `nsfg_height_sexless.csv`

| inches | height | n | pct_sexless | sexless_lo | sexless_hi | pct_virgin | virgin_lo | virgin_hi |
|---|---|---|---|---|---|---|---|---|
| 65 | ≤5'5 | 84 | 19.784 | 11.389 | 30.964 | 13.279 | 6.006 | 23.063 |
| 66 | 5'6 | 85 | 18.834 | 9.987 | 30.306 | 10.252 | 3.777 | 19.422 |
| 67 | 5'7 | 128 | 19.424 | 9.742 | 29.720 | 8.665 | 3.298 | 15.490 |
| 68 | 5'8 | 156 | 17.768 | 11.787 | 24.975 | 11.373 | 6.542 | 16.552 |
| 69 | 5'9 | 243 | 18.689 | 12.457 | 25.013 | 12.218 | 7.427 | 17.982 |
| 70 | 5'10 | 246 | 23.989 | 17.602 | 31.367 | 12.073 | 7.579 | 16.772 |
| 71 | 5'11 | 149 | 22.972 | 14.143 | 33.242 | 16.064 | 7.954 | 26.273 |
| 72 | 6'0 | 456 | 20.428 | 15.597 | 25.390 | 13.707 | 9.160 | 18.375 |
| 73 | 6'1 | 214 | 18.882 | 11.822 | 26.654 | 8.229 | 2.757 | 15.074 |
| 74 | 6'2 | 198 | 24.391 | 15.691 | 33.259 | 11.472 | 6.457 | 17.221 |
| 75 | 6'3 | 113 | 17.670 | 9.862 | 27.101 | 8.702 | 3.664 | 16.423 |
| 76 | ≥6'4 | 119 | 19.540 | 12.003 | 27.872 | 14.674 | 7.803 | 21.966 |
| 0 | all | 2191 | 20.525 |  |  | 11.988 |  |  |

### `paid_sex_profile.csv`

| measure | buyers | non_buyers | d | d_lo | d_hi | n_buyers | n_others |
|---|---|---|---|---|---|---|---|
| Lifetime partners | 27.476 | 11.174 | 0.675 | 0.597 | 0.757 | 1393 | 8063 |
| Height | 71.557 | 70.254 | 0.392 | -0.143 | 1.069 | 20 | 234 |
| Looks, SES-adjusted | 0.015 | -0.002 | 0.023 | -0.237 | 0.278 | 92 | 686 |
| Looks (interviewer) | 3.301 | 3.338 | -0.047 | -0.284 | 0.180 | 117 | 896 |
| Income rank | 49.794 | 49.586 | 0.007 | -0.064 | 0.079 | 1201 | 7210 |
| Prestige rank | 47.074 | 49.154 | -0.073 | -0.132 | -0.001 | 1473 | 8628 |
| Has a BA+ | 0.239 | 0.303 | -0.140 | -0.199 | -0.076 | 1493 | 8729 |
| Ever married | 0.727 | 0.756 | -0.065 | -0.125 | -0.009 | 1495 | 8733 |
| Currently married | 0.499 | 0.614 | -0.234 | -0.303 | -0.168 | 1495 | 8733 |

### `paid_sex_by_income.csv`

| decile | share_paid | lo | hi | n |
|---|---|---|---|---|
| 1 | 0.141 | 0.117 | 0.167 | 839 |
| 2 | 0.139 | 0.115 | 0.167 | 817 |
| 3 | 0.127 | 0.103 | 0.156 | 791 |
| 4 | 0.101 | 0.080 | 0.123 | 864 |
| 5 | 0.135 | 0.110 | 0.161 | 829 |
| 6 | 0.150 | 0.119 | 0.176 | 834 |
| 7 | 0.141 | 0.118 | 0.165 | 891 |
| 8 | 0.125 | 0.101 | 0.148 | 830 |
| 9 | 0.147 | 0.120 | 0.176 | 884 |
| 10 | 0.126 | 0.100 | 0.153 | 832 |
| 0 | 0.135 |  |  | 10231 |

### `bodycount_distribution.csv`

| stat | value |
|---|---|
| p10 | 1.000 |
| p20 | 2.000 |
| p30 | 3.000 |
| p40 | 4.000 |
| p50 | 6.000 |
| p60 | 10.000 |
| p70 | 12.000 |
| p80 | 20.000 |
| p90 | 40.000 |
| p95 | 70.000 |
| mean_capped100 | 14.587 |
| share_below_mean | 0.718 |
| fifth1_share | 0.011 |
| fifth1_mean | 0.828 |
| fifth2_share | 0.041 |
| fifth2_mean | 2.973 |
| fifth3_share | 0.084 |
| fifth3_mean | 6.162 |
| fifth4_share | 0.185 |
| fifth4_mean | 13.472 |
| fifth5_share | 0.679 |
| fifth5_mean | 49.499 |

### `bodycount_crosscuts.csv`

| group | family | n | median | p80 | p90 | mean_capped100 |
|---|---|---|---|---|---|---|
| All men 40-60 | all | 5435 | 6.000 | 20.000 | 40.000 | 14.587 |
| Bachelor's+ | status | 1811 | 5.000 | 19.872 | 30.000 | 12.380 |
| No degree | status | 3617 | 6.000 | 20.000 | 40.000 | 15.531 |
| Income top third | status | 1443 | 6.000 | 20.000 | 30.000 | 13.531 |
| Income bottom third | status | 1445 | 5.000 | 20.000 | 35.000 | 14.541 |
| Married | history | 3240 | 5.000 | 17.000 | 30.000 | 12.617 |
| Never married | history | 802 | 5.000 | 20.000 | 40.000 | 14.306 |
| Divorced/separated | history | 1286 | 11.000 | 30.000 | 50.000 | 22.177 |
| Never paid for sex | appetite | 4171 | 5.000 | 15.000 | 30.000 | 11.749 |
| Has paid for sex | appetite | 917 | 15.000 | 50.000 | 100.000 | 29.276 |
| No X-rated film, past yr | appetite | 2364 | 5.000 | 20.000 | 30.000 | 12.379 |
| Saw X-rated film, past yr | appetite | 1105 | 10.000 | 25.000 | 50.000 | 18.634 |

### `natsal_years_married.csv`

| adj | se | lo | hi | raw | n | years_married | sex | outcome |
|---|---|---|---|---|---|---|---|---|
| 5.898 | 0.790 | 4.349 | 7.447 | 6.550 | 42.000 | 0.000 | men | sex_per_month |
| 5.438 | 1.508 | 2.483 | 8.393 | 6.030 | 116.000 | 1.000 | men | sex_per_month |
| 5.546 | 1.038 | 3.512 | 7.580 | 6.114 | 80.000 | 2.000 | men | sex_per_month |
| 4.542 | 0.610 | 3.345 | 5.738 | 5.085 | 104.000 | 3.000 | men | sex_per_month |
| 3.246 | 0.441 | 2.382 | 4.111 | 3.705 | 78.000 | 4.000 | men | sex_per_month |
| 3.553 | 0.327 | 2.913 | 4.194 | 3.999 | 167.000 | 5.500 | men | sex_per_month |
| 3.792 | 0.408 | 2.993 | 4.591 | 4.085 | 172.000 | 8.000 | men | sex_per_month |
| 3.196 | 0.227 | 2.752 | 3.641 | 3.376 | 216.000 | 12.000 | men | sex_per_month |
| 3.661 | 0.254 | 3.162 | 4.159 | 3.694 | 181.000 | 17.000 | men | sex_per_month |
| 3.228 | 0.229 | 2.779 | 3.676 | 3.005 | 241.000 | 24.500 | men | sex_per_month |
| 2.649 | 0.293 | 2.074 | 3.223 | 1.945 | 571.000 | 35.000 | men | sex_per_month |
| 0.434 | 0.088 | 0.260 | 0.607 | 0.501 | 42.000 | 0.000 | men | want_more |
| 0.576 | 0.053 | 0.472 | 0.680 | 0.635 | 114.000 | 1.000 | men | want_more |
| 0.668 | 0.059 | 0.553 | 0.783 | 0.728 | 76.000 | 2.000 | men | want_more |
| 0.590 | 0.058 | 0.475 | 0.704 | 0.643 | 102.000 | 3.000 | men | want_more |
| 0.675 | 0.060 | 0.558 | 0.791 | 0.724 | 78.000 | 4.000 | men | want_more |
| 0.582 | 0.042 | 0.499 | 0.666 | 0.631 | 166.000 | 5.500 | men | want_more |
| 0.634 | 0.041 | 0.553 | 0.715 | 0.664 | 165.000 | 8.000 | men | want_more |
| 0.659 | 0.035 | 0.589 | 0.728 | 0.671 | 206.000 | 12.000 | men | want_more |
| 0.620 | 0.040 | 0.542 | 0.697 | 0.617 | 165.000 | 17.000 | men | want_more |
| 0.653 | 0.035 | 0.583 | 0.722 | 0.621 | 219.000 | 24.500 | men | want_more |
| 0.645 | 0.035 | 0.576 | 0.714 | 0.552 | 417.000 | 35.000 | men | want_more |
| 5.460 | 0.665 | 4.158 | 6.763 | 6.844 | 45.000 | 0.000 | women | sex_per_month |
| 4.418 | 0.525 | 3.389 | 5.448 | 5.646 | 108.000 | 1.000 | women | sex_per_month |
| 3.462 | 0.401 | 2.676 | 4.247 | 4.502 | 126.000 | 2.000 | women | sex_per_month |
| 3.588 | 0.421 | 2.763 | 4.413 | 4.586 | 146.000 | 3.000 | women | sex_per_month |
| 3.700 | 0.489 | 2.741 | 4.659 | 4.613 | 122.000 | 4.000 | women | sex_per_month |
| 3.014 | 0.283 | 2.460 | 3.567 | 3.850 | 250.000 | 5.500 | women | sex_per_month |
| 3.402 | 0.232 | 2.947 | 3.857 | 4.092 | 317.000 | 8.000 | women | sex_per_month |
| 3.235 | 0.261 | 2.723 | 3.748 | 3.561 | 351.000 | 12.000 | women | sex_per_month |
| 3.566 | 0.379 | 2.823 | 4.309 | 3.557 | 231.000 | 17.000 | women | sex_per_month |
| 3.645 | 0.192 | 3.269 | 4.022 | 3.204 | 401.000 | 24.500 | women | sex_per_month |
| 3.177 | 0.207 | 2.770 | 3.583 | 1.609 | 777.000 | 35.000 | women | sex_per_month |
| 0.561 | 0.089 | 0.387 | 0.734 | 0.573 | 44.000 | 0.000 | women | want_more |
| 0.531 | 0.057 | 0.420 | 0.643 | 0.540 | 107.000 | 1.000 | women | want_more |
| 0.566 | 0.052 | 0.464 | 0.668 | 0.574 | 122.000 | 2.000 | women | want_more |
| 0.520 | 0.050 | 0.422 | 0.618 | 0.528 | 146.000 | 3.000 | women | want_more |
| 0.503 | 0.054 | 0.396 | 0.609 | 0.508 | 117.000 | 4.000 | women | want_more |
| 0.547 | 0.038 | 0.473 | 0.621 | 0.553 | 243.000 | 5.500 | women | want_more |
| 0.481 | 0.033 | 0.416 | 0.546 | 0.485 | 307.000 | 8.000 | women | want_more |
| 0.459 | 0.030 | 0.401 | 0.517 | 0.461 | 333.000 | 12.000 | women | want_more |
| 0.421 | 0.036 | 0.352 | 0.491 | 0.419 | 212.000 | 17.000 | women | want_more |
| 0.402 | 0.029 | 0.345 | 0.458 | 0.394 | 356.000 | 24.500 | women | want_more |
| 0.320 | 0.033 | 0.254 | 0.386 | 0.306 | 530.000 | 35.000 | women | want_more |
| 0.586 |  |  |  |  | 2207.000 |  | single men | want_more |

_Every 1th row of 45 shown; full file: [natsal_years_married.csv](natsal_years_married.csv)._

### `natsal_desire_matching.csv`

| per_month | women_want_at_least | men_want_at_most |
|---|---|---|
| 1.000 | 0.876 | 0.078 |
| 1.500 | 0.790 | 0.141 |
| 2.000 | 0.712 | 0.202 |
| 2.500 | 0.643 | 0.258 |
| 3.000 | 0.583 | 0.310 |
| 3.500 | 0.531 | 0.357 |
| 4.000 | 0.485 | 0.400 |
| 4.500 | 0.444 | 0.439 |
| 5.000 | 0.409 | 0.474 |
| 5.500 | 0.377 | 0.506 |
| 6.000 | 0.349 | 0.535 |
| 6.500 | 0.324 | 0.561 |
| 7.000 | 0.301 | 0.586 |
| 7.500 | 0.281 | 0.608 |
| 8.000 | 0.262 | 0.629 |
| 8.500 | 0.245 | 0.648 |
| 9.000 | 0.230 | 0.666 |
| 9.500 | 0.216 | 0.682 |
| 10.000 | 0.204 | 0.697 |
| 10.500 | 0.192 | 0.711 |
| 11.000 | 0.181 | 0.725 |
| 11.500 | 0.171 | 0.737 |
| 12.000 | 0.162 | 0.748 |
| 12.500 | 0.154 | 0.759 |
| 13.000 | 0.146 | 0.769 |
| 13.500 | 0.138 | 0.779 |
| 14.000 | 0.132 | 0.788 |
| 14.500 | 0.125 | 0.796 |
| 15.000 | 0.119 | 0.804 |
| 15.500 | 0.114 | 0.812 |
| 16.000 | 0.109 | 0.819 |
| 16.500 | 0.104 | 0.826 |
| 17.000 | 0.099 | 0.832 |
| 17.500 | 0.095 | 0.838 |
| 18.000 | 0.091 | 0.844 |
| 18.500 | 0.087 | 0.849 |
| 19.000 | 0.083 | 0.854 |
| 19.500 | 0.080 | 0.859 |
| 20.000 | 0.077 | 0.864 |
| 20.500 | 0.074 | 0.869 |
| 21.000 | 0.071 | 0.873 |
| 21.500 | 0.068 | 0.877 |
| 22.000 | 0.066 | 0.881 |
| 22.500 | 0.063 | 0.884 |
| 23.000 | 0.061 | 0.888 |
| 23.500 | 0.059 | 0.891 |
| 24.000 | 0.057 | 0.895 |
| 24.500 | 0.055 | 0.898 |
| 25.000 | 0.053 | 0.901 |
| 25.500 | 0.051 | 0.904 |
| 26.000 | 0.049 | 0.906 |
| 26.500 | 0.047 | 0.909 |
| 27.000 | 0.046 | 0.912 |
| 27.500 | 0.044 | 0.914 |
| 28.000 | 0.043 | 0.916 |
| 28.500 | 0.042 | 0.919 |
| 29.000 | 0.040 | 0.921 |
| 29.500 | 0.039 | 0.923 |
| 30.000 | 0.038 | 0.925 |

_Every 2th row of 117 shown; full file: [natsal_desire_matching.csv](natsal_desire_matching.csv)._

### `aella_sex_porn_digitised.csv`

| years_together | sex_per_month | porn_per_month_men | porn_per_month_women | source |
|---|---|---|---|---|
| 0-1 | 12.150 | 10.800 | 3.560 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 1-2 | 8.790 | 11.710 | 2.910 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 2-4 | 6.890 | 12.720 | 3.240 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 4-6 | 5.430 | 13.840 | 3.280 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 6-8 | 4.750 | 14.580 | 2.570 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 8-10 | 4.410 | 13.600 | 2.680 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 10-12 | 4.410 | 13.840 | 2.240 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 12-14 | 4.160 | 13.760 | 2.510 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 14-16 | 4.320 | 13.540 | 2.680 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 16-22 | 3.870 | 13.340 | 2.670 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |
| 22+ | 4.160 | 12.290 | 1.690 | Aella relationships survey, Knowingless 2023-03-14 (digitised) |

### `aella_satisfaction_digitised.csv`

| agreement | pct | source |
|---|---|---|
| -3 | 13.713 | Aella relationships survey, straight non-poly men 12+ yrs, n=3,720 (digitised) |
| -2 | 21.786 | Aella relationships survey, straight non-poly men 12+ yrs, n=3,720 (digitised) |
| -1 | 17.914 | Aella relationships survey, straight non-poly men 12+ yrs, n=3,720 (digitised) |
| 0 | 6.018 | Aella relationships survey, straight non-poly men 12+ yrs, n=3,720 (digitised) |
| 1 | 13.852 | Aella relationships survey, straight non-poly men 12+ yrs, n=3,720 (digitised) |
| 2 | 20.609 | Aella relationships survey, straight non-poly men 12+ yrs, n=3,720 (digitised) |
| 3 | 6.108 | Aella relationships survey, straight non-poly men 12+ yrs, n=3,720 (digitised) |

## Notes and caveats

## Caveats

**Everything here is cross-sectional.** The duration curves compare different couples at different durations
rather than following the same couples, so couples who split early drop out of the later years. The looks, SES
and height panels compare different men at the same moment, and say nothing about what would happen if one man's
looks, income or height changed.

**Self-report.** Partner counts, sex frequency, height and paying for sex are all self-reported. Men report about
2.4 times as many opposite-sex partners as women do in the GSS, which cannot be literally true, so the levels
carry a reporting bias. Comparisons *within* men (by looks, rung or height) are less exposed to it, as long as the
bias doesn't itself vary by looks, rung or height.

**Looks is one stranger's rating.** The GSS interviewer rates attractiveness after an hour-long interview, only
in 2016, 2018 and 2022. Only 66 men got a 5 and 79 a 1 or 2, so the extreme bars have wide intervals.

**Height in the paid-sex profile rests on 20 buyers** (GSS asked height only in 2022). Read it as a curiosity.
The height panel in the first chart uses NSFG instead, with about 2,200 young men.

**The NSFG height panel is narrow by design.** It follows NuancePill's sample: heterosexual non-Hispanic white
men aged 18-29. Older men, other groups, and outcomes beyond past-year sexlessness and virginity aren't covered.

**Britain vs the US.** The duration and wanting-more charts use Natsal-3 (Britain, 2010-12), because it is the
only large probability survey that asks both how often people have sex, as a count, and whether they want more.
Its achieved frequencies are close to the GSS's, but it is still a different country and a decade old.

**"Wanting more" is a direction, not an amount.** Natsal asks whether people would like sex more or less often,
not how much more. The desired-frequency fit in the wanting-more chart caps "a bit more" at twice the achieved
count. The cap moves the levels in panel B, but not the gap between the sexes.

**The porn and satisfaction charts are Aella's data, redrawn.** We don't have her microdata. Values were read
off her published charts by pixel measurement, to within about 0.2 points. Her relationships survey is a
convenience sample, mostly her own followers, who skew liberal and sexually open, so treat its levels as
indicative. Its patterns (sex falling with relationship length, porn rising) agree with the probability
surveys. Her sex line is per respondent, not per couple.

## Code

Run in the order listed. Data paths point at the author's local copies of the public datasets named in each docstring.

### `sexfreq_data.py`

```python
"""Sex frequency extracts from the GSS (US) and Natsal-3 (Britain), for the married-vs-single and years-married charts.

Run:  uv run --with pandas,pyarrow python notebooks/sexfreq/sexfreq_data.py
Reads: notebooks/us_public_data/GSS_stata/gss7224_r3a.dta (GSS 1972-2024 cumulative, R3a)
       notebooks/us_public_data/NATSAL/UKDA-7799-stata/stata/stata11/eul_natsal_2010_for_archive.dta (Natsal-3, 2010-12)
Writes: gss_sexfreq.parquet, natsal_sexfreq.parquet (next to this script)

GSS
  sexfreq  past-year frequency in 7 bands (asked 1989-2024, self-completion). Mapped to times/year at the usual
           midpoints 0, 2, 12, 30, 52, 130, 208: banded, so means are only as good as the midpoints, and the top
           band ("more than 3 times a week") is open-ended.
  partners numwomen + nummen, partners since age 18 (asked 1989-2022, not 2024). 989 = "989 or more"; codes above
           are missing. "Body count" here = all partners since 18, both sexes.
  dur      years in the current marriage = age - agewed, kept only for the married who were never divorced/separated
           (divorce == 2) or widowed (widowed == 2), so the first marriage is the current one. agewed was asked only
           in 1988-94 and 2006, so dur exists for those years only; it is whole years and can be off by one.
  conrinc  respondent's own income, constant 2000 $ (rescaled to 2024 $). Used as the income control because family
           income (coninc) mechanically counts a spouse's pay, which would soak up part of the married effect.
  w        wtssps (person post-stratification weight) scaled to mean 1 within each year.
Natsal-3
  sex4wks  occasions of heterosexual sex in the past 4 weeks: a count, not a band (995+ = missing).
  dur      years married to the current spouse = interview year - year married, for "married & living with spouse"
           whose most recent live-in partner is the spouse: year cohabitation began if they were married at the start
           (pp1ms == 1), otherwise the year they later married (pp1mar == 1, pp1ymar). Whole years, off by up to one.
  frequent "would you like to have sex more often, less often, or is it about right": 1-2 = wants more.
  sffsatis "I feel satisfied with my sex life": 1-2 agree, 4-5 disagree.
  totlife  lifetime partners, both sexes (9995+ missing). income: household, 8 bands, top band 50,000+.
  w        total_wt (core + boost samples).
"""
from pathlib import Path

import numpy as np
import pandas as pd

HERE = Path(__file__).resolve().parent
DATA = HERE.parent / "us_public_data"
GSS = DATA / "GSS_stata" / "gss7224_r3a.dta"
NATSAL = DATA / "NATSAL" / "UKDA-7799-stata" / "stata" / "stata11" / "eul_natsal_2010_for_archive.dta"
CPI_2000_TO_2024 = 313.689 / 172.2
TIMES_PER_YEAR = {0: 0, 1: 2, 2: 12, 3: 30, 4: 52, 5: 130, 6: 208}


def gss():
    cols = ["year", "age", "sex", "marital", "sexfreq", "numwomen", "nummen", "agewed", "divorce", "widowed",
            "degree", "conrinc", "coninc", "wtssps", "hapmar", "sexornt"]
    d = pd.read_stata(GSS, columns=cols, convert_categoricals=False)
    d = d[(d.year >= 1989) & d.sexfreq.isin(TIMES_PER_YEAR) & d.age.between(18, 89) & d.wtssps.notna()
          & d.marital.isin([1, 2, 3, 4, 5])].copy()
    d["female"] = (d.sex == 2).astype(int)
    d["times"] = d.sexfreq.map(TIMES_PER_YEAR).astype(float)
    d["weekly"] = (d.sexfreq >= 4).astype(float)
    d["sexless"] = (d.sexfreq == 0).astype(float)
    for c in ("numwomen", "nummen"):
        d.loc[d[c] > 989, c] = np.nan
    d["partners"] = d.numwomen + d.nummen
    d["inc_own"] = d.conrinc * CPI_2000_TO_2024
    d["inc_fam"] = d.coninc * CPI_2000_TO_2024
    first = (d.marital == 1) & (d.divorce == 2) & (d.widowed == 2)
    d["dur"] = np.where(first & d.agewed.between(12, 80), d.age - d.agewed, np.nan)
    d.loc[d.dur < 0, "dur"] = np.nan
    d["w"] = d.wtssps / d.groupby("year").wtssps.transform("mean")
    keep = ["year", "age", "female", "marital", "sexfreq", "times", "weekly", "sexless", "numwomen", "nummen",
            "partners", "degree", "inc_own", "inc_fam", "agewed", "divorce", "widowed", "dur", "hapmar", "sexornt", "w"]
    return d[keep].reset_index(drop=True)


def natsal():
    cols = ["rsex", "dage", "dateyoi", "marstat", "partnerhh", "pp1ms", "pp1mar", "pp1ystrt", "pp1ymar", "sex4wks",
            "frequent", "sffsatis", "sfrint", "relsat", "totlife", "hetlife", "educ3", "income", "total_wt"]
    d = pd.read_stata(NATSAL, columns=cols, convert_categoricals=False)
    d = d[(d.total_wt > 0) & (d.sex4wks < 995)].copy()
    d["female"] = (d.rsex == 2).astype(int)
    d["age"] = d.dage
    d["sex4wks"] = d.sex4wks.astype(float)
    for c in ("totlife", "hetlife"):
        d[c] = d[c].where(d[c] < 9995).astype(float)
    for c in ("frequent", "sffsatis", "sfrint", "relsat"):
        d[c] = d[c].where(d[c].between(1, 7)).astype(float)
    d["educ3"] = d.educ3.where(d.educ3.between(1, 5))
    d["income"] = d.income.where(d.income.between(1, 8))
    married = d.marstat == 2
    yr = np.where((d.pp1ms == 1) & d.pp1ystrt.between(1930, 2012), d.pp1ystrt,
                  np.where((d.pp1mar == 1) & d.pp1ymar.between(1930, 2012), d.pp1ymar, np.nan))
    d["year_married"] = np.where(married, yr, np.nan)
    d["dur"] = d.dateyoi - d.year_married
    d.loc[d.dur < 0, "dur"] = np.nan
    d["w"] = d.total_wt / d.total_wt.mean()
    keep = ["age", "female", "dateyoi", "marstat", "partnerhh", "year_married", "dur", "sex4wks", "frequent",
            "sffsatis", "sfrint", "relsat", "totlife", "hetlife", "educ3", "income", "w"]
    return d[keep].reset_index(drop=True)


def main():
    g = gss()
    g.to_parquet(HERE / "gss_sexfreq.parquet", index=False)
    print(f"GSS {g.year.min()}-{g.year.max()}: {len(g):,} rows; partners known {g.partners.notna().sum():,}; "
          f"married with duration {g.dur.notna().sum():,} (years {sorted(g[g.dur.notna()].year.unique())})")
    n = natsal()
    n.to_parquet(HERE / "natsal_sexfreq.parquet", index=False)
    print(f"Natsal-3: {len(n):,} rows; married {(n.marstat == 2).sum():,}, with duration {n.dur.notna().sum():,}")


if __name__ == "__main__":
    main()
```

### `sexfreq_model.py`

```python
"""Regression helpers for the sex-frequency charts: weighted quintiles and regression-adjusted cell means.

adjusted_means fits one weighted least-squares model per call,
    y ~ cell dummies (group x quintile, no intercept) + controls
with HC1 sandwich SEs, and reports each cell's mean standardized to the controls of the whole sample passed in
(every person's controls, with the cell switched on), so all cells are compared at the same education, income, age
and survey-year mix. Being linear, that standardized mean is b_cell + mean(controls) @ b_controls, and its SE comes
straight from the coefficient covariance.
"""
import numpy as np
import pandas as pd


def wls_robust(y, X, w):
    """WLS with HC1 sandwich covariance. Returns coef, cov."""
    sw = np.sqrt(w)
    Xw, yw = X * sw[:, None], y * sw
    XtX_inv = np.linalg.pinv(Xw.T @ Xw)
    b = XtX_inv @ Xw.T @ yw
    e = yw - Xw @ b
    meat = (Xw * e[:, None]).T @ (Xw * e[:, None])
    n, k = X.shape
    return b, XtX_inv @ meat @ XtX_inv * n / (n - k)


def weighted_quintile(x, w):
    """Weighted quintile 1..5 of x; tied values share the quintile of their tie group's weighted midpoint."""
    x, w = pd.Series(x), pd.Series(w, index=x.index)
    order = x.sort_values().index
    cw = w[order].cumsum() / w.sum()
    mid = cw - w[order] / w.sum() / 2
    tie_mid = mid.groupby(x[order]).transform("mean")
    return np.clip((tie_mid * 5).astype(int) + 1, 1, 5).reindex(x.index)


def dummies(s, prefix, drop_first=True):
    return pd.get_dummies(s.astype("category"), prefix=prefix, drop_first=drop_first, dtype=float)


def adjusted_means(d, y, cell, controls):
    """Regression-adjusted mean of y for each value of `cell`, standardized to d's control mix.

    controls: DataFrame of control columns (already dummied), indexed like d.
    Returns a DataFrame indexed by cell value: adj, se, lo, hi (95%), raw (weighted), n.
    """
    C = dummies(d[cell], "c", drop_first=False)
    X = pd.concat([C, controls], axis=1)
    b, V = wls_robust(d[y].to_numpy(float), X.to_numpy(float), d.w.to_numpy(float))
    k = C.shape[1]
    cbar = controls.to_numpy(float).mean(axis=0)
    rows = {}
    for j, name in enumerate(C.columns):
        g = np.zeros(X.shape[1])
        g[j] = 1.0
        g[k:] = cbar
        m, se = g @ b, np.sqrt(g @ V @ g)
        lev = C[name] == 1
        rows[name[2:]] = dict(adj=m, se=se, lo=m - 1.96 * se, hi=m + 1.96 * se,
                              raw=np.average(d.loc[lev, y], weights=d.loc[lev, "w"]), n=int(lev.sum()))
    out = pd.DataFrame(rows).T
    out.attrs["coef"], out.attrs["cov"], out.attrs["cols"] = b, V, list(X.columns)
    return out


def contrast(res, a, b_):
    """Difference adj[a] - adj[b_] with its SE (cells are from one model, so use the joint covariance)."""
    cols, b, V = res.attrs["cols"], res.attrs["coef"], res.attrs["cov"]
    g = np.zeros(len(b))
    g[cols.index(f"c_{a}")], g[cols.index(f"c_{b_}")] = 1.0, -1.0
    return g @ b, np.sqrt(g @ V @ g)
```

### `master_post_data.py`

```python
"""Data for "The master post of sexual and relationship dynamics": every number behind the post's reworked charts.

    uv run --with pandas,pyarrow,numpy,scipy,statsmodels python notebooks/master_post/master_post_data.py

Writes small CSVs next to this script; `master_post_charts.py` draws them. Run from the repo root.

**Sources computed here.**

* **GSS 1972-2024 cumulative file** (`gss7224_r2.dta`), survey-weighted with `wtssps` (missing weight = 1).
  - *Looks* (`rlooks`, asked 2016, 2018 and 2022): the interviewer rates the respondent's physical attractiveness 1-5 at the end of the
    interview. Men aged 25-60, lifetime female partners since 18 (`numwomen`, 0-989), winsorised at 100.
    Ratings 1 and 2 are pooled ("1-2, below average") because rating 1 has too few men to stand alone.
  - *SES rungs*: own income (`conrinc`) and occupational prestige (`prestg10`), each ranked within survey year x
    sex and cut into thirds. "Bottom both" = bottom third on both; "bottom either" = bottom third on one; "high
    both" = top third on both; "high either" = top third on one (and not bottom on the other); the rest are
    "middle". Split by bachelor's degree or more (`degree` >= 3). Men 25-60, partners winsorised at 100.
  - *Paying for sex* (`evpaidsx`, men 25-60, 1991-2022): share who have ever paid, by own-income decile ranked
    within survey year; and standardised differences (buyers minus non-buyers, divided by the pooled SD) on
    degree, income and prestige rank, interviewer looks (raw and SES-adjusted: residual after regressing on age,
    age^2, income rank, prestige rank, degree and year), height (`rheight`, 2022 only), lifetime partners
    (capped 100) and marital status.
  - Intervals are percentile bootstraps of the weighted mean (800 draws; 600 for the standardised differences).

* **Natsal-3** (Britain 2010-12, `eul_natsal_2010_for_archive.dta`), weighted with `total_wt`.
  - *Duration*: respondents living with a partner (`partnerhh` = 1); duration = age - age at first sex with
    the current partner (`rafsmr`). Occasions of sex in the last 4 weeks (`sex4wks`, a count) are fitted with a
    Poisson GLM, and "would like sex more often" (`frequent` = much or a bit more) with a logistic GLM. Both use
    age (centred at 35, quadratic) and duration (log1p, a linear term to 5 years, and a slope after 5) as
    separate terms, and are drawn at age 35, so the curves show years together rather than ageing.
  - *Wanting more*: partnered (`partnerhh` = 1) men and women aged 25-40. Share wanting more, by fifths of
    achieved frequency within sex. Desired monthly frequency is fitted per sex as an interval-censored lognormal
    (achieved count plus direction of the wish; "a bit more" is capped at twice the achieved count), and the
    matching table gives, for a stated desired frequency, the share of each sex whose desired frequency is at
    least that high.

* **NSFG 2011-13, 2013-15, 2015-17 male files** (CDC public-use, `nsfg_2011_2017/`), pooled with each cycle's
  final weight divided by 3. Heterosexual (`ORIENT`/`ORIENT_A`/`ORIENT_B` = 1) non-Hispanic white (`HISPRACE2` = 2)
  men aged 18-29 with a self-reported height (`INCHES`): share with no opposite-sex partner in the past 12 months
  (`PARTS1YR` = 0, which includes virgins) and share who never had intercourse (`HADSEX` = 2), by height in
  inches (65 and under, 76 and over pooled). Replicates NuancePill, "The Bumble height filter graph and height's
  effect on male celibacy" (https://nuancepill.substack.com/p/the-bumble-height-filter-graph-and-heights-effect-on-male-celibacy),
  whose idea and sample definition this follows; our n is 2,191 against their 2,623 (they likely used a looser
  orientation filter), and the pattern is the same: flat across heights.

**Sources digitised, not recomputed** (we do not have the microdata; values read off the author's published
charts by pixel measurement against their axis gridlines, accurate to about +/-0.2 points):

* **Sex and porn by relationship length, and sex-life satisfaction**: Aella, "Monogamous men in long-term
  relationships aren't doin too hot" (Knowingless, 14 Mar 2023,
  https://aella.substack.com/p/monogamous-men-in-long-term-relationships). Her relationships survey (~25,000
  respondents, mostly her own followers). Monthly sex and porn occasions by relationship length in years; and
  agreement with "I'm satisfied with my sex life" (-3 to +3) for straight, non-poly men in relationships of
  12+ years (n = 3,720).
"""
import re
import sys
from pathlib import Path

import numpy as np
import pandas as pd
import statsmodels.api as sm
from scipy.optimize import minimize
from scipy.stats import norm

HERE = Path(__file__).resolve().parent
DATA = HERE.parent / "us_public_data"
GSS = DATA / "gss7224_r2.dta"
NAT = DATA / "NATSAL" / "UKDA-7799-stata" / "stata" / "stata11" / "eul_natsal_2010_for_archive.dta"


def wmean(x, w):
    x = np.asarray(x, float); w = np.asarray(w, float)
    return float((x * w).sum() / w.sum())


def boot_ci(x, w, n=800, seed=0):
    x = np.asarray(x, float); w = np.asarray(w, float)
    rng = np.random.default_rng(seed)
    out = [wmean(x[i], w[i]) for i in (rng.integers(0, len(x), len(x)) for _ in range(n))]
    return tuple(np.percentile(out, [2.5, 97.5]))


def read_gss(cols):
    return pd.concat(list(pd.read_stata(GSS, columns=cols, convert_categoricals=False, chunksize=20000)),
                     ignore_index=True)


# ---------------------------------------------------------------- GSS: looks and SES (men 25-60)
g = read_gss(["year", "age", "sex", "numwomen", "rlooks", "conrinc", "prestg10", "degree", "wtssps"])
g = g[g.age.between(25, 60) & g.sex.isin([1, 2])].copy()
g["w"] = g.wtssps.fillna(1.0)
g["inc_pct"] = g.groupby(["year", "sex"]).conrinc.rank(pct=True) * 100
g["pres_pct"] = g.groupby(["year", "sex"]).prestg10.rank(pct=True) * 100
men = g[g.sex.eq(1) & g.numwomen.between(0, 989)].copy()
men["bc"] = men.numwomen.clip(upper=100)

lk = men[men.rlooks.between(1, 5)].copy()
LOOKS_LAB = {1: "1 (lowest)", 2: "2", 3: "3", 4: "4", 5: "5 (highest)"}
rows = []
for k, lab in LOOKS_LAB.items():
    s = lk[lk.rlooks == k]
    lo, hi = boot_ci(s.bc, s.w)
    rows.append({"looks": lab, "code": k, "n": len(s), "mean_partners": wmean(s.bc, s.w), "lo": lo, "hi": hi})
LOOKS = pd.DataFrame(rows)
LOOKS.to_csv(HERE / "looks_partners.csv", index=False)
print("looks:\n", LOOKS.round(2).to_string(index=False))

bk = men[men.inc_pct.notna() & men.pres_pct.notna() & men.degree.between(0, 4)].copy()
bk["college"] = bk.degree.ge(3)
lo_i, hi_i = bk.inc_pct.lt(100 / 3), bk.inc_pct.gt(200 / 3)
lo_p, hi_p = bk.pres_pct.lt(100 / 3), bk.pres_pct.gt(200 / 3)
RUNGS = ["bottom both", "bottom either", "middle", "high either", "high both"]
bk["rung"] = np.select([lo_i & lo_p, hi_i & hi_p, lo_i | lo_p, hi_i | hi_p], [RUNGS[0], RUNGS[4], RUNGS[1], RUNGS[3]],
                       default="middle")
rows = []
for coll in (True, False):
    for r in RUNGS:
        s = bk[(bk.college == coll) & (bk.rung == r)]
        lo, hi = boot_ci(s.bc, s.w)
        rows.append({"college": coll, "rung": r, "n": len(s), "mean_partners": wmean(s.bc, s.w), "lo": lo, "hi": hi})
SES = pd.DataFrame(rows)
SES.to_csv(HERE / "ses_partners.csv", index=False)
print("SES rungs (men):\n", SES.round(2).to_string(index=False))

# ---------------------------------------------------------------- GSS: paying for sex (men 25-60)
p = read_gss(["year", "age", "sex", "evpaidsx", "rlooks", "rheight", "numwomen", "degree", "conrinc", "prestg10",
              "marital", "wtssps"])
p = p[p.sex.eq(1) & p.age.between(25, 60) & p.evpaidsx.isin([1, 2])].copy()
p["w"] = p.wtssps.fillna(1.0)
p["paid"] = p.evpaidsx.eq(1)
p["inc_pct"] = p.groupby("year").conrinc.rank(pct=True) * 100
p["pres_pct"] = p.groupby("year").prestg10.rank(pct=True) * 100
p["ba"] = p.degree.where(p.degree.between(0, 4)).ge(3)
p.loc[~p.degree.between(0, 4), "ba"] = np.nan
p["bc100"] = p.numwomen.where(p.numwomen.between(0, 989)).clip(upper=100)
mar = p.marital.where(p.marital.between(1, 5))
p["evermar"] = mar.ne(5).where(mar.notna())
p["curmar"] = mar.eq(1).where(mar.notna())
p["looks"] = p.rlooks.where(p.rlooks.between(1, 5))
p["height"] = p.rheight.where(p.rheight.between(48, 90))
PAID_SHARE = wmean(p.paid, p.w)

lkp = p[p.looks.notna() & p.inc_pct.notna() & p.pres_pct.notna() & p.ba.notna()].copy()
X = np.column_stack([np.ones(len(lkp)), (lkp.age - 42) / 10, ((lkp.age - 42) / 10) ** 2, lkp.inc_pct / 100,
                     lkp.pres_pct / 100, lkp.ba.astype(float), (lkp.year - 2019) / 10])
lkp["looks_resid"] = lkp.looks - sm.WLS(lkp.looks.astype(float), X, weights=lkp.w).fit().predict(X)

MEASURES = [("Lifetime partners", "bc100"), ("Height", "height"), ("Looks, SES-adjusted", "looks_resid"),
            ("Looks (interviewer)", "looks"), ("Income rank", "inc_pct"), ("Prestige rank", "pres_pct"),
            ("Has a BA+", "ba"), ("Ever married", "evermar"), ("Currently married", "curmar")]
rows = []
for lab, col in MEASURES:
    src = lkp if col == "looks_resid" else p
    d = src[src[col].notna()]
    a, b = d[d.paid], d[~d.paid]
    pooled = np.sqrt((np.var(a[col].astype(float)) + np.var(b[col].astype(float))) / 2)
    diff = wmean(a[col], a.w) - wmean(b[col], b.w)
    rng = np.random.default_rng(0); boots = []
    for _ in range(600):
        ia, ib = rng.integers(0, len(a), len(a)), rng.integers(0, len(b), len(b))
        boots.append(wmean(a[col].iloc[ia], a.w.iloc[ia]) - wmean(b[col].iloc[ib], b.w.iloc[ib]))
    lo, hi = np.percentile(boots, [2.5, 97.5])
    rows.append({"measure": lab, "buyers": wmean(a[col], a.w), "non_buyers": wmean(b[col], b.w), "d": diff / pooled,
                 "d_lo": lo / pooled, "d_hi": hi / pooled, "n_buyers": len(a), "n_others": len(b)})
PAIDP = pd.DataFrame(rows)
PAIDP.to_csv(HERE / "paid_sex_profile.csv", index=False)
print(f"ever paid for sex: {PAID_SHARE:.1%} of men 25-60 (n = {len(p):,})")
print(PAIDP.round(2).to_string(index=False))

inc = p[p.inc_pct.notna()].copy()
inc["dec"] = np.minimum((inc.inc_pct // 10).astype(int), 9) + 1
rows = []
for d_ in range(1, 11):
    s = inc[inc.dec == d_]
    lo, hi = boot_ci(s.paid, s.w)
    rows.append({"decile": d_, "share_paid": wmean(s.paid, s.w), "lo": lo, "hi": hi, "n": len(s)})
rows.append({"decile": 0, "share_paid": PAID_SHARE, "lo": np.nan, "hi": np.nan, "n": len(p)})   # 0 = all men
pd.DataFrame(rows).to_csv(HERE / "paid_sex_by_income.csv", index=False)

# ---------------------------------------------------------------- Natsal: by years married (reuses sexfreq/)
sys.path.insert(0, str(HERE.parent / "sexfreq"))
from sexfreq_model import adjusted_means  # noqa: E402

NATSAL_BINS = [-1, 0, 1, 2, 3, 4, 6, 9, 14, 19, 29, 80]
NATSAL_X = [0, 1, 2, 3, 4, 5.5, 8, 12, 17, 24.5, 35]
ns_ = pd.read_parquet(HERE.parent / "sexfreq" / "natsal_sexfreq.parquet")
mar = ns_[ns_.dur.notna()].copy()


def by_years_married(d, y):
    d = d.copy()
    d["db"] = pd.cut(d.dur, NATSAL_BINS, labels=False)
    ctrl = pd.DataFrame({"a": d.age - 40, "a2": (d.age - 40) ** 2 / 100}, index=d.index)
    r = adjusted_means(d, y, "db", ctrl)
    r.index = r.index.astype(int)
    r = pd.DataFrame(r.to_dict()).sort_index()
    r["years_married"] = [NATSAL_X[i] for i in r.index]
    return r


rows = []
for f, sx in [(0, "men"), (1, "women")]:
    d = mar[mar.female == f].assign(pm=lambda x: x.sex4wks)
    rows.append(by_years_married(d[d.pm.notna()], "pm").assign(sex=sx, outcome="sex_per_month"))
    d = mar[(mar.female == f) & mar.frequent.notna()].assign(more=lambda x: (x.frequent <= 2).astype(float))
    rows.append(by_years_married(d, "more").assign(sex=sx, outcome="want_more"))
single = ns_[(ns_.female == 0) & (ns_.partnerhh == 0) & ns_.marstat.isin([1, 4, 5, 6]) & ns_.frequent.notna()]
YM = pd.concat(rows)
YM = pd.concat([YM, pd.DataFrame([{"sex": "single men", "outcome": "want_more",
                                    "adj": np.average(single.frequent <= 2, weights=single.w), "n": len(single)}])])
YM.to_csv(HERE / "natsal_years_married.csv", index=False)
print(f"Natsal married with a known marriage year: n = {len(mar):,}")
print(YM[["sex", "outcome", "years_married", "adj", "lo", "hi", "n"]].round(2).to_string(index=False))

# ---------------------------------------------------------------- Natsal: desired frequency (partnered 25-40)
nat = pd.read_stata(NAT, columns=["rsex", "dage", "sex4wks", "frequent", "total_wt", "partnerhh"],
                    convert_categoricals=False)
nat = nat[(nat.sex4wks < 900) & nat.frequent.between(1, 5) & (nat.total_wt > 0)].copy()
nat["male"] = nat.rsex.eq(1); nat["w"] = nat.total_wt; nat["A"] = nat.sex4wks.astype(float)
nb = nat[nat.dage.between(25, 40) & nat.partnerhh.eq(1)]
WANT_ALL = {sx: wmean(nb[nb.male == (sx == "men")].frequent.le(2), nb[nb.male == (sx == "men")].w)
            for sx in ("men", "women")}


def fit_desired(s, k=2.0):
    A = np.clip(s.A.to_numpy(float), 0.25, None); w = s.w.to_numpy(); f = s.frequent.to_numpy()
    lo = np.where(f == 1, k * A, np.where(f == 2, A, np.where(f == 3, A * 0.999, np.where(f == 4, A / 2, 1e-6))))
    hi = np.where(f == 1, np.inf, np.where(f == 2, k * A, np.where(f == 3, A * 1.001, np.where(f == 4, A, A / 2))))

    def nll(par):
        mu, ls = par[0], np.exp(par[1])
        pr = norm.cdf((np.log(hi) - mu) / ls) - norm.cdf((np.log(lo) - mu) / ls)
        return -float((w * np.log(np.clip(pr, 1e-12, None))).sum())

    r = minimize(nll, [1.5, 0.0], method="Nelder-Mead", options=dict(maxiter=9000, xatol=1e-7, fatol=1e-7))
    return float(r.x[0]), float(np.exp(r.x[1]))


FIT = {sx: fit_desired(nb[nb.male == (sx == "men")]) for sx in ("men", "women")}
xs = np.round(np.arange(1, 30.01, 0.25), 2)
(mu_m, sg_m), (mu_w, sg_w) = FIT["men"], FIT["women"]
MATCH = pd.DataFrame({
    "per_month": xs,
    "women_want_at_least": norm.sf((np.log(xs) - mu_w) / sg_w),    # he wants x: women who want x or more
    "men_want_at_most": norm.cdf((np.log(xs) - mu_m) / sg_m),      # she wants x: men who want x or less
})
MATCH.to_csv(HERE / "natsal_desire_matching.csv", index=False)
print({sx: f"median wanted {np.exp(mu):.1f}/mo" for sx, (mu, sg) in FIT.items()}, WANT_ALL)
print(MATCH[MATCH.per_month.isin([2, 4.25, 4.5, 8.75, 13, 17.25, 30])].round(3).to_string(index=False))

# ---------------------------------------------------------------- GSS: men's lifetime partner distribution (40-60)
bcg = read_gss(["year", "age", "sex", "numwomen", "marital", "degree", "conrinc", "evpaidsx", "xmovie", "wtssps"])
bcg = bcg[bcg.sex.eq(1) & bcg.age.between(40, 60) & bcg.numwomen.between(0, 989)].copy()
bcg["w"] = bcg.wtssps.fillna(1.0)
bcg["inc"] = bcg.groupby("year").conrinc.rank(pct=True)


def wq(x, w, q):
    o = np.argsort(np.asarray(x, float)); x_, w_ = np.asarray(x, float)[o], np.asarray(w, float)[o]
    return float(np.interp(q, (np.cumsum(w_) - 0.5 * w_) / w_.sum(), x_))


PCTS = [10, 20, 30, 40, 50, 60, 70, 80, 90, 95]
dist = [{"stat": f"p{q}", "value": wq(bcg.numwomen, bcg.w, q / 100)} for q in PCTS]
dist.append({"stat": "mean_capped100", "value": wmean(bcg.numwomen.clip(upper=100), bcg.w)})
dist.append({"stat": "share_below_mean", "value": wmean(bcg.numwomen < wmean(bcg.numwomen.clip(upper=100), bcg.w), bcg.w)})
# fifths of men ranked by count (ties split by position), and each fifth's share of all partners (capped at 100)
o = bcg.sort_values("numwomen")
cum = (np.cumsum(o.w) - 0.5 * o.w) / o.w.sum()
o["fifth"] = np.minimum((cum * 5).astype(int), 4) + 1
tot = (o.numwomen.clip(upper=100) * o.w).sum()
for q, d in o.groupby("fifth"):
    dist.append({"stat": f"fifth{q}_share", "value": float((d.numwomen.clip(upper=100) * d.w).sum() / tot)})
    dist.append({"stat": f"fifth{q}_mean", "value": wmean(d.numwomen.clip(upper=100), d.w)})
DIST = pd.DataFrame(dist)
DIST.to_csv(HERE / "bodycount_distribution.csv", index=False)
print(f"men 40-60 with a partner count: n = {len(bcg):,}")
print(DIST.round(2).to_string(index=False))

CUTS = [("All men 40-60", bcg.index == bcg.index, "all"),
        ("Bachelor's+", bcg.degree.ge(3), "status"), ("No degree", bcg.degree.between(0, 2), "status"),
        ("Income top third", bcg.inc.gt(2 / 3), "status"), ("Income bottom third", bcg.inc.lt(1 / 3), "status"),
        ("Married", bcg.marital.eq(1), "history"), ("Never married", bcg.marital.eq(5), "history"),
        ("Divorced/separated", bcg.marital.isin([3, 4]), "history"),
        ("Never paid for sex", bcg.evpaidsx.eq(2), "appetite"), ("Has paid for sex", bcg.evpaidsx.eq(1), "appetite"),
        ("No X-rated film, past yr", bcg.xmovie.eq(2), "appetite"), ("Saw X-rated film, past yr", bcg.xmovie.eq(1), "appetite")]
rows = []
for lab, mask, grp in CUTS:
    d = bcg[mask]
    rows.append({"group": lab, "family": grp, "n": len(d), "median": wq(d.numwomen, d.w, .5),
                 "p80": wq(d.numwomen, d.w, .8), "p90": wq(d.numwomen, d.w, .9),
                 "mean_capped100": wmean(d.numwomen.clip(upper=100), d.w)})
CUT = pd.DataFrame(rows)
CUT.to_csv(HERE / "bodycount_crosscuts.csv", index=False)
print(CUT.round(1).to_string(index=False))

# ---------------------------------------------------------------- NSFG 2011-17: height (replicating NuancePill)
NSFG = DATA / "nsfg_2011_2017"
NEED = ["AGE_R", "HISPRACE2", "HADSEX", "PARTS1YR", "ORIENT", "ORIENT_A", "ORIENT_B", "INCHES"]


RE_COL = re.compile(r"\s*_column\((\d+)\)\s+\w+\s+(\w+)\s+%(\d+)[a-z]")   # start, name, width


def read_nsfg(cycle):
    """Read the needed fixed-width columns of an NSFG male file using its Stata .dct layout."""
    spec = {}
    for line in (NSFG / f"{cycle}_MaleSetup.dct").read_text(errors="ignore").splitlines():
        m = RE_COL.match(line)
        if m:
            spec[m.group(2)] = (int(m.group(1)) - 1, int(m.group(1)) - 1 + int(m.group(3)))
    names = [c for c in NEED + [f"WGT{cycle}"] if c in spec]
    df = pd.read_fwf(NSFG / f"{cycle}_MaleData.dat", colspecs=[spec[c] for c in names], names=names, header=None)
    df["w"] = df.pop(f"WGT{cycle}") / 3            # three pooled cycles
    df["cycle"] = cycle
    return df


ns = pd.concat([read_nsfg(c) for c in ("2011_2013", "2013_2015", "2015_2017")], ignore_index=True)
orient = ns.ORIENT.fillna(ns.get("ORIENT_A")).fillna(ns.get("ORIENT_B"))
ns = ns[ns.AGE_R.between(18, 29) & ns.HISPRACE2.eq(2) & orient.eq(1) & ns.INCHES.between(48, 90)
        & ns.PARTS1YR.between(0, 50)].copy()
ns["sexless"] = ns.PARTS1YR.eq(0)
ns["virgin"] = ns.HADSEX.eq(2)
ns["hbin"] = ns.INCHES.clip(65, 76).astype(int)
HLAB = {65: "≤5'5", 66: "5'6", 67: "5'7", 68: "5'8", 69: "5'9", 70: "5'10", 71: "5'11", 72: "6'0", 73: "6'1",
        74: "6'2", 75: "6'3", 76: "≥6'4"}
rows = []
for h, lab in HLAB.items():
    s = ns[ns.hbin == h]
    r = {"inches": h, "height": lab, "n": len(s)}
    for col in ("sexless", "virgin"):
        lo, hi = boot_ci(s[col], s.w)
        r.update({f"pct_{col}": 100 * wmean(s[col], s.w), f"{col}_lo": 100 * lo, f"{col}_hi": 100 * hi})
    rows.append(r)
rows.append({"inches": 0, "height": "all", "n": len(ns), "pct_sexless": 100 * wmean(ns.sexless, ns.w),
             "pct_virgin": 100 * wmean(ns.virgin, ns.w)})
HT = pd.DataFrame(rows)
HT.to_csv(HERE / "nsfg_height_sexless.csv", index=False)
print(f"NSFG 2011-17 het. NH white men 18-29 with height: n = {len(ns):,}")
print(HT.round(1).to_string(index=False))

# ---------------------------------------------------------------- digitised external series
pd.DataFrame({
    "years_together": ["0-1", "1-2", "2-4", "4-6", "6-8", "8-10", "10-12", "12-14", "14-16", "16-22", "22+"],
    "sex_per_month": [12.15, 8.79, 6.89, 5.43, 4.75, 4.41, 4.41, 4.16, 4.32, 3.87, 4.16],
    "porn_per_month_men": [10.80, 11.71, 12.72, 13.84, 14.58, 13.60, 13.84, 13.76, 13.54, 13.34, 12.29],
    "porn_per_month_women": [3.56, 2.91, 3.24, 3.28, 2.57, 2.68, 2.24, 2.51, 2.68, 2.67, 1.69],
}).assign(source="Aella relationships survey, Knowingless 2023-03-14 (digitised)").to_csv(
    HERE / "aella_sex_porn_digitised.csv", index=False)

sat = np.array([13.74, 21.83, 17.95, 6.03, 13.88, 20.65, 6.12])
pd.DataFrame({"agreement": [-3, -2, -1, 0, 1, 2, 3], "pct": sat / sat.sum() * 100}).assign(
    source="Aella relationships survey, straight non-poly men 12+ yrs, n=3,720 (digitised)").to_csv(
    HERE / "aella_satisfaction_digitised.csv", index=False)
print("digitised CSVs written")
```

### `master_post_charts.py`

```python
"""Charts for "The master post of sexual and relationship dynamics" (Performative Bafflement).

    uv run --with matplotlib,pillow,fonttools,pandas,numpy python notebooks/master_post/master_post_charts.py

Draws the post's five reworked figures from the CSVs that `master_post_data.py` writes (run that first), saves
the unwatermarked PNGs to ~/Desktop/Recent Projects/Substack Notes/Unwatermarked graphs/Sex/ and the watermarked
`_wm.png` copies to ~/Desktop/Recent Projects/Substack Notes/Sex/ (house watermark, style 7; this also rebuilds
the methods page pbmethods.github.io/sex-dynamics, unless PB_NO_PUSH=1).

1. `motivation_looks_ses_height` -- men's lifetime partners by interviewer-rated looks and by income x prestige
   rung (GSS), and the share of young men sexless in the past year by height (NSFG 2011-17).
2. `paid_sex_profile` -- how men who have paid for sex differ from those who have not (standardised
   differences), and the share who have paid by income decile (GSS).
3. `married_sex_porn` -- monthly sex and wanting more by years together at a fixed age of 35 (Natsal-3), and sex
   versus porn by relationship length (Aella's relationships survey, digitised).
4. `wants_more` -- who wants more sex by how much they already have, and how many people of the other sex want
   at least a given frequency (Natsal-3).
5. `sex_satisfaction` -- long-partnered men's agreement with "I'm satisfied with my sex life", in full and
   grouped (Aella's relationships survey, digitised).

Every number and method is in `master_post_data.py`'s docstring; caveats are in CAVEATS.md.
"""
import os
import subprocess
import sys
from pathlib import Path

import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

sys.path.insert(0, str(Path.home() / "_watermark"))
import pb_style as pb  # noqa: E402

HERE = Path(__file__).resolve().parent
NOTES = Path.home() / "Desktop" / "Recent Projects" / "Substack Notes"
OUT_WM, OUT_RAW = NOTES / "Sex", NOTES / "Unwatermarked graphs" / "Sex"
OUT_WM.mkdir(parents=True, exist_ok=True); OUT_RAW.mkdir(parents=True, exist_ok=True)

P = pb.apply()
INK, INK2, MUTED, NAVY, RED, MUTE, GRAY = P["INK"], P["INK2"], P["MUTED"], P["NAVY"], P["RED"], P["MUTE"], pb.GRAYS[0]
GRID_C = P["GRID"]
pct = plt.FuncFormatter(lambda v, _: f"{v:.0%}")
ppt = plt.FuncFormatter(lambda v, _: f"{v:.0f}%")
read = lambda name: pd.read_csv(HERE / name)


def header(fig, title, subtitle, y=1.0):
    fig.text(0.01, y + 0.075, title, fontsize=17, fontweight="bold", color=INK, ha="left", va="bottom")
    fig.text(0.01, y + 0.035, subtitle, fontsize=11, color=INK2, ha="left", va="bottom")


def panel_title(ax, letter, text):
    ax.set_title(f"{letter}   {text}", loc="left", fontsize=11.5, fontweight="bold", color=INK, pad=12)


def footnote(fig, text, y=-0.04):
    fig.text(0.01, y, text, fontsize=8.2, color=MUTED, ha="left", va="top", linespacing=1.45)


def errbars(ax, x, v, lo, hi, color=INK2):
    ax.errorbar(x, v, yerr=[np.asarray(v) - np.asarray(lo), np.asarray(hi) - np.asarray(v)], fmt="none",
                ecolor=color, elinewidth=0.9, capsize=2.5, zorder=3)


def save(fig, name):
    raw = OUT_RAW / f"{name}.png"
    fig.savefig(raw, dpi=200, bbox_inches="tight", pad_inches=0.25)
    plt.close(fig)
    wm = OUT_WM / f"{name}_wm.png"
    subprocess.run([sys.executable, str(Path.home() / "_watermark" / "watermark.py"), "7", str(raw), str(wm)],
                   check=True, env=os.environ.copy())
    print("wrote", wm)


# =============================================================== 1. looks, SES, height
looks, ses, ht = read("looks_partners.csv"), read("ses_partners.csv"), read("nsfg_height_sexless.csv")
fig, axes = plt.subplots(1, 3, figsize=(16, 5.4), gridspec_kw=dict(width_ratios=[1.0, 1.25, 1.35], wspace=0.24))

ax = axes[0]
x = np.arange(len(looks))
ax.bar(x, looks.mean_partners, width=0.62, color=NAVY)
errbars(ax, x, looks.mean_partners, looks.lo, looks.hi)
for xi, v, hi, n in zip(x, looks.mean_partners, looks.hi, looks.n):
    ax.text(xi, hi + 0.6, f"{v:.0f}", ha="center", fontsize=10.5, fontweight="bold", color=INK)
    ax.text(xi, 0.6, f"n={n}", ha="center", fontsize=7, color="white")
ax.set_xticks(x, ["1\nlowest", "2", "3", "4", "5\nhighest"], fontsize=9)
ax.set_ylim(0, 27); ax.set_ylabel("lifetime female partners (mean)")
ax.set_xlabel("interviewer's looks rating, 1-5")
panel_title(ax, "A", "Looks")

ax = axes[1]
rungs = ["bottom both", "bottom either", "middle", "high either", "high both"]
x = np.arange(len(rungs))
for i, (coll, colr, lab) in enumerate([(True, NAVY, "college grads"), (False, GRAY, "no degree")]):
    d = ses[ses.college == coll].set_index("rung").loc[rungs]
    xs = x + (i - 0.5) * 0.38
    ax.bar(xs, d.mean_partners, width=0.36, color=colr)
    errbars(ax, xs, d.mean_partners, d.lo, d.hi)
    for xi, v, hi in zip(xs, d.mean_partners, d.hi):
        ax.text(xi, hi + 0.6, f"{v:.0f}", ha="center", fontsize=9.5, fontweight="bold", color=INK)
    ax.text(-0.45, 24.5 - 2.0 * i, "■ " + lab, fontsize=9.5, color=colr, fontweight="bold", ha="left")
ax.set_xticks(x, ["bottom\nboth", "bottom\neither", "middle", "top\neither", "top\nboth"], fontsize=9)
ax.set_xlabel("income x occupational prestige (thirds, within year)")
ax.set_ylim(0, 27)
panel_title(ax, "B", "Income and job prestige")

ax = axes[2]
x = np.arange(12)
h = ht[ht.inches > 0]
ax.bar(x, h.pct_sexless, width=0.68, color=NAVY)
errbars(ax, x, h.pct_sexless, h.sexless_lo, h.sexless_hi)
avg = float(ht.loc[ht.inches == 0, "pct_sexless"].iloc[0])
ax.axhline(avg, color=INK, lw=1, ls=(0, (4, 3)))
ax.text(11.5, avg + 0.7, f"all heights\n{avg:.0f}%", ha="left", va="bottom", fontsize=9, color=INK, fontweight="bold")
ax.set_xlim(-0.6, 12.9)
ax.set_xticks(x, h.height, fontsize=8.5)
ax.yaxis.set_major_formatter(ppt); ax.set_ylim(0, 36)
ax.set_ylabel("no sex in the past year"); ax.set_xlabel("height, men 18-29")
panel_title(ax, "C", "Height")

header(fig, "Looks, money and height barely move men's sex lives",
       "Lifetime partners by looks (A) and by income and prestige (B), men 25-60; share of young men with no sex "
       "in the past year, by height (C)", y=0.92)
footnote(fig, "A, B: GSS, men 25-60, survey-weighted means, partners capped at 100, bootstrap 95% intervals. A: 2016-22, the only "
              "years with the interviewer's 1-5 looks rating. B: 1989-2022.\n"
              "B: income and occupational prestige each split into thirds within survey year; 'top both' = top third "
              "on both. College = bachelor's or more.\n"
              f"C: NSFG 2011-17, heterosexual non-Hispanic white men 18-29 (n = {int(ht.loc[ht.inches == 0, 'n'].iloc[0]):,}), "
              "weighted. Replicates NuancePill's analysis (nuancepill.substack.com); virginity by height is just as flat.",
         y=0.0)
save(fig, "motivation_looks_ses_height")

# =============================================================== 2. paying for sex
prof, inc = read("paid_sex_profile.csv"), read("paid_sex_by_income.csv")
fig, (ax, ax2) = plt.subplots(1, 2, figsize=(14.5, 5.6), gridspec_kw=dict(width_ratios=[1.1, 1], wspace=0.5))

d = prof.iloc[::-1].reset_index(drop=True)
y = np.arange(len(d))
cols = [RED if m == "Lifetime partners" else MUTE for m in d.measure]
ax.barh(y, d.d, color=cols, height=0.62)
ax.errorbar(d.d, y, xerr=[d.d - d.d_lo, d.d_hi - d.d], fmt="none", ecolor=INK2, elinewidth=0.9, capsize=2.5)
for yi, v, hi, lo in zip(y, d.d, d.d_hi, d.d_lo):
    xt = hi + 0.04 if v >= 0 else lo - 0.04
    ax.text(xt, yi, f"{v:+.2f}", va="center", ha="left" if v >= 0 else "right", fontsize=9, fontweight="bold", color=INK)
labels = [f"{m}  (n={nb:,})" if m != "Height" else "Height  (n=20, 2022 only)" for m, nb in zip(d.measure, d.n_buyers)]
ax.set_yticks(y, labels, fontsize=9.5)
ax.axvline(0, color=INK, lw=0.9)
ax.grid(axis="y", visible=False); ax.grid(axis="x", visible=True)
ax.set_xlim(-0.55, 1.15)
ax.set_xlabel("buyers minus everyone else, in standard deviations")
panel_title(ax, "A", "What sets buyers apart: partner count, not status")

dec = inc[inc.decile > 0]
allm = float(inc.loc[inc.decile == 0, "share_paid"].iloc[0])
ax2.bar(dec.decile, dec.share_paid, width=0.68, color=NAVY)
errbars(ax2, dec.decile, dec.share_paid, dec.lo, dec.hi)
ax2.axhline(allm, color=INK, lw=1, ls=(0, (4, 3)))
ax2.text(10.45, allm + 0.004, f"all men\n{allm:.0%}", fontsize=9, color=INK, fontweight="bold", va="bottom")
for xi, v in zip(dec.decile, dec.share_paid):
    ax2.text(xi, 0.006, f"{v:.0%}", ha="center", fontsize=8.5, color="white", fontweight="bold")
ax2.set_xticks(range(1, 11)); ax2.set_xlim(0.4, 11.6)
ax2.yaxis.set_major_formatter(pct); ax2.set_ylim(0, 0.2); ax2.set_yticks([0, .05, .1, .15, .2])
ax2.set_xlabel("own income decile within survey year (1 = poorest)")
ax2.set_ylabel("have ever paid for sex")
panel_title(ax2, "B", "Who pays, by income: basically flat")

header(fig, "Men who pay for sex aren't richer; they just have far more partners",
       f"Men 25-60 who have ever paid for sex ({allm:.1%}) against those who have not, GSS 1991-2022", y=0.92)
footnote(fig, "Survey-weighted, bootstrap 95% intervals. A: differences in means divided by the pooled SD so measures "
              "are comparable; partners capped at 100 (buyers 27 vs 11).\n'Looks, SES-adjusted' removes age, income "
              "rank, prestige rank, degree and year first. Height was asked only in 2022, so its bar rests on 20 buyers.",
         y=0.0)
save(fig, "paid_sex_profile")

# =============================================================== 3. married sex falls, porn rises
ym, ae = read("natsal_years_married.csv"), read("aella_sex_porn_digitised.csv")
fig, (ax, ax2) = plt.subplots(1, 2, figsize=(14.5, 5.4), gridspec_kw=dict(wspace=0.22))
XT = ([0, 5, 10, 15, 20, 25, 30, 35], ["0", "5", "10", "15", "20", "25", "30", "30+"])


def ym_line(ax, outcome, sex, colr, lw=2.4):
    d = ym[(ym.outcome == outcome) & (ym.sex == sex)].sort_values("years_married")
    ax.fill_between(d.years_married, d.lo, d.hi, color=colr, alpha=0.13, lw=0)
    ax.plot(d.years_married, d.adj, color=colr, lw=lw, marker="o", ms=4.5, zorder=3)
    return d.set_index("years_married").adj


m = ym_line(ax, "sex_per_month", "men", NAVY)
w = ym_line(ax, "sex_per_month", "women", GRAY, lw=2.0)
ax.annotate(f"Men, year 0: {m.loc[0]:.1f}/month", xy=(0, m.loc[0]), xytext=(3.5, 8.3), fontsize=9.5,
            fontweight="bold", color=INK, arrowprops=dict(arrowstyle="-", color=MUTED, lw=0.8))
ax.annotate(f"Years 5-6: {m.loc[5.5]:.1f}/month,\n{m.loc[5.5] / m.loc[0]:.0%} of year 0", xy=(5.5, m.loc[5.5]),
            xytext=(10, 6.6), fontsize=9.5, fontweight="bold", color=INK,
            arrowprops=dict(arrowstyle="-", color=MUTED, lw=0.8))
ax.text(36.5, m.iloc[-1] - 0.35, "men", color=NAVY, fontsize=10, fontweight="bold", ha="right", va="top")
ax.text(36.5, w.iloc[-1] + 0.3, "women", color=INK2, fontsize=10, fontweight="bold", ha="right", va="bottom")
ax.set_xticks(*XT); ax.set_xlim(-0.8, 37); ax.set_ylim(0, 10)
ax.set_xlabel("years married"); ax.set_ylabel("times a month (age-adjusted)")
panel_title(ax, "A", "Married sex falls 40% by year five, then levels off")

x = np.arange(len(ae))
ax2.plot(x, ae.porn_per_month_men, color=RED, lw=2.6, marker="o", ms=4.5)
ax2.plot(x, ae.sex_per_month, color=NAVY, lw=2.6, marker="o", ms=4.5)
ax2.plot(x, ae.porn_per_month_women, color=GRAY, lw=2.2, marker="o", ms=4)
ax2.text(10.25, ae.porn_per_month_men.iloc[-1], "his porn", color=INK, fontsize=9.5, fontweight="bold", va="center")
ax2.text(10.25, ae.sex_per_month.iloc[-1] + 0.5, "sex", color=INK, fontsize=9.5, fontweight="bold", va="center")
ax2.text(10.25, ae.porn_per_month_women.iloc[-1] - 0.5, "her porn", color=INK, fontsize=9.5, fontweight="bold",
         va="center")
ax2.set_xticks(x, ae.years_together, fontsize=8.5, rotation=45, ha="right", rotation_mode="anchor")
ax2.set_xlim(-0.4, 11.8); ax2.set_ylim(0, 16)
ax2.set_xlabel("years together"); ax2.set_ylabel("times a month")
panel_title(ax2, "B", "As sex fades, his porn use climbs")

header(fig, "As married sex falls, men's porn use rises",
       "Sex per month by years married, Britain (A); sex and porn per month by relationship length, Aella's "
       "relationships survey (B)", y=0.92)
footnote(fig, f"A: Natsal-3 (Britain 2010-12), married respondents with a known year of marriage (n = "
              f"{int(ym[(ym.outcome == 'sex_per_month')].n.sum()):,}), weighted; occasions in the last 4 weeks, each "
              "duration bin adjusted to the same age mix,\nso the line shows years married rather than ageing; shaded "
              "bands are 95% intervals. B: redrawn from Aella ('Monogamous men in long-term relationships aren't doin "
              "too hot',\naella.substack.com, 2023); values read off her chart. A different sample from A, so compare "
              "shapes, not levels.", y=0.0)
save(fig, "married_sex_porn")

# =============================================================== 4. wanting more
mt = read("natsal_desire_matching.csv")
fig, (ax, ax2) = plt.subplots(1, 2, figsize=(15, 5.8), gridspec_kw=dict(width_ratios=[1.2, 1], wspace=0.2))

ax.plot(mt.per_month, mt.women_want_at_least, color=NAVY, lw=2.8)
ax.plot(mt.per_month, mt.men_want_at_most, color=GRAY, lw=2.8)
he = mt.iloc[(mt.per_month - 104 / 12).abs().argmin()]
she = mt.iloc[(mt.per_month - 52 / 12).abs().argmin()]
ax.plot([he.per_month], [he.women_want_at_least], "o", ms=10, color=RED, mec=P["SURFACE"], mew=1.6, zorder=5)
ax.plot([she.per_month], [she.men_want_at_most], "o", ms=9, color=INK, mec=P["SURFACE"], mew=1.6, zorder=5)
ax.annotate(f"He wants it twice a week:\nonly {he.women_want_at_least:.0%} of women want it that often",
            xy=(he.per_month, he.women_want_at_least), xytext=(12.5, 0.36), fontsize=9.5, fontweight="bold",
            color=INK, arrowprops=dict(arrowstyle="-", color=MUTED, lw=0.8))
ax.annotate(f"She wants it once a week:\n{she.men_want_at_most:.0%} of men would be happy with that little",
            xy=(she.per_month, she.men_want_at_most), xytext=(9.5, 0.86), fontsize=9.5, fontweight="bold",
            color=INK, arrowprops=dict(arrowstyle="-", color=MUTED, lw=0.8))
ax.text(29.6, mt.women_want_at_least.iloc[-1] + 0.03, "women who want sex\nat least this often",
        color=NAVY, fontsize=9.5, fontweight="bold", ha="right", va="bottom")
ax.text(29.6, mt.men_want_at_most.iloc[-1] - 0.03, "men who want sex\nno more than this", color=INK2, fontsize=9.5,
        fontweight="bold", ha="right", va="top")
ax.set_xticks([52 / 12, 104 / 12, 156 / 12, 208 / 12, 30.4], ["once\na week", "twice\na week", "3x\na week",
                                                              "4x\na week", "daily"])
ax.grid(axis="x", visible=True)
ax.set_xlim(1, 31); ax.set_ylim(0, 1); ax.yaxis.set_major_formatter(pct)
ax.set_xlabel("how often you'd ideally like sex")
ax.set_ylabel("share of the other sex who'd be a match")
panel_title(ax, "A", "The more he wants, the fewer women match; the less she wants, the more men don't")

m = ym_line(ax2, "want_more", "men", NAVY)
w = ym_line(ax2, "want_more", "women", GRAY, lw=2.0)
single = float(ym.loc[ym.sex == "single men", "adj"].iloc[0])
ax2.axhline(single, color=MUTED, lw=1.0, ls=(0, (1, 2)))
ax2.text(36.5, single + 0.02, f"single men: {single:.0%}", color=MUTED, fontsize=9.5, ha="right", va="bottom")
ax2.plot([5.5], [m.loc[5.5]], "o", ms=9, color=RED, mec=P["SURFACE"], mew=1.5, zorder=5)
ax2.annotate(f"Men, years 5-6: {m.loc[5.5]:.0%}", xy=(5.5, m.loc[5.5]), xytext=(7.5, 0.88), fontsize=9.5,
             fontweight="bold", color=INK, arrowprops=dict(arrowstyle="-", color=MUTED, lw=0.8))
ax2.text(36.5, m.iloc[-1] - 0.03, "men", color=NAVY, fontsize=10, fontweight="bold", ha="right", va="top")
ax2.text(36.5, w.iloc[-1] - 0.03, "women", color=INK2, fontsize=10, fontweight="bold", ha="right", va="top")
ax2.set_xticks(*XT); ax2.set_xlim(-0.8, 37); ax2.set_ylim(0, 1); ax2.yaxis.set_major_formatter(pct)
ax2.set_xlabel("years married"); ax2.set_ylabel("would like sex more often")
panel_title(ax2, "B", "Wants sex more often, by years married")

header(fig, "Men want more sex than their partners, and marriage widens the gap",
       "How many of the other sex want sex as often as you do (A); share wanting more sex by years married (B). "
       "Britain, Natsal-3", y=0.92)
footnote(fig, "A: partnered men and women aged 25-40 (n = 1,097 / 1,797). Ideal frequency is estimated per sex from "
              "each person's actual count in the last 4 weeks plus whether they wanted sex more, less or about as "
              "often\n(a lognormal fit; 'a bit more' capped at twice the actual count). Blue: if a man wants sex x times "
              "a month, the share of women whose ideal is x or more. Gray: if a woman wants it x times, the share of "
              "men\nwhose ideal is x or less. B: married respondents with a known year of marriage, weighted, each bin "
              "adjusted to the same age mix; shaded bands are 95% intervals.", y=0.0)
save(fig, "wants_more")

# =============================================================== 5. sex-life satisfaction
sat = read("aella_satisfaction_digitised.csv")
fig, (ax, ax2) = plt.subplots(1, 2, figsize=(14.5, 5.2), gridspec_kw=dict(width_ratios=[1.45, 1], wspace=0.22))

DIV = ["#d7261e", "#e8796f", "#f2b8b0", "#e7e5e0", "#8fa9cf", "#4a6ea3", "#1f3a68"]
x = np.arange(7)
ax.bar(x, sat.pct, width=0.72, color=DIV)
for xi, v in zip(x, sat.pct):
    ax.text(xi, v + 0.5, f"{v:.0f}%", ha="center", fontsize=10.5, fontweight="bold", color=INK)
ax.set_xticks(x, ["-3\nstrongly\ndisagree", "-2", "-1", "0\nneutral", "+1", "+2", "+3\nstrongly\nagree"],
              fontsize=9.5)
ax.yaxis.set_major_formatter(ppt); ax.set_ylim(0, 25)
ax.set_ylabel("share of men")
panel_title(ax, "A", "“I'm satisfied with my sex life”")

groups = [("Neutral or\ndissatisfied\n(0 and under)", sat.pct[sat.agreement <= 0].sum(), RED),
          ("Satisfied\n(+1 or +2;\n5-6 of 7)", sat.pct[sat.agreement.isin([1, 2])].sum(), MUTE),
          ("Fully satisfied\n(+3; 7 of 7)", sat.pct[sat.agreement == 3].sum(), MUTE)]
x = np.arange(3)
ax2.bar(x, [g[1] for g in groups], width=0.62, color=[g[2] for g in groups])
for xi, (_, v, _) in zip(x, groups):
    ax2.text(xi, v + 1, f"{v:.0f}%", ha="center", fontsize=13, fontweight="bold", color=INK)
ax2.set_xticks(x, [g[0] for g in groups], fontsize=9.5)
ax2.yaxis.set_major_formatter(ppt); ax2.set_ylim(0, 70)
panel_title(ax2, "B", "Grouped")

header(fig, "Six in ten long-partnered men are neutral or unhappy about their sex life",
       "Straight, monogamous men in relationships of 12+ years, agreement on a -3 to +3 scale", y=0.92)
footnote(fig, "Redrawn from Aella's relationships survey (n = 3,720 men; 'Monogamous men in long-term relationships "
              "aren't doin too hot', aella.substack.com, 2023). Values read off her chart, to within about 0.2 points.\n"
              "Her sample is mostly her own followers, who skew liberal and sexually open, so treat levels as indicative.",
         y=-0.02)
save(fig, "sex_satisfaction")

# =============================================================== 6. men's lifetime partner distribution
dist = read("bodycount_distribution.csv").set_index("stat").value
cut = read("bodycount_crosscuts.csv")
fig, axes = plt.subplots(1, 3, figsize=(17.5, 5.8), gridspec_kw=dict(width_ratios=[1.15, 0.85, 1.25], wspace=0.32))

ax = axes[0]
qs = [10, 20, 30, 40, 50, 60, 70, 80, 90, 95]
vals = [dist[f"p{q}"] for q in qs]
x = np.arange(len(qs))
ax.bar(x, vals, width=0.68, color=NAVY)
for xi, v in zip(x, vals):
    ax.text(xi, v + 1.2, f"{v:.0f}", ha="center", fontsize=10, fontweight="bold", color=INK)
mean = dist["mean_capped100"]
ax.axhline(mean, color=RED, lw=1.6, ls=(0, (4, 3)))
ax.text(-0.4, mean + 1.5, f"mean {mean:.0f}: more than {dist['share_below_mean']:.0%} of men have had fewer",
        color=INK, fontsize=9.5, fontweight="bold", ha="left", va="bottom")
ax.set_xticks(x, [f"{q}th" if q != 50 else "median" for q in qs], fontsize=9)
ax.set_ylim(0, 80); ax.set_ylabel("lifetime female partners")
ax.set_xlabel("percentile of men")
panel_title(ax, "A", "The median man: 6. The 90th percentile: 40")

ax = axes[1]
x = np.arange(5)
shares = [dist[f"fifth{q}_share"] for q in range(1, 6)]
means = [dist[f"fifth{q}_mean"] for q in range(1, 6)]
ax.bar(x, shares, width=0.66, color=[MUTE] * 4 + [RED])
for xi, s_, m_ in zip(x, shares, means):
    ax.text(xi, s_ + 0.015, f"{s_:.0%}", ha="center", fontsize=11, fontweight="bold", color=INK)
    ax.text(xi, s_ + 0.06, f"avg {m_:.0f}", ha="center", fontsize=8.5, color=INK2)
ax.set_xticks(x, ["bottom\nfifth", "2nd", "middle", "4th", "top\nfifth"], fontsize=9)
ax.yaxis.set_major_formatter(pct); ax.set_ylim(0, 0.85)
ax.set_ylabel("share of all partners men report")
panel_title(ax, "B", "The top fifth: two-thirds of the total")

ax = axes[2]
c = cut.iloc[::-1].reset_index(drop=True)
y = np.arange(len(c)) * 1.0
gaps = {"history": 0.5, "status": 1.0, "all": 1.5}      # extra space above each family
for fam, extra in gaps.items():
    y[(c.family == fam).to_numpy()] += extra
y += np.where(c.family == "status", 0, 0)
for yi, row in zip(y, c.itertuples()):
    hot = row.group in ("Has paid for sex", "Saw X-rated film, past yr")
    ax.plot([row.median, row.p90], [yi, yi], color=GRID_C, lw=2.2, zorder=1)
    ax.plot(row.median, yi, "o", ms=8, color=GRAY, zorder=3)
    ax.plot(row.p90, yi, "o", ms=9, color=RED if hot else NAVY, zorder=3)
    ax.text(row.p90 + 3, yi, f"{row.p90:.0f}", va="center", fontsize=9, fontweight="bold", color=INK)
    ax.text(row.median - 2.5, yi, f"{row.median:.0f}", va="center", ha="right", fontsize=8.5, color=INK2)
ax.set_yticks(y, [f"{g}  (n={n:,})" for g, n in zip(c.group, c.n)], fontsize=9)
ax.grid(axis="y", visible=False); ax.grid(axis="x", visible=True)
ax.set_xlim(-6, 112); ax.set_xlabel("lifetime female partners")
ax.text(5, y.max() + 1.0, "● median", color=GRAY, fontsize=9.5, fontweight="bold")
ax.text(32, y.max() + 1.0, "● 90th percentile", color=NAVY, fontsize=9.5, fontweight="bold")
ax.set_ylim(-0.7, y.max() + 1.6)
panel_title(ax, "C", "Groups differ at the top, not in the middle")

header(fig, "The typical man has had six partners; the top fifth average fifty",
       "Lifetime female partners, US men aged 40-60: percentiles (A), each fifth's share of the total (B), and "
       "the middle vs the top across groups (C)", y=0.92)
footnote(fig, "GSS 1989-2022, men 40-60 (n = 5,435; older, so counts are close to lifetime), survey-weighted. Partner "
              "counts are self-reported; the mean and B cap answers at 100 so a few very large ones don't dominate.\n"
              "B ranks men by count and splits them into fifths. C: X-rated film = saw one in the past year; income "
              "thirds are ranked within survey year. Groups overlap and are not adjusted for each other.", y=0.0)
save(fig, "bodycount_distribution")
```
