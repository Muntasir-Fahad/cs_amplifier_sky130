# CS Amplifier (Resistive Load) — Design Specification

**Technology:** SkyWater SKY130 (sky130A PDK, TT corner for nominal)
**VDD:** 1.8 V
**Topology:** Common-Source amplifier with high-poly resistive load
**Author:** Muntasir Fahad
**Version:** 1.0 (practical, data-driven)
**Date:** 22-Oct-2026

---

## 1. Overview

Single-stage Common-Source amplifier with a high-poly resistor load,
implemented in SKY130. Targets moderate gain, wide bandwidth, and low
power for use as a first gain stage in an analog front-end.

All target values in this spec are **derived from simulation evidence**
(pre-layout OP + AC + DC sweeps on sky130 NMOS). Each parameter has:
- **Target** — what the design must hit
- **Acceptable min** — what is tolerated
- **Achieved (best candidate)** — measured in simulation for reference

---

## 2. Electrical Specifications

| Parameter | Symbol | Target | Acceptable min | Achieved (best) | Unit | Condition |
|-----------|--------|--------|----------------|-----------------|------|-----------|
| Supply voltage | VDD | 1.80 | 1.62–1.98 | 1.80 | V | ±10% |
| Midband gain | \|Av\| | ≥ 9.5 | ≥ 8.0 | **14.0** | dB | 1 MHz, CL = 10 fF |
| Midband gain (linear) | \|Av\| | ≥ 3.0 | ≥ 2.5 | **5.04** | V/V | 1 MHz, CL = 10 fF |
| Bandwidth (−3 dB) | BW | ≥ 10 | ≥ 8 | **13.1** | GHz | CL = 10 fF |
| Quiescent output | Vout,Q | 0.90 ± 0.10 | 0.80–1.00 | 0.90 (design target) | V | Vin = VBIAS_Q |
| Overdrive voltage | Vov | 0.15–0.25 | 0.10–0.30 | ~0.20 | V | NMOS in saturation |
| Supply current | IDD | ≤ 1.0 | ≤ 1.2 | **0.44** | mA | Q-point |
| Total power | Ptot | ≤ 1.8 | ≤ 2.2 | **0.80** | mW | VDD·IDD |
| Load resistor | R_L | 2.0 | 1.5–4.0 | 2.3–2.9 | kΩ | high-poly, ±20% |
| Load capacitance | CL | 10 | 5–20 | 10 | fF | testbench |
| THD | — | ≤ 1 | ≤ 2 | TBD | % | Vin,pp = 200 mV, 1 MHz |
| Output swing (linear) | — | ≥ ±400 | ≥ ±300 | TBD | mV | THD < 1% |

> Values marked "TBD" will be finalized in transient verification (§7).

---

## 3. Q-Point Targets

- NMOS biased in **saturation** with Vov ≈ 200 mV.
- Output node centered near VDD/2 for maximum symmetric swing:
  **`Vout,Q = 0.90 V ± 100 mV`**.
- Drain current **`ID ≤ 1.0 mA`** (power ≤ 1.8 mW).
- R_L chosen so that `ID · R_L ≈ VDD − Vout,Q ≈ 0.9 V`.
- With target ID and R_L, this gives Vout,Q ≈ 0.9 V automatically.

---

## 4. Candidate (W/L, R_L) Selection Rule

From the pre-layout sizing sweep (Vgs = 0.8 / 0.9 / 1.0 V, WN = 1..30,
L = 1..8), the acceptable candidates must satisfy:

1. `|Av| ≥ 3 V/V` (midband, CL = 10 fF)
2. `BW ≥ 10 GHz`
3. `ID ≤ 1.0 mA`
4. `Vout,Q ∈ [0.80, 1.00] V`
5. Saturation margin at Q-point: `VDS − Vdsat ≥ 100 mV` (small-signal
   linearity + robustness to Vth variation)

Candidates that pass all five are ranked by **FOM = |Av| · BW / Ptot**
(gain-bandwidth per unit power). The best candidate is locked for layout.

**Reference (pre-layout best, Vgs = 0.9 V):**
- **WN = 10, L = 4** → |Av| = 14.04 dB (5.04 V/V), BW = 13.09 GHz,
  ID = 442 µA, Ptot = 0.80 mW, PM ≈ 127°
- Alternative: WN = 9, L = 5 → |Av| = 14.3 dB, BW ≈ 12.3 GHz,
  ID = 388 µA, Ptot = 0.70 mW
- Swing-optimised alternative: WN = 9, L = 4 → |Av| = 13.6 dB,
  BW ≈ 14.2 GHz, Vout,Q ≈ 0.95 V

---

## 5. Verification Flow

Each step must pass before the next.

| # | Step | Deck location | Acceptance |
|---|------|--------------|------------|
| 1 | Operating point | `03_simulation/op` | Vout,Q 0.80–1.00 V, Vov 0.15–0.25 V, ID ≤ 1.0 mA |
| 2 | DC sweep | `03_simulation/dc` | Monotonic; linear region around Q; max-gain slope ≈ AC gain |
| 3 | AC (gain, BW) | `03_simulation/ac` | \|Av\| ≥ 3.0 V/V, BW ≥ 10 GHz |
| 4 | Transient (THD) | `03_simulation/tran` | THD ≤ 1% @ 200 mVpp, 1 MHz |
| 5 | W/L + R_L sweep | `04_sizing` | FOM = \|Av\|·BW/P maximised within constraints |
| 6 | Temperature sweep | `03_simulation/sweep/temp_sweep` | −40 to 125 °C, gain ≥ 2.5 V/V |
| 7 | Supply sweep | `03_simulation/sweep/vdd_sweep` | 1.62 / 1.80 / 1.98 V, gain ≥ 2.7 V/V |
| 8 | PVT corners | `03_simulation/corners` | All 5×3×3 = 45 corners, gain ≥ 2.5 V/V |
| 9 | Layout | `05_layout/magic` | Physical match to schematic |
| 10 | DRC | `06_drc` | Zero violations |
| 11 | LVS | `07_lvs` | MATCH, no property errors |
| 12 | PEX | `08_pex` | Parasitic netlist extracted |
| 13 | Post-layout sim | `08_pex/post_layout_results` | \|Av\| ≥ 2.7 V/V, BW ≥ 8 GHz |

---

## 6. PVT Corner Matrix

Full factorial: **5 process × 3 temperature × 3 voltage = 45 corners.**

- **Process:** tt, ff, ss, fs, sf
- **Temperature:** −40 °C, 27 °C, 125 °C
- **Supply:** 1.62 V, 1.80 V, 1.98 V

Per-corner pass criteria:
- NMOS in saturation (VDS > Vov)
- \|Av\| ≥ 2.5 V/V (midband, 1 MHz)
- BW ≥ 8 GHz (CL = 10 fF)
- ID ≤ 1.2 mA

---

## 7. Sizing Strategy

Two-dimensional sweep with power as hard constraint:

1. **Fix R_L ≈ 2.5 kΩ** (mid-range high-poly), sweep WN from 5 to 20
   (fine step near optimum), record |Av|, BW, ID, Vout,Q, gm, gds, ro.
2. **Fix best W/L** from step 1, sweep R_L from 1.5 kΩ to 4.0 kΩ,
   re-record the same metrics.
3. **Select** (W/L, R_L) maximising `FOM = |Av|·BW / Ptot` subject to:
   - |Av| ≥ 3 V/V
   - BW ≥ 10 GHz
   - ID ≤ 1.0 mA
   - Vout,Q ∈ [0.80, 1.00] V
   - Sat margin ≥ 100 mV

Sweep scripts live in `04_sizing/wl_sweep/` and `04_sizing/rd_sweep/`.

---

## 8. Layout Notes

- High-poly resistor with dummy ends for matching.
- NMOS in multi-finger layout to reduce gate resistance.
- Guard ring around NMOS (substrate noise).
- Symmetric placement of R_L and NMOS to minimize output parasitics.
- Top metal for output pad; keep output-node capacitance low.
- Aim to preserve (post-layout) ≥ 90% of pre-layout gain and ≥ 80% of
  pre-layout BW.

---

## 9. Post-Layout Acceptance

| Parameter | Pre-layout | Post-layout min | Max degradation |
|-----------|------------|-----------------|-----------------|
| Midband gain | ≥ 3.0 V/V | ≥ 2.7 V/V | ≤ 10% |
| Bandwidth | ≥ 10 GHz | ≥ 8 GHz | ≤ 20% |
| Vout,Q | 0.80–1.00 V | 0.75–1.05 V | ±50 mV extra |
| ID | ≤ 1.0 mA | ≤ 1.1 mA | ≤ 10% |

---

## 10. Tool Flow

| Tool | Role |
|------|------|
| xschem | Schematic capture, symbol generation |
| ngspice | OP / DC / AC / Tran / Sweep / Corner simulation |
| Magic | Layout, DRC, parasitic extraction |
| Netgen | LVS (layout vs schematic) |
| sky130A | PDK (TT/FF/SS/FS/SF models, standard cells) |

---

## 11. Deliverables

- [ ] Final schematic (`01_schematic/`)
- [ ] Sizing sweep report (`04_sizing/`)
- [ ] Clean DRC report (`06_drc/`)
- [ ] Clean LVS report (`07_lvs/`)
- [ ] PEX netlist + post-layout results (`08_pex/`)
- [ ] Final GDS (`09_gds/final_gds/`)
- [ ] README + this design spec

---

## 12. Revision History

| Date | Rev | Notes |
|------|-----|-------|
| 22-Oct-2026 | 1.0 | Initial practical spec: target / acceptable-min / achieved columns; data-driven from OP + AC + DC pre-layout simulations; candidate selection rule + FOM added |
