# Transparency vs. Accuracy in Financial AI

A study of what drives people to prefer explainable ("glass-box") financial AI systems over purely accurate ("black-box") ones, using mixture models built to separate *sentiment* from *uncertainty* in survey responses.

> Original title: *Trasparenza vs accuratezza nell'IA finanziaria: un'analisi con modelli CUB/GeCUB e Proportional Odds*

## Overview

Financial institutions increasingly face a trade-off between AI systems that are highly accurate but opaque, and systems that are less powerful but easy to explain. This project analyzes survey data (n = 303) on how people weigh that trade-off, using an ordinal target variable (`preferenza_IA`, 0–10) where low scores favor efficiency/accuracy and high scores favor transparency.

The core methodological question is whether a standard ordinal regression is enough to explain these preferences, or whether part of what looks like "indecision" is actually a distinct, measurable component of the response. To test this, the analysis pits a **GeCUB** mixture model (which explicitly separates *feeling* — attraction toward transparency — from *uncertainty* — how unsure a respondent is) against a classical **Proportional Odds Model (POM)** used as a benchmark.

## Dataset

Survey data (n = 303) with:
- **Target**: `preferenza_IA`, an ordinal 0–10 scale (rescaled to 1–11 for the mixture models), where 0–3 = preference for black-box accuracy, 4–6 = compromise/indecision, 7–10 = preference for glass-box transparency.
- **Predictors**: over twenty variables spanning demographics (age, gender, education), technology awareness (prior use of AI-based financial apps, awareness of black-box models), and trust/expectations (clarity of criteria, right to explanation, human oversight).

The sample skews young and educated (69.6% aged 18–34; 60.3% hold at least a bachelor's degree), which is noted as a limitation on generalizability.

## Methodology

1. **Exploratory analysis** — sample composition, technology awareness, and the shape of the target variable, which shows negative skew toward transparency and a "shelter effect" (a cluster of neutral responses at the midpoint).
2. **GLM framework recap** — the three components of a generalized linear model, and where CUB/GeCUB sits relative to that framework (it extends the spirit of GLMs — linking latent parameters to covariates via logistic links — without technically belonging to the exponential family, so estimation uses EM / direct log-likelihood maximization rather than IRLS).
3. **GeCUB model** — a finite mixture of a shifted Binomial (the *feeling* component, ξ) and a discrete Uniform (the *uncertainty* component, 1−π), each linked to covariates via `logit(π) = y'γ` and `logit(ξ) = w'β`.
4. **Model selection** — five candidate specifications compared by BIC; the final model links uncertainty to black-box awareness and feeling to age.
5. **POM benchmark** — a standard proportional-odds ordinal regression on the same two drivers, to test whether the extra complexity of GeCUB is actually earning its keep.
6. **Manual MLE validation** — a hand-built CUBE + shelter-effect likelihood, optimized directly (BFGS), to stress-test whether the residual anomalies seen in the GeCUB fit call for a richer model. The result is a degenerate, unidentifiable fit — which reinforces rather than undermines the simpler GeCUB choice.
7. **Stepwise POM** — a best-fit proportional-odds model selected automatically from an extended predictor set, as a further point of comparison.

## Key Findings

- **Age drives the "feeling" component**: older respondents show a systematically stronger preference for transparency (Wald = −4.32, p < 0.001).
- **The "paradox of the expert"**: awareness of black-box models does *not* push preferences in either direction — instead, it increases the *uncertainty* component (Wald = −2.36, p = 0.018). Knowing that a system is opaque doesn't make people's opinion more decisive; it makes them more cautious.
- The **Proportional Odds Model misses this entirely**: in the POM, the same black-box-awareness variable is statistically insignificant (p = 0.943), because POM can only capture where a response sits on the scale, not how confidently it was given.
- On pure information criteria (BIC), GeCUB and POM are close, and a POM built with different covariates (age + clarity of factors) edges GeCUB out slightly — but only GeCUB is able to isolate *why* the sample clusters at the midpoint, which the raw BIC comparison doesn't capture.

## Repository Contents

```
glm-transparency/
├── README.md
├── report.qmd          # Quarto source: full narrative + code
├── progetto_Glm.pdf    # compiled report, PDF (Italian)
└── LICENSE
```

`report.qmd` is the actual analysis: it loads the data, fits both the GeCUB and POM models, runs the manual MLE stress-test, and generates every table and figure that appears in `progetto_Glm.pdf`. The PDF is kept alongside it as the ready-to-read compiled version.

## Reproducing the analysis

The report renders to PDF via Quarto, so on top of R you'll need a LaTeX distribution — the easiest route is `quarto install tinytex`.

The script expects a cleaned survey file named `dataset_glm_pulito (3).csv` in the working directory (it also checks a couple of author-specific fallback paths, which you can ignore). That file isn't included here — survey data with respondent-level demographics is best kept out of a public repo, or anonymized first if you want to include it. Drop your own copy in the project folder under that exact name and the script will pick it up automatically.

Required R packages (the report stops with a clear message if any are missing): `dplyr`, `ggplot2`, `tidyr`, `knitr`, `MASS`, `ordinal`. Two more are used but optional — the report degrades gracefully if they're absent: `CUB` + `Formula` (for the CUB/GeCUB models) and `brant` (for the parallel-lines test). Then render with:

```bash
quarto render report.qmd
```

## Tools

R and Quarto, with the `CUB` package (Iannario, Piccolo & Simone) for the CUB/GeCUB models, `MASS` and `ordinal` for the Proportional Odds benchmark, and `brant` for the parallel-lines test.

## Limitations

The sample is a convenience sample, skewed young and highly educated, so results should not be generalized to the broader population. CUB/GeCUB models can be sensitive to convergence issues on smaller or sparser datasets, as the manual MLE stress-test in this project directly demonstrates. Statistical significance and practical relevance don't always align — some effects here are consistent but modest in size.

## Author

Antonio Verde

## License

MIT — see [LICENSE](./LICENSE).
