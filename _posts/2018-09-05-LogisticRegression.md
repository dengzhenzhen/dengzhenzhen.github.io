---
layout:     post
title:      Logistic回归的数学推导和python实现
subtitle:   
date:       2018-09-05
author:     dengzhenzhen
header-img: img/post-bg-sigmoid.png
catalog: 	 false
tags:
    - Machine Learning
    - Logistic Regression
---

```python
import numpy as np
```

$$
h_\theta(x) = {1 \over {1+e^{-x}}} \\
{\partial h(x)\over \partial x  }= h(x)(1-h(x))
$$


```python
def sigmoid(x):
    return 1/(1 + np.math.exp(-x))

def sigmoid_derive(x):
    return sigmoid(x) * (1 - sigmoid(x))
```

$$
cost(h_\theta(x),y)=\sum_{i=1}^m−y_ilog(h_θ(x_i))−(1−y_i)log(1−h_θ(x_i))
$$


```python
def cost(x, y, theta):
    x = np.array(x)
    y = np.array(y).astype('int')
    sig = x.dot(theta)
    return sum([-np.math.log(sigmoid(sig[i])) if y[i]==1 else -np.math.log(1 - sigmoid(sig[i])) for i in range(len(y))])
```

$$
\begin{split}
{\partial cost(h(\theta,x), y)\over \partial\theta_i} &= 
{\sum_{i=1}^m {y_i\over h(\theta,x)}{\partial h(\theta,x)\over\partial\theta} }
+{1-y_i\over{1-h(\theta,x)}}{\partial h(\theta,x)\over\partial\theta}\\
&=
{\sum_{i=1}^m {y_i\over h(\theta,x)}{(h(\theta x)(1-h(\theta x)))}x_i }
+{1-y_i\over{1-h(\theta,x)}}{(h(\theta x)(1-h(\theta x)))x_i}\\
&=
\sum_{i=1}^m y_i(1-h(\theta x))x_i + (1-y_i)h(\theta x)x_i\\
grad(cost(h_\theta(x),y))
&=({\partial cost(h(\theta,x), y)\over \partial\theta_1},{\partial cost(h(\theta,x), y)\over \partial\theta_2},...,
{\partial cost(h(\theta,x), y)\over \partial\theta_m})
\end{split}
$$


```python
def grad(x, y, theta):
    x = np.array(x)
    y = np.array(y).astype('int')
    sig = np.array([sigmoid(i) for i in x.dot(theta)])
    return np.array([(1-sig[i])*x[i] for i in range(len(y))]).sum(axix=0)
    
```


```python
def logistic_grad_descent(x,y, num=1000):
    x = np.array(x)
    y = np.array(y).astype('int')
    theta = np.zeros(x.shape[1])
    for i in range(num):
        g = grad(x, y, theta)
        theta = theta - g
        
        if i % (num//10) == 0:
            print(cost(x, y, theta))
```


```python
class LogisticClassfier():
    
    def __init__(self):
        pass
        
    def fit(self, x, y):
        x = np.array(x)
        y = np.array(y).astype('int')
        theta = np.zeros(x.shape[1])
        for i in range(num):
            g = grad(x, y, theta)
            theta = theta - g

            if i % (num//10) == 0:
                print(cost(x, y, theta))
        self.theta = theta
        
    def predict(x):
        return np.array([sigmoid(i) for i in np.dot(theta,x)] )
```


```python

```


```python

```


```python

```
