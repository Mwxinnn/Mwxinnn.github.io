---
title: Optimizer
layout: single
date: 2024-08-09
priority: 1
---

> *Contents of this section are partially from [PyTorch official documentation](https://pytorch.org/docs/stable/optim.html).*

## Optimizer Overview
 Notation: $\theta_t$ the weights of $t$-th iteration, $\theta_{t-1}$ the weights of $t-1$-th iteration, $\gamma$ learning rate. We assume gradient $g_t<0$ here.

| PyTorch Doc | Note | Ada. LR | Elem. wise | Opt. Momentum |
|-------------|------|-------------|--------------|-------------------|
|[**Stochastic Gradient Descent** (SGD)](https://pytorch.org/docs/stable/generated/torch.optim.SGD.html#torch.optim.SGD) | $\theta_t=\theta_{t-1}+\gamma g_t$ (with optional first-order momentum) | ❌ | ⭕️ | ⭕️ |
|[**Resilient Propagation** (Rprop)](https://pytorch.org/docs/stable/generated/torch.optim.Rprop.html#torch.optim.Rprop) | Focusing on the sign of the gradient rather than its magnitude. | ❌ | ⭕️ | ❌ | 
| [**Adaptive Gradient** (Adagrad)](https://pytorch.org/docs/stable/generated/torch.optim.Adagrad.html#torch.optim.Adagrad) | Adaptive LR: accumulated sum of the squared gradients. LR keeps decaying. | ⭕️  | ⭕️ | ❌ | 
| [**Adadelta** (Adadelta)](https://pytorch.org/docs/stable/generated/torch.optim.Adadelta.html#torch.optim.Adadelta) |Adaptive LR: with the help of another parameter. It maintains a moving average of both the squared gradients and the squared parameter updates. | ⭕️ | ⭕️ | ❌ | 
|[**RMSprop**](https://pytorch.org/docs/stable/generated/torch.optim.RMSprop.html#torch.optim.RMSprop) | Adaptive LR: moving average of the squared gradients as normalization for learning rate. | ⭕️ | ⭕️ | ⭕️ |


### Adaptive LRs
Most of these adaptive methods takes historical gradients (or their statistic) as a normalizer to LR.

#### Adagrad:
$$
sum_t \leftarrow sum_{t-1} + {g_t}^2 
$$
$$
\theta_t=\theta_{t-1}+\frac{\tilde{\gamma}}{\sqrt{sum_t}+\epsilon}\  g_t
$$
where $\tilde{\gamma}$ means optionally decayed learning rate.

Adagrad maintains a running sum of the squared gradients for each parameter. This accumulated value is used to scale the learning rate, which helps in adapting to the relative importance of each parameter. However, **it leads to monotonically decreasing learning rate problem**.

#### Adadelta
$$
\nu_t \leftarrow \nu_{t-1}\ \rho + {g_t}^2\ (1-\rho) 
$$
$$
\Delta\kappa_t \leftarrow \frac{\sqrt{\upsilon_{t-1}+\epsilon}}{\sqrt{\nu_t+\epsilon}}\ g_t
$$
$$
\upsilon_t \leftarrow \upsilon_{t-1}\ \rho + {\Delta\kappa_t}^2\ (1-\rho) 
$$
where $\rho$


#### RMSprop 

adjusts the learning rate for each parameter individually based on the average of recent magnitudes of the gradients for that parameter. This means that parameters with larger gradients will have their learning rates reduced, while those with smaller gradients will have their learning rates increased.

| **Adam**       | Adam algorithm, which combines benefits of AdaGrad and RMSProp.  |
| **AdamW**      | AdamW algorithm, a variant of Adam with weight decay applied directly to weights. |
| **Adamax**     | Adamax, a variant of Adam based on the infinity norm.             |
| **NAdam**      | NAdam, combining Adam with Nesterov Accelerated Gradient.         |
| **Adadelta**   | Adadelta algorithm, which reduces the aggressive decrease in learning rates of Adagrad. |
| **LBFGS**      | L-BFGS algorithm, an optimization method based on limited-memory quasi-Newton methods. |
| **SparseAdam** | SparseAdam, a masked version of Adam suitable for sparse gradients. |
| **RAdam**      | RAdam, a variant of Adam with a dynamic adjustment of the learning rate. |

