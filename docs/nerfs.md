---
layout: default
nav: nerfs
permalink: /nerfs/
title: All About NeRFs
---

## Vanilla NeRF (2020)

If you want to truly understand what a NeRF is and how they work, then you should become VERY familiar with this paper. While undoubtedly the worst of all modern NeRF models, this is by far the simplest to understand and train, although I do not recommend training a Vanilla NeRF model yourself unless you are interested, they take a long time to train and only perform well on simple artificial datasets.  
**Key paper:** Mildenhall et al. (2020), *Representing Scenes as Neural Radiance Fields for View Synthesis* — https://arxiv.org/abs/2003.08934

### Basic Idea

The basic idea of the first NeRF model itself is pretty simple: every point in space has the potential to interact with light in some way, and we can use a neural network to approximate it. This makes NeRF a **volumetric renderer**, since it assumes that there are points anywhere in space that can emit or absorb light. You can compare this to other methods you know, one of the most popular being **surface-based renderers**, which use meshes (often triangles) to represent surfaces in a 3D scene which can then be illuminated and transformed to a 2D image (this is how most video games work).

The neural network in a NeRF model learns a continuous representation for the underlying radiance field, mapping each point in space to a position-dependent density value $$\sigma(\mathbf{x})$$ and a position- and view-dependent color value $$\mathbf{c}(\mathbf{x}, \mathbf{d})$$. In other words, NeRF takes a **5D input** (3D position $$\mathbf{x}$$ and 2D direction on the sphere $$\mathbf{d}$$) and predicts density and color. This covers the “3D” representation of the scene, but all the images we view on our monitors are 2D, so we need a rendering step to get pixels from the field.

#### Virtual Cameras

Regardless of the rendering technique, **virtual cameras** are used to capture the views present within a virtual scene. Modeling these cameras the same way as we do in real life comes with some benefits too, since it can allow us to modify their intrinsic values to achieve different viewing properties, such as changing the focal length to zoom in or out, or changing distortion parameters to make final images appear warped or curved. There are plenty of different virtual camera types we can use, but for our purposes we stick to the simplest one: the **pinhole**. This camera is modeled using a single hole and an image plane, where the light from the scene travels in a straight line through the hole onto the image plane, providing an inverted image of the scene. This makes the geometry of the problem a bit simpler to imagine, and later on it makes the task of training a NeRF model easier. 

> **Pinhole camera (for intuition)**
>
> ![Pinhole camera diagram](https://commons.wikimedia.org/wiki/Special:FilePath/Pinhole-camera.svg)

To represent these cameras in a virtual scene we use two matrices — the **intrinsic and extrinsic camera matrices**. The **intrinsic matrix** maps a 3D camera-centered point to a 2D one on the image plane (imagine a point following a ray back to the camera). This is a 3×3 upper-triangular matrix defined as:

$$
\mathbf K =
\begin{bmatrix}
f_x & s & c_x\\
0   & f_y & c_y\\
0   & 0   & 1
\end{bmatrix},
\qquad
\tilde{\mathbf p}_{\text{pix}} \propto \mathbf K\,\tilde{\mathbf x}_c .
$$

Here $$f_x$$ and $$c_x$$ are the x-axis focal length and optical center (in pixel units) with $$f_y$$ and $$c_y$$ representing the same for the y-axis. $$s$$ is usually $$0$$, and represents sensor skew. This matrix takes any camera-frame point $$ \mathbf p_c = [X,Y,Z]^\top $$ (or $$ \tilde{\mathbf x}_c $$ in homogeneous coordinates) and maps it to pixel space. **Remember,** $$ \mathbf K $$ is expressed in the **camera** reference frame.

The **extrinsic matrix** maps a 3D point centered at some arbitrary point in the world to a 3D point centered in the camera reference frame. This can be seen as the combination of a rotation and a translation, and is represented as:

$$
\mathbf T_{cw} =
\begin{bmatrix}
\mathbf R & \mathbf t\\
\mathbf 0^\top & 1
\end{bmatrix},
\qquad
\tilde{\mathbf x}_c = \mathbf T_{cw}\,\tilde{\mathbf x}_w .
$$

Here $$\mathbf R$$ is a rotation matrix and $$\mathbf t$$ is the translation from world to camera space (if the world is centered at the origin, this is the camera’s position expressed in the camera-from-world transform). If you have camera-to-world $$ \mathbf T_{wc} $$, then $$ \mathbf T_{cw} = \mathbf T_{wc}^{-1} = \big[\mathbf R_{wc}^\top \mid -\mathbf R_{wc}^\top \mathbf t_{wc}\big] $$.

This is mostly all we need to represent a camera in a virtual scene and generate images from it. In the case of NeRFs we will also incorporate **ray marching**, meaning that from every pixel on our camera we will cast a ray into the scene that we sample across. Once you have the virtual camera set up this is as simple as plotting a line that passes through the camera position and the center of each pixel in the projected image. A ray in world coordinates is:

$$
\mathbf r(t) = \mathbf o + t\,\mathbf d, \qquad t \in [t_n, t_f],
$$

with origin $$ \mathbf o $$ (the camera center) and unit direction $$ \mathbf d $$ computed from $$ \mathbf K $$ and $$ \mathbf T_{cw} $$.

> **Ray marching / volume ray casting (illustrative)**
>
> ![Volume ray casting schematic](https://commons.wikimedia.org/wiki/Special:FilePath/Volume_Ray_Casting-en.svg)

#### Training Vanilla NeRF

So, we know how NeRF represents density and color in space, and we have a virtual camera we can use to capture an image in that space, now we can put it all together. To get a pixel value for each ray we cast into the scene, we would ideally like to use this integral from classical volume rendering:

**NeRF rendering integral (Eq. 1 in the paper)**

$$
\mathbf C(\mathbf r)
= \int_{t_n}^{t_f}
T(t)\,\sigma\!\big(\mathbf r(t)\big)\,
\mathbf c\!\big(\mathbf r(t),\mathbf d\big)\,dt,
\qquad
T(t) = \exp\!\Big(-\!\!\int_{t_n}^{t} \sigma\!\big(\mathbf r(s)\big)\,ds\Big).
$$

where $$t_n$$ and $$t_f$$ are the near and far bounds along a ray, $$T(t)$$ is the **accumulated transmittance** (the probability a ray travels from $$t_n$$ to $$t$$ without hitting another particle), $$\mathbf r(t)$$ is the camera ray characterized by origin $$\mathbf o$$, direction $$\mathbf d$$ and length $$t$$ $$(\mathbf r(t)=\mathbf o + t\,\mathbf d)$$, $$\sigma(\mathbf r(t))$$ is the position-dependent density, and $$\mathbf c(\mathbf r(t), \mathbf d)$$ is the position- and view-dependent color.

Sadly we cannot do this exactly in practice, as it would require infinite sampling, so we instead estimate it using **quadrature with stratified sampling** (consult Mildenhall et al., 2020). In the end we are able to estimate the color values of rays using:

**Discrete alpha-compositing (Eq. 2)**

$$
\hat{\mathbf C}(\mathbf r) =
\sum_{i=1}^{N}
\underbrace{\exp\!\Big(-\sum_{j=1}^{i-1}\sigma_j\,\Delta_j\Big)}_{T_i}
\;\underbrace{\big(1-e^{-\sigma_i \Delta_i}\big)}_{\alpha_i}\;
\mathbf c_i,
\qquad
\Delta_i = t_{i+1}-t_i .
$$

where $$\Delta_i$$ is the distance between adjacent samples, $$\sigma_i=\sigma(\mathbf r(t_i))$$, and $$\mathbf c_i=\mathbf c(\mathbf r(t_i),\mathbf d)$$.

To recap, we have a way to represent the color and density at each point in these scenes, and using the rays cast from our virtual cameras we can now approximate full images from these scenes! To optimize this network we randomly sample and approximate rays from many different images and compare them to their ground truth pixel counterparts, constructing a loss function from their mean squared error. In the end the loss function looks like this:

**Photometric loss (MSE)**

$$
\mathcal L \;=\; \sum_{\mathbf r\in\mathcal R}
\Big\|
\hat{\mathbf C}(\mathbf r) - \mathbf C^{\mathrm{gt}}(\mathbf r)
\Big\|_2^2 .
$$

#### Additional details

A little more is required to get this model functioning to the level it did in 2020, the first is called **positional encoding**. Since deep networks are biased towards learning lower frequency functions, our NeRF will struggle to capture high frequency details present in the scene with its normal 5D inputs. This can be solved by mapping our inputs to a higher dimensional space using high frequency functions. NeRF (2020) covers their solution to the problem; today a popular choice is Fourier features for position and spherical harmonics for direction. For a scalar \(p\), a simple Fourier mapping is:

$$
\gamma(p) =
\big(\sin(2^0\pi p), \cos(2^0\pi p), \ldots,
\sin(2^{L-1}\pi p), \cos(2^{L-1}\pi p)\big).
$$

The final addition is a **hierarchical volume sampling** method. Without any guidance as to where we sample along our rays, there's a good chance that most of them land in empty space, harming reconstruction quality and slowing training time. NeRF (2020) guides sampling by simultaneously training two NeRF fields, one **coarse** and one **fine**. Using the outputs from the coarse NeRF we construct a PDF along the ray representing where density likely is and isn’t. By sampling additionally from this PDF and including these new samples, then running a **fine** NeRF pass, we greatly increase the number of meaningful samples per step, boosting reconstruction accuracy and convergence speed. Modern models use a flavor of this called a **proposal network**. In essence, proposal networks attempt to predict where density in the scene is, and optimize themselves by comparing their approximated density outputs to those produced by the NeRF. When these networks learn where the objects present in a scene are, it allows us to consistently sample at or near where we want to (as opposed to empty space). A common weight used for this PDF is
$$w_i = T_i\,\alpha_i$$.

## More NeRFs

Vanilla NeRF was just the beginning in 2020, and nowadays there are MANY models, each with their pros and cons depending on their intended application space. I won't cover these in detail, but I will list some of the important ones below and give a short description on why they are important:

- **NeRF in the Wild / NeRF-W (2021)** — Introduced **transient and appearance embeddings**, allowing NeRFs to learn scenes from images taken under different lighting conditions and with various transient objects clouding space.  
  Paper: https://arxiv.org/abs/2008.02268
- **Mip-NeRF (2021)** — Added anti-aliased rays through integrated positional encoding and cone tracing.  
  Abs: https://arxiv.org/abs/2103.13415 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Barron_Mip-NeRF_A_Multiscale_Representation_for_Anti-Aliasing_Neural_Radiance_Fields_ICCV_2021_paper.pdf
- **KiloNeRF (2021)** — Uses thousands of tiny MLPs instead of one big one, offering larger scene representations and faster renders.  
  Abs: https://arxiv.org/abs/2103.13744 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Reiser_KiloNeRF_Speeding_Up_Neural_Radiance_Fields_With_Thousands_of_Tiny_ICCV_2021_paper.pdf
- **PlenOctrees (2021)** — Transfer a NeRF into an octree for REALLY fast rendering.  
  Abs: https://arxiv.org/abs/2103.14024 · PDF: https://openaccess.thecvf.com/content/ICCV2021/papers/Yu_PlenOctrees_for_Real-Time_Rendering_of_Neural_Radiance_Fields_ICCV_2021_paper.pdf
- **Instant-NGP (2022)** — Applied **multi-resolution hash encoding** and occupancy grids, speeding up training and rendering greatly.  
  Paper: https://arxiv.org/abs/2201.05989 · Project: https://nvlabs.github.io/instant-ngp/
- **Mip-NeRF 360 (2022)** — Robust unbounded scene optimization introducing a number of changes, namely the now-standard proposal network.  
  Abs: https://arxiv.org/abs/2111.12077 · PDF: https://openaccess.thecvf.com/content/CVPR2022/papers/Barron_Mip-NeRF_360_Unbounded_Anti-Aliased_Neural_Radiance_Fields_CVPR_2022_paper.pdf
- **K-Planes (2023)** — Uses $$d \choose 2$$ planes to represent a radiance field, allowing for 4D representation of NeRFs (time-varying scenes), while remaining competitive on static 3D scenes.  
  Paper: https://arxiv.org/abs/2301.10241 · Project: https://sarafridov.github.io/K-Planes/
- **Zip-NeRF (2023)** — Builds on Mip-NeRF 360 while incorporating Instant-NGP-style advances, achieving much faster training on unbounded scenes.  
  Paper: https://arxiv.org/abs/2304.06706 · PDF: https://openaccess.thecvf.com/content/ICCV2023/papers/Barron_Zip-NeRF_Anti-Aliased_Grid-Based_Neural_Radiance_Fields_ICCV_2023_paper.pdf
- **Gaussian Splatting (2023)** — Not technically a NeRF model, but uses Gaussian blobs to quickly learn a scene. Since these use Gaussians rather than an MLP, this works well with rasterization, allowing **exceptionally high frame rates** on modern GPUs.  
  Paper: https://arxiv.org/abs/2308.04079