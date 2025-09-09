---
layout: default
nav: smurf
permalink: /smurfstudio/
title: Smurfstudio — Training & Tools
---


Smurfstudio extends the Nerfstudio ecosystem with utilities for data prep, training, and rendering.


## Data & Poses


1. Generate a `transforms.json` that lists image files and per‑image intrinsics/extrinsics.
2. Double‑check axis conventions. If poses are flipped or rotated, training converges to a blurry blob.


### Camera Recap


**Intrinsics** \(\mathbf K\) and **Extrinsics** \(\mathbf T_{cw}\) as defined on the Intro page apply here. Rays are \(\mathbf r(t)=\mathbf o+t\mathbf d\).


## Training


- Use a config that sets scene bounds, samplers (image‑space vs. scene‑box), and proposal networks.
- Keep an eye on PSNR/SSIM and the weight histogram of \(w_i=T_i\alpha_i\) to ensure samples cover geometry.


## Rendering & I/O


- Batch rendering helper converts a set of camera poses to images for a trained model.
- Expose the viewer port in your container to interactively inspect training.


## Closing Remarks


Open an issue/PR if you want a depth‑supervision section, dynamic scenes, or distillation guides.