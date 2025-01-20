---
{"dg-publish":true,"permalink":"/AI/concepts/softmax/","noteIcon":"3"}
---

#math
Softmax is a mathematical function that converts a vector of real numbers into a probability distribution. It is commonly used in machine learning and statistics, especially in tasks where you want to interpret the outputs of a model as probabilities.

Given an input vector $( z = (z_1, z_2, \dots, z_k) )$ of \( k \) real numbers, the softmax function computes the output vector $( \sigma(z) = (\sigma(z_1), \sigma(z_2), \dots, \sigma(z_k)) )$ where each component $( \sigma(z_i) )$ is calculated as follows:


$$
[ \sigma(z_i) = \frac{e^{z_i}}{\sum_{j=1}^k e^{z_j}} ]
$$

In other words:
$$
[ \sigma(z_i) = \frac{e^{z_i}}{e^{z_1} + e^{z_2} + \dots + e^{z_k}} ]
$$


Here's a step-by-step breakdown of how softmax works:

1. **Exponentiation**: Compute the exponential of each element in the input vector \( z \). This results in a new vector .

$$
( e^{z} = (e^{z_1}, e^{z_2}, \dots, e^{z_k}) ) 
$$

3. **Summation**: Calculate the sum of all elements in the new vector $( e^{z} )$, denoted as $( \sum_{j=1}^k e^{z_j} )$.

4. **Normalization**: For each element $( z_i )$ in the input vector $( z )$, divide $( e^{z_i} )$ by the sum $( \sum_{j=1}^k e^{z_j} )$ obtained in the previous step. This normalization ensures that the output vector $( \sigma(z) )$ represents a valid probability distribution, where each element is between 0 and 1, and the sum of all elements is equal to 1.

The softmax function is useful in various machine learning tasks, including classification problems. It transforms the raw scores (logits) produced by a model into probabilities that can be interpreted as the model's confidence scores for each class. The class with the highest probability according to the softmax output is typically chosen as the predicted class.

In summary, softmax function is defined as:
$$
 \sigma(z_i) = \frac{e^{z_i}}{\sum_{j=1}^k e^{z_j}} 
    

$$

for $( i = 1, 2, \dots, k )$, where $( z = (z_1, z_2, \dots, z_k) )$ is the input vector and $( \sigma(z) )$ is the output vector representing probabilities.