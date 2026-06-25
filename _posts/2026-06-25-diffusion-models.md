---
layout: post
title: Denoising Diffusion Probabilistic Models
description: My current understanding on current state of the art image generation
  models (diffusion models), the math behind them and my intuitive view of the dynamics.
categories:
- Research
- Diffusion and RL
tags:
- diffusion
- AI
- image generation
- research
hidden: false
math: true
date: 2026-06-25 21:52 +0530
---
## Introduction

With the recent advances in Generative AI and Image generation specifically, we all are able to play and generate images for various tasks. These tasks vary across a large spectrum starting from generating presentations using AI to generating and editing images just for fun (we all are aware of the ghibili trend). In this context and for my research as well I started exploring how actually these images are generated. A few of them are VAEs (Variational Auto Encoders), GANs (Generative Adversarial Networks), Autoregressive Generation and the current state of the art Diffusion Models. I read the paper which first introduced diffusion for image generation - [DDPM](https://arxiv.org/abs/2006.11239)

## What do Diffusion Models do?

I was very surprised when I first heard about diffusion models and how they actually generate images. The generative process goes as follows - they first start with a random noisy image (a bunch of random pixels) then they predict the noise in the current image and then removes it to get a less noisy image (ideally) and this process is repeated for a finite number of steps (pre-determined during the training phase) to finally arrive at the clean image. Sounds ridiculous right? How can we generate images from noise? At least it was not clear for me for the first time I heard this.

Now let us look at what happens during the training phase. We take clean image (natural images or some image dataset) then for each image we (systematically) add a small noise at each step (for some finite number of steps) so that we will finally arrive at a completely random noisy image (recall that, in the generating phase we started from random noise). Since we know how we are noising the images at each step we learn to denoise the image given a noisy image at each step and this helps us to sample a random noisy image and then generate the desired images (from the image distribution of the training data).

At first this sounded like magic but if we look at the math behind it, that makes much more sense and also tells us why this works.

## DDPM - Denoising Diffusion Probabilistic Models

Now we will see how the Diffusion models are mathematically modelled and what is the loss function used and a final form of the loss function and what does that intuitively mean.

### Mathematical Model of Diffusion Models

As mentioned earlier the diffusion process occurs over a finite number of steps which is pre-determined or chosen by us. Let us consider a diffusion process with $T$ steps.

Starting from $x_0$ the image from the data set we destroy this image to pure noise $x_T ~ N(0, I)$ similar to that sampled from a standard Gaussian. We call the process of noising the image as the forward process and the denoising process as the reverse process.

The forward process is described as follows:
- $x_0$ is sampled from the image data.
- $x_{t+1} = \sqrt{1 - \beta_t}x_t + \sqrt{\beta_t}\epsilon$ where $\epsilon \sim N(0, I)$ which came from the reparametrization of the sampling probability distribution $q(x_{t+1} \mid x_t) = N(\sqrt{1 - \beta_t}x_t, \beta_tI)$.
- This is modelled as the markov chain i.e., $q(x_{t+1}\mid x_t) = q(x_{t+1}\mid x_t, x_{t-1}, \dots, x_0)$
- Therefore we can write $q(x_{1:T} \mid x_0) = \prod\limits_{i=1}^{T} q(x_i \mid q_{i-1})$

The reverse process is described as follows:
- $x_T ~ p(x_T)$ where p(x_T) is a simple random distribution, in most cases $N(0. I)$.
- The reverse process is also modelled as the markov process given by $p(x_{0:T}) = p(x_T)\cdot \prod\limits_{t=1}^{T} p_{\theta}(x_{t-1} \mid x_t)$

Now we should learn these reverse probabilities so that we can randomly sample a noising image using $p(x_T)$ and iteratively denoise it to obtain a neat image $x_0$.

### Loss Function for Diffusion Models

The loss function for diffusion models is taken to be the log-likelihood of generation of the images from the sample distribution given by $$\mathcal{L} = \mathbb{E}_{x_T \sim p(x)}[-\log{p(x_{0:T})}]$$. This loss function is hard to estimate therefore we use an alternative for this loss function using variational bound as follows:

$$
\begin{align*}
\mathcal{L} &= \mathbb{E}_{x_T \sim p(x)}[-\log{p(x_{0:T})}] \tag{1} \label{eq:start} \\
&\ge \mathbb{E}_{x_0 \sim q}\left[-\log{\frac{p(x_{0:T})}{q(x_{1:T} \mid x_0)}}\right] \tag{2} \label{eq:elbo} \\
&= \mathbb{E}_{x_0 \sim q}\left[-\log{p(x_T)}-\sum\limits_{t\ge 1}\log{\frac{p(x_{t-1} \mid x_t)}{q(x_t \mid x_{t-1})}}\right] \\
&= \mathbb{E}_{x_0 \sim q}\left[-\log{p(x_T)}-\sum\limits_{t > 1}\log{\frac{p(x_{t-1} \mid x_t)}{q(x_t \mid x_{t-1})}} -\log{\frac{p(x_0 \mid x_1)}{q(x_1 \mid x_0)}} \right] \tag{3} \label{eq:pre-bayes-step}\\
&= \mathbb{E}_{x_0 \sim q}\left[-\log{\frac{p(x_T)}{q(x_T \mid x_0)}}-\sum\limits_{t > 1}\log{\frac{p(x_{t-1} \mid x_t)}{q(x_{t-1} \mid x_{t}, x_0)}} -\log{p(x_0 \mid x_1)} \right] \tag{4} \label{eq:bayes-step} \\
&= \mathbb{E}_{x_0 \sim q}\left[ \underbrace{D_{KL}(q(x_T \mid x_0) \| p(x_T))}_{L_T} +\sum\limits_{t > 1} \underbrace{D_{KL}(q(x_{t-1} \mid x_t, x_0) \| p(x_{t-1} \mid x_t))}_{L_{t-1}} -\underbrace{\log{p(x_0 \mid x_1)}}_{L_0} \right] \tag{5} \label{eq:final}
\end{align*}
$$

We use modified form of equation ($\ref{eq:final}$) as the final loss function to train the diffusion models. Before intuitively understanding the final equation we will first see how we arrived at equation ($\ref{eq:bayes-step}$) from equation ($\ref{eq:pre-bayes-step}$). This is a simple application of bayes rule that is the probability that $x_{t-1}$ is generated given that $x_t$ is generated starting from $x_0$.

$$q(x_{t-1} \mid x_t, x_0) = \frac{q(x_t \mid x_{t-1}) \cdot q(x_{t-1} \mid x_0)}{q(x_t \mid x_0)} \implies \frac{1}{q(x_t \mid x_{t-1})} = \frac{1}{q(x_{t-1} \mid x_t, x_0)}\cdot \frac{q(x_{t-1} \mid x_0)}{q(x_t \mid x_0)}$$

Substituting the above in equation ($\ref{eq:pre-bayes-step}$) and simplification we get to equation ($\ref{eq:bayes-step}$).

### Understanding the Final Loss Equation

The loss function in the above equation ($\ref{eq:final}$) looks as follows:

$$L = \mathbb{E}_{x_0 \sim q}\left[ \underbrace{D_{KL}(q(x_T \mid x_0) \| p(x_T))}_{L_T} +\sum\limits_{t > 1} \underbrace{D_{KL}(q(x_{t-1} \mid x_t, x_0) \| p(x_{t-1} \mid x_t))}_{L_{t-1}} -\underbrace{\log{p(x_0 \mid x_1)}}_{L_0} \right]$$

Intuitively, KL Divergence can be seen as a proxy for how far are the two distributions. With this perspective let us look at the various terms of the loss function.

* Minimising $L_T$ is equivalent to bringing $p(x_T)$ closer $q(x_T \mid x_0)$ which means we are trying to bring the distribution from which we are sampling the noise for generation closer to final distribution obtained by noising the images.
* Minimising the term $L_{t-1}$ is equivalent to bringing $p(x_{t-1} \mid x_t)$ closer to $q(x_{t-1} \mid x_t, x_0)$ which means we are trying to bring the denoising distribution at time step $t$ to the one which is used to noise in the reverse process.
* The last term in the loss $L_0$ indicates the likelihood of generating an image from the original distribution given the last but one noisy image.

### Diffusion as Noise Prediction

The [DDPM](https://arxiv.org/abs/2006.11239) paper makes some interesting assumptions on the noising process and further simplifies the Diffusion loss $L$ as given in equation ($\ref{eq:final}$) to show that the loss is equivalent to predict the noise at each step of the reverse or generative process.

### Current Landscape of Diffusion
In the current day almost all the SOTA image generation models use Diffusion Models for image generation, image editing and Text-to-Image generation which mainly falls into the paradigm of conditional Diffusion. Recently, there has been lot of work focusing on fine-tuning diffusion models to improve the quality of images generated which also gave rise to some interesting RL algorithms like DDPO, DPOK. Another close cousin of diffusion models are flow matching models and they are also being extensively used in the image generation.

There are a lot of interesting questions to answer here which include (but not limited to) fine-tuning diffusion models, diffusion models for video generation, ensuring character consistency in image generation, correct text rendering in images and Diffusion in latent spaces which can be used to create synthetic data for robot training.

---

PS: I will be adding the further simplication given by the diffusion paper soon.

