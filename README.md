# iDNS — Intelligent Adaptive Policy Solver for Navier–Stokes
[![DOI](https://zenodo.org/badge/1089748208.svg)](https://doi.org/10.5281/zenodo.17730872) 

#iDNS Minimal Kolmogorov / Taylor-Green Flow Solver — Public Release
# ----------------------------------------------------
# Author: Jeffrey Camlin
 © 2025 Red Dawn Academic Press & AI Lab, Milwaukee WI
 License: CC BY-NC 4.0 (Attribution–NonCommercial)
 https://creativecommons.org/licenses/by-nc/4.0/

<p align="center">
  <img src="https://i0.wp.com/reddawnacademicpress.org/wp-content/uploads/2025/11/teaser-5.png?w=681&ssl=1" alt="iDNS Taylor-Green Vortex" width="700">
</p>

# iDNS — Intelligent Adaptive Policy Solver for Navier–Stokes
➡️ [![Paper: iDNS RASI Policy](https://img.shields.io/badge/PDF-PhilPapers-blue.svg)](https://philpapers.org/archive/CAMIIA-3.pdf)

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

## All solvers are in iDNS_public.zip

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
<p align="center">
  <img src="https://[i0.wp.com/reddawnacademicpress.org/wp-content/uploads/2025/11/teaser-5.png](https://reddawnacademicpress.org/idns-ad-hoc-page/)?w=681&ssl=1" alt="iDNS Taylor-Green Vortex" width="700">
</p>

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

Example: Run at Re = 10⁸ (Kolmogorov) with dt = 0.006, t_end = 100, Re = 1e8.

In the code config set your resolution:
N = 1024

Then command line with parameters:

```bash
python idns_v1_public_kolmogorov.py --mode idns --Re 1e8 --t_end 100
```

There will be console updates you can watch at every step with various info.

Results are saved to `./results/` by default.

---

## Kolmogorov Flow Parameters

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

### What to adjust:

- **Re** — Set your target Reynolds number  
- **N** — Increase for higher Re (256 for 10⁴–10⁶, 512 for 10⁷, 1024 for 10⁸)  
- **T_final** — Integration duration  
- **dt_base** — normally **0.006**, decrease if instability (rare)

---

## Taylor–Green Parameters

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

### What to adjust:

- **Re** — validated at 10⁵  
- **N** — 128³ ≈ 1.5GB RAM; 256³ ≈ 12GB RAM  
- **T_final** — t = 5 captures full cascade at Re = 10⁵  

---

## Temporal Lift Controller (All Solvers)

The sigmoid parameters control adaptive timestepping:

| Parameter     | Default | Effect |
|---------------|---------|--------|
| lift_A        | 1.0     | Max lift factor (φ' ranges 1 → 1+A) |
| lift_k        | 12.0    | Steepness of activation |
| lift_center   | 0.6     | Stiffness threshold for activation |

Leave these at defaults unless you're experimenting. They've been validated across all benchmark regimes.


## Citation

If you use iDNS in your research, please cite:

@article{Camlin2025_iDNS_RASI_Code,
  author       = {Camlin, Jeffrey},
  title        = {iDNS: Intelligent Regularity-Adaptive Spectral Integration (RASI) Policy for Deterministic Flow Dynamics -- Code and Data Repository},
  year         = {2025},
  publisher    = {Red Dawn Academic Press},
  doi          = {10.5281/zenodo.17730872},
  url          = {https://doi.org/10.5281/zenodo.17730872},
  orcid        = {0000-0002-5740-4204},
  note         = {Code and dataset release accompanying the RASI/iDNS solver methodology}
}

```

---

## License

CC BY-NC 4.0 (Attribution-NonCommercial)

This software is provided for scientific and academic reproducibility. Commercial use requires separate licensing — contact reddawnacademicpress.org.

---
## 🔹 Reference Papers (Theory, Math, and Computer Science)

---

If the whole block suddenly becomes normal text →  
**YES, the issue was a missing closing fence BEFORE this section.**

## 🔹 Citation

⭐ 📚 Citation Block (3 Papers — For README / GitHub / Papers)

1. iDNS: Intelligent Regularity-Adaptive Spectral Integration (RASI) Policy for Deterministic Flow Dynamics (forthcoming arXiv)

Jeffrey Camlin (2025)
Published: 27 November 2025
PhilPapers ID: CAMIIA-3
ORCID: 0000-0002-5740-4204
➡️ [![Paper: iDNS RASI Policy](https://img.shields.io/badge/PDF-PhilPapers-blue.svg)](https://philpapers.org/archive/CAMIIA-3.pdf)


iDNS (RASI Policy) provides a deterministic spectral–temporal integration method capable of evolving high–Reynolds number flows with globally smooth lifted trajectories. The method merges spectral continuation, temporal lifting, and stability-aware timestep negotiation to produce Reynolds-invariant computational cost for stiff fluid systems.

Theory Papers

2.  Global Regularity for Navier–Stokes on T³ via Bounded Vorticity–Response Functionals
Author: Jeffrey Camlin
DOI: 10.63968/post-bio-ai-epistemics.v1n2.012 
Published: 1 December 2025
➡️[![Paper:] ((https://recursion-intelligence.org/post-bio-ai-epistemics-v1n2-012.html))

We prove global regularity for the incompressible Navier–Stokes equations on T³ by constructing a bounded vorticity-response functional Φ : ℝ≥0 → [φ_min, φ_max] that defines a temporal lifting generalizing Sundman's regularization from celestial mechanics. Uniform-in-N bounds on Galerkin approximations combined with coordinate invariance of the Beale–Kato–Majda integral establish finiteness of ∫₀ᵀ ‖ω‖_L∞ dt for all T > 0; the BKM contrapositive then yields global classical solutions u ∈ C^∞(T³ × [0,∞)) that coincide with Leray–Hopf weak solutions by weak-strong uniqueness. This resolves Fefferman's Clay Millennium Problem Statement (B), with numerical validation via iDNS confirming the framework at Reynolds numbers up to 10⁸.

3. Temporal Lifting as Latent-Space Regularization for Continuous-Time Flow Models in AI Systems

Jeffrey Camlin (2025)
arXiv preprint arXiv:2510.09805
Published: 10 October 2025
ORCID: 0000-0002-5740-4204
➡️[![Paper: Temporal Lifting](https://img.shields.io/badge/PDF-arXiv%3A2510.09805-red.svg)](https://arxiv.org/abs/2510.09805)


Temporal lifting introduces a smooth, monotone reparametrization of time that regularizes stiff flows while preserving conservation laws. In the lifted coordinate, trajectories of systems such as incompressible Navier–Stokes on T3
 become globally smooth. For AI systems, temporal lifting acts as continuous-time normalization for PINNs, neural ODEs, and latent-flow architectures, stabilizing turbulent or near-singular evolution.

4. SpectralTemporal-NS: Neural-Inspired Spectral–Temporal Continuation for Smooth Global Navier–Stokes Solutions on T³

Jeffrey Camlin (2025)
Red Dawn Academic Press, Milwaukee, WI
Publication Date: 2025
ORCID: 0000-0002-5740-4204
➡️ [![Paper: SpectralTemporal-NS](https://img.shields.io/badge/Article-Red%20Dawn%20Press-purple.svg)](https://recursion-intelligence.org/post-bio-ai-epistemics-v1n2-010a.html)


This paper introduces a neural-inspired spectral–temporal continuation method for the incompressible Navier–Stokes equations on the torus. By combining lifted-time operators with adaptive spectral filtering, the solver achieves smooth global trajectories even at extreme Reynolds numbers. The method bridges analytic fluid regularity theory with continuous-time machine learning architectures and PINN-style latent dynamics.

---

@article{camlin2025globalregularity,
  author    = {Camlin, Jeffrey},
  title     = {Global Regularity for {Navier--Stokes} on {$\mathbb{T}^3$} via Bounded Vorticity--Response Functionals},
  journal   = {The Scholarly Journal of Post-Biological Epistemics},
  volume    = {1},
  number    = {2},
  pages     = {1--14},
  year      = {2025},
  doi       = {10.63968/post-bio-ai-epistemics.v1n2.012},
  url       = {https://doi.org/10.63968/post-bio-ai-epistemics.v1n2.012},
  issn      = {3069-499X},
  keywords  = {Navier-Stokes equations, global regularity, Clay Millennium Problem, Beale-Kato-Majda criterion, temporal lifting, vorticity-response functional}
}

@article{Camlin2025_TemporalLifting,
  author        = {Camlin, Jeffrey},
  title         = {Temporal Lifting as Latent-Space Regularization for Continuous-Time Flow Models in AI Systems},
  journal       = {arXiv preprint},
  eprint        = {2510.09805},
  archivePrefix = {arXiv},
  primaryClass  = {cs.LG},
  year          = {2025},
  month         = oct,
  orcid         = {0000-0002-5740-4204},
  url           = {https://arxiv.org/abs/2510.09805}
}

@article{Camlin2025_iDNS_RASI,
  author        = {Camlin, Jeffrey},
  title         = {iDNS: Intelligent Regularity-Adaptive Spectral Integration (RASI) Policy for Deterministic Flow Dynamics},
  journal       = {PhilPapers Archive},
  year          = {2025},
  month         = nov,
  url           = {https://philpapers.org/archive/CAMIIA-3.pdf},
  orcid         = {0000-0002-5740-4204}
}

@article{Camlin2025_SpectralTemporalNS,
  author    = {Camlin, Jeffrey},
  year      = {2025},
  title     = {SpectralTemporal-NS: Neural-Inspired Spectral--Temporal Continuation for Smooth Global Navier--Stokes Solutions on T^3},
  publisher = {Red Dawn Academic Press},
  address   = {Milwaukee, WI},
  url       = {https://recursion-intelligence.org/post-bio-ai-epistemics-v1n2-010a.html},
  orcid     = {0000-0002-5740-4204}
}

---
## License

This repository is released under the Creative Commons
Attribution–NonCommercial 4.0 International License (CC BY-NC 4.0).

You are free to use, modify, and distribute this work for academic
and scientific purposes with attribution. Commercial use is prohibited.

Full license text:
https://creativecommons.org/licenses/by-nc/4.0/


