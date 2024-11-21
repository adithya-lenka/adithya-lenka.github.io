---
title: 'Image Metrics'
date: 2023-12-30
permalink: /posts/2023/12/blog-post-1/
tags:
  - metrics
  - PSNR
  - SSIM
  - FSIM
  - LPIPS
---

Formulae for some important image quality metrics

Peak Signal-to-Noise Ratio (PSNR)
======
Compares the pixel values of the original image to the degraded image.
Given a noise-free m×n monochrome image I and its noisy approximation K,
Mean Squared Error (MSE) is defines as

$$
{\displaystyle {\mathit {MSE}}={\frac {1}{m\,n}}\sum _{i=0}^{m-1}\sum _{j=0}^{n-1}[I(i,j)-K(i,j)]^{2}.}
$$

The PSNR (in dB) is defined as 

$$
{\displaystyle
  {\begin{aligned}
  {\mathit {PSNR}}  &=10\cdot \log _{10} \left({\frac {\mathit {MAX}_{I}^{2}}{\mathit {MSE}}} \right) \\
                    &=20\cdot \log _{10} \left({\frac {\mathit {MAX}_{I}}{\sqrt {\mathit {MSE}}}} \right).
  \end{aligned}
  }
}
$$

Structural Similarity Index Measure (SSIM)
======
Compares the luminance, contrast, and structure of the original image to the degraded image.
SSIM is calculated on various windows of an image. The measure between two windows **x** and **y**
of common size 
$$ 
N \times N $$
is

$$
{\displaystyle {\hbox{SSIM}}(x,y)={\frac {(2\mu _{x}\mu _{y}+c_{1})(2\sigma _{xy}+c_{2})}{(\mu _{x}^{2}+\mu _{y}^{2}+c_{1})(\sigma _{x}^{2}+\sigma _{y}^{2}+c_{2})}}}
$$

with 
* $$ {\mu _{x}} $$ the pixel sample mean of **x**
* $$ {\mu _{y}} $$ the pixel sample mean of **y**
* $$ {\sigma _{x}^2} $$ the variance of **x**
* $$ {\sigma _{y}^2} $$ the variance of **y**
* $$ {\sigma _{xy}} $$ the covariance of **x** and **y**
* $$ c_1 = (k_1 L)^2, c_2 = (k_2 L)^2 $$ two variables to stabilize the division with weak denominator
* **L** the dynamic range of the pixel-values (typically this is $$2^{\#bits\ per\ pixel} - 1$$)
* $$ k_1 = 0.01, k_2 = 0.03 $$ by default

Decomposition
-----

$$
{\displaystyle {\text{SSIM}}(x,y)=l(x,y)^{\alpha }\cdot c(x,y)^{\beta }\cdot s(x,y)^{\gamma }}
$$

SSIM weighted combination of comparison measurements: luminance (**l**), contrast (**c**) and structure (**s**)

$$
{\displaystyle l(x,y)={\frac {2\mu _{x}\mu _{y}+c_{1}}{\mu _{x}^{2}+\mu _{y}^{2}+c_{1}}}}
$$

$$
{\displaystyle c(x,y)={\frac {2\sigma _{x}\sigma _{y}+c_{2}}{\sigma _{x}^{2}+\sigma _{y}^{2}+c_{2}}}}
$$

$$
{\displaystyle s(x,y)={\frac {\sigma _{xy}+c_{3}}{\sigma _{x}\sigma _{y}+c_{3}}}}
$$

where $$ c_3 = \frac {c_2}{2}  $$

Setting the weights $$\alpha, \beta, \gamma $$ to 1, the formula can be reduced to the first form.

A more advanced form of SSIM, called Multiscale SSIM (MS-SSIM) is conducted over multiple scales through a process of multiple stages of sub-sampling.

Feature Similarity Index (FSIM)
=====

* Human Visual System understands an image mainly according to its low-level features. 
* FSIM uses phase congruency (PC), a dimensionless measure of the significance of a local structure ,as the primary feature.
* Since PC is contrast invariant, the image gradient magnitude (GM) is employed as the secondary feature in FSIM. 
* PC is used again on the local quality map as a weighting function to derive a single quality score.
![Illustration for the FSIM index computation](/images/fsim_illustration.png)

$$
\mathit {S}_{PC}(x) = \frac{2\mathit{PC}_1(x) \cdot \mathit{PC}_2(x) + \mathit {T}_1}{\mathit{PC}_1^2(x) + \mathit{PC}_2^2(x) + \mathit {T}_1}
$$

$$
\mathit {S}_{G}(x) = \frac{2\mathit{G}_1(x) \cdot \mathit{G}_2(x) + \mathit {T}_2}{\mathit{G}_1^2(x) + \mathit{G}_2^2(x) + \mathit {T}_2}
$$

Citation: [FSIM: A Feature Similarity Index for Image Quality Assessment](https://ieeexplore.ieee.org/document/5705575)


Learned Perceptual Image Patch Similarity (LPIPS)
=====

Computes a distance measure between the activations of two image patches for some pre-defined network (Eg. VGG, AlexNet)

![LPIPS - Computing Distance](/images/lpips-1.png)

To compute a distance $$ \mathit{d}_0 $$ between two patches, $$ \mathit{x}, \mathit{x}_0 $$, given a network F, first compute deep embeddings, normalize the activations in the channel dimension, scale each channel by vector w, and take the $$ \mathit{l}_2 $$ distance, and then average across spatial dimension and across all layers.

$$ d(x, x_0) = \sum_{l} \frac{1}{H_l\,W_l} \sum_{h,w} \lVert w_l \odot (\hat{y}_{hw}^l - \hat{y}_{0hw}^l)\rVert_2^2$$

![LPIPS - Predicting Perceptual Judgement](/images/lpips-2.png)

A small network G is trained to predict perceptual judgment $$ \mathit{h} $$ from distance pair $$ (d_0, d_1) $$.

Variants of training with our perceptual judgements:
* ***lin*** **configuration:** Pretrained network weights $$ \mathit{F} $$ fixed, linear weights $$ \mathit{w} $$ learnt
* ***tune*** **configuration:** Pretrained weights used for initialization and all weights finetuned
* ***scratch*** **configuration:** Initialize network from random Gaussian weights and train.

Citation: [The Unreasonable Effectiveness of Deep Features as a Perceptual Metric](https://arxiv.org/abs/1801.03924)