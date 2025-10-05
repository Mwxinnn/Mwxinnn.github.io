---
title: Diffusion
layout: single
date: 2025-04-04
priority: 1
---

> [Material Source1](https://www.bilibili.com/video/BV13J4m1K73z/), [Material Source2](https://www.bilibili.com/video/BV1UdYCzzEeo/). Thanks for the open-source lecture!

Before Diffusion, the generation models are mostly based on autoregressive methods like VQ-VAE2. Diffusion has made below changes:
- Autoregressive Prior -> Diffusion Prior
- VAE-like Decoder -> Diffusion Decoder

## Basic Diffusion Model
![](MHVAE.png)
Diffusion is built on Markovian Hierarchical VAE, that each latent is from the last latent. There are three constrains for diffusion model:
1. All the data sample $x$ and latent at time $t$ ($z_t$) have same channels.
2. Encoder $f(z_t|z_{t-1})$ doea not need learning. It is determistric that $z_t\sim N(\mu_{z_{t-1}},1)$.
3. Last latent $z_T\sim N(0,1)$.

We only have to learn a decoder to restore the image from a standard gaussian distribution. Each decoder $Dec_t(\cdot)$ predicts the mean value of $x_{t-1}$ the previous time step $\mu_{t-1}$.

## DDPM
![](DDPM.png)
The prediction of $x_{t-1}$ is changed to predicting the mean value of the noise at $t$, so that $x_t-noise_{t}\rarr x_{t-1}$. This helps training because the noise is restritced to gaussian distribution, and the searching space for the decoder model is much smaller.
> Moreover, there is also an optimization for the loss function.
![](DDPM_algorithm.png)

Given $\epsilon\sim N(0,1)$, the prediction process is shown in $\epsilon_ \theta\left(\sqrt{\bar{\alpha}_t}x_0 + \sqrt{1 - \bar{\alpha}_t}\epsilon,\ t\right)$. 

After predicting the noise for current step, as shown in stage2, we can sample the previous $x_{t-1}$ according to the noise. $\sigma _t z$ is a term to ensure diversity, where $z\sim N(0,1)$. 

The whole process for denoising in one step can be written as: $n_t=f_ \theta (x_t,\ t)$.

## Diffusions Beat GANs
This work optimizes the structural design of the decoder (based on U-Net + attention layer), including depth heads, attn resolutions. 

More importantly, this work proposes classifier guidance. Incorporating guidance by cross-attention based mechanism can be ineffective because there is a shortcut (making the weights in attention zero). This work ensures the direction for sampling $\mu$ is toward a given class $y$ by incorporating a classifer for each $x_t$, with a $s$ serving as a hyper-parameter for balance.
![](DbG.png)

## GLIDE: Classifier-free Guidance
For classifier guidance, the classifiers need to be trained from scratch each time because it is not possible to load a pre-trained classifier which has never seen noisy pictures.

Classifier-free Guidance (CFG) uses this strategy: during training, there are samples trained both with (90\%) and without text (10\%) during adding noise & denoise stages, so the model learns how to predict the corresponding $x_t$ give $t$ with text condition.

During sampling, $x_t$ is $$\hat{\epsilon}_ \theta(x_t|x_{t+1},y)=\epsilon_ \theta(x_t|x_{t+1},\Phi)+s(\epsilon_ \theta(x_t|x_{t+1},y)-\epsilon_ \theta(x_t|x_{t+1},\Phi))$$, a weighted sum of generation results with/without text condition. The direction choice is controlled by hyper-parameter $s$. 

In this way, the classifier is omitted, while the influence of text can be included during both training and sampling stages. This design can facilitate the balance between fidelity and diveristy.

However, this method requires double inference (with/without text condition). Moreover, with a large $s$, the diversity of generated image would decrease because the image feature space tends to be smaller.

## DALL$\cdot$E 2 (unCLIP)
![](DALLE2.png)
Overall design: generate image based on the image embeddings of CLIP and text prior. The model aligns text and visual embedding at first, then the text embedding is input to a prior network to predict the image embedding. The predicte image embedding works as the condition for decoding.

There are two choices for the prior network. Comparison between Auto-Regressive Model and Diffusion Model as a prior:
- AR: given text embedding, predict the next image embedding one by one.
- Diffusion: using a transformer model to predict unknown and noise-free image (not the noise). The training target is $$L\=E_{z_0​,ϵ,t}​\[∥ϵ−ϵ_θ​(z_t​,t∣\text{text})∥^2\]$$
  
The decoder design is shown below. It uses the image embedding as the initial condition, then does conditional super-resolution gradually.
![](DALLE2_decoder.png)

Benefits of using an image embedding: 
![](DALLE2_benefits.png)
1. Variation: The generated image follows the same structure and position as the original image (instead of using a random noise in each step); 
2. Interpolation: using the image embedding gained from two images to ensure the consistency between original images and generated images.
3. Text Diffs: Because the image features are aligned with text features, the difference between original and target text features can represent the difference between original and target image features.

CFG: The text prior is necessary. OpenAI team compares using pure text (without CLIP), using CLIP text embedding as the condition, and using text prior to generate image embedding as the condition, eventually chooses the last one.

However, the text-matching accuracy of generated image is still low. The generated image can lack of details. It is also hard to directly generate text in images. It is also hard to generate images that are conflicting with reality.

## Imagen
![](Imagen.png)
The decoder structure of Imagen is similar to DALLE 2, with low resolution image **added noise** as the conditon for super-resolution. However, there is no prior network. The text encoder is replaced by a frozen text encoder (T5) to generate text embeddings, added to the sampling model as condition by cross-attention. They also prove that T5 is more effective than CLIP text encoder. There're other techniques such as dynamic thresholding and efficient U-Net.

> This work has also proven that increasing the text model's scale is more effective than increasing the scale of denoise model.

## Stable Diffusion (LDM)
![](StableDiffusion.png)
Motivation: previous works require multi-level decoding, which is not computationally friendly. Stable Diffusion moves their attention onto latent space of VAE, diffusing and denoising on latent space. 

Additionally, various condition types are allowed by training different encoders.

### SDXL
![](SDXL.png)
This model uses two text encoders and the extracted embeddings are concatenated together. It uses a Base and a Refiner connected together as a decoder, in which the output of Base decoder is the input of the Refiner decoder (same resolution). Refiner is a denoise model for denoising the blurry part in the generated results of Base model.

It adds the resolution of the image as the condition. Because during augmentation for training, some images are upsampled and some are downsampled. With explicit resolution information, some confusion can be avoided to improve generation quality. It is also possible to generate image with different resolution. To avoid the confusion caused by crop operation, the coordinate of the left-top point is input to the model at the same time.

The problem of this model includes: hard to generate complex scenarios, concept bleeding, fidelity...

### DDIM
Effiecient generation (DDPM: 1000 steps, DDIM: 50 steps).

#### Why DDPM has to sample in 1000 steps?
They want to make the process of converting the distribution from one to another Markovian (which also prohibits skipping steps during sampling). They want to ensure the continuous diffusion process (limit of small step size $\beta$) and the reversal of the diffusion process have identical functional form.

#### DDIM
If we want to skip steps during sampling, we have to remove the Markovian Hypothesis during the diffusion process.

In DDPM, we always want to get $P(x_{t-1}|x_t, x_0)$ using Bayes's theorem, with $P(x_{t-1}|x_t, x_0)=P(x_{t-1}|x_t)$ given the Markovian diffusion process. In DDIM, we can now add culmulated noise across steps in diffusion process, and we directly calculate $P(x_{t-1}|x_t, x_0)$. We make new hypothesis that $P(x_{t-1}|x_t, x_0)$ is from a Gaussian distribution.

![](DDIM.png)

The training of DDIM is inditical to DDPM (1000 steps). During inference, denoising for 1000 steps is no longer necessary. Moreover, with $\deta_t$ not engaging in any of the diffusion process, it enables deterministic generation and interpolation between images when $\delta_t=0$.

However, one-step generation is still impossible. The DDIM denoise process is highly dependent on the Gaussian hypothesis which requires the denoise step to be not too large, only when the denoise formulation is valid. In another perspective, the sampling process is non-linear, it is almost impossible to get a linear result with huge step gap.

### SDXL-Turbo
Includes distillation and adversial techniques to improve training efficiency (reduce sampling steps). It learns to achieve how to sample in the best direction from a well-pretrained teacher model.
![](SDTurbo.png)

1. Distillation Loss: The distillation process is in noise space (the distillation loss is caculated between predicted noises).
2. Adversarial Loss: Discriminator between student network's prediction and the real image, which is actually a DINO encoder. This loss ensures high fidelity. This process is in image space.

#### Score Distillation Sample Loss
Key idea: use a pretrained and frozen Diffusion model to be a scorer of a generation model. It adds noise to a generated picture and uses pretrained SD to predict the added noise. If the predicted image of the generation model has good quality, the predicted noise should be the same (or at least very similar) to the added noise. If the gap between two noise images is large, it considers the generated image bad. 
> It is similar to discriminator in GAN but different because SD is frozen.

Using it as a loss, the actual idea is using a pretrained SD to distill a generator. In DreamFusion, the generator is a Text-to-3D model. For a single-step generation model, we can calculate the $x_0$ under each predicted noise $\epsilon_t$ at time step $t$ given the culmulated noise adding equation during diffusion process, and thus we can calulate the SDS loss for each predicted noise to train the model.

![](SDSLoss1.png)
![](SDSLoss2.png)

To prevent redundant and complicated gradient caculation, the term $\hat{\epsilon}_ \phi(x_t, t, c_ \theta) - \epsilon$ is stop-gradient.

## SD3
### Rectified Flow
A newer paradigms in generative modeling that builds on diffusion models but tries to simplify and accelerate the generation process.

Modeling the generation process as the motion field (along time $t$) of each pixel. The neural network has to predict the motion direction and velocity of each pixel at time $t$. Imagining a generation process in a continuous time period:

$$x_t=x_0+\int_0^1 \mu_t(x_s,s)ds$$
where $x_0$ is the original image, $x_t$ is the final generated image, $s$ is the time step, and $\mu_t(x_s,s)$ gives the velocity and direction (since it is a vector), which is the target of the modeling. During this whole process, noise is not necessary.

Forward process: $x_t = a_t \times x_0 + b_t\times \epsilon, \ t\in [0,1]$ \\
Velocity field: $\mu_t(\epsilon, x_0)=\frac{dx_t}{dt}=a'_t\times x_0+b'_t \times \epsilon$. In practice, $a_t$ and $b_t$ are defined  manually.

![](RectifiedFlow.png)

In the shown example, the direction is irrelevant to time $t$, which means the velocity and direction is consistent (that is why it is called RECTIFIED flow; otherwise it is called Flow Matching).

We can reformulate the definition from reverse direction: 
$$x_0=\frac{x_t-b_t}{a_t}\epsilon$$
$$\mu_t(\epsilon, x_0)=-\frac{x_t-b_t}{a_t}\epsilon +\epsilon=\frac{a_t'}{a_t}x_t+b_t(-\frac{a_t'}{a_t}+\frac{b_t'}{b_t})\epsilon$$
making the network a **noise predictor** again.

The difference between diffusion and rectified flow:
![](FlowSampling.png)

> SNR is defined as $\lambda_t=log(\frac{a_t^2}{b_t^2})$, s.t. $\lambda'_t=2(\frac{a_t'}{a_t}-\frac{b_t'}{b_t})$, which is proportional to the coefficient term in above equation

The loss function is defined as:
![](RectifiedFlowLoss.png)

### DiT
![](DiT.png)
It replaces the U-Nets by Transformers, proving the scalability of transformers. The scale is controlled by the condition (encoded by MLP).

## SD3: Multi-modal DiTs
![](MMDiTs.png)
Motivation: text and image are the same important.
- Input: text enbedded by different text encoders + noisy latent;
- MM-DiT block: image and text go through different structure, fused by cross-attention

## Summary
![](Summary.png)