# Methodology

The approach this project takes, and why.

This is a **working plan**. The literature survey is complete and the baseline data
is in hand; the manifold parameterisation and everything downstream is the work
still to be done. Anything marked `TBD` has not been decided yet, and is recorded as
open rather than filled in speculatively.

Motivation and the studies behind these choices:
[`literature-survey.md`](literature-survey.md).

---

## 1. What is being built

Two enhancement strategies for microchannel heat sinks are well established
individually:

**Converging–diverging channels** periodically accelerate and decelerate the
coolant. Favourable pressure gradients in the converging section thin the thermal
boundary layer and raise wall shear; controlled deceleration in the diverging
section increases coolant residence time. The net effect is repeated boundary-layer
redevelopment rather than the monotonic decay seen in uniform channels.

**Manifold layers** split the coolant into several short parallel passes instead of
one long one. The effective flow length collapses to a fraction of the footprint,
which cuts both the streamwise temperature rise and the pressure drop, and the
jet-impingement action under each inlet slot raises the local heat transfer
coefficient.

This project **combines them** — a manifold placed above a converging–diverging
channel bank — and optimises the combined geometry.

---

## 2. Baseline: what is inherited and what is not

The channel bank follows Meher, Agrawal and Saha (*Appl. Therm. Eng.* **293**, 2026,
art. 130472): an experimentally validated conjugate model of a converging–diverging
multi-microchannel sink under a fixed footprint.

**Inherited**

| Item | Value |
|---|---|
| Footprint | 6.35 × 17.5 mm² |
| Applied heat flux | 200 W/cm² |
| Coolant mass flow rate | 0.0035 kg/s |
| Inlet temperature | 300 K |
| Solid thermal conductivity | 380 W/m·K |
| Coolant | Water, single phase, temperature-dependent viscosity |
| Channel-level starting geometry | 27 channels, 2° fin angle, 0.8 mm depth, 0.24 mm fin width |

Holding the heat flux, flow rate and footprint fixed is deliberate: it makes the
manifold design directly comparable against the non-manifold benchmark.

**Not inherited: the claim that the channel geometry is optimal**

The baseline optimum was found for a **single 6.35 mm pass**, where the flow is
close to fully developed over most of the channel length. Introducing a manifold
collapses the pass length, so entrance effects dominate instead and the balance
between conduction through the fin, convection in the fluid, and hydraulic loss
shifts.

Concretely, the reasoning behind two of the baseline choices weakens under a
manifold:

- Channel count was limited to 27 because thinner fins beyond that weaken solid
  conduction and raise flow resistance. Shorter passes reduce the per-pass pressure
  penalty, which may move that limit.
- Depth of 0.8 mm was chosen partly because deeper channels reduce viscous
  resistance over a long pass. Over a short pass that benefit is smaller.

So the channel parameters are treated as **open**, not fixed, and the manifold and
channel parameters are optimised together.

Reference cases: [`data/baseline_cd_mchs.csv`](../data/baseline_cd_mchs.csv). Their
role is validation and benchmarking — they are **not** the surrogate training set,
because the flow configuration they describe is not the one being designed.

---

## 3. Why a surrogate

The combined manifold-plus-channel space is larger than the baseline study's, and
its parameters interact. One-factor-at-a-time sweeps are not adequate for it —
published work across manifold microchannel geometries repeatedly finds the optimum
at combinations that no single-parameter sweep points to, and Sobol analyses on
these sinks report total-effect indices well above unity.

Covering that space directly with CFD-in-the-loop optimisation is not affordable;
published work reports on the order of 50 hours of cluster time for a full
CFD-coupled genetic algorithm. Running CFD once on a designed set of points and
fitting a regression model brings evaluation cost down to milliseconds, with CFD
returning only to verify the final selections.

---

## 4. Manifold parameterisation and design of experiments — to be decided

**Status: this is the immediate next piece of work.** The decisions below are open,
and are recorded here so the reasoning is visible once they are made.

### Manifold parameters under consideration

| Parameter | Why it matters | Status |
|---|---|---|
| Number of inlet/outlet pairs | Sets the effective flow length — the primary manifold effect | `TBD` |
| Inlet-to-outlet slot width ratio | Controls jet velocity into the channel and flow distribution along the bank | `TBD` |
| Manifold layer height | Affects distribution uniformity and manifold-side pressure loss | `TBD` |
| Manifold arrangement (U-type / Z-type) | Z-type arrangements are reported to suffer flow maldistribution | `TBD` |

### Channel parameters under consideration

Channel count, fin angle, depth and fin width — the same four the baseline study
varied, now re-opened for the reason given in section 2.

### Sampling decisions

| Decision | Status |
|---|---|
| Which parameters to carry, and which to freeze at baseline values | `TBD` |
| Number of levels per parameter | `TBD` |
| Sampling scheme — orthogonal array, Latin hypercube, or full factorial on a reduced set | `TBD` |
| Total run budget | `TBD` |

Notes feeding these decisions:

- Carrying all eight parameters is almost certainly beyond the run budget. A
  screening stage may be needed to decide what enters the main design.
- Several parameters are **coupled by the fixed footprint** — changing fin width
  changes how many channels fit, which changes the per-channel flow rate. The design
  has to be laid out on the constrained space, not on an idealised independent one.
- The baseline study finds non-monotonic behaviour in channel count and fin angle,
  so at least three levels are needed on those to capture curvature.

Once fixed, the design will be generated by a script under `src/doe/` so that it is
reproducible and its balance can be checked, rather than assembled by hand.

---

## 5. CFD model

Solver: ANSYS Fluent.

**Conjugate heat transfer** — solid conduction and fluid convection coupled at the
wetted interface — under steady, laminar, incompressible flow.

```
Continuity      ∇ · u = 0
Momentum        ρf (u · ∇) u = −∇p + μf ∇²u
Fluid energy    ρf cp,f (u · ∇Tf) = kf ∇²Tf
Solid energy    ks ∇²Ts = 0
```

The setup follows the baseline study wherever the physics is unchanged, so that
reproduction of its cases is a meaningful check. Settings specific to the manifold
configuration to be recorded here as the model is built:

| Item | Value |
|---|---|
| Computational domain — full sink, or periodic unit cell | `TBD` |
| Manifold inlet and outlet boundary treatment | `TBD` |
| Discretisation schemes and convergence criteria | `TBD` |
| Meshing strategy and adopted cell count | `TBD` |

Because the design variables alter the local geometry, the same **meshing controls**
should be carried case to case rather than the same absolute cell count.

---

## 6. Quantities extracted and objectives

| Quantity | Role |
|---|---|
| Maximum substrate temperature | Primary thermal measure |
| Thermal resistance | Objective |
| Pressure drop | Objective |
| Temperature uniformity factor | Reported, and a constraint candidate |
| Flow uniformity factor | Diagnostic — manifold designs are prone to maldistribution |
| Performance evaluation criterion (PEC) | Derived comparison against the non-manifold benchmark |

Thermal resistance and pressure drop conflict — any change that accelerates the
coolant to improve convection also raises the pressure loss — which is why the
optimisation is multi-objective rather than scalarised.

---

## 7. Validation

Three checks, in order:

**Reproduction of the baseline cases.** Since the source study is experimentally
validated, agreement means this model inherits that validation rather than needing a
separate experimental campaign. This is done on the non-manifold geometry, before
the manifold is added.

**Energy balance.** Coolant enthalpy rise against the applied heat load, confirming
the conjugate interface and heat flux boundary are handled consistently. Applied to
every run, manifold or not.

**Grid independence on the manifold configuration.** The manifold introduces
features the baseline mesh study never covered — the slot-to-channel transition and
the impingement region — so grid independence has to be re-established rather than
assumed from the baseline.

Outcomes recorded in [`results/validation.md`](../results/validation.md).

---

## 8. Surrogate modelling (planned)

Inputs: the design variables chosen in section 4. Outputs: thermal resistance and
pressure drop, with derived measures such as pumping power and PEC computed from
those rather than predicted separately, so the model cannot produce a
thermodynamically inconsistent set.

**Model selection by comparison, not by default.** Several regressors fitted on
identical splits and compared on the same metrics — polynomial response surface as a
baseline, random forest, gradient-boosted ensemble, Gaussian process, small neural
network. Published evidence suggests tree ensembles are competitive with almost no
tuning while a tuned network may edge ahead on a large enough dataset. Worth
establishing rather than assuming.

**Sample size demonstrated, not asserted.** A learning curve of cross-validated R²
against training-set size, generated before any optimisation is attempted. Most of
the surveyed studies skip this, and without it a reported R² only shows how well the
model interpolates its own sampling density.

**Interpretability alongside the result.** SHAP attribution computed on the selected
model and reported with the Pareto front. Here it carries extra weight: whether the
manifold parameters or the channel parameters dominate the response is one of the
questions the project exists to answer.

---

## 9. Optimisation and design selection (planned)

The trained surrogate coupled to **NSGA-II**, minimising thermal resistance and
pressure drop simultaneously to produce a Pareto front.

A compromise design then selected from the front by **TOPSIS with entropy-derived
weights**, so the weighting follows the actual spread of each objective across the
front rather than an arbitrary judgement.

Selected designs re-simulated in Fluent. Agreement within a few percent between
surrogate prediction and direct simulation is the acceptance criterion for the model.

The final comparison is against the non-manifold benchmark at the same heat flux,
flow rate and footprint. If the manifold design does not beat it by a margin that
justifies the added fabrication complexity, that is a result worth reporting too.

---

## 10. Open questions

Recorded so the reasoning stays visible:

- Which manifold and channel parameters to carry into the DOE, and at how many
  levels (section 4)
- Whether a screening stage is needed before the main design
- Whether the optimal channel geometry does in fact move once a manifold is added —
  this is the central technical question, and the surrogate is what answers it
- Whether a periodic unit cell is valid for the manifold configuration, or whether
  flow maldistribution along the bank forces a full-sink domain
- How many CFD points the surrogate will need — the learning curve decides this,
  not a target chosen up front
