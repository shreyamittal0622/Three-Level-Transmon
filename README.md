<h1 align="center">Three-Level Transmon</h1>
<p align="center">
  <b>Pulse simulation, qubit calibration and drift management for a driven three-level transmon</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/python-3.9%2B-blue" alt="Python">
  <img src="https://img.shields.io/badge/numpy-scipy-013243" alt="NumPy/SciPy">
  <img src="https://img.shields.io/badge/tests-passing-brightgreen" alt="Tests">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License">
</p>

<p align="center">
  <img src="results/original_notebook/fig1_static_pi_pulse.png" width="520" alt="Gaussian pi-pulse populations">
</p>

---

## Table of Contents

1. [About the Project](#about-the-project)
2. [Features](#features)
3. [Getting Started](#getting-started)
4. [Usage](#usage)
5. [Project Structure](#project-structure)
6. [Methodology](#methodology)
7. [Results](#results)
8. [Known Limitations](#known-limitations)
9. [Future Work](#future-work)
10. [License](#license)

---

## About the Project

A transmon qubit is never perfectly a two-level system: the third level $|2\rangle$ causes **leakage**, and the qubit frequency and drive amplitude **drift** during the day. This project simulates a driven **three-level transmon** and studies how to keep a gate good under those effects. It has four stages:

| # | Stage | Question |
|---|---|---|
| 1 | Static π-pulse | How much error and leakage does a 40 ns Gaussian pulse with DRAG have? |
| 2 | Calibration scans | Can amplitude, frequency and DRAG be tuned from noisy shot data? |
| 3 | Adaptive Bayesian Ramsey | Can a hidden detuning be estimated efficiently with a grid posterior? |
| 4 | 24-hour drift | When should the qubit be recalibrated: never, on a schedule, or on a health check? |

The original analysis is a QuTiP notebook (`notebooks/3_level_transmon_original.ipynb`). This repository adds a NumPy re-implementation that reproduces the notebook's static result exactly and fixes the issues listed in [Known Limitations](#known-limitations).

## Features

- Three-level transmon with Gaussian + DRAG pulses (no QuTiP dependency)
- Leakage-aware gate error: state-based and average-fidelity formulas
- Amplitude, frequency and DRAG calibration scans with fit-based uncertainties
- Adaptive Bayesian Ramsey estimation on a detuning grid, with a simulated fixed-delay baseline
- 24-hour random-walk drift with three recalibration policies and a cost model
- One command regenerates every figure and number

## Getting Started

### Prerequisites

- Python 3.9 or newer
- `numpy`, `scipy`, `matplotlib` (and `pytest` for the tests)

### Installation

```bash
git clone https://github.com/shreyamittal0622/three-level-transmon-calibration.git
cd three-level-transmon-calibration
pip install -r requirements.txt
```

## Usage

```bash
python scripts/run_all.py   # regenerates results/reproduced/ (figures + summary.json)
pytest -q                   # sanity tests
```

Use the modules directly:

```python
from transmon.model import TransmonParams, static_pi_pulse

t, P = static_pi_pulse(TransmonParams())      # populations P0, P1, P2 vs time
print(P[-1])                                  # [0.04829 0.94617 0.00554]
```

## Project Structure

```
.
├── README.md
├── requirements.txt
├── LICENSE
├── src/transmon/
│   ├── model.py               Hamiltonian, Gaussian + DRAG pulse, propagator
│   ├── measurement.py         binomial shot noise
│   ├── calibration.py         amplitude / frequency / DRAG scans, error amplification
│   ├── bayesian.py            Ramsey model, grid posterior, adaptive delay rule
│   └── drift.py               random-walk drift, oracle error, policies P0-P2
├── scripts/run_all.py         reproduces all results and figures
├── tests/test_model.py
├── notebooks/
│   └── 3_level_transmon_original.ipynb
├── docs/original_report.pdf
└── results/
    ├── original_notebook/     figures + numbers from the original notebook
    └── reproduced/            summary.json + figures 01-07
```

## Methodology

**Units.** Time in ns; energies and Rabi rates are angular frequencies in rad/ns (the notebook calls them "GHz"), so a detuning Δ gives a Ramsey phase Δ·τ.
Parameters: α = −0.210, t_g = 40 ns, σ = t_g/4 = 10 ns, t₀ = 20 ns, θ = π.

### 1. Driven transmon

$$H(t)=\Delta(t)\,a^\dagger a+\frac{\alpha}{2}\,a^\dagger a^\dagger a a+\frac{g(t)}{2}\Big[\Omega_x(t)(a+a^\dagger)+\Omega_y(t)\,i(a^\dagger-a)\Big]\qquad(1)$$

Δ(t) is the drifting detuning, α the anharmonicity (it shifts |2⟩ so that 0↔1 can be driven without 1↔2), g(t) the drive gain, and Ω_x, Ω_y the drive quadratures. In the qubit subspace a+a† → σₓ and i(a†−a) → σ_y.

### 2. Gaussian pulse and DRAG

$$\Omega_x(t)=A\,e^{-(t-t_0)^2/2\sigma^2},\qquad \int\Omega_x\,dt=\theta\ \Rightarrow\ A=\frac{\theta}{\sigma\sqrt{2\pi}}\qquad(2a)$$

$$\Omega_y(t)=-\beta\,\dot\Omega_x(t)\qquad(2b)$$

For θ = π, A ≈ 0.1253. The derivative quadrature suppresses the 1↔2 transition. First-order theory gives |β| ≈ 1/|α| ≈ 4.76; the true optimum is found numerically, and its sign depends on convention.

### 3. Gate error including leakage

$$\bar F=\frac{\mathrm{Tr}(M_VM_V^\dagger)+|\mathrm{Tr}\,M_V|^2}{d(d+1)},\quad M_V=V^\dagger M,\ M=PUP,\ d=2,\qquad \epsilon=1-\bar F\qquad(3a)$$

M is the 2×2 qubit block of the 3×3 propagator U, so leakage counts as error automatically. State-based version used in the notebook (|0⟩ → |1⟩):

$$F=|\langle 1|\psi(t_g)\rangle|^2=P_1(t_g),\quad \epsilon=1-P_1(t_g),\quad \text{leakage}=P_2(t_g),\quad P_0+P_1+P_2=1\qquad(3b)$$

### 4. Amplitude calibration (error amplification)

$$\langle Z\rangle=-(-1)^n\sin(n\pi\eta)\qquad(4)$$

Each nominal π pulse over-rotates by η. The signal is linear in η and amplified n-fold.

### 5. Ramsey frequency measurement

$$P(0\mid\Delta,\tau)=\tfrac12\Big[1+e^{-\tau/T_2}\cos(2\pi\Delta\tau+\varphi_0)\Big]\qquad(5)$$

A single delay is unambiguous only for |Δ| < 1/(2τ₀) (aliasing); a ladder τ_k = 2^k τ₀ resolves precision and ambiguity. The notebook simulates the full 3-level sequence with two 20 ns Gaussian π/2 pulses and reads out P(|1⟩).

### 6. Cramér–Rao bound

$$\mathrm{Var}(\hat\Delta)\ \ge\ \frac{1}{(2\pi\tau V)^2\,N},\qquad V=e^{-\tau/T_2}\qquad(6)$$

Variance falls as 1/τ² but visibility V shrinks with τ, so the optimum is near τ ≈ T₂.

### 7. DRAG signature

Leakage is invisible to a binary readout, so indirect signatures such as the apparent P(1) after trains of identity pairs [X_π X_{−π}]ⁿ are used. The notebook scans P₂ directly, an optimistic idealisation.

### 8. Bayesian update

$$P(\Delta\mid\text{data})\ \propto\ P(\Delta)\binom{N}{k}\,p(\Delta,\tau_k)^k\,[1-p(\Delta,\tau_k)]^{N-k},\qquad \sum_i P(\Delta_i\mid\text{data})=1\qquad(8)$$

301-point grid on [−0.08, 0.08], uniform prior, N = 50 shots per step. p(Δ, τ) is the simulated P(|1⟩) of the Ramsey sequence for trial detuning Δ, which is the likelihood model.

### 9. Posterior moments and next-delay rule

$$\bar\Delta=\sum_i\Delta_iP(\Delta_i),\quad \sigma_\Delta=\sqrt{\sum_iP(\Delta_i)(\Delta_i-\bar\Delta)^2},\quad \tau_{\rm next}=\mathrm{clip}\!\left(\frac{1}{\sigma_\Delta},\,5\text{ ns},\,200\text{ ns}\right)\qquad(9)$$

Broad posterior → short, unambiguous delay; narrow posterior → long, sensitive delay.

### 10. Measurement model

$$k\sim\mathrm{Binomial}(S,p),\qquad \hat p=\frac{k}{S},\qquad \sigma_{\hat p}=\sqrt{\frac{\hat p(1-\hat p)}{S}}\qquad(10)$$

### 11. Drift model

$$\Delta(t_m)=\Delta(0)+\sum_{j=1}^{m}\xi_j,\qquad \xi_j\sim\mathcal N(0,\sigma_\xi^2),\quad m=0,\dots,1439\qquad(11)$$

Random walk, one step per minute; Δ(0) = 0.02, σ_ξ = 0.0002 per minute. Seed 42 is the notebook trace, seed 2026 the problem statement's.

### 12. Oracle gate error

$$\epsilon(t)=\epsilon_0+\tfrac12\big(\Delta(t)-\Delta_{\rm cal}\big)^2,\qquad \epsilon_0=10^{-3}\qquad(12)$$

### 13. Recalibration policies

| Policy | Rule |
|---|---|
| **P0** | Calibrate once at t = 0, never again |
| **P1** | Recalibrate every 120 min |
| **P2** | Health check every 5 min; recalibrate if the shift exceeds 0.004 |

Costs: 2 min per full calibration, 0.2 min per health check.

## Results

Two sets of results are given: **(A) the original notebook output**, and **(B) the corrected NumPy re-implementation**. Only the static pulse and the drift ordering from (A) are fully valid; the reasons are in [Known Limitations](#known-limitations). All numbers in (B) are in [`results/reproduced/summary.json`](results/reproduced/summary.json).

### At a glance

| Stage | Quantity | (A) Notebook | (B) Corrected |
|---|---|---|---|
| Static pulse | P₁ / P₂ (leakage) / ε | 0.94617 / 0.00554 / 0.05383 | identical |
| Calibration | π-amplitude | 0.1504 ± 0.0200 | 1.055 ± 0.012 × A |
| Calibration | Frequency | −0.1000 ± 0.0100 | drive offset 0.0062 ± 0.0009 |
| Calibration | DRAG β | 1.75 ± 0.05 | 3.94 ± 0.20 |
| Bayesian Ramsey | Estimate (true 0.025) | 0.0277 | 0.0245 (σ_Δ = 6.1×10⁻⁴) |
| Bayesian Ramsey | Shot saving vs fixed | −900 % (assumed baseline) | none: 250 vs 150 shots |
| Drift (seed 42) | Recalibrations P0 / P1 / P2 | 0 / 12 / 2 | 0 / 11 / 2 |

### 1. Static π-pulse

| P₀(t_g) | P₁(t_g) | P₂(t_g) | ε = 1 − P₁ | Peak leakage |
|---|---|---|---|---|
| 0.04829 | 0.94617 | 0.00554 | 0.05383 | 0.0784 at 24.75 ns |

<p align="center">
  <img src="results/original_notebook/fig1_static_pi_pulse.png" width="560" alt="Static pi-pulse (notebook)"><br>
  <em>Notebook output. The NumPy re-simulation gives identical numbers.</em>
</p>

|2⟩ is used transiently: leakage peaks mid-pulse (≈ 0.08 near 25 ns) and mostly returns. Of the 5.4 % error, 4.8 % is population left in |0⟩ and 0.55 % is in |2⟩.

### 2. Calibration scans

**(A) Notebook** (400 shots per point, hard-coded uncertainties; every scan point simulated the same Hamiltonian, so these are shot noise, not calibrated values):

| Routine | Scan | Rule | Value | Uncertainty | Shots |
|---|---|---|---|---|---|
| π-amplitude | 11 scales, 0.8–1.2 × A | max P₁ | 0.1504 | 0.0200 | 4,400 |
| Frequency | 11 detunings, −0.1 to 0.1 | min P₁ | −0.1000 | 0.0100 | 4,400 |
| DRAG β | 13 values, 0 to 3 | min P₂ | 1.75 | 0.0500 | 5,200 |

**(B) Corrected** (each point simulates its own pulse; parabola fit with shot-noise weights):

| Routine | Estimate (± fit) | Noise-free optimum | Shots |
|---|---|---|---|
| π-amplitude scale (× A) | 1.055 ± 0.012 | 1.038 | 8,400 |
| Drive-frequency offset | 0.0062 ± 0.0009 | 0.0060 | 8,400 |
| DRAG β (min P₂) | 3.94 ± 0.20 | 3.925 | 10,000 |

<p align="center">
  <img src="results/reproduced/figures/02_calibration_scans.png" width="900" alt="Corrected calibration scans"><br>
  <em>Corrected scans: shots with error bars, noise-free simulation (black), parabola fit (red).</em>
</p>

<p align="center">
  <img src="results/reproduced/figures/03_amplitude_error_amplification.png" width="380" alt="Error amplification"><br>
  <em>Eq. (4) for η = 0, 0.02, 0.05.</em>
</p>

- The hidden qubit offset is 0.025, but the frequency response peak is not there. The DRAG quadrature and drive-induced Stark shift move the pulse's resonance (effective detuning +0.019 for β = 4.76, −0.026 for β = 0), so the scan finds the best *drive offset for this pulse* (≈ 0.006). Ramsey, which uses no DRAG, recovers the bare offset.
- The leakage-minimising β ≈ 3.9 differs from 1/|α| = 4.76 and from the P₁-maximising value (≈ 2.9).
- Applying all three estimates together gives ε ≈ 0.015 (leakage ≈ 1.1 %), versus 0.054 for the nominal pulse at zero detuning.

### 3. Adaptive Bayesian Ramsey

True detuning 0.025; 15 steps × 50 shots = 750 shots.

**(A) Notebook:** final estimate 0.0277. The update sat outside the loop, so only one posterior update was recorded: τ stayed at 21.58 ns, the width stayed at 0.0463, and the fixed baseline was an assumed 0.04/√N curve (hence the meaningless −900 %).

<p align="center">
  <img src="results/original_notebook/fig2_bayesian_ramsey_original.png" width="760" alt="Bayesian Ramsey (notebook)"><br>
  <em>Notebook output.</em>
</p>

**(B) Corrected** (update inside the loop, simulated fixed-delay baseline with 10 delays from 10 to 200 ns):

<p align="center">
  <img src="results/reproduced/figures/04_bayesian_ramsey.png" width="760" alt="Bayesian Ramsey (corrected)"><br>
  <em>(a) posterior narrowing, (b) adaptive delays, (c) posterior std vs shots, (d) shots to reach σ<sub>Δ</sub> ≤ 0.005.</em>
</p>

Single run (seed 42): MAP = **0.0245**, σ_Δ = 6.1×10⁻⁴. Delays grow 21.6 → 26.5 → 27.6 → 69.0 → 77.3 → 200 ns (clipped).

Over 60 random seeds (750 shots each):

| Metric | Adaptive | Fixed-delay |
|---|---|---|
| RMSE of posterior mean | 4.7×10⁻⁴ | 3.6×10⁻⁴ |
| Median shots to σ_Δ ≤ 0.005 | 250 | 150 |
| Runs reaching σ_Δ ≤ 0.005 | 98 % | 100 % |

<p align="center">
  <img src="results/reproduced/figures/05_bayesian_multiseed.png" width="480" alt="Multi-seed comparison"><br>
  <em>RMSE versus total shots, 60 seeds.</em>
</p>

In this model the heuristic rule shows **no advantage** over a fixed ladder: there is no T₂ decay, so long delays are always better, and the rule needs a few steps to ramp up.

### 4. 24-hour drift and recalibration policies

**(A) Notebook** (seed 42): recalibrations P0 = 0, P1 = 12, P2 = 2. P0 stays on the 0.0010 floor for ~15 h then climbs to ≈ 0.00106; P1 resets at each scheduled recalibration; P2 resets only twice (near 15 h and 19 h). The P1 count should be 11: resets occur at 120 … 1320 min.

<p align="center">
  <img src="results/original_notebook/fig3_drift_policies_original.png" width="760" alt="Drift policies (notebook)"><br>
  <em>Notebook output, seed 42.</em>
</p>

**(B) Corrected** (actual event counts, stated costs applied):

| Seed | Policy | Recal. | Health checks | Downtime (min) | Mean ε | Max ε |
|---|---|---|---|---|---|---|
| 42 | P0 | 0 | 0 | 0 | 1.0131e-3 | 1.0647e-3 |
| 42 | P1 | 11 | 0 | 22.0 | 1.0015e-3 | 1.0145e-3 |
| 42 | P2 | 2 | 287 | 61.4 | 1.0011e-3 | 1.0092e-3 |
| 2026 | P0 | 0 | 0 | 0 | 1.0563e-3 | 1.1531e-3 |
| 2026 | P1 | 11 | 0 | 22.0 | 1.0014e-3 | 1.0163e-3 |
| 2026 | P2 | 5 | 287 | 67.4 | 1.0020e-3 | 1.0093e-3 |

<p align="center">
  <img src="results/reproduced/figures/06_drift_policies_seed42_notebook.png" width="760" alt="Drift, seed 42"><br>
  <em>Corrected run, seed 42 (▼ marks recalibrations).</em>
</p>

<details>
<summary>Seed 2026 trace and 200-day Monte Carlo</summary>

<p align="center">
  <img src="results/reproduced/figures/06_drift_policies_seed2026_problem_statement.png" width="720" alt="Drift, seed 2026">
</p>

"Realistic" assumes health-check noise σ = 0.001 and residual calibration error σ = 0.0005 (illustrative assumptions, not from the problem statement).

| Model | Policy | Recal. | Downtime (min/day) | Mean ε |
|---|---|---|---|---|
| ideal | P0 | 0 | 0 | 1.01446e-3 |
| ideal | P1 | 11 | 22.0 | 1.00122e-3 |
| ideal | P2 | 3.2 | 63.8 | 1.00148e-3 |
| realistic | P1 | 11 | 22.0 | 1.00133e-3 |
| realistic | P2 | 6.5 | 70.4 | 1.00101e-3 |

<p align="center">
  <img src="results/reproduced/figures/07_drift_monte_carlo.png" width="700" alt="Monte Carlo">
</p>

</details>

Both active policies remove the late-day error growth seen in P0. P2 needs far fewer full calibrations than P1, but its 287 health checks cost ≈ 57 min against 22 min for all of P1's calibrations, and the error difference is ~10⁻⁶. With these costs and this small drift rate, **P1 is the better trade-off**.

## Known Limitations

Issues in the original notebook, and how this repository handles them:

| Issue in notebook | Effect | Fix in `src/` |
|---|---|---|
| Scans called `mesolve` with the base Hamiltonian | All points identical; reported values were shot noise | Each point simulates its own pulse |
| Hard-coded uncertainties | Error bars unrelated to data | Weighted parabola-fit covariance |
| Frequency scan took the minimum of P₁ | Picked the scan edge | Fit the response peak; hidden offset added |
| Bayesian update outside the 15-step loop | One update, constant τ, flat σ | Measurement, likelihood, update inside the loop |
| Fixed baseline was 0.04/√N | −900 % saving meaningless | Simulated baseline, 60 seeds |
| P1 count `1440 // 120 = 12` | Resets occur at 11 times | Count actual events |
| Costs and noisy checks ignored | Optimistic comparison | Downtime reported; noisy Monte Carlo |

Remaining limitations of the model:

- Closed-system dynamics: no T₁/T₂ decay, so the Cramér–Rao optimum τ ≈ T₂ never appears.
- The DRAG scan reads P₂ directly, which a binary readout cannot do.
- The frequency scan gives the best drive offset for the pulse, not the bare qubit frequency.
- Eq. (12) is a stand-in; amplitude drift g(t) is not simulated.
- The "realistic" noise levels are assumptions.

## Future Work

- Add T₁/T₂ decay and re-test the adaptive delay rule with a T₂-aware or information-gain criterion
- Replace the direct P₂ readout with the [X_π X_{−π}]ⁿ signature
- Iterate frequency and DRAG calibration jointly
- Simulate gain drift g(t) and a measured (noisy) health-check statistic

## License

Distributed under the MIT License. See [`LICENSE`](LICENSE).
