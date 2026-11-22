# When either spouse could leave: unilateral divorce laws and women's suicide, violence and spousal homicide

<p class="meta">Methodology for a chart from <a href="https://performativebafflement.substack.com/">Performative Bafflement</a>. Updated 2026-11-22.</p>

![chart 1](chart.png)

<p class="ai">For AI fact-checking: the plain-text version of this page is at <a href="methods.md">https://pbmethods.github.io/divorce-exit/methods.md</a>. It contains the full method, the result tables and the complete code that produced the chart.</p>

## Method

### `unilateral_divorce_chart.py`

```text
When US states let either spouse end a marriage alone, did fewer women die? Stevenson & Wolfers, and a reanalysis.

What this chart shows. No new estimation: every number is transcribed from two papers' tables into
`unilateral_divorce_estimates.csv` (one row per coefficient, with source table and column), and plotted.

The natural experiment. Between 1969 and 1985, 37 US states moved from fault-based or mutual-consent divorce to
unilateral divorce, where either spouse can end the marriage without the other's agreement. Each state switched in a
different year (some before 1964, some never), so states that have not yet switched serve as the comparison for
states that have. Stevenson & Wolfers ask whether the law change, which gave the unhappy spouse an exit and shifted
bargaining power inside every marriage, changed the rate of suicide, domestic violence and spousal homicide.

A. Female and male suicide, by years since the reform. Stevenson & Wolfers (2004, NBER w10175; published as
"Bargaining in the Shadow of the Law", QJE 2006), Table 2. State-by-year panel 1964-96 (n = 1,683 state-years),
NCHS suicide counts. Dependent variable: aggregate state suicide rate for all women (or all men), married or not;
mean 54 per million women, 202 per million men. Regressors: dummies for 0, 1-2, 3-4, ... 17-18, 19+ years since the
state adopted unilateral divorce, state and year fixed effects, plus state controls (AFDC maximum, log income per
capita, unemployment, female/male employment ratio, age and race shares). Coefficients are percent changes at the
cell mean. Solid line: column 2f (the authors' preferred specification, OLS with controls; average over 20 years
-8.3%, SE 2.3). Dashed line: column 4f, which adds state-specific linear time trends (average -5.4%, SE 5.6; the
joint test of the dummies has p = 0.82). Gray: men, column 2m (average -2.0%, SE 1.3). Bands are +/-1.96 SE (the
paper plots +/-1 SE). The paper's figure also shows pre-reform leads, flat, which its tables do not list; they are not
plotted here.

B. The same question under other reasonable models. Goodman-Bacon (2018, NBER w25018; published in J. Econometrics
2021), Table 2 and Section III. He replicates the female-suicide result in levels (suicides per million women,
age-adjusted; his mean 52 per million, 1964-96, 48 states excluding Alaska and Hawaii, unweighted, no covariates):
two-way fixed effects estimate -3.08 (about -6%). He then shows that a third of that estimate comes from comparing
states that reformed later with states that reformed earlier, comparisons that are biased when the effect grows over
time (as in panel A); removing them gives -5.44. The other rows are his alternative specifications: population
weights, inverse propensity-score weights on 1960 sex ratio, income, fertility and infant mortality, state-specific
linear trends, group pre-trends from 1964-68, region-by-year fixed effects, and dropping never-reform and pre-1964
states. Bars are +/-1.96 times his reported standard errors; the -5.44 has no reported SE. Percent labels divide by
his mean of 52.

C. Other outcomes, Stevenson & Wolfers. Domestic violence: 1976 and 1985 National Family Violence Surveys, any
violent act between spouses in the past year (Gelles-Straus conflict tactics), difference-in-differences between
states that adopted unilateral divorce between the surveys and states that did not (Table 3). The paper reports
-4.3 points (SE 1.9) husband-to-wife and -2.7 (SE 1.8) wife-to-husband, which it converts to -36% and -24% at the
cell mean; the SEs here are converted the same way (1.9/11.9 and 1.8/11.25, both about 16%). Intimate homicide: FBI
Supplementary Homicide Reports 1968-94 (Illinois and DC excluded), percent change in the rate of women and men killed
by a spouse, with controls (Table 5, column 2). In the paper's triple difference against non-intimate homicide the
women's estimate is -7.2% (SE 6.9), not significant.

Run:  uv run --with pandas,matplotlib,pillow,fonttools python notebooks/marital_happiness/unilateral_divorce_chart.py
```

## Results

### `unilateral_divorce_estimates.csv`

| panel | source | table | series | label | x | estimate | se | unit |
|---|---|---|---|---|---|---|---|---|
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | Year of change | 0 | 1.300 | 3.400 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 1-2 years | 1.500 | -1.400 | 3.500 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 3-4 years | 3.500 | -1.100 | 3.100 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 5-6 years | 5.500 | -2.000 | 2.900 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 7-8 years | 7.500 | -6.600 | 3.000 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 9-10 years | 9.500 | -8.500 | 3.000 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 11-12 years | 11.500 | -10.200 | 3.200 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 13-14 years | 13.500 | -11.100 | 3.100 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 15-16 years | 15.500 | -11.700 | 3.600 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 17-18 years | 17.500 | -13.900 | 3.600 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2f | women | 19+ years | 19.500 | -16.400 | 3.300 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | Year of change | 0 | 2.200 | 3.200 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 1-2 years | 1.500 | -0.400 | 3.700 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 3-4 years | 3.500 | 0.300 | 4.000 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 5-6 years | 5.500 | -0.300 | 4.200 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 7-8 years | 7.500 | -4.800 | 4.800 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 9-10 years | 9.500 | -6.200 | 5.300 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 11-12 years | 11.500 | -7.000 | 6.100 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 13-14 years | 13.500 | -7.300 | 6.900 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 15-16 years | 15.500 | -7.300 | 7.700 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 17-18 years | 17.500 | -9.500 | 8.700 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 4f | women_trends | 19+ years | 19.500 | -11.200 | 9.600 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | Year of change | 0 | -1.400 | 2.100 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 1-2 years | 1.500 | 0.500 | 1.400 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 3-4 years | 3.500 | -0.900 | 1.500 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 5-6 years | 5.500 | -0.200 | 1.500 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 7-8 years | 7.500 | -1.300 | 1.800 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 9-10 years | 9.500 | -3.900 | 1.700 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 11-12 years | 11.500 | -2.600 | 2.000 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 13-14 years | 13.500 | -3.600 | 2.000 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 15-16 years | 15.500 | -2.000 | 1.900 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 17-18 years | 17.500 | -1.900 | 2.000 | pct |
| A | Stevenson & Wolfers 2004 NBER w10175 | Table 2 col 2m | men | 19+ years | 19.500 | -4.300 | 2.000 | pct |
| B | Goodman-Bacon 2018 NBER w25018 | Table 2 col 1 | spec | Baseline two-way fixed effects |  | -3.080 | 1.270 | per_million |
| B | Goodman-Bacon 2018 NBER w25018 | Section III.A | spec | Drop the biased later-vs-earlier comparisons |  | -5.440 |  | per_million |
| B | Goodman-Bacon 2018 NBER w25018 | Table 2 col 6 | spec | Subtract each group's 1964-68 trend |  | -6.520 | 2.980 | per_million |
| B | Goodman-Bacon 2018 NBER w25018 | Table 2 col 7 | spec | Region-by-year fixed effects |  | -1.160 | 1.370 | per_million |
| B | Goodman-Bacon 2018 NBER w25018 | Table 2 col 3 | spec | Weight states by population |  | -0.350 | 1.970 | per_million |
| B | Goodman-Bacon 2018 NBER w25018 | Table 2 col 5 | spec | State-specific linear trends |  | 0.590 | 1.350 | per_million |
| B | Goodman-Bacon 2018 NBER w25018 | Table 2 col 4 | spec | Propensity-score weights (1960 traits) |  | 1.040 | 1.780 | per_million |
| B | Goodman-Bacon 2018 NBER w25018 | Table 2 col 2 | spec | Reform states only (timing variation) |  | 2.420 | 1.810 | per_million |
| C | Stevenson & Wolfers 2004 NBER w10175 | Table 3 panel A | outcome | Husband-to-wife violence |  | -36 | 16.000 | pct |
| C | Stevenson & Wolfers 2004 NBER w10175 | Table 3 panel B | outcome | Wife-to-husband violence |  | -24 | 16.000 | pct |
| C | Stevenson & Wolfers 2004 NBER w10175 | Table 5 col 2 | outcome | Women killed by a spouse |  | -12.600 | 6.000 | pct |
| C | Stevenson & Wolfers 2004 NBER w10175 | Table 5 col 2 | outcome | Men killed by a spouse |  | 3.900 | 9.000 | pct |

_Every 1th row of 45 shown; full file: [unilateral_divorce_estimates.csv](unilateral_divorce_estimates.csv)._

## Notes and caveats

# Caveats: unilateral divorce and female suicide

- **Transcribed, not re-estimated.** Every number on the chart comes from the published tables of Stevenson & Wolfers
  (NBER w10175; QJE 2006) and Goodman-Bacon (NBER w25018; J. Econometrics 2021). The working-paper versions were used;
  check the journal versions before quoting a figure as final.
- **The outcome is all women in the state, not married women.** The suicide rate covers every woman, married or not,
  so the estimated effect per affected (married, unhappily married) woman is larger than the percentages shown, but
  the data cannot say by how much.
- **The headline result is sensitive to the model.** Adding state-specific linear trends leaves the women's
  coefficients negative but no longer jointly significant (p = 0.82). In Goodman-Bacon's reanalysis, the two-way fixed
  effects estimate (-3.08 per million, about -6%) moves to between -6.52 and +2.42 under reasonable alternatives:
  population weighting, propensity-score weights and state trends all push it to about zero. His decomposition shows
  part of this comes from a known bias in staggered-timing difference-in-differences when the effect grows over time
  (removing the biased comparisons gives -5.44, about -10%), and part from California's influence and imperfect
  balance between early and late reform states (1960 income and sex ratio).
- **Modern staggered-adoption estimators** (Callaway-Sant'Anna, Sun-Abraham, de Chaisemartin-D'Haultfoeuille) are
  designed for exactly this problem. A re-estimation with one of them on the NCHS data would settle more than either
  paper's numbers; none is shown here.
- **Domestic violence** rests on two survey waves (1976, 1985), around 6,000 households, and self-reported violence;
  the wife-to-husband estimate is not significant. **Spousal homicide** of women falls 12.6% (marginally significant),
  but in the paper's triple difference against non-intimate homicide the drop is 7.2% and not significant.
- **Mechanism.** The law could lower suicide by letting people leave bad marriages, or by improving marriages that
  continue (more bargaining power for the spouse who could now leave). The paper argues the steady decline without a
  divorce spike favors the second, but the data cannot cleanly separate them.

## Code

Run in the order listed. Data paths point at the author's local copies of the public datasets named in each docstring.

### `unilateral_divorce_chart.py`

```python
"""When US states let either spouse end a marriage alone, did fewer women die? Stevenson & Wolfers, and a reanalysis.

What this chart shows. No new estimation: every number is transcribed from two papers' tables into
`unilateral_divorce_estimates.csv` (one row per coefficient, with source table and column), and plotted.

The natural experiment. Between 1969 and 1985, 37 US states moved from fault-based or mutual-consent divorce to
unilateral divorce, where either spouse can end the marriage without the other's agreement. Each state switched in a
different year (some before 1964, some never), so states that have not yet switched serve as the comparison for
states that have. Stevenson & Wolfers ask whether the law change, which gave the unhappy spouse an exit and shifted
bargaining power inside every marriage, changed the rate of suicide, domestic violence and spousal homicide.

A. Female and male suicide, by years since the reform. Stevenson & Wolfers (2004, NBER w10175; published as
"Bargaining in the Shadow of the Law", QJE 2006), Table 2. State-by-year panel 1964-96 (n = 1,683 state-years),
NCHS suicide counts. Dependent variable: aggregate state suicide rate for all women (or all men), married or not;
mean 54 per million women, 202 per million men. Regressors: dummies for 0, 1-2, 3-4, ... 17-18, 19+ years since the
state adopted unilateral divorce, state and year fixed effects, plus state controls (AFDC maximum, log income per
capita, unemployment, female/male employment ratio, age and race shares). Coefficients are percent changes at the
cell mean. Solid line: column 2f (the authors' preferred specification, OLS with controls; average over 20 years
-8.3%, SE 2.3). Dashed line: column 4f, which adds state-specific linear time trends (average -5.4%, SE 5.6; the
joint test of the dummies has p = 0.82). Gray: men, column 2m (average -2.0%, SE 1.3). Bands are +/-1.96 SE (the
paper plots +/-1 SE). The paper's figure also shows pre-reform leads, flat, which its tables do not list; they are not
plotted here.

B. The same question under other reasonable models. Goodman-Bacon (2018, NBER w25018; published in J. Econometrics
2021), Table 2 and Section III. He replicates the female-suicide result in levels (suicides per million women,
age-adjusted; his mean 52 per million, 1964-96, 48 states excluding Alaska and Hawaii, unweighted, no covariates):
two-way fixed effects estimate -3.08 (about -6%). He then shows that a third of that estimate comes from comparing
states that reformed later with states that reformed earlier, comparisons that are biased when the effect grows over
time (as in panel A); removing them gives -5.44. The other rows are his alternative specifications: population
weights, inverse propensity-score weights on 1960 sex ratio, income, fertility and infant mortality, state-specific
linear trends, group pre-trends from 1964-68, region-by-year fixed effects, and dropping never-reform and pre-1964
states. Bars are +/-1.96 times his reported standard errors; the -5.44 has no reported SE. Percent labels divide by
his mean of 52.

C. Other outcomes, Stevenson & Wolfers. Domestic violence: 1976 and 1985 National Family Violence Surveys, any
violent act between spouses in the past year (Gelles-Straus conflict tactics), difference-in-differences between
states that adopted unilateral divorce between the surveys and states that did not (Table 3). The paper reports
-4.3 points (SE 1.9) husband-to-wife and -2.7 (SE 1.8) wife-to-husband, which it converts to -36% and -24% at the
cell mean; the SEs here are converted the same way (1.9/11.9 and 1.8/11.25, both about 16%). Intimate homicide: FBI
Supplementary Homicide Reports 1968-94 (Illinois and DC excluded), percent change in the rate of women and men killed
by a spouse, with controls (Table 5, column 2). In the paper's triple difference against non-intimate homicide the
women's estimate is -7.2% (SE 6.9), not significant.

Run:  uv run --with pandas,matplotlib,pillow,fonttools python notebooks/marital_happiness/unilateral_divorce_chart.py
"""
import subprocess
import sys
from pathlib import Path

import matplotlib.pyplot as plt
import pandas as pd

sys.path.insert(0, str(Path.home() / "_watermark"))
import pb_style as pb  # noqa: E402

HERE = Path(__file__).resolve().parent
NOTES = Path.home() / "Desktop" / "Recent Projects" / "Substack Notes" / "Marriage"
OUT = NOTES.parent / "Unwatermarked graphs" / "Marriage" / "unilateral_divorce.png"
Z = 1.96
GB_MEAN = 52  # Goodman-Bacon's mean female suicide rate, per million

est = pd.read_csv(HERE / "unilateral_divorce_estimates.csv")
A = {s: d for s, d in est[est.panel == "A"].groupby("series")}
B = est[est.panel == "B"].reset_index(drop=True)
C = est[est.panel == "C"].reset_index(drop=True)

pb.apply()
fig = plt.figure(figsize=(14, 11.6))
axA = fig.add_axes([0.07, 0.53, 0.90, 0.30])
axB = fig.add_axes([0.285, 0.13, 0.26, 0.28])
axC = fig.add_axes([0.765, 0.13, 0.175, 0.28])

fig.text(0.03, 0.955, "When either spouse could leave, women's suicides fell. How much is less certain",
         fontsize=21, fontweight="bold", color=pb.INK)
fig.text(0.03, 0.918, "US states adopted unilateral divorce at different times, 1969–85. Female suicide fell in "
         "reform states, gradually, by up to 16% after 19 years; men's barely moved.\nA later reanalysis of the "
         "same data finds the estimate swings from clearly negative to zero depending on reasonable modeling choices.",
         fontsize=12.5, color=pb.INK2, va="top", linespacing=1.45)


def title(ax, text, x, y=1.1):
    from matplotlib.transforms import blended_transform_factory
    ax.text(x, y, text, transform=blended_transform_factory(fig.transFigure, ax.transAxes), fontsize=14,
            fontweight="bold", color=pb.INK, ha="left")


# A: event study
title(axA, "A  Change in suicide rate, by years since the state allowed unilateral divorce", 0.03, y=1.07)
axA.axhline(0, color=pb.MUTED, lw=1)
for key, color, ls, lw in [("men", pb.GRAYS[0], "-", 2), ("women_trends", pb.RED, (0, (4, 3)), 1.6),
                           ("women", pb.RED, "-", 2.8)]:
    d = A[key]
    if key == "women":
        axA.fill_between(d.x, d.estimate - Z * d.se, d.estimate + Z * d.se, color=pb.RED, alpha=0.12, lw=0)
    axA.plot(d.x, d.estimate, color=color, ls=ls, lw=lw, marker="o" if key == "women" else None, ms=5)
w, wt, m = A["women"].iloc[-1], A["women_trends"].iloc[-1], A["men"].iloc[-1]
axA.text(19.9, w.estimate, f"Women: {w.estimate:+.0f}%", color=pb.INK, fontweight="bold", va="center", fontsize=12)
axA.text(19.9, wt.estimate, f"Women, with state trends: {wt.estimate:+.0f}%\n(not significant)", color=pb.INK2,
         va="center", fontsize=11)
axA.text(19.9, m.estimate, f"Men: {m.estimate:+.0f}%", color=pb.INK2, va="center", fontsize=11.5)
axA.text(9.6, -24.5, "Shaded: 95% interval for women (preferred model)", color=pb.MUTED, fontsize=10.5)
axA.set_xlim(-0.5, 26)
axA.set_ylim(-26, 10)
axA.set_xticks(A["women"].x)
axA.set_xticklabels(["0", "1–2", "3–4", "5–6", "7–8", "9–10", "11–12", "13–14", "15–16", "17–18", "19+"])
axA.set_xlabel("Years since reform", color=pb.INK2)
axA.yaxis.set_major_formatter(lambda v, _: f"{v:+.0f}%" if v else "0")
axA.grid(axis="y")
axA.spines[["left", "bottom"]].set_visible(False)

# B: specification curve
title(axB, "B  Female suicide under other reasonable models (reanalysis)", 0.03)
axB.axvline(0, color=pb.MUTED, lw=1)
ys = range(len(B))[::-1]
for y, (_, r) in zip(ys, B.iterrows()):
    base = r.label.startswith("Baseline")
    color = pb.NAVY if base else pb.GRAYS[1]
    if pd.notna(r.se):
        axB.plot([r.estimate - Z * r.se, r.estimate + Z * r.se], [y, y], color=color, lw=2, alpha=0.55)
    axB.plot(r.estimate, y, "o", color=color, ms=8 if base else 7)
    axB.text(-11.4, y, r.label, ha="right", va="center", fontsize=11, color=pb.INK,
             fontweight="bold" if base else "normal")
    axB.text(9.2, y, f"{r.estimate / GB_MEAN * 100:+.0f}%", ha="right", va="center", fontsize=11,
             color=pb.INK, fontweight="bold")
axB.set_xlim(-11, 9.5)
axB.set_ylim(-0.7, len(B) - 0.3)
axB.set_yticks([])
axB.set_xlabel("Suicides per million women per year (mean 52)", color=pb.INK2)
axB.grid(axis="x")
axB.spines[["left", "bottom"]].set_visible(False)

# C: other outcomes
title(axC, "C  Other outcomes in reform states", 0.60)
axC.axvline(0, color=pb.MUTED, lw=1)
ys = range(len(C))[::-1]
for y, (_, r) in zip(ys, C.iterrows()):
    women = r.label.startswith(("Husband", "Women"))
    color = pb.NAVY if women else pb.GRAYS[0]
    axC.barh(y, r.estimate, color=color, height=0.55)
    axC.plot([r.estimate - Z * r.se, r.estimate + Z * r.se], [y, y], color=pb.INK, lw=1.2)
    axC.text(-73, y, r.label, ha="right", va="center", fontsize=11, color=pb.INK)
    axC.text(42, y, f"{r.estimate:+.0f}%", ha="right", va="center", fontsize=11, color=pb.INK, fontweight="bold")
axC.set_xlim(-70, 43)
axC.set_ylim(-0.6, len(C) - 0.4)
axC.set_yticks([])
axC.xaxis.set_major_formatter(lambda v, _: f"{v:+.0f}%" if v else "0")
axC.set_xlabel("Change after reform (95% interval)", color=pb.INK2)
axC.grid(axis="x")
axC.spines[["left", "bottom"]].set_visible(False)

fig.text(0.03, 0.045,
         "Sources: A and C, Stevenson & Wolfers, NBER w10175 (QJE 2006), Tables 2, 3 and 5. Suicide rates are for all "
         "women or men in the state, married or not, 1964–96; dashed line adds state-specific trends.\nB, Goodman-Bacon, "
         "NBER w25018 (J. Econometrics 2021), Table 2 and section III; same suicide data in levels. Violence: 1976 vs "
         "1985 family violence surveys; homicide: FBI reports 1968–94. Intervals are ±1.96 SE.",
         fontsize=9.5, color=pb.MUTED, linespacing=1.5)

OUT.parent.mkdir(parents=True, exist_ok=True)
fig.savefig(OUT, dpi=200, facecolor=pb.SURFACE)
subprocess.run([sys.executable, str(Path.home() / "_watermark" / "watermark.py"), "7", str(OUT),
                str(NOTES / (OUT.stem + "_wm.png"))], check=True)
```
