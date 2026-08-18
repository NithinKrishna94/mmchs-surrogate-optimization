# Literature Survey

**Machine-learning surrogate models for thermal–fluid design optimisation of
manifold microchannel heat sinks**

Survey conducted as the opening stage of this project. It is organised around the
four decisions a surrogate-based design study has to make: which geometric
parameters are worth carrying; how the training data should be sampled; which
regression family to fit; and how the fitted model should be driven by an optimiser
and then interrogated for physical insight.

Five recent studies are examined in detail [6–10], supported by the broader
microchannel literature.

---

## 1. Background and scope

Chip-level heat fluxes in power electronics, laser diodes and high-density logic
now routinely exceed 100 W/cm², and localised values beyond 1000 W/cm² are reported
for GaN devices and laser sources. Air cooling stopped being a credible answer to
this a long time ago. Single-phase liquid cooling through microchannels, first
demonstrated by Tuckerman and Pease [1], remains the most practical route, but the
conventional straight-channel heat sink carries two structural weaknesses: the
coolant heats up along a long flow path, which produces a strong streamwise
temperature gradient, and the pressure drop scales badly as the hydraulic diameter
shrinks.

The **manifold microchannel heat sink (MMCHS)**, introduced by Harpole and Eninger
[2] and analysed numerically by Ryu and co-workers [3], addresses both problems at
once. A distribution layer above the channels splits the coolant into many short
parallel passes, so the effective flow length collapses to a fraction of the
footprint. The pressure drop falls, the streamwise temperature rise falls with it,
and the jet-impingement action under each inlet slot raises the local heat transfer
coefficient. Devices built on this principle have been co-integrated directly with
the semiconductor die [4], and hierarchical manifold arrays have been characterised
experimentally at high heat flux [5].

The difficulty is that an MMCHS has many more geometric handles than a
straight-channel sink: channel width and depth, fin width, manifold slot widths and
heights, inlet-to-outlet asymmetry, and any streamwise tapering of either the
manifold or the channel. These handles interact, they trade thermal resistance
against pumping power, and each candidate design costs a three-dimensional
conjugate heat transfer simulation to evaluate. Direct coupling of a genetic
algorithm to CFD is therefore prohibitively slow — Sikirica et al. [6] report
roughly 55 hours of cluster time for 20,000 case evaluations on a comparatively
simple geometry. This is the gap that machine-learning surrogate models fill.

---

## 2. What is worth parameterising in a manifold heat sink

### 2.1 Manifold and channel inlet/outlet asymmetry

Tang et al. [7] proposed a diverging/converging MMCHS and reduced the design space
to two dimensionless ratios: the manifold inlet-to-outlet length ratio
α = Lin/Lout and the channel inlet-to-outlet width ratio β = Wc,in/Wc,out, each
varied from 1/9 to 9 on a periodic unit cell in COMSOL.

Their single-parameter results are already informative: at a fixed pumping power of
5 × 10⁻⁴ W, a moderately diverging manifold (α = 1/3) lowered thermal resistance by
about 11 %, and a strongly diverging channel (β = 1/9) by about 13.5 %, relative to
the equal-width baseline. Both effects trace back to the same mechanism: a narrower
inlet accelerates the coolant, so the jet impinges on the channel floor harder,
thins the boundary layer, and sets up a vortex near the outlet that keeps the fluid
mixed.

The more important result is what happened when both ratios were varied together
across 99 combinations. **The single-parameter optima did not survive.** As α was
raised from 1/9 to 9 the best-performing β moved in the opposite direction, and the
overall optimum sat at α = 1/9 with β = 9 — a 19.2 % reduction in thermal
resistance, and a combination that neither one-factor-at-a-time sweep would have
suggested. The worst combination, α = β = 9, more than tripled the thermal
resistance.

For a surrogate-based study this is the single most useful finding in the set: the
response surface of an MMCHS has strong two-factor interaction, so sequential
parameter tuning is not merely inefficient but wrong.

### 2.2 Tapered manifolds and variable cross-section channels

Li et al. [9] attacked temperature uniformity rather than peak temperature. Their
tapered-manifold, variable-cross-section design (TMVC-MCHS) narrows the inlet
manifold along its length so that coolant is pushed further towards the far
channels, and simultaneously contracts the channel cross-section along the flow
direction so that the fluid accelerates where it has already been heated.

A single-factor screening over six candidate ratios showed that the width
inclination of the inlet manifold matters more than the height inclination by a
factor above 2.5, and that reducing the *outlet* manifold inclination actually
hurts — so the common practice of tapering inlet and outlet identically is not
optimal. The screening reduced six candidates to three design variables, which is a
sensible pattern to copy: use cheap one-factor runs to decide *what enters* the
surrogate, then spend the expensive sampling budget on the survivors.

### 2.3 Channel count, depth and fin scaling under a fixed footprint

Meher, Agrawal and Saha [10] report an experimentally validated conjugate model of
a converging–diverging multi-microchannel sink at 200 W/cm² with the footprint,
inlet temperature and mass flow rate held fixed. Sweeping channel count (19–35),
depth (0.5–0.9 mm), fin angle (1–4°) and fin width, they found that **none of the
trends is monotonic** once the footprint constraint is enforced. Temperature
uniformity improves with channel count up to about 27 channels, after which the
fins become too thin to conduct and flow resistance climbs; the fin angle that
minimises substrate temperature is 2°, with larger angles introducing adverse
pressure gradients that decelerate the flow in the diverging section. Their
converging–diverging arrangement delivered roughly 20–25 % lower thermal resistance
than uniform channels at the higher flow rates examined.

The methodological lesson is that fixed-footprint parameterisation couples
variables implicitly — changing channel width changes channel count, which changes
per-channel flow rate — and a surrogate must be trained on the constrained space,
not on an idealised independent one.

### 2.4 Which parameters actually carry the variance

Two complementary attribution methods appear in this literature.

Xia et al. [8] applied **variance-based Sobol analysis** to a diamond-substrate
U-type counter-flow MMCHS with five geometric variables. Fin width dominated
thermal resistance at roughly 71 % of the total variance, while microchannel width
dominated both entropy generation (about 75 %) and pumping power (about 52 %).
Crucially, the total-effect indices summed to well above 100 % for every objective —
a direct quantitative confirmation of the interaction effects that Tang et al.
observed geometrically.

Sikirica et al. [6] instead used **SHAP (SHapley Additive exPlanations)** on their
trained networks and found channel width and the number of secondary channels or
ribs to control thermal resistance, with pumping power set principally by channel
width and channel count. SHAP is much cheaper here because it interrogates the
surrogate rather than requiring the thousands of extra model evaluations a Sobol
estimate needs.

---

## 3. Constructing the training set

Sampling strategy is where the reviewed studies differ most, and where the weakest
reporting is found. Latin hypercube sampling (LHS) is the common choice because it
stratifies each variable independently and keeps inter-variable correlation low, but
the budgets vary by more than an order of magnitude.

| Study | Design variables | Sampling | Data points | Objectives |
|---|---|---|---|---|
| Xia et al. [8] | Wch, Hch, Wf, Lin, Hm (5) | LHS, differentiated step sizes | 200 train + 500 validation | Rt, pumping power, entropy generation |
| Sikirica et al. [6] | channel count, rib/secondary-channel count, widths, angle (6–7) | LHS + IQR outlier filter, 80/20 split | 1500 per design | Rt, pumping power |
| Li et al. [9] | αw,in, β, ε (3, after screening 6) | Structured full factorial (5×4×5) | 100 | Tmax, ΔP |
| Tang et al. [7] | α, β (2) | Exhaustive grid, 9 levels each | 99 (no surrogate used) | Rt at fixed pumping power |
| Meher et al. [10] | channel count, depth, fin angle, fin width (4) | One-factor-at-a-time sweeps | ~20 (no surrogate used) | Rt, ΔP, PEC, uniformity |

*The two studies that do not fit a surrogate are included because they define the
geometric parameterisation used later in this project.*

**Only Sikirica et al. justify the budget rather than asserting it.** Their learning
curves show the cross-validated coefficient of determination exceeding 0.95 once the
sample size passes roughly 1000, and degrading sharply below 500 — which implies
that several published surrogates trained on a hundred or two points are being
validated on a test split drawn from the same small, possibly unrepresentative
sample.

Any study that reports R² without a learning curve is reporting how well the model
interpolates its own sampling density, not how well it covers the design space.
Reporting a learning curve should be treated as mandatory.

---

## 4. Surrogate model families and reported accuracy

Three families dominate.

**Back-propagation neural networks** are the most common. Xia et al. [8] fitted a
5-17-5-3 architecture with Bayesian regularisation and input normalisation, mapping
five geometric inputs directly onto three objectives, and reported R² above 0.99 for
all three with mean absolute percentage errors of 0.79 %, 4.87 % and 0.82 % for
thermal resistance, pumping power and entropy generation respectively. The pumping
power error being six times the others is worth noting and recurs elsewhere:
pressure drop is the harder target because it responds to local geometry more sharply
than the temperature field does.

**Gradient-boosted trees** are the second family. Li et al. [9] used XGBoost, chosen
for its regularisation and its tolerance of small datasets, and obtained R² of 0.986
for maximum temperature and 0.996 for pressure drop from only 100 training
conditions, with worst-case relative errors of 0.29 % and 7.24 %. Again the hydraulic
target is the noisier one.

The third contribution is **comparative** rather than a single family. Sikirica et
al. [6] benchmarked random forests, CatBoost, LightGBM and a tuned neural network on
identical data. The ensemble methods gave respectable results with little effort —
CatBoost reached about 4.6 % MAPE untuned in relative terms — but the network, with
its architecture, batch size, learning rate, activation and initialiser all tuned by
a metaheuristic search, was clearly best at roughly 1.4 % MAPE and R² of 0.983.

The practical conclusion is that tree ensembles are the right starting point because
they are nearly hyperparameter-free, but a properly tuned network will beat them if
the dataset is large enough to support the tuning.

| Study | Surrogate | Reported accuracy | Reported benefit |
|---|---|---|---|
| Xia et al. [8] | BP-ANN, 5-17-5-3, Bayesian regularisation | R² > 0.99; MAPE 0.79 / 4.87 / 0.82 % | Pareto solutions verified against CFD within ~4 % |
| Sikirica et al. [6] | Tuned ANN (vs RF, CatBoost, LightGBM) | MAPE 1.41–1.93 %; R² 0.980–0.983 | Optimisation completed in about one-fifth of the CFD-loop time |
| Li et al. [9] | XGBoost regression | R² 0.986 (Tmax), 0.996 (ΔP) | Compromise design verified by CFD within 3 % |

*Hydraulic objectives are consistently harder to fit than thermal ones.*

---

## 5. From surrogate to design: optimisation and decision making

Every reviewed study that fits a surrogate then drives it with **NSGA-II**, and the
settings converge on a narrow band: population sizes between 100 and 400, crossover
probability 0.8, mutation between 0.05 and 0.2, and 100 to 200 generations. The
algorithm is chosen for the usual reasons — it needs no gradients, parallelises
trivially and maintains diversity through non-dominated sorting with crowding
distance — but the deeper reason it suits this problem is that thermal resistance and
pumping power are genuinely conflicting, so a single scalarised objective would hide
the trade-off the designer actually needs to see.

The Pareto front is not itself an answer, and all three surrogate studies resolve this
the same way: **entropy-weighted TOPSIS**. The entropy method assigns each objective a
weight from the information content of its own spread across the front, so a metric
that barely varies gets little say, and TOPSIS then ranks candidates by relative
closeness to the ideal point. Li et al. [9] used this to select αw,in = 0.40,
β = 0.63, ε = 0.52, and the resulting design cut the peak substrate temperature from
337.8 K to 320.0 K and the substrate temperature difference from 17.5 K to 3.5 K for
only about 3.5 kPa of additional pressure drop — an unusually favourable trade because
tapering redistributes flow rather than simply forcing more of it.

Xia et al. [8] add a third objective, entropy generation, and the comparison with
their two-objective run is instructive. The bi-objective scheme cut thermal resistance
by 44.1 % for a 13.3 % pumping power penalty; the tri-objective scheme achieved 46.7 %
together with a 25.2 % reduction in entropy generation, and produced the more uniform
temperature field. Their single-objective runs, by contrast, showed exactly the
pathology that motivates multi-objective work — minimising thermal resistance alone
raised pumping power by roughly twenty times.

The last step in all these workflows is re-simulating the selected Pareto points in
CFD; reported surrogate-to-CFD deviations sit below 3–5 %, and this verification
should be regarded as part of the method rather than an optional check.

---

## 6. Gaps identified, and how this project is positioned

Reading these studies together, five gaps stand out and each one shapes a decision in
the present work.

**Operating conditions are usually frozen.** Almost every surrogate in this literature
is trained at a single flow rate or a single pumping power, so it can answer geometry
questions but not the more practical question of how a given geometry behaves across a
pump curve. Both Tang et al. [7] and Meher et al. [10] show that the ranking of
geometries changes with flow rate. The surrogate built here therefore takes volumetric
flow rate as a model input alongside the geometric ratios, so the trained model can be
queried at matched flow rate *or* at matched pumping power without retraining.

**Sample size is asserted rather than demonstrated.** A learning curve of
cross-validated R² against training-set size is generated before any optimisation is
run, following [6], and the dataset expanded by infill sampling until the curve
flattens.

**Model choice is rarely justified by comparison.** Several regressors are fitted on
identical splits and compared on the same metrics rather than one being adopted by
default.

**Interaction is real and one-factor-at-a-time screening is unsafe.** Tang et al. found
the optimum at a corner of the design space that neither single-parameter sweep pointed
to, and Xia et al. quantified the same effect with total sensitivity indices above
unity. The sampling plan used here is a structured multi-level design that varies all
factors simultaneously, and interaction terms are examined explicitly.

**Interpretability is treated as an afterthought.** SHAP attribution is reported
alongside the Pareto front rather than as a separate curiosity, because the practical
output of a design study is a set of usable design rules, not only a single optimised
geometry.

The case study adopted is the diverging/converging manifold microchannel unit cell of
Tang et al. [7], chosen because its geometry is fully specified in the open literature,
its baseline results give an independent check on the CFD setup, and its two-parameter
design space is small enough that the interaction structure can be verified against a
published exhaustive sweep before the method is extended to a larger parameter set.

---

## 7. Summary

The manifold microchannel heat sink has a design space rich enough to reward
optimisation and expensive enough to make direct CFD-in-the-loop optimisation
impractical. The literature has converged on a stable four-stage recipe: sample the
design space with a space-filling or structured plan, evaluate the samples by conjugate
CFD, fit a regression surrogate, and drive that surrogate with NSGA-II before selecting
a compromise design by entropy-weighted TOPSIS and re-verifying it in CFD. Reported
accuracies are high and the computational saving is roughly fivefold.

What is less settled is the evidence base underneath those accuracies — sample sizes
are often unjustified, model families are seldom compared, and operating conditions are
usually excluded from the input vector. Those are the aspects this project sets out to
handle more carefully, on a geometry whose published results provide an independent
benchmark.

---

## References

[1] D. B. Tuckerman and R. F. W. Pease, "High-performance heat sinking for VLSI,"
*IEEE Electron Device Letters*, vol. 2, no. 5, pp. 126–129, 1981.

[2] G. M. Harpole and J. E. Eninger, "Micro-channel heat exchanger optimization,"
*Proc. 7th IEEE Semiconductor Thermal Measurement and Management Symposium*,
pp. 59–63, 1991.

[3] J. H. Ryu, D. H. Choi and S. J. Kim, "Three-dimensional numerical optimization of
a manifold microchannel heat sink," *International Journal of Heat and Mass Transfer*,
vol. 46, pp. 1553–1562, 2003.

[4] R. van Erp, R. Soleimanzadeh, L. Nela, G. Kampitsis and E. Matioli, "Co-designing
electronics with microfluidics for more sustainable cooling," *Nature*, vol. 585,
pp. 211–216, 2020.

[5] K. P. Drummond et al., "A hierarchical manifold microchannel heat sink array for
high-heat-flux two-phase cooling of electronics," *International Journal of Heat and
Mass Transfer*, vol. 117, pp. 319–330, 2018.

[6] A. Sikirica, L. Grbčić and L. Kranjčević, "Machine learning based surrogate models
for microchannel heat sink optimization," arXiv:2208.09683v2 [physics.flu-dyn], 2022.

[7] K. Tang, G. Lin, Y. Guo, J. Huang, H. Zhang and J. Miao, "Simulation and
optimization of thermal performance in diverging/converging manifold microchannel heat
sink," *International Journal of Heat and Mass Transfer*, vol. 200, art. 123495, 2023.

[8] Y. Xia, M. Wu, L. Lin, C. Wang, Z. Zhang, H. Cui, Y. Chen and Q. Zhang,
"Multi-objective optimization of a diamond-based U-type counter-flow manifold
microchannel heat sink using neural network-assisted NSGA-II algorithm," *Applied
Thermal Engineering*, vol. 279, art. 127482, 2025.

[9] J.-B. Li, T.-Y. Zhang, Z.-D. Li, L. Chen and W.-Q. Tao, "Multi-objective parameter
optimization design of tapered-type manifold/variable cross-section microchannel heat
sink," *Applied Thermal Engineering*, vol. 251, art. 123587, 2024.

[10] A. K. Meher, A. Agrawal and S. K. Saha, "Constrained design of converging–diverging
multi-microchannel heat sinks for high heat flux loading," *Applied Thermal
Engineering*, vol. 293, art. 130472, 2026.

[11] K. Deb, A. Pratap, S. Agarwal and T. Meyarivan, "A fast and elitist multiobjective
genetic algorithm: NSGA-II," *IEEE Transactions on Evolutionary Computation*, vol. 6,
no. 2, pp. 182–197, 2002.

[12] S. M. Lundberg and S.-I. Lee, "A unified approach to interpreting model
predictions," *Advances in Neural Information Processing Systems*, vol. 30, 2017.

[13] M. E. Polat and S. Cadirci, "Artificial neural network model and multi-objective
optimization of microchannel heat sinks with diamond-shaped pin fins," *International
Journal of Heat and Mass Transfer*, vol. 194, art. 123015, 2022.

[14] Z. Wang, M. Li, F. Ren, B. Ma, H. Yang and Y. Zhu, "Sobol sensitivity analysis and
multi-objective optimization of manifold microchannel heat sink considering entropy
generation minimization," *International Journal of Heat and Mass Transfer*, vol. 208,
art. 124046, 2023.

[15] Y.-H. Pan, R. Zhao, X.-H. Fan, Y.-L. Nian and W.-L. Cheng, "Study on the effect of
varying channel aspect ratio on heat transfer performance of manifold microchannel heat
sink," *International Journal of Heat and Mass Transfer*, vol. 163, art. 120461, 2020.
