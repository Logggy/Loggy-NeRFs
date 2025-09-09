---
layout: default
nav: nerfs
permalink: /nerfs/
title: All About NeRFs Page
---

# All About NeRFs Page

## Vanilla NeRF (2020)

If you want to truly understand what a NeRF is and how they work, then you should become VERY familiar with this paper. While undoubtedly the worst of all modern NeRF models, this is by far the simplest to understand and train, although I do not recommend training a Vanilla NeRF model yourself unless you are interested, they take a long time to train and only perform well on simple artificial datasets.  
**Key paper:** Mildenhall et al. (2020), *Representing Scenes as Neural Radiance Fields for View Synthesis* — https://arxiv.org/abs/2003.08934

### Basic Idea

The basic idea of the first NeRF model itself is pretty simple: every point in space has the potential to interact with light in some way, and we can use a neural network to approximate it. This makes NeRF a **volumetric renderer**, since it assumes that there are points anywhere in space that can emit or absorb light. You can compare this to other methods you know, one of the most popular being **surface-based renderers**, which use meshes (often triangles) to represent surfaces in a 3D scene which can then be illuminated and transformed to a 2D image (this is how most video games work).

The neural network in a NeRF model learns a continuous representation for the underlying radiance field, mapping each point in space to a position-dependent density value \( \sigma(\mathbf{x}) \) and a position- and view-dependent color value \( \mathbf{c}(\mathbf{x}, \mathbf{d}) \). In other words, NeRF takes a **5D input** (3D position \( \mathbf{x} \) and 2D direction on the sphere \( \mathbf{d} \)) and predicts density and color. This covers the “3D” representation of the scene, but all the images we view on our monitors are 2D, so we need a rendering step to get pixels from the field.

#### Virtual Cameras

Regardless of the rendering technique, **virtual cameras** are used to capture the views present within a virtual scene. Modeling these cameras like real ones lets us tweak intrinsics for different viewing properties (e.g., focal length to zoom). We’ll stick to the simplest model: the **pinhole**.

> **Pinhole camera (diagram)**  
> ![Pinhole camera diagram](https://commons.wikimedia.org/wiki/Special:FilePath/Pinhole-camera.svg)  
> *Credit: Wikimedia Commons.* 

We represent a calibrated camera using **intrinsics** and **extrinsics**.

**Intrinsics** (pinhole model, maps camera-frame 3D to pixel coordinates):
$$
\mathbf K = 
\begin{bmatrix}
f_x & s & c_x\\
0 & f_y & c_y\\
0 & 0 & 1
\end{bmatrix}, 
\quad
\tilde{\mathbf p}_\text{pix} \propto \mathbf K\,\tilde{\mathbf x}_c.
$$

Here \(f_x,f_y\) are focal lengths in pixels, \((c_x,c_y)\) the principal point, and \(s\) the (usually zero) skew.

**Extrinsics** (rigid transform world→camera):
$$
\mathbf T_{cw} = 
\begin{bmatrix}
\mathbf R & \mathbf t\\
\mathbf 0^\top & 1
\end{bmatrix}, 
\qquad 
\tilde{\mathbf x}_c = \mathbf T_{cw}\,\tilde{\mathbf x}_w.
$$

**Ray through a pixel** (in world coords) is
$$
\mathbf r(t) = \mathbf o + t\,\mathbf d, \qquad t\in [t_n, t_f],
$$
with camera origin \( \mathbf o \) and unit direction \( \mathbf d \) computed from intrinsics/extrinsics.

This is mostly all we need to represent a camera in a virtual scene and generate images from it. In NeRF we also incorporate **ray marching**: from every pixel we cast a ray into the scene and sample along it.

> **Ray marching / volume ray casting (single-ray sampling)**  
> ![Volume ray casting diagram](https://commons.wikimedia.org/wiki/Special:FilePath/Volume_Ray_Casting-en.svg)  
> *Credit: Wikimedia Commons.*  
> *(Prefer a NeRF-specific schematic? See SNeRG Fig. 3 for a NeRF ray-marching overview.)*

#### Training Vanilla NeRF

We have density and color everywhere, and we have camera rays; now we put it together with **classical volume rendering**.

**Continuous rendering equation** (NeRF Eq. 1):
$$
\mathbf C(\mathbf r)
= \int_{t_n}^{t_f}
T(t)\,\sigma\!\big(\mathbf r(t)\big)\,
\mathbf c\!\big(\mathbf r(t),\mathbf d\big)\,dt,
\quad
T(t)=\exp\!\Big(-\!\!\int_{t_n}^{t}\sigma\!\big(\mathbf r(s)\big)\,ds\Big).
$$

Because we can’t integrate exactly, we use **quadrature with stratified sampling**.

**Discrete alpha compositing** (NeRF Eq. 2):
$$
\hat{\mathbf C}(\mathbf r)=
\sum_{i=1}^{N}
\underbrace{\exp\!\Big(-\sum_{j=1}^{i-1}\sigma_j\,\Delta_j\Big)}_{T_i}
\;\underbrace{\big(1-e^{-\sigma_i\Delta_i}\big)}_{\alpha_i}\;
\mathbf c_i,
\qquad
\Delta_i = t_{i+1}-t_i.
$$

Here \(t_n, t_f\) are near/far bounds, \( \sigma_i = \sigma(\mathbf r(t_i)) \), \( \mathbf c_i = \mathbf c(\mathbf r(t_i),\mathbf d) \), and \( \Delta_i \) is the spacing between adjacent samples.

To **optimize** the network we randomly sample rays from the training images and compare rendered colors to ground truth pixels using MSE:

**Photometric loss**:
$$
\mathcal L = \sum_{\mathbf r\in\mathcal R}
\big\|
\hat{\mathbf C}(\mathbf r)-\mathbf C^{\mathrm{gt}}(\mathbf r)
\big\|_2^2.
$$

#### Additional details

**Positional encoding.** Deep nets are biased toward low frequencies, so a plain 5D input struggles with high-frequency detail. We map inputs to a higher-dimensional, high-frequency space—e.g., Fourier features for positions and spherical harmonics for directions:
$$
\gamma(p)=\big(
\sin(2^{0}\pi p),\cos(2^{0}\pi p),\ldots,
\sin(2^{L-1}\pi p),\cos(2^{L-1}\pi p)
\big).
$$

**Hierarchical / proposal sampling.** Without guidance, many ray samples land in empty space. NeRF (2020) trains **coarse** and **fine** fields: coarse weights \(w_i=T_i\alpha_i\) define a PDF along the ray and we resample where density is likely, then render with a fine pass. Modern pipelines replace the coarse MLP with **proposal networks** that directly predict densities to place samples efficiently.

---

## More NeRFs

Vanilla NeRF was just the beginning in 2020, and nowadays there are MANY models, each with their pros and cons depending on their intended application space. I won't cover these in detail, but I will list some of the important ones below and give a short description of why they are important:

- **NeRF-W (2021)** — **Transient/appearance embeddings** for Internet photo collections. Paper: https://arxiv.org/abs/2008.02268  
- **Mip-NeRF (2021)** — Anti-aliased rays via integrated positional encoding & cone tracing. Abs: https://arxiv.org/abs/2103.13415 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Barron_Mip-NeRF_A_Multiscale_Representation_for_Anti-Aliasing_Neural_Radiance_Fields_ICCV_2021_paper.pdf
- **KiloNeRF (2021)** — Thousands of tiny MLPs for faster rendering. Abs: https://arxiv.org/abs/2103.13744 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Reiser_KiloNeRF_Speeding_Up_Neural_Radiance_Fields_With_Thousands_of_Tiny_ICCV_2021_paper.pdf
- **PlenOctrees (2021)** — Bake a NeRF into an octree for **real-time** rendering. Abs: https://arxiv.org/abs/2103.14024 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Yu_PlenOctrees_for_Real-Time_Rendering_of_Neural_Radiance_Fields_ICCV_2021_paper.pdf
- **Instant-NGP (2022)** — **Multi-resolution hash encoding** + occupancy grids; big speedups. Paper: https://arxiv.org/abs/2201.05989 · Project: https://nvlabs.github.io/instant-ngp/
- **Mip-NeRF 360 (2022)** — Robust **unbounded** scenes, commonly paired with proposal networks. Abs: https://arxiv.org/abs/2111.12077 · PDF: https://openaccess.thecvf.com/content/CVPR2022/papers/Barron_Mip-NeRF_360_Unbounded_Anti-Aliased_Neural_Radiance_Fields_CVPR_2022_paper.pdf
- **K-Planes (2023)** — Plane-factorized fields for 4D (space-time-appearance). Paper: https://arxiv.org/abs/2301.10241 · Project: https://sarafridov.github.io/K-Planes/
- **Zip-NeRF (2023)** — Anti-aliased, grid-based NeRF with fast, high-quality unbounded scenes. Paper: https://arxiv.org/abs/2304.06706 · PDF: https://openaccess.thecvf.com/content/ICCV2023/papers/Barron_Zip-NeRF_Anti-Aliased_Grid-Based_Neural_Radiance_Fields_ICCV_2023_paper.pdf
- **Gaussian Splatting (2023)** — Not technically a NeRF; explicit Gaussians + rasterization for **real-time**. Paper: https://arxiv.org/abs/2308.04079
