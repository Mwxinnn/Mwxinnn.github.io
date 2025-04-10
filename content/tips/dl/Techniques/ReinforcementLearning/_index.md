---
title: Reinforcement Learning
layout: single
date: 2025-04-10
priority: 1
---
[Material Source](http://t.cn/RBtg1O2)
## Overview
![](definition.png)
The model learns to take the action that maximizes the expected reward in an episode. However, the reward is sparse (can only acquired at certain steps / after some steps). 

- supervised learning: learn from the teacher
- Reinforcement learning: learn from experience

Challenges:
- There can be randomness in environment.
- Reward delay
- An action affects the subsequent data.

### Method
![](category.png)
The prediction is stochastic instead of deterministic to prevent repeat action.
![](method.png)

## Target
### Actor $\pi(\cdot)$
![](target.png)
![](target2.png)
![](target3.png)
![](target4.png)
![](target5.png)
![](target6.png)
![](target7.png)

Understanding: optimizing a weighted sum of multiple sequential classification targets (weighted by overall reward).

### Critic $V^\pi(s)$
A critic does not determine the action. Given an actor $\pi$, it evaluates how good it is.
#### State value function
It outputs the expected *cumulated* reward for actor $\pi$ after taking an action and reaching state $s$. It takes the state $s$ after action as the input. Different actor can have different output after a same observation and action. We can use **Monte-Carlo based approach** to formulate it as a regression problem. We can also use **Temporal-difference approach**.
![](TemporalDifference.png)

#### State-action value function
It outputs the expected *cumulated* reward for actor $\pi$ taking an action $a$ in state $s$. It takes the state $s$ and the action $a$ as the input. One can use this function for finding a better actor $\pi$, whihc is termed **Q-Learning**.
![](q-learning.png)
$\pi'$ takes the better action $a^*$ as the next action.

## PPO: Proximal Policy Optimization
![](baseline.png)
If all the actions have positive rewards, the model will favor the actions which have been sampled more frequently, leading to the decline of probablity of less-frequent samples. Adding a baseline can ensure not every action is positive. The baseline can be a model or a statistic value. We define advantage function as: $$A^\theta(s_t,a_t)=R(\tau^t)-b$$

Except using a **baseline** to prevent the rewards for every actions being positive, it is also reasonable to assign different credict for different action step. Because the step will influence the subsequent steps but will not influence the steps before, it is not optimal to use the total reward as the weight for summing up steps. Moreover, current step will have less effects on steps further away and more effects on next a few steps. A **decay** factor is applied.
![](decay.png)

### off-policy
![](onoff-policy.png)
On-policy learning is very time-consuming since the agent $\pi_{\theta}$ has to interact with the environment to sample the data before each update. Off-policy aims to use another actor $\pi_{\theta'}$ to sample data and provide information for the learning process of $\pi_{\theta}$.

#### Importance Sampling
![](ImportanceSampling.png)
However, if there is a huge difference between $p(x)$ and $q(x)$, the variance between two distribution would be large and the training will be influenced hugely by sampling.

In off-policy learning, this technique is employed:
![](is4off.png)

### PPO1
![](PPO.png)
Notice: The KL divergence is between different action distributions instead of parameter distributions.
![](PPO-al.png)

### PPO2
![](PPO2.png)
If $A>0$, we want to maximize 