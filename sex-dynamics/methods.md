# The master post of sexual and relationship dynamics: looks, money, height, paying for sex, marriage, porn and wanting more

<p class="meta">Methodology for a chart from <a href="https://performativebafflement.substack.com/">Performative Bafflement</a>. Updated 2026-11-14.</p>

![chart 1](chart.png)

![chart 2](chart2.png)

![chart 3](chart3.png)

![chart 4](chart4.png)

![chart 5](chart5.png)

<p class="ai">For AI fact-checking: the plain-text version of this page is at <a href="methods.md">https://pbmethods.github.io/sex-dynamics/methods.md</a>. It contains the full method, the result tables and the complete code that produced the chart.</p>

## Method

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

| looks | code | n | mean_partners | lo | hi | n_rating_1 |
|---|---|---|---|---|---|---|
| 1-2 below avg | 2 | 79 | 12.472 | 8.522 | 17.509 | 19 |
| 3 average | 3 | 520 | 11.431 | 9.822 | 13.086 | 0 |
| 4 above avg | 4 | 294 | 14.499 | 10.689 | 19.172 | 0 |
| 5 very good | 5 | 66 | 16.112 | 10.348 | 23.359 | 0 |

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

### `natsal_want_more.csv`

| sex | fifth | per_month_lo | per_month_hi | per_month_median | n | want_more |
|---|---|---|---|---|---|---|
| men | 1 | 0.000 | 1.000 | 1.000 | 224 | 0.770 |
| men | 2 | 1.000 | 3.000 | 2.000 | 210 | 0.767 |
| men | 3 | 3.000 | 5.000 | 4.000 | 211 | 0.713 |
| men | 4 | 5.000 | 8.000 | 6.000 | 210 | 0.666 |
| men | 5 | 8.000 | 100.000 | 12.000 | 242 | 0.428 |
| men | 0 |  |  |  | 1097 | 0.669 |
| women | 1 | 0.000 | 1.000 | 0.000 | 348 | 0.676 |
| women | 2 | 1.000 | 3.000 | 2.000 | 359 | 0.596 |
| women | 3 | 3.000 | 4.000 | 3.000 | 348 | 0.552 |
| women | 4 | 4.000 | 8.000 | 6.000 | 364 | 0.481 |
| women | 5 | 8.000 | 45.000 | 10.000 | 378 | 0.321 |
| women | 0 |  |  |  | 1797 | 0.525 |

### `natsal_desire_matching.csv`

| wants | per_month | men_at_least | women_at_least |
|---|---|---|---|
| once a week | 4.333 | 0.574 | 0.457 |
| twice a week | 8.667 | 0.346 | 0.240 |
| 3x a week | 13.000 | 0.231 | 0.146 |
| 4x a week | 17.333 | 0.164 | 0.096 |
| daily | 30.400 | 0.073 | 0.037 |

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
lk["looks"] = lk.rlooks.clip(lower=2).astype(int)          # pool 1 with 2
LOOKS_LAB = {2: "1-2\nbelow avg", 3: "3\naverage", 4: "4\nabove avg", 5: "5\nvery good"}
rows = []
for k, lab in LOOKS_LAB.items():
    s = lk[lk.looks == k]
    lo, hi = boot_ci(s.bc, s.w)
    rows.append({"looks": lab.replace("\n", " "), "code": k, "n": len(s), "mean_partners": wmean(s.bc, s.w),
                 "lo": lo, "hi": hi, "n_rating_1": int((s.rlooks == 1).sum())})
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

# ---------------------------------------------------------------- Natsal: duration curves
nat = pd.read_stata(NAT, columns=["rsex", "dage", "sex4wks", "frequent", "total_wt", "partnerhh", "rafsmr"],
                    convert_categoricals=False)
nat = nat[(nat.sex4wks < 900) & nat.frequent.between(1, 5) & (nat.total_wt > 0)].copy()
nat["male"] = nat.rsex.eq(1); nat["w"] = nat.total_wt; nat["A"] = nat.sex4wks.astype(float)

nd = nat[nat.partnerhh.eq(1) & nat.rafsmr.between(10, 80)].copy()
nd["dur"] = nd.dage - nd.rafsmr
nd = nd[nd.dur.between(0, 45)]


def design(age, dur):
    age = np.asarray(age, float); dur = np.asarray(dur, float)
    return np.column_stack([np.ones(len(age)), age - 35, (age - 35) ** 2 / 100, np.log1p(dur), np.minimum(dur, 5),
                            np.clip(dur - 5, 0, None) / 10])


GRID = np.linspace(0, 17, 69)        # a 35-year-old cannot have been with someone (since 18) for longer
rows = {"years": GRID}
for male, sx in [(True, "men"), (False, "women")]:
    s = nd[nd.male == male]
    for y, lab, fam in [(s.sex4wks, "per_month", sm.families.Poisson()),
                        (s.frequent.le(2).astype(float), "want_more", sm.families.Binomial())]:
        f = sm.GLM(np.asarray(y, float), design(s.dage, s.dur), family=fam, freq_weights=s.w.to_numpy()).fit()
        eta = design(np.full(len(GRID), 35.0), GRID) @ f.params
        rows[f"{sx}_{lab}"] = np.exp(eta) if lab == "per_month" else 1 / (1 + np.exp(-eta))
DUR = pd.DataFrame(rows)
DUR.to_csv(HERE / "natsal_duration.csv", index=False)
print(f"Natsal couples with a datable start: n = {len(nd):,}")
print(DUR.iloc[[0, 4, 20, 40, 68]].round(2).to_string(index=False))

# ---------------------------------------------------------------- Natsal: wanting more (partnered 25-40)
nb = nat[nat.dage.between(25, 40) & nat.partnerhh.eq(1)]
rows = []
for male, sx in [(True, "men"), (False, "women")]:
    s = nb[nb.male == male].sort_values("A").copy()
    s["cum"] = (np.cumsum(s.w) - 0.5 * s.w) / s.w.sum()
    s["fifth"] = np.minimum((s.cum * 5).astype(int), 4)
    for q, d in s.groupby("fifth"):
        rows.append({"sex": sx, "fifth": q + 1, "per_month_lo": d.A.min(), "per_month_hi": d.A.max(),
                     "per_month_median": float(np.median(d.A)), "n": len(d),
                     "want_more": wmean(d.frequent.le(2), d.w)})
    rows.append({"sex": sx, "fifth": 0, "n": len(s), "want_more": wmean(s.frequent.le(2), s.w)})    # 0 = all
WANT = pd.DataFrame(rows)
WANT.to_csv(HERE / "natsal_want_more.csv", index=False)
print(WANT.round(2).to_string(index=False))


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
rows = []
for lab, x in [("once a week", 52 / 12), ("twice a week", 104 / 12), ("3x a week", 156 / 12),
               ("4x a week", 208 / 12), ("daily", 30.4)]:
    rows.append({"wants": lab, "per_month": x,
                 **{f"{sx}_at_least": float(norm.sf((np.log(x) - mu) / sg)) for sx, (mu, sg) in FIT.items()}})
MATCH = pd.DataFrame(rows)
MATCH.to_csv(HERE / "natsal_desire_matching.csv", index=False)
print({sx: f"median wanted {np.exp(mu):.1f}/mo" for sx, (mu, sg) in FIT.items()})
print(MATCH.round(3).to_string(index=False))

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
fig, axes = plt.subplots(1, 3, figsize=(16, 5.4), gridspec_kw=dict(width_ratios=[0.8, 1.25, 1.35], wspace=0.24))

ax = axes[0]
x = np.arange(len(looks))
ax.bar(x, looks.mean_partners, width=0.62, color=NAVY)
errbars(ax, x, looks.mean_partners, looks.lo, looks.hi)
for xi, v, hi, n in zip(x, looks.mean_partners, looks.hi, looks.n):
    ax.text(xi, hi + 0.6, f"{v:.0f}", ha="center", fontsize=10.5, fontweight="bold", color=INK)
    ax.text(xi, 0.6, f"n={n}", ha="center", fontsize=7.5, color="white")
ax.set_xticks(x, ["1-2\nbelow avg", "3\naverage", "4\nabove avg", "5\nvery good"], fontsize=9)
ax.set_ylim(0, 27); ax.set_ylabel("lifetime female partners (mean)")
ax.set_xlabel("interviewer's looks rating")
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
ax.text(11.55, avg, f"all\nheights\n{avg:.0f}%", ha="left", va="center", fontsize=9, color=INK, fontweight="bold")
ax.set_xlim(-0.6, 12.9)
ax.set_xticks(x, h.height, fontsize=8.5)
ax.yaxis.set_major_formatter(ppt); ax.set_ylim(0, 36)
ax.set_ylabel("no sex in the past year"); ax.set_xlabel("height, men 18-29")
panel_title(ax, "C", "Height")

header(fig, "Looks, money and height barely move men's sex lives",
       "Lifetime partners by looks (A) and by income and prestige (B), men 25-60; share of young men with no sex "
       "in the past year, by height (C)", y=0.92)
footnote(fig, "A, B: GSS, men 25-60, survey-weighted means, partners capped at 100, bootstrap 95% intervals. A: 2016-22, the only "
              "years with the interviewer's 1-5 looks rating (1 and 2 pooled; only 19 men got a 1). B: 1989-2022.\n"
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

# =============================================================== 3. sex and porn after marriage
dur, ae = read("natsal_duration.csv"), read("aella_sex_porn_digitised.csv")
fig, axes = plt.subplots(1, 3, figsize=(17, 5.4), gridspec_kw=dict(wspace=0.28))

ax = axes[0]
ax.plot(dur.years, dur.men_per_month, color=NAVY, lw=2.6)
ax.plot(dur.years, dur.women_per_month, color=GRAY, lw=2.6)
ax.axvspan(0, 5, color=pb.NAVY_RAMP[0], alpha=0.12, lw=0)
ax.text(2.5, 0.5, "first 5 years", ha="center", fontsize=8.5, color=INK2)
for col, colr, lab, dy in [("men_per_month", NAVY, "men", 0.25), ("women_per_month", GRAY, "women", -0.45)]:
    ax.text(17.3, dur[col].iloc[-1] + dy, f"{lab} {dur[col].iloc[-1]:.1f}", fontsize=9.5, color=INK, fontweight="bold",
            va="center")
ax.text(0.3, dur.men_per_month.iloc[0] + 0.3, f"{dur.men_per_month.iloc[0]:.1f}", fontsize=9.5, fontweight="bold")
ax.set_xlim(0, 21); ax.set_ylim(0, 8)
ax.set_xlabel("years with this partner"); ax.set_ylabel("times per month, at age 35")
panel_title(ax, "A", "Sex falls by a quarter in five years")

ax = axes[1]
ax.plot(dur.years, dur.men_want_more, color=NAVY, lw=2.6)
ax.plot(dur.years, dur.women_want_more, color=GRAY, lw=2.6)
ax.axvspan(0, 5, color=pb.NAVY_RAMP[0], alpha=0.12, lw=0)
for col, lab in [("men_want_more", "men"), ("women_want_more", "women")]:
    ax.text(17.3, dur[col].iloc[-1], f"{lab} {dur[col].iloc[-1]:.0%}", fontsize=9.5, color=INK, fontweight="bold",
            va="center")
    ax.text(-0.2, dur[col].iloc[0], f"{dur[col].iloc[0]:.0%}", fontsize=9, color=INK2, ha="right", va="center")
ax.yaxis.set_major_formatter(pct); ax.set_ylim(0.3, 0.8); ax.set_xlim(-1.6, 21)
ax.set_xlabel("years with this partner"); ax.set_ylabel("want sex more often, at age 35")
panel_title(ax, "B", "...and men's wish for more ratchets up")

ax = axes[2]
x = np.arange(len(ae))
ax.plot(x, ae.porn_per_month_men, color=RED, lw=2.6, marker="o", ms=4.5)
ax.plot(x, ae.sex_per_month, color=NAVY, lw=2.6, marker="o", ms=4.5)
ax.plot(x, ae.porn_per_month_women, color=GRAY, lw=2.2, marker="o", ms=4)
ax.text(10.25, ae.porn_per_month_men.iloc[-1], "his porn", color=INK, fontsize=9.5, fontweight="bold", va="center")
ax.text(10.25, ae.sex_per_month.iloc[-1] + 0.5, "sex", color=INK, fontsize=9.5, fontweight="bold", va="center")
ax.text(10.25, ae.porn_per_month_women.iloc[-1] - 0.5, "her porn", color=INK, fontsize=9.5, fontweight="bold",
        va="center")
ax.set_xticks(x, ae.years_together, fontsize=8.5, rotation=45, ha="right", rotation_mode="anchor")
ax.set_xlim(-0.4, 11.6); ax.set_ylim(0, 16)
ax.set_xlabel("years together"); ax.set_ylabel("times per month")
panel_title(ax, "C", "As sex fades, his porn rises")

header(fig, "After the honeymoon: less sex, more wanting, more porn",
       "Monthly sex and the wish for more by years with a partner, at a fixed age of 35 (A, B); sex and porn by "
       "relationship length (C)", y=0.92)
footnote(fig, "A, B: Natsal-3 (Britain 2010-12), people living with a partner (n = 5,899), weighted. Poisson (A) and "
              "logistic (B) fits with age and duration as separate terms, drawn at age 35,\nso they show years together "
              "rather than ageing; they stop at 17 years because a 35-year-old cannot have been with someone longer.\n"
              "C: redrawn from Aella's relationships survey ('Monogamous men in long-term relationships aren't doin too "
              "hot', aella.substack.com, 2023); values read off her chart.", y=0.0)
save(fig, "married_sex_porn")

# =============================================================== 4. wanting more
want, match = read("natsal_want_more.csv"), read("natsal_desire_matching.csv")
fig, (ax, ax2) = plt.subplots(1, 2, figsize=(14.5, 5.4), gridspec_kw=dict(width_ratios=[1.15, 1], wspace=0.3))

x = np.arange(5)
for i, (sx, colr) in enumerate([("men", NAVY), ("women", GRAY)]):
    d = want[(want.sex == sx) & (want.fifth > 0)].sort_values("fifth")
    xs = x + (i - 0.5) * 0.38
    ax.bar(xs, d.want_more, width=0.36, color=colr)
    for xi, v, med in zip(xs, d.want_more, d.per_month_median):
        ax.text(xi, v + 0.015, f"{v:.0%}", ha="center", fontsize=9.5, fontweight="bold", color=INK)
        ax.text(xi, 0.025, f"{med:.0f}/mo", ha="center", fontsize=7.2, color="white", fontweight="bold")
allm = float(want.loc[(want.sex == "men") & (want.fifth == 0), "want_more"].iloc[0])
allw = float(want.loc[(want.sex == "women") & (want.fifth == 0), "want_more"].iloc[0])
ax.text(4.55, 0.93, f"■ men  (all: {allm:.0%})", color=NAVY, fontsize=9.5, fontweight="bold", ha="right")
ax.text(4.55, 0.86, f"■ women  (all: {allw:.0%})", color=GRAY, fontsize=9.5, fontweight="bold", ha="right")
ax.set_xticks(x, ["least sex\n(bottom fifth)", "2nd", "middle", "4th", "most sex\n(top fifth)"], fontsize=9)
ax.yaxis.set_major_formatter(pct); ax.set_ylim(0, 1)
ax.set_ylabel("would like sex more often")
ax.set_xlabel("how much sex they are already having (fifths, within each sex; median inside the bar)")
panel_title(ax, "A", "Most want more, even when they're getting plenty")

y = np.arange(len(match))[::-1]
ax2.barh(y + 0.19, match.women_at_least, height=0.36, color=NAVY)
ax2.barh(y - 0.19, match.men_at_least, height=0.36, color=GRAY)
for yi, vw, vm in zip(y, match.women_at_least, match.men_at_least):
    ax2.text(vw + 0.01, yi + 0.19, f"{vw:.0%}", va="center", fontsize=9.5, fontweight="bold", color=INK)
    ax2.text(vm + 0.01, yi - 0.19, f"{vm:.0%}", va="center", fontsize=9, color=INK2)
ax2.set_yticks(y, [f"{w}" for w in match.wants], fontsize=10)
ax2.grid(axis="y", visible=False); ax2.grid(axis="x", visible=True)
ax2.xaxis.set_major_formatter(pct); ax2.set_xlim(0, 0.7)
ax2.text(0.69, y[-1] + 0.75, "■ women", color=NAVY, fontsize=9.5, fontweight="bold", ha="right")
ax2.text(0.69, y[-1] + 0.35, "■ men", color=GRAY, fontsize=9.5, fontweight="bold", ha="right")
ax2.set_xlabel("share who want sex at least this often")
panel_title(ax2, "B", "If you want it this often, who else does?")

header(fig, "Two-thirds of partnered men want more sex than they get",
       "Partnered men and women aged 25-40: who wants more (A), and how many people want at least a given "
       "frequency (B)", y=0.92)
footnote(fig, "Natsal-3 (Britain 2010-12), living with a partner, 25-40 (men n = 1,097, women n = 1,797), weighted. "
              "A: 'would like sex much or a bit more often', by fifths of occasions in the last 4 weeks.\n"
              "B: desired frequency fitted per sex from each person's actual count plus whether they wanted more, the "
              "same or less ('a bit more' capped at 2x actual), as a lognormal.", y=0.0)
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
```
