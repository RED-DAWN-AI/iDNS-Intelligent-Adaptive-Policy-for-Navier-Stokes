# iDNS — Intelligent Adaptive Policy Solver for Navier–Stokes

#iDNS Minimal Kolmogorov / Taylor-Green Flow Solver — Public Release
# ----------------------------------------------------
# Author: Jeffrey Camlin
 © 2025 Red Dawn Academic Press & AI Lab, Milwaukee WI
 License: CC BY-NC 4.0 (Attribution–NonCommercial)
 https://creativecommons.org/licenses/by-nc/4.0/
#
 This minimal solver is provided for scientific and academic
 reproducibility of the Kolmogorov flow and Taylor-Green 
 benchmarks reported in the iDNS validation paper:
 "iDNS: Intelligent Adaptive Policy for Turbulence Simulation via Temporal Lifting"
 (Citation is below)


**Red Dawn AI Lab — Jeffrey Camlin (2025)**

iDNS is a deterministic adaptive-control framework for stable, high–Reynolds number 
integration of the incompressible Navier–Stokes equations on periodic domains, combining
Fourier spectral methods with geometric temporal lifting to prevent CFL-induced timestep collapse.

This repository provides the testing and replication code and data used for
benchmark validation in the iDNS paper. 

# iDNS — Intelligent Adaptive Policy Solver for Navier–Stokes

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

## Contents

- `idns_kolmogorov2d.py` — Minimal 2D Kolmogorov flow solver.
- `idns_tg3d.py` — Minimal 3D Taylor–Green vortex solver.
- `kolmogorov_512.csv` — Benchmark data from the N=512² iDNS run at Re = 10⁸.
- `taylor_green_128.csv` — Benchmark data from the N=128³ iDNS run at Re = 10⁵.

### Quick Start

Run any benchmark without editing code:

# Regime I and II (Re = 2×e⁴)
python idns_kolmogorov2d.py --preset 2e4
For the "conservative" solver change line 19 in "Benchmark Presets" from ("dt": 0.006) to ("dt": 0.001)

# Regime II (Re = 1e6)
python idns_kolmogorov2d.py --preset 1e6

# Extreme iDNS test (Re = 1e8, 512²)
python idns_kolmogorov2d.py --preset 1e8_512

# Resolution validation (Re = 1e8, 1024²)
python idns_kolmogorov2d.py --preset 1e8_1024


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


