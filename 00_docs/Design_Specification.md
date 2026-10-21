# CS Amplifier (Resistive Load) — Design Specification

**Technology :** SKY130
**Supply Voltage (VDD) :** 1.8 V

**Topology :** Common Source Amplifier
**Load :** Resistive Load

**Target Midband Gain :** ≥ 3 V/V (≈ 9.5 dB)
**Target Bandwidth :** ≥ 10 GHz

**Power Consumption :** As low as practical (within bias stability)

**Region of Operation :**
NMOS must remain in saturation at quiescent point

**Output Swing :**
Maximum symmetrical swing around Q-point without distortion (≤ 1% THD preferred)

**Key Design Trade-off :**
Gain–Bandwidth tradeoff must be optimized, not maximized individually

**Verification Flow :**
- DC Operating Point (OP)
- DC Sweep (Vout vs Vin)
- AC Analysis (Gain, BW, phase margin trend)
- Transient Analysis (sine + step response)
- W/L Optimization sweep
- Temperature Sweep (-40°C to 125°C)
- Supply Sweep (1.62 V / 1.8 V / 1.98 V)
- Process Corner Analysis (tt, ff, ss, fs, sf if available)
- Post-layout Parasitic Extraction (PEX)
- Layout Design
- DRC
- LVS
- Post-layout Simulation (AC + Transient)
