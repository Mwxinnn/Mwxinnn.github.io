---
title: Momentum, Moment and Nesterov Momentum
layout: single
date: 2024-08-12
priority: 5
---

## Momentum
Momentum is a technique used in optimization to accelerate the convergence of gradient descent by smoothing out the oscillations in the optimization path. The core idea of momentum is to accumulate **a moving average of past gradients** and use this average to update the parameters, rather than relying solely on the current gradient. It helps in pushing the parameters in the consistent direction of descent by giving more weight to gradients from previous steps.

Momentum calculation: $v_t = \beta v_{t-1} + (1 - \beta) g_t$ \
Parameter update: $\theta_{t} = \theta_{t-1} - \gamma v_t$

## Moment
Moment in the context of optimization, particularly in algorithms like Adam, refers to the **statistics** of gradients over time, typically their mean and variance. These statistics help in adapting the learning rate for each parameter based on how the gradient behaves over time.
### First Moment (Mean of Gradients):
$$ m_t = \beta_1 m_{t-1} + (1 - \beta_1) g_t$$
The first moment estimate $m_t$ is an exponentially weighted moving average (EWMA) of the gradients. It captures the mean or expected value of the gradients, smoothing out the noise from individual gradient estimates.

### Second Moment (Mean Square of Gradients)
$$ v_t = \beta_2 v_{t-1} + (1 - \beta_2) g_t^2$$

### Bias Correction
Both $m_t$​ and $v_t$​ are **biased towards zero**, especially in the early stages of training, because they are initialized at zero. To correct this bias, we should apply bias-correction factors, resulting in $\hat{m_t}$​ and $\hat{v_t}$​:
$$\hat{m_t}\leftarrow \frac{m_t}{1-\beta_1^t},\quad \hat{ v_t}\leftarrow \frac{ v_t}{1-\beta_2^t}$$
This correction ensures that the moments reflect the true statistics of the gradients, especially during the early stages of training.

### Adam Analysis
Expectation and Variance have this relationship:
$$\frac{E^2[t]}{E[t^2]}=\frac{E[t^2]-Var[t]}{E[t^2]}=1-\frac{Var[t]}{E[t^2]}$$
Accordingly, we can decomposite the variance part in $v_t$,$$\frac{m_t}{\sqrt{v_t}}\approx \sqrt{1-\frac{Var[g]}{v_t}}$$

This helps us to understand Adam optimizer, which updates parameters by this equation (ignoring bias correction and $\epsilon$):
$$\theta_t = \theta_{t-1} - \gamma \frac{m_t}{\sqrt{v_t}} =\theta_{t-1} -\gamma \sqrt{1-\frac{Var\[g\]}{v_t}}$$
where $Var\[g\]$ refers to the historical variance.

It can be obviously seen that Adam **stablizes training** by adjusting the learning rate inversely proportional to the variance, so that parameters with high variance gradients are updated more cautiously (lower learning rate).

## Nesterov Momentum
Nesterov Momentum is an enhancement of the standard momentum method that introduces a "lookahead" mechanism. Instead of applying the momentum to the current gradient, Nesterov momentum first moves the parameters in the direction of the accumulated momentum, **then calculates the gradient at this new, "lookahead" position**. This allows the optimizer to make more informed updates.

Current momentum calculation: $v_t = \beta v_{t-1} + (1 - \beta) g_t$ \
Lookahead: $\tilde{v_t} = v_t = \beta v_{t} + (1 - \beta) g_t$ \
Parameter update: $\theta_{t} = \theta_{t-1} - \gamma \tilde{v_t}$

The "lookahead" step in Nesterov momentum allows the optimizer to anticipate where the parameters are heading, which often results in faster convergence and more stable updates.

