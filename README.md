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

Let $`U`$ be the $`3 \times 3`$ propagator of a gate and $`M = PUP`$ its $`2 \times 2`$ block on the qubit subspace, which is non-unitary in the presence of leakage. For a target unitary $`V`$, the average gate fidelity of the subspace operation is
