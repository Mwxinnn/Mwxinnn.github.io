---
title: Optimizer
layout: single
date: 2024-08-09
priority: 1
---

> *Contents of this section are partially from [PyTorch official documentation](https://pytorch.org/docs/stable/optim.html).*

## Optimizer Overview

| Optimizer      | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| **SGD**        | Stochastic Gradient Descent, optionally with momentum.           |
| **Adam**       | Adam algorithm, which combines benefits of AdaGrad and RMSProp.  |
| **AdamW**      | AdamW algorithm, a variant of Adam with weight decay applied directly to weights. |
| **Adamax**     | Adamax, a variant of Adam based on the infinity norm.             |
| **NAdam**      | NAdam, combining Adam with Nesterov Accelerated Gradient.         |
| **RMSprop**    | RMSprop, adapts learning rates based on recent gradient magnitudes. |
| **Adagrad**    | Adagrad algorithm, adjusts learning rates based on frequency of updates. |
| **Adadelta**   | Adadelta algorithm, which reduces the aggressive decrease in learning rates of Adagrad. |
| **Rprop**      | the Resilient Propagation algorithm, which adapts learning rates based on the sign of the gradient. |
| **ASGD**       | Averaged Stochastic Gradient Descent, averaging the parameters over time. |
| **LBFGS**      | L-BFGS algorithm, an optimization method based on limited-memory quasi-Newton methods. |
| **SparseAdam** | SparseAdam, a masked version of Adam suitable for sparse gradients. |
| **RAdam**      | RAdam, a variant of Adam with a dynamic adjustment of the learning rate. |

