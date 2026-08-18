# Data

## `baseline_cd_mchs.csv`

Nineteen reference cases for a converging–diverging multi-microchannel heat sink
**without a manifold**, covering four single-parameter sweeps plus one
uniform-channel reference.

These come from the parametric study of **Meher, Agrawal and Saha**, *Applied
Thermal Engineering* **293** (2026), art. 130472, supplied by the corresponding
author (Prof. S. K. Saha, Department of Mechanical Engineering, IIT Bombay).

---

## What this data is for

This project adds a **manifold layer** above the channel bank. The cases here
describe the geometry *before* that addition, so they are used in three ways:

**Validating the CFD setup.** The baseline study is experimentally validated.
Reproducing its cases means the numerical model used here inherits that validation
rather than needing a separate test rig.

**Fixing the starting point.** Its optimum — 27 channels, 2° fin angle, 0.8 mm
depth, 0.24 mm fin width — is the channel geometry the manifold is first built on.

**Setting the benchmark.** The manifold design has to beat these numbers at the same
heat flux, flow rate and footprint, or the added fabrication complexity is not
justified.

## What this data is not for

**It is not the surrogate training set.** These cases describe a single 6.35 mm
flow pass. A manifold splits that into several short passes, so entrance effects
dominate where fully-developed flow used to, and the response surface is a different
one. Training on this data and predicting manifold performance would be
extrapolation across a change in flow regime, not interpolation.

The training set comes from the designed experiment on the manifold configuration,
described in [`docs/methodology.md`](../docs/methodology.md), section 4.

---

## Fixed conditions

Held constant across every case in the file, and carried forward into the manifold
study so that the comparison stays like-for-like:

| Quantity | Value |
|---|---|
| Applied heat flux | 200 W/cm² on the bottom wall |
| Coolant mass flow rate | 0.0035 kg/s |
| Inlet temperature | 300 K |
| Channel length | 6.35 mm |
| Footprint | 6.35 × 17.5 mm² |
| Solid thermal conductivity | 380 W/m·K |
| Solid-to-fluid thickness ratio | 2.25 |
| Coolant | Water, single phase, temperature-dependent viscosity |
| Flow regime | Laminar, Re(max) = 1485 |

Baseline configuration: **27 channels, 2° fin angle, 0.8 mm depth, 0.24 mm fin
width.**

---

## Columns

| Column | Units | Description |
|---|---|---|
| `case_id` | — | Short identifier |
| `sweep` | — | Which single-parameter sweep the case belongs to |
| `N` | — | Number of channels |
| `fin_angle_deg` | ° | Converging–diverging fin angle |
| `depth_mm` | mm | Channel depth |
| `fin_width_mm` | mm | Fin width (blank where not reported for that case) |
| `delta_p_Pa` | Pa | Pressure drop across the microchannel domain |
| `Ts_max_K` | K | Maximum substrate temperature near the outlet |
| `Tf_out_K` | K | Outlet fluid temperature |
| `sigma_m_pct` | % | Flow uniformity factor |
| `sigma_temp_C` | °C | Temperature uniformity factor |
| `PEC` | — | Performance evaluation criterion |
| `fin_effectiveness` | — | Overall fin effectiveness, εo |
| `fin_efficiency` | — | Overall fin efficiency, ηo |
| `pec_reference` | — | What PEC is normalised against for that case |

### The `pec_reference` column matters

PEC is not normalised consistently across the source study. In the channel-count,
depth and fin-width sweeps it is referenced to the uniform-channel design; in the
fin-angle sweep it is referenced to the 2° configuration. **PEC values are therefore
not comparable across sweeps without renormalising**, which is why the reference is
carried as an explicit column rather than assumed.

---

## Known discrepancies in the source data

The baseline case (27 channels, 2°, 0.8 mm, 0.24 mm) appears in three of the four
sweeps, which gives a free consistency check. Most quantities agree exactly — ΔP
382 Pa, Ts,max 364 K, σm 2.12 %, σtemp 0.85, PEC 1.00 — but two do not:

| Quantity | Depth sweep | Fin-width sweep |
|---|---|---|
| Outlet fluid temperature | 349 K | 347 K |
| Overall fin effectiveness | 4.85 | 4.81 |

Both differences are small and likely rounding or a slightly different extraction
plane. They are recorded here rather than silently reconciled, and the values in the
CSV are kept as reported in each sweep.

`N27`, `H08` and `A2`/`W024` describe the same physical case. Deduplicate before any
statistical use, or the baseline is weighted four times relative to every other
point.

---

## Structure of this dataset

Every case varies **one parameter at a time** from the baseline — 19 points lying on
four lines through a single centre in a four-dimensional space. That design shows
what each parameter does in isolation and nothing about how they interact.

Adding a manifold introduces further parameters on top of these, and the manifold
and channel parameters are expected to be coupled: shorter flow passes change the
balance that set the channel optimum in the first place. Neither the existing sweeps
nor an extension of them in the same style would resolve that. See
[`docs/methodology.md`](../docs/methodology.md), sections 2 and 3.

---

## Citation

> A. K. Meher, A. Agrawal and S. K. Saha, "Constrained design of converging–diverging
> multi-microchannel heat sinks for high heat flux loading," *Applied Thermal
> Engineering*, vol. 293, art. 130472, 2026.
> https://doi.org/10.1016/j.applthermaleng.2026.130472
