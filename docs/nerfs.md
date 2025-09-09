---
layout: default
nav: nerfs
permalink: /nerfs/
title: All About NeRFs
---


## Vanilla NeRF (2020)


**Core idea:** A neural field maps \((\mathbf x,\mathbf d)\mapsto(\sigma,\mathbf c)\), and images are formed by **volume rendering**.


### The Rendering Equation


Given a ray \(\mathbf r(t)=\mathbf o+t\mathbf d\):


\[\mathbf C(\mathbf r)=\int_{t_n}^{t_f} T(t)\,\sigma(\mathbf r(t))\,\mathbf c(\mathbf r(t),\mathbf d)\,\mathrm dt,\quad T(t)=\exp\Big(-\int_{t_n}^{t}\!\sigma(\mathbf r(s))\,\mathrm ds\Big).\tag{1}\]


Discretized (alpha compositing over samples \(t_1<\dots<t_N\)):


\[\hat{\mathbf C}(\mathbf r)=\sum_{i=1}^N \underbrace{\exp\big(-\sum_{j=1}^{i-1}\sigma_j\,\Delta_j\big)}_{T_i}\;\underbrace{\big(1-e^{-\sigma_i\Delta_i}\big)}_{\alpha_i}\;\mathbf c_i,\quad \Delta_i=t_{i+1}-t_i.\tag{2}\]


Training loss over ray batch \(\mathcal R\):


\[\mathcal L=\sum_{\mathbf r\in\mathcal R}\big\|\hat{\mathbf C}(\mathbf r)-\mathbf C^{\mathrm{gt}}(\mathbf r)\big\|_2^2.\tag{3}\]


### Positional Encoding


For scalar \(p\), Fourier features (applied component‑wise):


\[\gamma(p)=\big(\sin(2^0\pi p),\cos(2^0\pi p),\ldots,\sin(2^{L-1}\pi p),\cos(2^{L-1}\pi p)\big).\]


### Hierarchical / Proposal Sampling


Compute per‑sample weights \(w_i=T_i\alpha_i\) and resample along the ray proportional to this PDF. Modern pipelines replace the coarse MLP with small **proposal networks** that predict densities for efficient sample allocation.


> **Image:** Conceptual ray‑marching
> ![ray-marching](https://upload.wikimedia.org/wikipedia/commons/7/72/Visualization_of_SDF_ray_marching_algorithm.png)


---


## More NeRFs (with links)


- **NeRF (2020).** *Representing Scenes as Neural Radiance Fields for View Synthesis.*
Paper: https://arxiv.org/abs/2003.08934 · Project: https://www.matthewtancik.com/nerf
- **NeRF‑W (2021).** *NeRF in the Wild: Neural Radiance Fields for Unconstrained Photo Collections.*
Paper: https://arxiv.org/abs/2008.02268 · Project: https://nerf-w.github.io/
- **Mip‑NeRF (2021).** *A Multiscale Representation for Anti‑Aliasing Neural Radiance Fields.*
Abs: https://arxiv.org/abs/2103.13415 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Barron_Mip-NeRF_A_Multiscale_Representation_for_Anti-Aliasing_Neural_Radiance_Fields_ICCV_2021_paper.pdf
- **KiloNeRF (2021).** *Speeding Up Neural Radiance Fields With Thousands of Tiny MLPs.*
Abs: https://arxiv.org/abs/2103.13744 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Reiser_KiloNeRF_Speeding_Up_Neural_Radiance_Fields_With_Thousands_of_Tiny_ICCV_2021_paper.pdf
- **PlenOctrees (2021).** *Real‑Time Rendering of Neural Radiance Fields.*
Abs: https://arxiv.org/abs/2103.14024 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Yu_PlenOctrees_for_Real-Time_Rendering_of_Neural_Radiance_Fields_ICCV_2021_paper.pdf
- **Instant‑NGP (2022).** *Instant Neural Graphics Primitives with a Multiresolution Hash Encoding.*
Paper: https://arxiv.org/abs/2201.05989 · Project: https://nvlabs.github.io/instant-ngp/
- **Mip‑NeRF 360 (2022).** *Unbounded Anti‑Aliased Neural Radiance Fields.*
Abs: https://arxiv.org/abs/2111.12077 · PDF: https://openaccess.thecvf.com/content/CVPR2022/papers/Barron_Mip-NeRF_360_Unbounded_Anti-Aliased_Neural_Radiance_Fields_CVPR_2022_paper.pdf
- **K‑Planes (2023).** *Explicit Radiance Fields in Space, Time, and Appearance.*
Paper: https://arxiv.org/abs/2301.10241 · Project: https://sarafridov.github.io/K-Planes/
- **Zip‑NeRF (2023).** *Anti‑Aliased Grid‑Based Neural Radiance Fields.*
Paper: https://arxiv.org/abs/2304.06706 · PDF: https://openaccess.thecvf.com/content/ICCV2023/papers/Barron_Zip-NeRF_Anti-Aliased_Grid-Based_Neural_Radiance_Fields_ICCV_2023_paper.pdf
- **3D Gaussian Splatting (2023).** *For Real‑Time Radiance Field Rendering.*
Paper: https://arxiv.org/abs/2308.04079


---


## Notes & Tips


- Validate poses early (viewer sanity checks).
- Watch \(\sigma\) scale and near/far bounds—both affect stability.
- Anti‑aliasing (mip / cone tracing / IPE) matters with large FOVs.