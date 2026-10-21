# Three-Level-Transmon

## 1. Driven Transmon with Frequency and Gain Drift

In the frame rotating at the nominal drive frequency $`\omega_d`$, truncated to three levels, the driven transmon Hamiltonian is

```math
H(t) = \Delta(t)\, a^\dagger a + \frac{\alpha}{2}\, a^\dagger a^\dagger a a + \frac{g(t)}{2}\Big[\Omega_x(t)\,(a + a^\dagger) + \Omega_y(t)\, i(a^\dagger - a)\Big] \qquad (1)
```

where $`\Delta(t) = \omega_q(t) - \omega_d`$ is the drifting detuning, $`\alpha`$ is the anharmonicity, $`\Omega_{x,y}`$ are the programmed drive quadratures and $`g(t)`$ is the drifting drive gain. Within the qubit subspace, $`a + a^\dagger \to \sigma_x`$ and $`i(a^\dagger - a) \to \sigma_y`$.

**Gaussian pulse and DRAG.** A Gaussian pulse of duration $`t_g`$ and width $`\sigma = t_g/4`$ has $`\Omega_x(t) = A\, e^{-(t-t_0)^2/2\sigma^2}`$, with $`A`$ fixed by $`\int \Omega_x\, dt = \theta`$ for a rotation by angle $`\theta`$. The DRAG correction adds a derivative term on the orthogonal quadrature,

```math
\Omega_y(t) = -\beta\, \dot{\Omega}_x(t) \qquad (2)
```

which suppresses the $`|1\rangle \to |2\rangle`$ transition. First-order theory gives $`|\beta| \approx 1/|\alpha|`$; the true optimum depends on pulse duration and amplitude and must be determined empirically. The sign of $`\beta_{\mathrm{opt}}`$ depends on convention and should be established from your own simulator.

## 2. Gate Error Including Leakage

Let $U$ be the $3 \times 3$ propagator and $M = PUP$ its $2 \times 2$ block on the qubit subspace (non-unitary when there is leakage). For a target unitary $V$:

$$\bar{F} = \frac{\mathrm{Tr}\!\left(M_V M_V^\dagger\right) + \left|\mathrm{Tr}\, M_V\right|^2}{d(d+1)}, \qquad M_V = V^\dagger M, \quad d = 2 \qquad (3)$$
$$\epsilon = 1 - \bar{F}$$

Leakage out of the subspace counts as error automatically. $\epsilon$ is the oracle score of a policy and is not available to the calibration agent.

State-based version used in the notebook

For the state $|0\rangle \to |1\rangle$ with target $|1\rangle$:

$$F = |\langle 1 | \psi(t_g) \rangle|^2 = P_1(t_g), \qquad \epsilon = 1 - P_1(t_g), \qquad \text{leakage} = P_2(t_g)$$
$$P_k(t) = |\langle k | \psi(t) \rangle|^2, \qquad P_0 + P_1 + P_2 = 1$$

## 3. Calibration Primitives
Amplitude (error amplification)

Each nominal $\pi$ pulse rotates by $\pi(1+\eta)$. Prepare $|0\rangle$, apply $R_X(\pi/2)$, then $n$ such pulses:

$$\langle Z \rangle = -(-1)^n \sin(n\pi\eta) \qquad (4)$$

This is linear in $\eta$ for small $\eta$ and amplified $n$-fold.

Frequency (Ramsey)

After $R_X(\pi/2) - \tau - R_X(\pi/2)$:

$$P(0 \mid \Delta, \tau) = \frac{1}{2}\Big[1 + e^{-\tau/T_2} \cos\!\left(2\pi\Delta\tau + \varphi_0\right)\Big] \qquad (5)$$
A single delay $\tau_0$ is unambiguous only for $|\Delta| < 1/(2\tau_0)$ (aliasing).
A ladder of delays $\tau_k = 2^k \tau_0$ resolves both precision and ambiguity, as in iterative phase estimation.

For $N$ shots at delay $\tau$ with visibility $V = e^{-\tau/T_2}$, the Cramér–Rao bound is

$$\mathrm{Var}(\hat{\Delta}) \ge \frac{1}{(2\pi\tau V)^2\, N} \qquad (6)$$

which is optimised near $\tau \approx T_2$.

DRAG coefficient

Leakage is invisible to a binary readout, so indirect signatures are used, for example the growth of the apparent $P(1)$ after long trains of nominally identity pairs:

$$\left[X_\pi X_{-\pi}\right]^n$$

## 4. Adaptive (Bayesian) Experiment Design

Posterior over detuning on a grid, updated after each batch of $N$ shots with $k$ successes at delay $\tau_k$:

$$P(\Delta \mid \text{data}) \;\propto\; P(\Delta)\, \binom{N}{k}\, p(\Delta, \tau)^k \left[1 - p(\Delta, \tau)\right]^{N-k}$$

Normalisation:

$$\sum_i P(\Delta_i \mid \text{data}) = 1$$

Posterior mean and standard deviation:

$$\bar{\Delta} = \sum_i \Delta_i\, P(\Delta_i), \qquad \sigma_\Delta = \sqrt{\sum_i P(\Delta_i)\,(\Delta_i - \bar{\Delta})^2}$$

Next-delay rule used in the notebook (clipped to a range):

$$\tau_{\text{next}} = \mathrm{clip}\!\left(\frac{1}{\sigma_\Delta},\; 5\ \text{ns},\; 200\ \text{ns}\right)$$

Notebook Ramsey pulses are two Gaussian $\pi/2$ pulses of 20 ns each:

$$A_{\pi/2} = \frac{\pi/2}{\sigma_r \sqrt{2\pi}}, \qquad \sigma_r = \frac{20}{4}\ \text{ns}$$


## 5. Measurement Model

Simulated binary readout with $S$ shots from a true population $p$:

$$k \sim \mathrm{Binomial}(S, p), \qquad \hat{p} = \frac{k}{S}, \qquad \sigma_{\hat{p}} = \sqrt{\frac{\hat{p}(1-\hat{p})}{S}}$$


## 6. Drift Model: the Simulated Day

Detuning as a random walk sampled once per minute (seed 2026 defines the official trace in the problem statement; the notebook used seed 42):

$$\Delta(t_m) = \Delta(0) + \sum_{j=1}^{m} \xi_j, \qquad \xi_j \sim \mathcal{N}(0, \sigma_\xi^2), \qquad m = 0, \dots, 1439$$

Notebook values: $\Delta(0) = 0.02$ GHz and $\sigma_\xi = 0.0002$ GHz per minute.

Oracle gate error model (notebook)
$$\epsilon(t) = \epsilon_0 + \frac{1}{2}\left(\Delta(t) - \Delta_{\text{cal}}\right)^2, \qquad \epsilon_0 = 0.001$$
Recalibration policies
Policy	Rule
P0	Calibrate once at $t=0$, never again
P1	Recalibrate every $T_{\text{int}}$ minutes ($T_{\text{int}} = 120$ in the notebook)
P2	Health check every $T_{\text{chk}}$ minutes; recalibrate if the test statistic exceeds a threshold
