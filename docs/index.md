---
layout: default
nav: home
permalink: /
title: Introduction to NeRFs
---


## What is a NeRF?


Neural Radiance Fields (NeRFs) are 3D scene representations that learn a function mapping position and view direction to density and color. With posed images as supervision, they synthesize novel views by **volumetric rendering**.


> If you’re already familiar, jump to **[All About NeRFs]({{ "/nerfs/" | relative_url }})** or **[Smurfstudio]({{ "/smurfstudio/" | relative_url }})**.


### Volumetric vs. Surface Rendering


- *Volumetric (NeRF)*: any point in space can absorb/emit light.
- *Surface (rasterization)*: triangles + shaders on surfaces.


### Camera Model (Pinhole)


**Intrinsics**
\[\mathbf K=\begin{bmatrix}f_x&s&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}\]


**Extrinsics** (world→camera)
\[\mathbf T_{cw}=\begin{bmatrix}\mathbf R&\mathbf t\\\mathbf 0^\top&1\end{bmatrix},\quad \mathbf T_{cw}=\mathbf T_{wc}^{-1}=\big[\,\mathbf R_{wc}^\top\mid-\mathbf R_{wc}^\top\mathbf t_{wc}\,\big].\]


**Ray through a pixel**
\[\mathbf r(t)=\mathbf o+t\,\mathbf d,\quad t\in[\,t_n,t_f\,].\]


> **Image:** Pinhole camera geometry
> ![pinhole](https://upload.wikimedia.org/wikipedia/commons/3/3b/Pinhole-camera.svg)


### Rendering (High Level)


Color along a ray integrates emitted color times differential opacity, attenuated by transmittance (details in the next page).


---


## Why NeRFs Matter


- *Photo‑realistic view synthesis* from sparse views.
- *Compact continuous representation* instead of heavy meshes.
- *Generalizable frameworks*: many extensions handle unbounded scenes, dynamic content, and fast training.


---


## Quick Links


- **All About NeRFs →** {{ "/nerfs/" | relative_url }}
- **Smurfstudio →** {{ "/smurfstudio/" | relative_url }}