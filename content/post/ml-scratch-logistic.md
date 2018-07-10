---
author: "Donghua (Lyla) CAI"
date: 2018-01-20
title: "Machine Learning from Scratch: Logistic Regression"
tags: [
    "Machine Learning",
	"Logistic Regression"
]
categories: [
    "Machine Learning from Scratch"
]
math: true
---

## Introduction

First, do not be fooled by its name. Logistic regression is not a regression algorithm but a classification one. It is used to assign data into **discrete** set of classes, for example, two classes (binary logistic regression) or multiple classes. 

Real world application includes: given a dataset describing patients' tumor sizes, predict if the tumor is benign, yes (label=1) or no (label=0). This is a binary classification problem. First, we need to pass the data to a prediction function. It should give us a probability of a data sample being assigned yes. The probability is ranging from $0$ and $1$. Then we can classify the data by, for example, set the threshold 0.5:

$$p\geq 0.5, class = 1$$
$$p< 0.5, class = 0$$

This prediction function is called **logic function** or **sigmoid function**. *Sigmoid* means S-shaped. The sigmoid function for $z\in(-\infty,\infty)$ looks like below.
$$sigmoid(z) = \sigma(z) =\frac{1}{1+\exp(-z)}$$


{{< figure src="/img/sigmoid.png" title="Figure 1. Sigmoid function (From Machine learning, Stanford, Coursera)" >}}

```{python}
# Easy to implement in python

def sigmoid(z)
	return 1.0 / (1 + numpy.exp(-z))
```

With sigmoid function, when $z$ is very large, $\exp(-z)$ is small so that $sigmoid(z)$ is close to $1$; On the other hand, when $z$ is very small, $\exp(-z)$ is large so that $sigmoid(z)$ is close to $0$. Hence, we map all possible $z$ value to $[0,1]$.

## Cost Function 
It is also called **Cross Entropy**. 

The cost function in linear regression (Mean Squared Error, $MSE=\frac{1}{2}(\hat{y}-y)^2)$ can not be used in logistic regression. Because prediction function in logistic regression is not linear due to sigmoid transform. If we use MSE, the cost function is non-convex and has many local optima. Instead, we use log-like function below:

$$J(\theta) = -\frac{1}{m} \sum_{i=1}^{m}[y^{(i)}\log(h)+(1-y^{(i)})\log{(1-h)}]$$

where $h(x) = \sigma(X\theta)$, $z=X\theta$, $\theta$ is a vector of model parameters. 

```{python}
def cost(h, y):
	return np.mean((-y * np.log(h) - (1 - y) * np.log(1 - h)))

```

## Gradient Descent

To find the global minimum of the cost function, we need to compute the derivative of $J(\theta)$. Use the derivative of sigmoid function $\sigma(x)' =\sigma(x)(1-\sigma(x))$ and apply chain rule, we will have the result below:

<div>
$$
\begin{align}\frac{\partial J}{\partial\mathbf{\theta}} &=	-\frac{1}{m}\sum_{i=1}^{m}\big[\frac{\partial}{\partial\mathbf{\theta}}y\log\sigma(\mathbf{\theta}^{T}x)+\frac{\partial}{\partial\mathbf{\theta}}(1-y)\log[1-\sigma(\mathbf{\theta}^{T}x)]\big] \\
  &=	-\frac{1}{m}\sum_{i=1}^{m}\big[\frac{y}{\sigma(\mathbf{\theta}^{T}x)}-\frac{1-y}{1-\sigma(\mathbf{\theta}^{T}x)}\big]\frac{\partial}{\partial\mathbf{\theta}}\sigma(\mathbf{\theta}^{T}x) \\
  &=	-\frac{1}{m}\sum_{i=1}^{m}\big[\frac{y}{\sigma(\mathbf{\theta}^{T}x)}-\frac{1-y}{1-\sigma(\mathbf{\theta}^{T}x)}\big]\sigma(\mathbf{\theta}^{T}x)[1-\sigma(\mathbf{\theta}^{T}x)]\frac{\partial}{\partial\mathbf{\theta}}(\mathbf{\theta}^{T}x) \\
  &=	-\frac{1}{m}\sum_{i=1}^{m}\big[y-\sigma(\mathbf{\theta}^{T}x)\big]\frac{\partial}{\partial\mathbf{\theta}}(\mathbf{\theta}^{T}x)
\end{align}$$
</div>

Vectorized version:
$$\nabla J(\theta)=-\frac{1}{m}X^T(y-\sigma(X\theta))$$

```{python}
h = sigmoid(np.dot(X, theta))
gradient = -1.0 / y.size * np.transpose(X) * (y - h)
```

## Stochastic Gradient Descent

(to be continued.)


