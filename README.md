# DMDc: Improved Algorithm Implementation

Implementation and analysis of the **Improved Dynamic Mode Decomposition with Control (DMDc)** algorithm proposed by Nedzhibov (2023), benchmarked against the standard DMDc (Proctor et al., 2016).

> **Reference:** Nedzhibov, G. *An Improved Approach for Implementing Dynamic Mode Decomposition with Control.* Computation, 2023, 11(10), 201. https://doi.org/10.3390/computation11100201

---

## What is DMDc?

Dynamic Mode Decomposition with Control (DMDc) is a data-driven method that identifies a linear model of a dynamical system from observations, explicitly accounting for external control inputs:

$$\mathbf{x}_{k+1} \approx A\,\mathbf{x}_k + B\,\mathbf{u}_k$$

Given snapshot matrices $X$, $X'$ (states) and $\Upsilon$ (control inputs), DMDc recovers operators $A$ and $B$ using SVD-based dimensionality reduction — **without any knowledge of the underlying equations**.

---

## What this project does

| Component | Description |
|-----------|-------------|
| `StandardDMDc` | Baseline implementation of Algorithm 1 (Proctor et al., 2016) |
| `ImprovedDMDc` | Implementation of Algorithm 3 (Nedzhibov, 2023) |
| Three test systems | Lotka–Volterra, Lorenz, Damped oscillator with sensors |
| Benchmarks | RMSE, Relative Error, computational speedup |

---

## Key idea of Algorithm 3

Standard DMDc requires two expensive SVDs — one of the augmented matrix $\Omega \in \mathbb{R}^{(n+l)\times m}$ and one of the output matrix $X' \in \mathbb{R}^{n\times m}$.

Algorithm 3 replaces the second SVD with a decomposition of a **much smaller** matrix $\tilde{H} = \tilde{\Sigma}^{-1}\tilde{U}_1^* \in \mathbb{R}^{p\times n}$, where $p \leq n+l \ll m$:

| | Algorithm 1 | Algorithm 3 |
|---|---|---|
| Second SVD | $X'$, size $n \times m$ | $\tilde{H}$, size $p \times n$ |
| Matrix multiplications for $\tilde{A}$ | 5 | 3 |
| Matrices to store | 6 | 4 |
| Results | baseline | **identical** ✓ |

Both algorithms produce the **same eigenvalues and DMD modes** (verified numerically to machine precision).

---

## Results

### Eigenvalue agreement (Standard vs Improved)

| System | max \|λ_std − λ_imp\| |
|--------|----------------------|
| Lotka–Volterra | `6.66e-16` — machine precision |
| Lorenz | `5.55e-16` — machine precision |
| Damped oscillator | `4.44e-16` — machine precision |

### RMSE comparison

**Lotka–Volterra** (nonlinear, $n=2$, $m=1000$):

| | Standard DMDc | Improved DMDc |
|---|---|---|
| RMSE($x_1$) | 142.45 | 142.45 |
| RMSE($x_2$) | 28.23 | 28.23 |

**Lorenz system** (chaotic, $n=3$, $m=10000$):

| | Standard DMDc | Improved DMDc |
|---|---|---|
| RMSE($x_1$) | 11.49 | 11.49 |
| RMSE($x_2$) | 13.05 | 13.05 |
| RMSE($x_3$) | 9.35 | 9.35 |

**Damped oscillator with 50 sensors** (linear, $n_{obs}=50$, $m=500$):

| | Standard DMDc | Improved DMDc |
|---|---|---|
| RMSE (mean over 50 sensors) | ≈ 0 | ≈ 0 |

> Linear system → exact recovery → RMSE at machine precision.

### Computational speedup

| System | $n$ | $m$ | Standard | Improved | Speedup |
|--------|-----|-----|----------|----------|---------|
| Lotka–Volterra | 2 | 1000 | 0.12 ms | 0.10 ms | **1.20×** |
| Lorenz | 3 | 10000 | 1.87 ms | 1.02 ms | **1.83×** |
| Oscillator (50 sensors) | 50 | 500 | 5.22 ms | 2.51 ms | **2.08×** |

Speedup grows with the observation dimension $n$ — the larger $n$, the more expensive the second SVD in Algorithm 1 and the bigger the advantage of Algorithm 3.

---

## Test systems

### 1. Lotka–Volterra (predator–prey)
Weakly nonlinear system with sinusoidal control $u(t) = (2\sin t\,\sin(t/10))^2$:

$$\dot{x}_1 = \tfrac{1}{2}x_1 - \tfrac{1}{40}x_1 x_2, \qquad \dot{x}_2 = -\tfrac{1}{2}x_2 + \tfrac{1}{200}x_1 x_2 + u(t)$$

### 2. Lorenz system (chaotic)
Chaotic system with random control $u \in [-50, 50]$ updated every 10 steps:

$$\dot{x}_1 = 10(x_2 - x_1) + u, \qquad \dot{x}_2 = x_1(28 - x_3) - x_2, \qquad \dot{x}_3 = x_1 x_2 - \tfrac{8}{3}x_3$$

### 3. Damped oscillator with 50 sensors (linear)
Mass-spring-damper system observed through 50 virtual sensors $\mathbf{y}_k = C\,\mathbf{x}_k \in \mathbb{R}^{50}$:

$$m\ddot{q} + c\dot{q} + kq = u(t), \quad m=1,\; k=1,\; c=0.1$$

DMDc recovers the true eigenvalues exactly: $\lambda_{1,2} = 0.9901 \pm 0.0992j$.

---

## Project structure

```
Data-Driven-project/
├── project.ipynb        # Main notebook: code, experiments, visualizations
├── requirements.txt     # Python dependencies
└── README.md            # This file
```

---

## Getting started

```bash
# Clone the repository
git clone https://github.com/rkonurin-ops/Data-Driven-project.git
cd Data-Driven-project

# Create and activate virtual environment
python3 -m venv dmdc_env
source dmdc_env/bin/activate       # Linux/macOS
# dmdc_env\Scripts\activate        # Windows

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook project.ipynb
```

---

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| numpy | ≥ 1.24 | SVD, matrix operations |
| scipy | ≥ 1.10 | ODE integration (`solve_ivp`), matrix exponential |
| matplotlib | ≥ 3.7 | Visualization |
| pydmd | ≥ 0.4 | Baseline comparison |
| pandas | ≥ 2.0 | Metrics tables |

---

## References

1. **Nedzhibov, G.** (2023). An Improved Approach for Implementing Dynamic Mode Decomposition with Control. *Computation*, 11(10), 201.
2. **Proctor, J.L., Brunton, S.L., Kutz, J.N.** (2016). Dynamic Mode Decomposition with Control. *SIAM Journal on Applied Dynamical Systems*, 15(1), 142–161.
3. **Schmid, P.J.** (2010). Dynamic mode decomposition of numerical and experimental data. *Journal of Fluid Mechanics*, 656, 5–28.
