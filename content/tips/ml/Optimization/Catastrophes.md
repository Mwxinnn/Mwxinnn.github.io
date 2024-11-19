---
title: Catastrophes of NN
layout: single
date: 2024-09-12
priority: 2
---

## Vanishing/Exploding Gradients

## Catastrophic interference

Catastrophic interference, also known as catastrophic forgetting, is the tendency of an artificial neural network to abruptly and drastically forget previously learned information upon learning new information. When a network learns a new task, it modifies its weights to reduce the error for that particular task. This modification can dramatically alter the knowledge representation of prior tasks, leading to the "forgetting" phenomenon.

One example of the catastrophic forgetting problem is when training a model on the MNIST classification task with digits 0-4 and testing it on all digits 0-9. The model can effectively classify 0-4 but misclassifies 5-9 as one of the digits 0-4. However, when the same model is retrained using only digits 5-9, it can then correctly classify 5-9 but misclassifies 0-4 as one of the digits 5-9. In other words, after the second training process, the model forgets how to classify 0-4.

### When does it happen?
This phenomenon is particularly prevalent in the following situations:

1. **Sequential Learning**: When a neural network is trained on a sequence of tasks one after another without revisiting previous tasks, it tends to forget the earlier tasks as it adapts to the new ones.

2. **Overlapping Data Distributions**: When new data has a distribution that overlaps or conflicts with the distribution of previously learned data, the network may overwrite old knowledge in favor of the new information.

3. **Insufficient Regularization**: Without proper regularization techniques (e.g., dropout, weight decay, or specific algorithms designed to mitigate forgetting), the network may prioritize learning the new data too aggressively, leading to the forgetting of prior knowledge.

4. **Limited Model Capacity**: If the model does not have sufficient capacity (e.g., too few parameters) to store and generalize knowledge from multiple tasks, it may erase old information to accommodate new learning.

5. **Online Learning**: In scenarios where a model continuously learns from a stream of data without retraining on previous data, it might forget older information as it focuses on more recent inputs.

These situations highlight the challenge of preserving knowledge in neural networks, especially in environments where learning is ongoing and data is non-stationary.

My intuition regarding this phenomenon is that it tends to occur **when the tasks of the training process are not sufficiently challenging.**
1. First, if the initial training process is easy, the information the model learned during the initial training phase may be inadequate and does not encompass the knowledge needed for the subsequent training phases. As a result, the model must acquire new information for the later tasks. However, the trained model may be already converged to a condense space, introducing new information may lead the model to other directions out of this space, in other words, forget stored information.
2. Also, if the gap between two data distributions is too large, the model has to renew in a huge range which could probably lead to drastic change in parameters.
2. Second, if the subsequent task is easy to learn, the model may overfit to it, causing all trainable parameters to converge towards this new solution easily, thereby forgetting the information retained from earlier tasks.


### Possible Solutions

#### Regularization Techniques:
1. Elastic Weight Consolidation (EWC): This technique **penalizes significant changes** to the weights that are important for previous tasks by adding a regularization term to the loss function. It helps the network retain knowledge from earlier tasks.
2. Synaptic Intelligence (SI): Similar to EWC, SI **accumulates importance weights** for each parameter based on how much they contributed to previous tasks and discourages changes to critical parameters.
3. L2 Regularization: This is a simpler regularization method that can help **prevent drastic changes** to the model parameters, thereby reducing forgetting.

However, these methods do not always work since a slight change in NN weights can lead to drastic change in its output.

#### Replay Methods:
1. Experience Replay: In this method, the network is periodically retrained on a mix of new and old data (stored in a memory buffer), which helps in maintaining knowledge from previous tasks.
2. Generative Replay: Instead of storing old data, a generative model (e.g., a GAN or VAE) is trained to generate examples from previous tasks, which are then used to replay and reinforce old knowledge.

This method regularizes output instead of model's weights. However, either way above is memory consuming and time consuming. It's better to sample a few representative samples to store rather than store all of previous data. We can use gaussian process to sample "memorable" old data and throw away "forgettable" data.

#### Parameter Isolation:
1. Progressive Neural Networks: New tasks are handled by creating new subnetworks that are connected to existing ones, preventing interference with previously learned tasks.
2. PathNet: This approach involves freezing a subset of the network’s parameters that are important for previous tasks, while new tasks are learned using the remaining parameters.

This is the most popular method in large foundation model era, which appears in another form, Parameter-Efficient Fine-Tuning (PEFT).
#### Meta-Learning:
Learning to Learn: Meta-learning algorithms optimize the learning process itself so that the network can adapt to new tasks quickly without forgetting previous ones. Methods like MAML (Model-Agnostic Meta-Learning) aim to find an initialization that is suitable for learning multiple tasks.

#### Architectural Solutions:
1. Dynamic Architectures: Networks can be designed to grow their architecture by adding new neurons or layers as new tasks are learned, which helps in accommodating new knowledge without overwriting old information.
2. Continual Learning Networks: These architectures are explicitly designed for lifelong learning, often involving a combination of the above techniques to balance plasticity (learning new tasks) and stability (retaining old tasks).