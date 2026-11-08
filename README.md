# Data-Driven Identification of Bifurcation Normal Forms with SINDy

Using **SINDy** (Sparse Identification of Nonlinear Dynamics, via [PySINDy](https://github.com/dynamicslab/pysindy)) to recover the governing equations of dynamical systems from simulated trajectories, with a focus on parametric systems near bifurcations (Hopf and saddle-node normal forms).

Work done during a research internship at the University of Washington. The full write-up is in [`report/SINDy_Bifurcation.pdf`](report/SINDy_Bifurcation.pdf).

---

## The question

Can SINDy recover a bifurcation normal form when the bifurcation parameter μ is itself part of the data? The system studied is the 2-D Hopf normal form, with μ treated as a third state that drifts slowly at rate ε:

```
ẋ = μx + y − x(x² + y²)
ẏ = −x + μy − y(x² + y²)
μ̇ = ε
```

The write-up compares ε = 0 (μ constant) with ε = 0.1.

## Key findings (from the report)

- **ε = 0:** the correct system could not be identified accurately. With μ constant, terms such as μ·y are almost collinear with y, so the fit can split a coefficient between them. In [`hopf_normal_form_mu_dot_0.ipynb`](notebooks/hopf_normal_form_mu_dot_0.ipynb) the fit returns `0.962 y + 0.192 yμ` (with μ = 0.2) in place of a single `1.0 y`, while still achieving R² ≈ 0.99999 on the derivatives.
- **ε = 0.1:** with the STLSQ threshold at 0.1, the correct structure is recovered, with coefficients within about 1% of the true values (for example `ẋ = 0.999y + 1.003xμ − 0.998x³ − 0.996xy²`).
- **Model selection:** for each ε the report evaluates about 1.5 million models, built by varying the initial conditions (x, y ∈ [−1, 1], μ₀ ∈ [−0.2, 0.6]), the time span (10–25 s) and the time step (0.01–0.05 s), all with STLSQ and threshold 0.1. Models are compared by error (1 − R², log scale) against the number of non-zero terms, and the lowest-error sparse model is selected. Best errors are on the order of 10⁻⁵ to 10⁻⁶.
- **Takeaway:** a small drift in μ (ε > 0) removes the degeneracy, but ε must stay at or below the STLSQ threshold; if ε exceeds it, the correct system would not be identified.

## Notebooks

The notebooks demonstrate the identification setup on single trajectories; the large-scale sweep and its figures are described in the report.

| Notebook | System | What it shows |
|---|---|---|
| [`hopf_normal_form_mu_dot_0.ipynb`](notebooks/hopf_normal_form_mu_dot_0.ipynb) | Hopf normal form, μ̇ = 0 | The collinearity issue above, plus the R² score on the derivatives. |
| [`hopf_normal_form_mu_dot_0.1.ipynb`](notebooks/hopf_normal_form_mu_dot_0.1.ipynb) | Hopf normal form, μ̇ = 0.1 | Recovery of the full cubic structure. |
| [`lorenz_custom_library.ipynb`](notebooks/lorenz_custom_library.ipynb) | Lorenz system | SINDy with a custom feature library; recovers the Lorenz coefficients (σ ≈ 10, ρ ≈ 28, β ≈ 8/3). |
| [`saddle_node_parameterized_library.ipynb`](notebooks/saddle_node_parameterized_library.ipynb) | Saddle-node normal form (ẋ = α − x², ẏ = −y) | A parameterized library with α supplied as a control input; recovers `ẋ = 1.002 − 1.002x²`, `ẏ = −1.000y`. |

## Setup

```bash
pip install -r requirements.txt
jupyter notebook
```

## Reference

- S. L. Brunton, J. L. Proctor and J. N. Kutz, *Discovering governing equations from data by sparse identification of nonlinear dynamical systems*, PNAS 113(15), 2016.
