# UrgencyDDMonSpecies

Code and data for:

> Asadpour, A., Tan, H., Lenfesty, B., & Wong-Lin, K. (2024). **Of Rodents and Primates: Time-Variant Gain in Drift–Diffusion Decision Models.** *Computational Brain & Behavior*. https://doi.org/10.1007/s42113-023-00194-1
> ([paper](https://link.springer.com/article/10.1007/s42113-023-00194-1))

## Purpose

In many drift–diffusion models (DDMs), a time-variant gain (an "urgency signal") multiplies both the drift rate (signal) and the noise. It is therefore unclear what the gain does on each component separately. This repository fits four variants of a time-variant-gain DDM (model 4 of Ditterich, 2006) to random-dot motion discrimination data from **monkeys** (Roitman & Shadlen, 2002), **humans** and **rats** (Shevinsky & Reinagel, 2019), and plots the results.

The decision variable follows `dX = (μ_initial·γ(t))dt + √dt·(γ(t)·σ)·η`, where `μ_initial = k·c` (c = motion coherence) and γ(t) is a logistic gain function with parameters `d`, `s_x`, `s_y` (γ(0) = 1). Bounds are constant at ±1. There are seven fitted parameters: `k, s_y, s_x, d, σ0, t_residual, σ_residual`.

The four cases compared:

| Case | Gain on drift | Gain on noise | Noise depends on coherence |
|------|:---:|:---:|:---:|
| (i)   | yes | yes | yes (control, as in Ditterich 2006) |
| (ii)  | yes | yes | no |
| (iii) | no  | yes | no |
| (iv)  | yes | no  | no |

## Repository layout

```
Monkey/   monkey_squared_loss_fixed.py, solve_model.py, roitman_rts.csv, fixedIC/ (fitted models, .pkl)
Human/    human_squared_loss_fixed.py,  solve_model.py, BestUnbiasedPsychoEpochsHUMAN200502.csv, fixedIC/
rat/      rat_squared_loss_fixed.py,    solve_model.py, BestUnbiasedPsychoEpochsRAT200502.csv, fixedIC/
Matlab Code/   analysis and plotting scripts (Figs 2–4 and Fig S1)
requirements.txt
```

In each species folder:
- `*_squared_loss_fixed.py` fits all four cases with PyDDM (`LossRobustLikelihood`, T = 5 s, dt = 0.01 s) using multiple parallel runs, and saves the results as `.pkl` files.
- `solve_model.py` solves a fitted model and computes its psychometrics/chronometrics, saving `solved_model_*.pkl`.
- `fixedIC/` holds the pre-computed outputs used for the paper's figures, so you can reproduce the figures without refitting.

Data: only epochs with RT between 0.1 and 1.65 s (monkey) or 0.1 and 2.5 s (human, rat) are used. The original datasets come from the studies cited above.

## Setup

Python 3 is required.

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

`pyddm==0.7.0` is pinned because the scripts rely on its API.

## How to run

### 1. Fit the models (Python)

Run from inside the species folder, because the scripts read the CSV with a relative path:

```bash
cd Monkey && python monkey_squared_loss_fixed.py
cd Human  && python human_squared_loss_fixed.py
cd rat    && python rat_squared_loss_fixed.py
```

Each script fits cases (i)–(iv) with repeated runs (the paper used 18 runs per case; the scripts default to 40 for human/rat and `cpu_count()-2` for monkey) in parallel via `multiprocessing`, and writes `restricted_models_*_LossRobustLikelihood.pkl` files. Fitting is stochastic and can take a long time, so run it on a multi-core machine.

Notes:
- `rat_squared_loss_fixed.py` opens `BestUnbiasedPsychoEpochsRat200502.csv`, but the file is named `...RAT200502.csv`. On case-sensitive file systems (Linux) change the filename in the script or rename the file.
- Edit `run_numbers` in the script to change the number of runs.

To compute the psychometrics of a solved model, run `python solve_model.py` in the same folder.

### 2. Reproduce the figures (MATLAB)

MATLAB must be configured to call Python (`pyenv`), because the model `.pkl` files are loaded with `py.pickle.load` and require PyDDM to be importable from that Python environment.

In MATLAB, from the `Matlab Code` folder:

```matlab
plot_within_species_figures      % Figs 2–4 (monkey, human, rat)
plot_between_species_figures     % Fig S1 (supplementary)
```

The paper used MATLAB R2023a.

## Results

Main findings (see the paper for figures and statistics):

- **Monkeys:** time-variant gain on both drift and noise (cases i and ii) fits best. Gain only on the noise (case iii) still produces slower error RTs than correct RTs, whereas gain only on the drift (case iv) fits worst and gives faster errors.
- **Rats:** time-variant gain on the drift rate only (case iv) fits best, with the lowest squared error and AIC, reproducing the faster error RTs seen in rodents.
- **Humans:** time-variant gain has minimal and largely indistinguishable effects across cases. The fitted gain for case (iv) stays close to 1.
- **Overall:** gain on noise alone gives slower errors and gain on drift alone gives faster errors, so the component the gain acts on shapes choice behaviour and may differ between species.

Model comparison used the squared error across runs (Kruskal–Wallis with Tukey HSD) and AIC = 14 − 2·log-likelihood.

## Citation

```bibtex
@article{asadpour2024rodents,
  title   = {Of Rodents and Primates: Time-Variant Gain in Drift--Diffusion Decision Models},
  author  = {Asadpour, Abdoreza and Tan, Hui and Lenfesty, Brendan and Wong-Lin, KongFatt},
  journal = {Computational Brain \& Behavior},
  year    = {2024},
  doi     = {10.1007/s42113-023-00194-1}
}
```

## References

- Ditterich, J. (2006). Computational approaches to visual decision making. 
- Roitman, J. D., & Shadlen, M. N. (2002). *J Neurosci*.
- Shevinsky, C. A., & Reinagel, P. (2019). *Front Neurosci*.
- Shinn, M., Lam, N. H., & Murray, J. D. (2020). A flexible framework for simulating and fitting generalized drift-diffusion models. *eLife*.
