# iDNS — Intelligent Adaptive Policy Solver for Navier–Stokes

#iDNS Minimal Kolmogorov / Taylor-Green Flow Solver — Public Release
# ----------------------------------------------------
# Author: Jeffrey Camlin
 © 2025 Red Dawn Academic Press & AI Lab, Milwaukee WI
 License: CC BY-NC 4.0 (Attribution–NonCommercial)
 https://creativecommons.org/licenses/by-nc/4.0/

 ![iDNS Banner]([https://your-url.com](https://i0.wp.com/reddawnacademicpress.org/wp-content/uploads/2025/11/vorticity_evolution-f.png?resize=2048%2C1038&ssl=1)/image.png)

# iDNS — Intelligent Adaptive Policy Solver for Navier–Stokes

Abstract
We introduce iDNS, a deterministic spectral-temporal algorithmic solver that stabilizes direct numerical simulation of incompressible Navier–Stokes by lifting the Fourier–Galerkin weak solution to a uniformly sampled computational manifold. Instead of advancing the flow on physical time t, the method evolves the solution curve in a latent parameter τ through a diffeomorphic map t = φ(τ), with φ'(τ) set by a sigmoid policy that concentrates sampling where geometric curvature is high. An intelligent adaptive policy
∂τU(τ) = (1/φ'(τ)) N(U(τ))
produces dual-scaling control: the simulation timestep expands as Δt = φ'Δτ while the integration step contracts as ΔU ∝ (1/φ')NΔτ. Large φ' increases sampling rate along the trajectory without modifying the physics, enabling conservation-preserving integration on coarse grids (Δx ≫ η).
Benchmarks confirm effectiveness: Kolmogorov flow computed stably from Re = 2×10⁴ to 10⁸ with correct dissipation scaling (0.07% error) and exponential spectral decay; Taylor–Green vortex at Re = 10⁵ integrated through the full cascade with bounded Beale–Kato–Majda integral (BKM = 37.1) and 33.7× speedup.
Although nominal grids (512² Kolmogorov, 128³ Taylor–Green) appear under-resolved, temporal lifting introduces a resolution multiplier: Nτ ≈ 33 pullback samples yield effective resolutions of 512√33 ≈ 2,941² and 128 × 33^(1/3) ≈ 411³, spectral super-resolution without artificial dissipation. Unlike heuristic timestep controllers, ours monitors the Beale–Kato–Majda functional in real-time, concentrating samples where the solution curve bends hardest. Sampling density scales with local stiffness, not Reynolds number.

This repository contains the minimal research reference implementation for the iDNS
(Intelligent Direct Numerical Simulation) framework, along with the benchmark CSV files
used in the validation of 2D Kolmogorov and 3D Taylor–Green test cases.

This code is intentionally minimal and corresponds only to the reproducibility elements
required for the associated research papers.

iDNS is a deterministic spectral-temporal algorithmic solver for stable, high-Reynolds number integration of the incompressible Navier–Stokes equations on periodic domains. The method lifts the Fourier–Galerkin weak solution to a uniformly sampled computational manifold via geometric temporal lifting, preventing CFL-induced timestep collapse without artificial viscosity.

**Key result:** Temporal lifting introduces a resolution multiplier. $N_\tau \approx 33$ pullback samples yield effective resolutions far beyond nominal grid size:

| Benchmark | Nominal Grid | Effective Resolution | Reynolds |
|-----------|--------------|---------------------|----------|
| Kolmogorov | $512^2$ | $\approx 2941^2$ | $10^8$ |
| Taylor–Green | $128^3$ | $\approx 411^3$ | $10^5$ |

Same grid. 33× more information.

---

# iDNS: Intelligent Direct Numerical Simulation

**Deterministic Spectral-Temporal Solver for High-Reynolds Turbulence**

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)

iDNS achieves stable integration of the incompressible Navier-Stokes equations at extreme Reynolds numbers (up to 10⁸) on consumer laptop hardware through **temporal lifting** — a geometric reparameterization that decouples numerical stability from spatial resolution.

## Key Results

| Benchmark | Reynolds | Grid | Result | Hardware |
|-----------|----------|------|--------|----------|
| Kolmogorov Flow | 10⁸ | 512² | Stable to t=200, 22 min | Laptop (i5, 8GB) |
| Taylor-Green Vortex | 10⁵ | 128³ | BKM=37.1, full cascade | Laptop (i3, 8GB) |
| Dissipation Scaling | 10⁴ → 10⁶ | 256² | 0.07% error vs theory | Same hardware |

## How It Works

Traditional DNS fails at high Reynolds numbers due to CFL-induced timestep collapse. iDNS solves this by evolving the solution in a **computational time** τ while tracking physical time t through a diffeomorphic map:

```
∂τU(τ) = (1/φ'(τ)) N(U(τ))
```

where φ'(τ) is determined by a sigmoid controller responding to local flow stiffness. Large φ' → dense sampling in stiff regions, maintaining stability without modifying the Navier-Stokes operator.

---

## Files

### `idns_v1_public_kolmogorov.py`
**2D Kolmogorov Flow Solver** — Regimes I, II, IV

Forced 2D turbulence on periodic torus T². Use for:
- Computational efficiency benchmarks (33.7× speedup)
- Reynolds scaling validation (10⁴ to 10⁸)
- Dissipation rate verification

**Outputs:** CSV (time series), PNG (diagnostic plots)

### `idns_v1_public_tg.py`
**3D Taylor-Green Vortex Solver** — Regime III

Decaying 3D turbulence benchmark. Use for:
- Vortex-stretching cascade validation
- BKM regularity criterion verification
- High-Reynolds 3D stability demonstration

**Outputs:** CSV (time series), PNG (diagnostic plots)

### `idns_v1_public_tg_snapshots.py`
**3D Taylor-Green with Field Snapshots**

Same as above, plus saves full 3D velocity fields at t = 3.0, 4.0, 4.5, 5.0 for visualization (Brachet-style vorticity contours, energy spectra, etc.)

**Outputs:** CSV, PNG, NPZ files (ux, uy, uz arrays)

---

## Parameters

All solvers use a `Config` dataclass. Edit these values directly in the script:

### Kolmogorov Flow Parameters

```python
@dataclass
class Config:
    N: int = 256              # Grid resolution (N×N modes)
    L: float = 2 * np.pi      # Domain size
    Re: float = 20_000.0      # Reynolds number
    T_final: float = 100.0    # Integration time
    dt_base: float = 6e-3     # Base timestep (τ units)
    
    # Forcing
    F: float = 1.0            # Forcing amplitude
    kf: int = 4               # Forcing wavenumber
    
    # Temporal lift controller
    lift_A: float = 1.0       # Sigmoid amplitude
    lift_k: float = 12.0      # Sigmoid steepness
    lift_center: float = 0.6  # Activation threshold
    
    # Output
    out_dir: str = "results"
    print_every: int = 100
```

**What to adjust:**
- `Re` — Set your target Reynolds number
- `N` — Increase for higher Re (256 for 10⁴-10⁶, 512 for 10⁷, 1024 for 10⁸)
- `T_final` — Integration duration
- `dt_base` — Decrease if you see instability (rare)

### Taylor-Green Parameters

```python
@dataclass
class Config:
    N: int = 128              # Grid resolution (N³ modes)
    L: float = 2 * np.pi      # Domain size  
    Re: float = 100_000.0     # Reynolds number
    T_final: float = 5.0      # Integration time
    dt_base: float = 1e-4     # Base timestep (τ units)
    
    # Temporal lift controller
    lift_A: float = 1.0       # Sigmoid amplitude
    lift_k: float = 12.0      # Sigmoid steepness
    lift_center: float = 0.6  # Activation threshold
    
    # Output
    out_dir: str = "results"
    print_every: int = 100
```

**What to adjust:**
- `Re` — Reynolds number (validated at 10⁵)
- `N` — Grid size (128³ ≈ 1.5GB RAM, 256³ ≈ 12GB RAM)
- `T_final` — Integration time (t=5 captures full cascade at Re=10⁵)

### Temporal Lift Controller (All Solvers)

The sigmoid parameters control adaptive timestepping:

| Parameter | Default | Effect |
|-----------|---------|--------|
| `lift_A` | 1.0 | Max lift factor (φ' ranges from 1 to 1+A) |
| `lift_k` | 12.0 | Steepness of activation (higher = sharper transition) |
| `lift_center` | 0.6 | Stiffness threshold for activation |

**Leave these at defaults** unless you're experimenting. They've been validated across all benchmark regimes.

---

## Quick Start

```bash
# Clone the repository
git clone https://github.com/reddawnacademicpress/iDNS.git
cd iDNS

# Run Taylor-Green benchmark (takes ~80 hours at Re=10⁵ on laptop)
python idns_v1_public_tg.py

# Run Kolmogorov benchmark (takes ~22 minutes at Re=10⁴)
python idns_v1_public_kolmogorov.py
```

Results are saved to `./results/` by default.

---

## Requirements

- Python 3.8+
- NumPy
- Matplotlib

```bash
pip install numpy matplotlib
```

No GPU required. No parallelization required. Runs on consumer hardware.

---

## Citation

If you use iDNS in your research, please cite:

```bibtex
@article{camlin2025idns,
  title={iDNS: Intelligent Adaptive Policy for Turbulence Simulation via Temporal Lifting},
  author={Camlin, Jeffrey},
  year={2025},
  publisher={Red Dawn Academic Press}
}
```

---

## License

CC BY-NC 4.0 (Attribution-NonCommercial)

This software is provided for scientific and academic reproducibility. Commercial use requires separate licensing — contact reddawnacademicpress.org.

---

## Author

**Jeffrey Camlin**  
Red Dawn Academic Press & AI Lab  
Milwaukee, WI

© 2025 Red Dawn Academic Press

---

## Citation

If you use this repository, cite the corresponding paper:

Camlin, J. (2025). *iDNS: Intelligent Adaptive Policy for Navier–Stokes.*  
Red Dawn Academic Press.



---

## 🔹 Reference Papers

| Paper | Description | Local PDF | Online Link |
|:--|:--|:--|:--|
| **Temporal Lifting as Latent-Space Regularization** | Adaptive time-scaling for latent and PDE models. | [📄 PDF](Temporal_Lift-AI-CS-stamped.pdf) | [🔗 doi.org/10.48550/arXiv.2510.09805](https://doi.org/10.48550/arXiv.2510.09805) |
| **Neural-Inspired Spectral–Temporal Continuation** | Unified SC + TL global smoothness construction. | [📄 PDF](arxiv-stamp-Neural-Inspired%20Spectral-Temporal.pdf) | [🔗 recursion-intelligence.org/post-bio-ai-epistemics-v1n2-010a.html](https://recursion-intelligence.org/post-bio-ai-epistemics-v1n2-010a.html) |
| **XXXX: Intelligent Direct Numerical Simulation** | Validation paper with benchmark results at Re up to 10⁸. | [📄 PDF](XXXX_validation_paper.pdf) | [🔗 arXiv](https://arxiv.org) |

---

## 🔹 Citation

If you reference this work, please cite as follows (APA 7th edition):

> Camlin, J. (2025). *SpectralTemporal-NS: Neural-Inspired Spectral–Temporal Continuation for Smooth Global Navier–Stokes Solutions on T³.*  
> Red Dawn Academic Press, Milwaukee, WI.  
> https://recursion-intelligence.org/post-bio-ai-epistemics-v1n2-010a.html  
> ORCID 0000-0002-5740-4204

---

**BibTeX**

```bibtex
@article{Camlin2025_SpectralTemporalNS,
  author    = {Camlin, Jeffrey},
  year      = {2025},
  title     = {SpectralTemporal-NS: Neural-Inspired Spectral--Temporal Continuation for Smooth Global Navier--Stokes Solutions on T^3},
  publisher = {Red Dawn Academic Press},
  address   = {Milwaukee, WI},
  url       = {https://recursion-intelligence.org/post-bio-ai-epistemics-v1n2-010a.html},
  orcid     = {0000-0002-5740-4204}
}
```

---
## License

This repository is released under the Creative Commons
Attribution–NonCommercial 4.0 International License (CC BY-NC 4.0).

You are free to use, modify, and distribute this work for academic
and scientific purposes with attribution. Commercial use is prohibited.

Full license text:
https://creativecommons.org/licenses/by-nc/4.0/


