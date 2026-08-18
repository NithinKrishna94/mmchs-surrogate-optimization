# Machine Learning Surrogate Model for Thermal-Fluid Design Optimisation

Adding a **manifold distribution layer** to a converging–diverging microchannel heat
sink, and using a CFD-trained surrogate model to optimise the combined geometry.

**Nithin** — M.Tech, Thermal and Fluids Engineering, IIT Bombay
---

## The idea

A converging–diverging microchannel bank improves heat transfer by periodically
accelerating and decelerating the coolant, which repeatedly redevelops the thermal
boundary layer. A manifold layer does something different: it splits the coolant
into several short parallel passes, collapsing the effective flow length and cutting
both the streamwise temperature rise and the pressure drop.

The two enhancements have been studied separately. This project combines them —
placing a manifold above an experimentally validated converging–diverging channel
bank — and optimises the resulting geometry.

## Why this needs a surrogate rather than parameter tuning

The channel geometry cannot simply be inherited from the non-manifold design.

The baseline study optimised channel count, depth, fin angle and fin width for a
single long pass, where the flow is close to fully developed over most of the
channel. Under a manifold the pass length collapses and entrance effects dominate
instead. **The optimum channel geometry is therefore expected to move**, and the
manifold and channel parameters have to be optimised together rather than in
sequence.

That combined space is too large for one-factor-at-a-time sweeps and too expensive
for CFD-in-the-loop optimisation — published work reports on the order of 50 hours
of cluster time for a full CFD-coupled genetic algorithm. Running CFD once on a
designed set of points and fitting a regression model to it brings evaluation cost
down to milliseconds, with CFD returning only to verify the final selections.

```
   baseline validation  →  designed CFD dataset  →  surrogate  →  NSGA-II optimisation
                                                                          ↓
                                                          CFD verification of selections
```

## Baseline

The channel bank follows Meher, Agrawal and Saha (*Appl. Therm. Eng.* **293**, 2026,
art. 130472) — an experimentally validated converging–diverging multi-microchannel
sink at 200 W/cm², 0.0035 kg/s and 300 K inlet, on a 6.35 × 17.5 mm² footprint.

Their reported cases serve three purposes here:

- **Validating the CFD setup.** Reproducing their results means this model inherits
  their experimental validation rather than needing its own test rig.
- **Fixing the starting channel geometry.** Their optimum — 27 channels, 2° fin
  angle, 0.8 mm depth, 0.24 mm fin width — is the point the manifold is built on.
- **Setting the benchmark.** The manifold design has to beat their non-manifold
  performance at the same heat flux, flow rate and footprint, or it isn't worth
  the added fabrication complexity.

Reference cases: [`data/baseline_cd_mchs.csv`](data/baseline_cd_mchs.csv).
They are a validation and benchmark set, **not** the training set.

## Approach notes

**No one-factor-at-a-time.** Published work across manifold microchannel geometries
consistently finds the optimum where no single-parameter sweep points, and Sobol
analyses report total-effect indices well above unity. Sequential tuning of
interacting parameters converges on the wrong answer — and the manifold–channel
coupling here is exactly that situation.

**Model choice by comparison.** Several regressors fitted on identical splits and
compared on the same metrics, rather than one adopted by default.

**Sample size demonstrated, not asserted.** A learning curve of cross-validated R²
against training-set size, before any optimisation is run. Most of the surveyed
studies skip this.

## Status

| Stage | Status |
|---|---|
| Literature survey on surrogate modelling and CFD-based design optimisation | Done |
| Baseline data collected; benchmark and channel-level starting point fixed | Done |
| Reproduction of baseline cases to validate the CFD setup | Done |
| Manifold geometry parameterisation | In progress |
| Design of experiments — parameters, levels, sampling scheme | In progress |
| Generation of the designed CFD dataset | In progress |
| Surrogate fitting, benchmarking, learning curve | Not started |
| SHAP attribution | Not started |
| NSGA-II coupling, Pareto front, TOPSIS selection | Not started |
| CFD verification of selected designs | Not started |

Open decisions are tracked in [`docs/methodology.md`](docs/methodology.md),
sections 4 and 9.

## Repository layout

```
docs/     literature survey and methodology
data/     baseline reference cases, and the designed dataset as it is generated
results/  validation of the numerical setup
src/      DOE generation, surrogate training, optimisation
```

## Key references

Meher, Agrawal & Saha, *Appl. Therm. Eng.* **293** (2026) 130472 · Tang et al.,
*Int. J. Heat Mass Transfer* **200** (2023) 123495 · Xia et al., *Appl. Therm. Eng.*
**279** (2025) 127482 · Li et al., *Appl. Therm. Eng.* **251** (2024) 123587 ·
Sikirica et al., arXiv:2208.09683 (2022)

Full survey in [`docs/literature-survey.md`](docs/literature-survey.md).
