---
title: Overview
layout: single
priority: 2
---
## BP Overview
BP involves calculating the gradient of the loss function with respect to each weight by applying the chain rule, then updating the weights to minimize the loss. This process allows the model to learn from data.

## Computational Graph
A computational graph is a representation of the operations and variables involved in a function or a network. It shows how each variable is computed from its predecessors and how they contribute to the final output. This is a typical computational graph in NN during forward propagation:
![a simple computational graph](/tips/pytorch/BP/fp_graph.png) 
where each line represents an weight tensor and each node represents an input/intermediate tensor. Different tensors are added together. 

During backward propagation, the computational graph is reversed:![backward pass](/tips/pytorch/BP/bp.png)

## Gradient Calculation
For a parameter on a specific line, its gradient is calculated as the multiplication of the input to this line in FP graph and the input to this line in BP graph. For example, the gradient of $W_{1,3}^3$ is calculated by multiplying $Z_2^1$ (the input in FP graph which is left to $W_{1,3}^3$) and $S_3^1$ (the input in BP graph which is right to $W_{1,3}^3$).

### Components need to be calculated
To accomplish an iteration, the values of each intermediate output in both FP and BP graph have to be stored. 

## Pytorch Inplementation
In forward propagation, the value of each node in FP graph is calculated and stored during ```model.forward()```, and by calling ```loss.backward()``` the value of each node in BP graph is calculated. In the same time, the gradient of each parameter is calculated according to the rule discussed above, which is saved in ```.grad``` attributed of each parameter with ```required_grad==True```. 

The next step is calling ```optimizer.zero_grad()``` function to reset the gradients of all model parameters calculated in last iteration to zero. Afterwards, calling ```optimizer.step()``` updates every parameters according to the specific optimization algorithm (e.g., SGD, Adam, etc., see [Optimizer]({{<relref "/tips/ml/Optimization/Optimizer">}}) for details).