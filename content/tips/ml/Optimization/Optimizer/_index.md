---
title: Optimizer
layout: single
date: 2024-08-09
priority: 1
---

> *Contents of this section are partially from [PyTorch official documentation](https://pytorch.org/docs/stable/optim.html).*

## Optimizer Overview
 Notation: $\theta_t$ the weights of $t$-th iteration, $\theta_{t-1}$ the weights of $t-1$-th iteration, $\gamma$ learning rate. We assume gradient $g_t>0$ here.
 > `Opt. Momentum` means if PyTorch has an implementation of momentum tracking the direction of update.

| PyTorch Doc | Note | Ada. LR | Elem. wise | Opt. Momentum |
|-------------|------|-------------|--------------|-------------------|
|[**Stochastic Gradient Descent** (SGD)](https://pytorch.org/docs/stable/generated/torch.optim.SGD.html#torch.optim.SGD) | $\theta_t=\theta_{t-1}-\gamma g_t$ (with optional first-order momentum) | ❌ | ⭕️ | ⭕️ |
|[**Resilient Propagation** (Rprop)](https://pytorch.org/docs/stable/generated/torch.optim.Rprop.html#torch.optim.Rprop) | Focusing on the sign of the gradient rather than its magnitude. | ❌ | ⭕️ | ❌ | 
| [**Adaptive Gradient** (Adagrad)](https://pytorch.org/docs/stable/generated/torch.optim.Adagrad.html#torch.optim.Adagrad) | Adaptive LR: accumulated sum of the squared gradients. LR keeps decaying. | ⭕️  | ⭕️ | ❌ | 
| [**Adadelta** (Adadelta)](https://pytorch.org/docs/stable/generated/torch.optim.Adadelta.html#torch.optim.Adadelta) |Adaptive LR: with the help of another parameter. It maintains a moving average of both the squared gradients and the squared parameter updates. | ⭕️ | ⭕️ | ❌ | 
|[**RMSprop**](https://pytorch.org/docs/stable/generated/torch.optim.RMSprop.html#torch.optim.RMSprop) | Adaptive LR: moving average of the squared gradients as normalization for learning rate. | ⭕️ | ⭕️ | ⭕️ |
|[**Adaptive Moment Estimation** (Adam)](https://pytorch.org/docs/stable/generated/torch.optim.Adam.html#torch.optim.Adam) | RMSprop + First and Second Momentum | ⭕️ | ⭕️ | ❌ |
|[**Decoupled Weight Decay Regularization** (AdamW)](https://pytorch.org/docs/stable/generated/torch.optim.AdamW.html#torch.optim.AdamW) | RMSprop + First and Second Momentum | ⭕️ | ⭕️ | ❌ |


## Adaptive LRs
Most of these adaptive methods takes historical gradients (or their statistic) as a normalizer to LR, including:
### Adagrad:
Adaptation normalizer:
$$
sum_t \leftarrow sum_{t-1} + {g_t}^2 
$$
Parameter update:
$$
\theta_t\leftarrow \theta_{t-1}-\frac{\tilde{\gamma}}{\sqrt{sum_t}+\epsilon}\  g_t
$$
where $\tilde{\gamma}$ means optionally decayed learning rate.

Adagrad maintains a running sum of the squared gradients for each parameter. This accumulated value is used to scale the learning rate, which helps in adapting to the relative importance of each parameter. However, **it leads to monotonically decreasing learning rate problem**.


### RMSprop 
Adaptation normalizer:
$$
\begin{aligned}
& v_t \leftarrow  v_{t-1}\ \rho + {g_t}^2\ (1-\rho) \\\
&if\ centered: \\\
&\quad g_t^{avg} \leftarrow g_t^{avg} \alpha +(1-\alpha)g_t \\\
&\quad \tilde{ v_t} \leftarrow  v_t -(g_t^{avg})^2 \\\
&else: \\\
&\quad \tilde{ v_t} \leftarrow  v_t
\end{aligned}
$$
where $\alpha$ is the hyper-parameter controling the running average (usually close to 1, e.g., 0.9).

Parameter update:
$$
\theta_t \leftarrow \theta_{t-1}-\frac{\gamma}{\sqrt{\tilde{ v_t}}+\epsilon}\  g_t
$$

Parameters with larger gradients will have their learning rates reduced, while those with smaller gradients will have their learning rates increased.

### Adadelta
Adaptation normalizer:
$$
 v_t \leftarrow  v_{t-1}\ \rho + {g_t}^2\ (1-\rho) \\\
\Delta x_t \leftarrow \frac{\sqrt{ u_{t-1}+\epsilon}}{\sqrt{ v_t+\epsilon}}\ g_t \\\
 u_t \leftarrow  u_{t-1}\ \rho + {\Delta x_t}^2\ (1-\rho) 
$$
where $\rho$ is a hyper-parameter (set to 0.9 as default), and $\Delta x_t$ is a auto-updated parameter for adapting learning rate. The adapted learning rate can be taken as:
$$
\Delta x_t \leftarrow V_{t-1}[ x]\cdot\frac{g_t}{E_t[g]}
$$
If a parameter's gradient is very large at iteration $t$, it would be constrained by $E_t[g]$.

Parameter update:
$$
\theta_t\leftarrow \theta_{t-1}-\gamma \Delta x_t
$$
Adadelta aims to address the diminishing learning rate problem inherent in AdaGrad by restricting the window of accumulated past gradients and introducing a mechanism to **adjust learning rates on a per-dimension basis** without requiring a global learning rate.


## Adam and Varients
### Adam
Adam aims to combine the benefits of RMSprop and Momentum by using both the **first momentum (mean)** and **second momentum (uncentered variance)** of the gradients to adapt the learning rate for each parameter. The algorithm can be summaried as:
1. Update First Momentum: $$m_t\leftarrow \beta_1m_{t-1}+(1-\beta_1)g_t$$
2. Update Second Momentum: $$ v_t\leftarrow \beta_2 v_{t-1}+(1-\beta_2)g_t^2$$
3. Bias Correction: 
$$
\hat{m_t}\leftarrow \frac{m_t}{1-\beta_1^t},\quad \hat{ v_t}\leftarrow \frac{ v_t}{1-\beta_2^t}
$$
4. Normalize LR and Update Parameters:
$$
\theta_{t+1} = \theta_t - \hat{m}_t \frac{\gamma}{\sqrt{\hat{v}_t} + \epsilon}
$$

For the step of Bias Correction, both $m_t,\ \hat{m_t}$​ and $v_t,\ \hat{v_t}$​ are **biased towards zero**, especially in the early stages of training, because they are initialized at zero. To correct this bias, Adam applies bias-correction factors, resulting in $\hat{m_t}$​ and $\hat{v_t}$​.\
By taking the first moment $\hat{m}_t$​ into account, the update rule accounts for the average gradient **direction**, ensuring that the learning step is proportional to the overall trend of the gradients rather than just the most recent gradient.\
The second moment, $\hat{v_t}$​, is similar to RMSprop above, tracks the average magnitude of gradient.

> Note: There is momentum option in RMSprop which is actually tracking the average update direction instead of gradient direction.
There are some problems of Adam:
1. Second momentum diminishing leads to explosive LR -- Solution: Amsdrad
2. Interaction of Weight Decay with Adaptive Learning Rates -- Solution: AdamW

#### Amsdrad in Adam

In Adam, the adaptive learning rate can become too large due to the second moment estimates (the squared gradients) being dominated by large gradients in non-convex optimization problems. The second moment estimate $\hat{v}_t$​ (which keeps track of the moving average of squared gradients) can decrease even if the gradient values are large, because the moving average can be dominated by earlier, smaller gradients. This can cause the effective learning rate to become too high, leading to instability in training.
> RMSprop inherently assumes that the second moment's scaling is sufficient.

AMSGrad modifies Adam by maintaining the maximum of past squared gradients (second moment estimate) rather than the exponentially decaying average. This ensures that the effective learning rate does not increase over time, leading to more stable and reliable convergence.

Second Momentum Calculation:
$$v_t\leftarrow \beta_2 v_{t-1}+(1-\beta_2)g_t^2\\\
\hat{v_t}=max(\hat{v_{t-1}},v_t) $$

### AdamW

| **Adamax**     | Adamax, a variant of Adam based on the infinity norm.             |
| **NAdam**      | NAdam, combining Adam with Nesterov Accelerated Gradient.         |
| **LBFGS**      | L-BFGS algorithm, an optimization method based on limited-memory quasi-Newton methods. |
| **SparseAdam** | SparseAdam, a masked version of Adam suitable for sparse gradients. |
| **RAdam**      | RAdam, a variant of Adam with a dynamic adjustment of the learning rate. |

