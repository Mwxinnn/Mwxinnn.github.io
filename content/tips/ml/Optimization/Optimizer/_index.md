---
title: Optimizer
layout: single
date: 2024-08-09
priority: 1
---

> *Contents of this section are partially from [PyTorch official documentation](https://pytorch.org/docs/stable/optim.html).*
## Pre-requisites
[Momentum, Moment and Nesterov Momentum]({{<relref "/tips/ml/Optimization/Optimizer/Prerequisite.md">}})

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
|[*Adamax*](https://pytorch.org/docs/stable/generated/torch.optim.Adamax.html#torch.optim.Adamax) | For sparse gradients or  optimizers with an unbounded infinity norm $L_\infin $ for the second moment | ⭕️ | ⭕️ | ❌ |
|[*Nesterov-accelerated Adaptive Moment Estimation* (NAdam)](https://pytorch.org/docs/stable/generated/torch.optim.NAdam.html#torch.optim.NAdam) |  Nesterov momentum + Adam | ⭕️ | ⭕️ | ❌ |
|[*Limited memory BFGS* (LBFGS)](https://pytorch.org/docs/stable/generated/torch.optim.LBFGS.html#torch.optim.LBFGS) | quasi-Newton methods; approximates the inverse of the Hessian matrix |  ❌ | ❌ | ❌ |

## Adaptive LRs
Most of these adaptive methods takes historical gradients (or their statistic) as a normalizer to LR, but differs in the way they normalize (called normalizer below). These methods including:
### Adagrad:
Adaptation normalizer:
$$sum_t \leftarrow sum_{t-1} + {g_t}^2 $$
Parameter update:
$$\theta_t\leftarrow \theta_{t-1}-\frac{\tilde{\gamma}}{\sqrt{sum_t}+\epsilon}\  g_t$$
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
$$\theta_t \leftarrow \theta_{t-1}-\frac{\gamma}{\sqrt{\tilde{ v_t}}+\epsilon}\  g_t$$

Parameters with larger gradients will have their learning rates reduced, while those with smaller gradients will have their learning rates increased.

### Adadelta
Adaptation normalizer:
$$
 v_t \leftarrow  v_{t-1}\ \rho + {g_t}^2\ (1-\rho) \\\
\Delta x_t \leftarrow \frac{\sqrt{ u_{t-1}+\epsilon}}{\sqrt{ v_t+\epsilon}}\ g_t \\\
 u_t \leftarrow  u_{t-1}\ \rho + {\Delta x_t}^2\ (1-\rho) 
$$
where $\rho$ is a hyper-parameter (set to 0.9 as default), and $\Delta x_t$ is a auto-updated parameter for adapting learning rate. The adapted learning rate can be taken as:
$$\Delta x_t \leftarrow V_{t-1}[ x]\cdot\frac{g_t}{E_t[g]}$$
If a parameter's gradient is very large at iteration $t$, it would be constrained by $E_t[g]$.

Parameter update:
$$\theta_t\leftarrow \theta_{t-1}-\sigma \Delta x_t$$
where $\sigma$ is a coefficient that scale delta before it is applied to the parameters (default=1).

Adadelta is similar to RMSprop, aiming to address the diminishing learning rate problem inherent in AdaGrad. But it does not require an explicitly set learning rate. It adjust learning rates ($\Delta x_t$) on a per-dimension basis without requiring a global learning rate.


## Adam and Varients
These methods also incorporate adaptive LR implementation, but they also includes moment operations.
### Adam
Adam aims to combine the benefits of RMSprop and Moment by using both the **first moment (mean)** and **second moment (uncentered variance)** of the gradients to adapt the learning rate for each parameter. The algorithm can be summaried as:
1. Update First Moment: $$m_t\leftarrow \beta_1m_{t-1}+(1-\beta_1)g_t$$
2. Update Second Moment: $$ v_t\leftarrow \beta_2 v_{t-1}+(1-\beta_2)g_t^2$$
3. Bias Correction: 
$$\hat{m_t}\leftarrow \frac{m_t}{1-\beta_1^t},\quad \hat{ v_t}\leftarrow \frac{ v_t}{1-\beta_2^t}$$
4. Normalize LR and Update Parameters:
$$\theta_{t} \leftarrow \theta_{t-1} - \hat{m}_t \frac{\gamma}{\sqrt{\hat{v}_t} + \epsilon}$$

By taking the first moment $\hat{m}_t$​ into account, the update rule accounts for the average gradient **direction**, ensuring that the learning step is proportional to the overall trend of the gradients rather than just the most recent gradient.\
The second moment, $\hat{v_t}$​, is similar to RMSprop above, tracks the average magnitude of gradient.\
Some further analysis is in [Moment]({{<relref "/tips/ml/Optimization/Optimizer/Prerequisite.md">}}).

> Compare to RMSprop: There is momentum option in RMSprop which is actually tracking the average update direction instead of gradient direction. Also, the actual value of the gradients of $t$-th iteration is no longer taken into consideration in Adam, which is replaced by the moving average of gradients (i.e. frist moment).

There are some problems of Adam:
1. Second moment diminishing leads to explosive LR ---- Solution: Amsgrad
2. Interaction of Weight Decay with Adaptive Learning Rates ---- Solution: AdamW

#### Amsgrad in Adam

In Adam, the adaptive learning rate can become too large due to the second moment estimates (the squared gradients) being dominated by large gradients in non-convex optimization problems. The second moment estimate $\hat{v}_t$​ (which keeps track of the moving average of squared gradients) can decrease even if the gradient values are large, because the moving average can be dominated by earlier, smaller gradients. This can cause the effective learning rate to become too high, leading to instability in training.
> RMSprop inherently assumes that the second moment's scaling is sufficient.

AMSGrad modifies Adam by maintaining the maximum of past squared gradients (second moment estimate) rather than the exponentially decaying average. This ensures that the effective learning rate does not increase over time, leading to more stable and reliable convergence.

Second Moment Calculation:
$$v_t\leftarrow \beta_2 v_{t-1}+(1-\beta_2)g_t^2\\\
\hat{v_t}\leftarrow max(\hat{v_{t-1}},v_t) $$

### AdamW
There is an optional weight decay ([L2 regularization]({{< relref "/tips/ml/Optimization/Overfitting.md">}})) in Adam, and also other optimizers, to prevent overfitting. It is implemented as:
$$
g_t\leftarrow g_t +\lambda \theta_{t-1}
$$
where $\lambda$ is a hyper-parameter controls weight decay rate. When updating parameters, a portion or $\theta_{t-1}$ is also taken in to account to limit the magnitude of $\theta_{t}$. Each update is slightly "pulled" back towards zero. 

The weight decay term is not directly scaled by the second moment estimate; instead, it is added to the gradient before applying the update. This can lead to inconsistent regularization effects because the weight decay interacts with the adaptive learning rates in a way that may not be optimal. The regularization strength can vary.

AdamW addresses these issues by decoupling weight decay from the gradient updates. Instead of adding the weight decay term to the gradient, **AdamW applies the weight decay directly to the parameters**. The algorithm can be summaried as:
1. Apply Weight Decay: $$\theta_t \leftarrow \theta_{t-1} - \gamma \lambda\theta_{t-1}$$
1. Update First Moment: $$m_t\leftarrow \beta_1m_{t-1}+(1-\beta_1)g_t$$
2. Update Second Moment: $$ v_t\leftarrow \beta_2 v_{t-1}+(1-\beta_2)g_t^2$$
3. Bias Correction: 
$$
\hat{m_t}\leftarrow \frac{m_t}{1-\beta_1^t},\quad \hat{ v_t}\leftarrow \frac{ v_t}{1-\beta_2^t}
$$
4. Normalize LR and Update Parameters:
$$
\theta_{t} \leftarrow \theta_{t} - \hat{m}_t \frac{\gamma}{\sqrt{\hat{v}_t} + \epsilon}
$$
The only difference is the first step, which also means weight decay in AdamW is no longer optional.

### Adamax
Adam uses the $L_2$​ norm (Euclidean norm) to compute the second moment of the gradients.
Adamax uses the $L_\infin$​ norm (maximum absolute value of the gradients), which can be more stable, especially when dealing with very large or very small gradient values.

The first moment estimation is the same with Adam: $$m_t\leftarrow \beta_1m_{t-1}+(1-\beta_1)g_t$$
while the second moment estimation is changed to: $$u_t = \text{max}(\beta_2 u_{t-1}, |g_t|)$$
and the update of the parameter is still: $$\theta_{t} \leftarrow \theta_{t-1} - m_t \frac{\gamma}{(1-\beta_1^t)u_t}$$

This optimizer is specially useful in large gradients and sparse gradients situations, where the $L_\infin$​ norm leads to more reliable convergence than the $L_2$​ norm used in Adam (large outliers).

### NAdam
NAdam is a variant of the Adam optimizer that incorporates [**Nesterov momentum**]({{<relref "/tips/ml/Optimization/Optimizer/Prerequisite.md">}}) into the Adam update rule. Ignoring bias correction, the idea can be summarized as:
$$ \theta_{t+1} = \theta_t - \eta \left( \frac{\beta_1 \hat{m}_t + (1 - \beta_1) g_t}{\sqrt{\hat{v}_t} + \epsilon} \right)$$
Decoupled Weight Decay is also optionally incorporated.


