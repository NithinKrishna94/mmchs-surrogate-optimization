# Validation

Checks on the numerical setup, in the order they are carried out.

Procedure described in [`docs/methodology.md`](../docs/methodology.md), section 7.

---

## 1. Reproduction of the baseline (non-manifold) cases

Carried out **before** the manifold is added. Because the source study is
experimentally validated, agreement here means the present model inherits that
validation rather than needing a separate experimental campaign.

Reference values: [`data/baseline_cd_mchs.csv`](../data/baseline_cd_mchs.csv).

### Baseline configuration

27 channels, 2° fin angle, 0.8 mm depth, 0.24 mm fin width, at 200 W/cm²,
0.0035 kg/s and 300 K inlet.

| Quantity | Reported | Present model | Deviation |
|---|---|---|---|
| Pressure drop (Pa) | 382 | | |
| Max substrate temperature (K) | 364 | | |
| Outlet fluid temperature (K) | 349 / 347 * | | |
| Flow uniformity factor σm (%) | 2.12 | | |
| Temperature uniformity factor σtemp (°C) | 0.85 | | |

\* The source study reports two slightly different values for this case across two
sweeps — see [`data/README.md`](../data/README.md).

### Across the sweeps

| Sweep | Cases re-run | Worst deviation in ΔP | Worst deviation in Ts,max |
|---|---|---|---|
| Channel count (19–35) | | | |
| Fin angle (1–4°) | | | |
| Channel depth (0.5–0.9 mm) | | | |
| Fin width (0.20–0.30 mm) | | | |

### Trend reproduction

Absolute agreement matters, but so does reproducing the shape of the response, since
the surrogate is fitted to that shape.

| Expected behaviour | Reproduced? | Notes |
|---|---|---|
| Pressure drop falls as channel count rises | | |
| Pressure drop falls as channel depth rises | | |
| Substrate temperature and uniformity both improve with channel count, with diminishing returns past 27 | | |
| Substrate temperature is minimised near a 2° fin angle, rising for larger angles | | |
| Pressure drop rises sharply for fin widths above 0.27 mm | | |

---

## 2. Energy balance

Coolant enthalpy rise between inlet and outlet compared against the applied heat
load, confirming the conjugate interface and heat flux boundary are handled
consistently. Applied to every run, manifold or not.

**Acceptance criterion:** closure within 1 %

**Worst-case deviation:** _(to be filled in)_

---

## 3. Grid independence — manifold configuration

The manifold introduces flow features the baseline mesh study never covered: the
slot-to-channel transition, the impingement region under each inlet slot, and the
turning losses at the outlet. Grid independence therefore has to be **re-established
on the manifold geometry**, not inherited from the baseline.

| Grid | Cells | Ts,max (K) | ΔP (Pa) | Change in Ts,max (%) | Change in ΔP (%) |
|---|---|---|---|---|---|
| | | | | — | — |
| | | | | | |
| | | | | | |
| | | | | | |

**Grid adopted:** _(to be filled in)_

**Reasoning:** _(state the change between the adopted grid and the next finer one)_

Because the design variables alter the local geometry, the same meshing controls
rather than the same absolute cell count are carried across the design space.

---

## 4. Domain validity for the manifold configuration

An open question rather than a completed check. The baseline study simulates the
full channel bank. For the manifold configuration, a periodic unit cell would be far
cheaper — but only if flow distribution along the bank is uniform enough for
periodicity to hold. Manifold designs are known to be prone to maldistribution,
particularly in Z-type arrangements.

**Test:** compare a full-bank simulation against a periodic unit cell at the
baseline manifold design, on both ΔP and Ts,max.

**Outcome:** _(to be filled in)_

If the unit cell is not valid, the run budget for the DOE has to be sized against
full-bank simulations, which affects the sampling decisions in
[`docs/methodology.md`](../docs/methodology.md), section 4.

---

## Status

| Check | Status |
|---|---|
| Baseline reproduction | In progress |
| Energy balance | Not started |
| Grid independence (manifold) | Not started |
| Unit cell vs full bank | Not started |
