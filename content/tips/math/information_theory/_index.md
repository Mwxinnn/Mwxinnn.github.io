---
title: Information Theory
layout: single
date: 2025-03-13
priority: 1
---

## Quantities of information
 $I(p_i)=log_a(\frac{1}{p_i})$, $a$ is the encoding bit number.
## Information entropy
$H(X)=\mathbb{E}[-log\ p(X)]$. 

In other words: (discrete) $H(X)=-\sum_{i=0}^{n}p_i\ log\ p_i$; (continuous) $H(X)=-\int p(x)\ log\ p(x)dx$. 

It measures the expected quantity of information, or the expected uncertainty of a distribution. Uniform distribution has the max entropy.

### Properties
1. $H(X)\geq 0$, with the minimum entropy value being 0, which occurs when the random variable is a one-hot vector.
2. $H(X)\leq log\ n$, with equality holding if and only if $X_i\sim Uniform(\frac1n)$。
3. **Maximum Entropy Principle:** Given precisely stated prior information, the probability distribution that best represents the current state of the system is the one with the highest entropy.
    - If both the mean and variance are given as prior information → Normal distribution
    - If only the mean is given as prior information → Exponential distribution
    - If no prior information is given → Uniform distribution

### The entropy of $y=f(x)$
1. If $y=g(x)$, then $H(Y)≤H(X)$, because multiple values of $X$ may map to the same $Y$. If $g(⋅)$ is invertible, then $H(Y)=H(X)$.
2. $H(Y∣X)=0$ if and only if $Y=f(X)$ (i.e., $Y$ is a deterministic function of $X$).
3. $H(X,g(X))=H(X)$.


## Conditional entropy
$H(X|Y)\leq H(X); \ H(X|Y)=H(X)\ if\ X\perp Y;$

With the information of $Y$, the uncertainty of $X$ would decrease, unless $X$ and $Y$ are independent.
$$H(X|Y)=\sum_xp(x)H(Y|X=x)=-\sum_xp(x)\sum_yp(y|x)log\ p(y|x)=-\sum_{x,y}p(x,y)log\ p(x|y);$$

### Properties
1. $H(X|Y)+H(Y|Z)\geq H(X|Z);$
1. $H(X,Y)=-\sum_{x,y}p(x,y)\ log\ p(x,y)=H(X)+H(X|Y);$
1. $H(X_2,...,X_{n}|X_1)=-\sum_{x_1,x_2,...,x_n}p(x_1,x_2,...,x_n)log\ p(x_2,...,x_n|x_1)$
    - (Chain Rule) $H(X_1,...,X_n)=H(X_1)+H(X_2|X_1)+H(X_3|X_1,X_2)+...+H(X_n|X_1,...,X_{n-1});$
    - For all $X_i$ independent from each other, $H(X_1,X_2,...,X_n)=\sum_iH(X_i);$
2. $H(Y|X)$ is not equal to $H(X|Y)$, but $H(X)-H(X|Y)=H(Y)-H(Y|X);$

## The variants of entropy
### Cross Entropy
Measure the similarity between two distributions. (Discrete)  $CE(p,q)=-\sum_{i=1}^np_i\ log\ q_i$, (Continuous) $CE(p,q)=-\int p(x)\ log\ q(x)dx$.

### Relative Entropy (KL divergence)
$$D(p||q)=\sum_x\ p(x)\ log\ \frac{p(x)}{q(x)}=\mathbb{E}_p[log\ \frac{p(X)}{q(X)}]=-H(X)-\mathbb{E}_p[log\ q(X)]=CE(p(X),q(X))-H(X)$$

> Relative Entropy is not symmetric, $D(p||q)$ is not equal to $D(q||p)$

### Intuitive Understanding
If $p(X)$ is the true data distribution and $q(X)$ is a predicted distribution, Cross-Entropy (CE) measures the expected information content assuming the data follows $q(X)$.
KL Divergence (KL divergence) quantifies the extra information required when assuming data follows $q(X)$, the additional information from $q(X) & p(x)$ divergence, beyond the inherent information in $p(X)$ itself.

In this process, $p(X)$ and $q(X)$ do not hold equal status, which is why both CE and KL divergence are asymmetric.

> $H(X)=-DL(p(X)||1_{Uni(X)})$, which means entropy can be understood as the KL divergence between $p(X)$ and a uniform distribution over $X$.

### **Jensen-Shannon Divergence**
$$JSD(p||q)=\frac12\ D(p||t)+\frac12\ D(q||t)=H(t)-\frac12[H(p)+H(q)], \quad t=\frac{p+q}2$$

### Jesen inequality
$\sum_i p_i\ f(x_i)\geq\ f(\sum_ip_i\ x_i)$

This is an important property of a convex function, and its proof follows directly from the definition of convexity. For $i=2$, if $p_1+p_2=1$, then: $$f(p_1\ x_1+p_2\ x_2)\leq p_1f(x_1)+p_2f(x_2)$$.

For more terms, the result can be derived using mathematical induction. If $f(x)$ is concave, the inequality is reversed.

### Properties
1. $D(p||q)=\mathbb{E}_p[log\frac{p(X)}{q(X)}]=\mathbb{E}_p[-log\frac{q(X)}{p(X)}]$
   
   $\geq-log\mathbb{E}_p[\frac{q(X)}{p(X)}]$ *(Jensen inequality)*
   
   $=-log\sum_ip(x_i)\frac{q(x_i)}{p(x_i)}$
   
   $=-log\sum_iq(x_i)= -log1=0$
   
   Equality holds if and only if $p(X)=q(X)$ (since Jensen’s inequality is applied to a linear function).
   
   This property can also be used to prove that the entropy of a uniform distribution is the maximum: Let $q(X)$ be a discrete uniform distribution, i.e. $q(X)=\frac1n$. Then, $-H(X)=D(p(X)||1)=\mathbb{E}_p[-log\frac{1}{np(X)}]\geq -log\mathbb{E}_p[\frac{1}{np(X)}]=-log\ n$. Equality holds if and only if $p(X)=\frac1n$.
   

2. Convexity of $D(p||q)$ holds with respect to both $p,q$.
   
   Consider two distributions $\vec{x_1}=(p_1,q_1),$ and $\vec{x_2}=(p_2,q_2)$, and let $a\in[0,1]$ such that：
   
   Define $f(\vec{x})=D(x_1||x_2)$. The function $f(\cdot)$ is convex, meaning: $$f(p_1\vec{x_1}+p_2\vec{x_2})\leq p_1f(\vec{x_1})+p_2f(\vec{x_2})$$. That is $$D(ap_1+(1-a)p_2||aq_1+(1-a)q_2)\leq aD(p_1||q_1)+(1-a)D(p_2||q_2)$$

## Mutual Information 
$I(X;Y)=D(p(X,Y)||p(X)p(Y))$  measures the similarity between the joint distribution $p(X,Y)$ and the product of marginal distributions $p(X)p(Y)$.

### Alternative Forms of Mutual Information

$$I(X;Y)=H(X)+H(Y)-H(X,Y)$$ $$=H(X)-H(X|Y)=H(Y)-H(Y|X)$$
$I(X;Y)\geq0\ \rArr\ H(X)\geq H(X|Y);H(Y)\geq H(Y|X)$. Equality holds if and only if $X\perp Y$ (i.e., $X$ and $Y$ are independent). This provides further proof that imposing conditions on a random variable generally reduces entropy.

## Conditional Mutual Information
The conditional mutual information of $X$ and $Y$ given $Z$ is defined as:$$I(X;Y|Z)=H(X|Z)-H(X|Y,Z);$$. Unlike entropy, mutual information can either increase or decrease when conditioned on another variable. 

Examples:
- Markov Chain Property: If $X\rarr Y \rarr Z$ forms a Markov chain, then: $$H(X;Y)\geq H(X;Y|Z)$$
- Establishing Dependencies: For independent random variables $X$v and $Y$, conditioning on a common variable $Z$ can introduce a dependency. For example, if $Z=X+Y$, then $I(X;Y∣Z)$ may be positive, even if $I(X;Y)=0$.