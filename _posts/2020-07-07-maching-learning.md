---
layout: post
title:  "机器学习笔记-GDA"
date:   2020-07-07 00:00:00 +0000
tags: ML
---



生成模型（**generative** model）是另外一种思想。根据贝叶斯原理，在得知观察量X对于预测量Y的概率、以及Y的先验概率后，可以计算得到后验概率。生成模型使用的就是这个后验概率。在生成模型中，最基本的是高斯判别分析模型（Gaussian discriminant analysis, GDA）。

资料：

[斯坦福CS229机器学习课程笔记四：GDA、朴素贝叶斯、多项事件模型 - Logos - 博客园](https://www.cnblogs.com/logosxxw/p/4651435.html)

[Generative Learning Algorithm - Wei’s Homepage](https://wei2624.github.io/MachineLearning/sv_generative_model/)



## 1 高斯判别分析

在高斯判别分析中，观察量呈多元高斯分布，预测量呈离散分布（我们后面用最简单的0、1分布举例）。

我们拥有的数据有：

* 预测量的先验概率数据
* 观察量关于每个预测量的概率

我们的目标是通过调整概率公式中的参数，使得目标函数（似然估计）达到最大。

根据贝叶斯原理，我们也可以得到预测量关于观察量的概率。



### 1.1 多元高斯分布

多元高斯分布（multivariate normal distribution）是一元高斯分布的扩展，通过声明一些符号，可将多元高斯分布公式化成和一元高斯分布类似的形式：

$$
p(x;\mu,\Sigma) = \frac{1}{(2\pi)^{d/2}|\Sigma|^{1/2}}
\exp\left(
-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)
\right)
$$

其中，向量 $$\mu$$ 为每一个元素的均值，矩阵 $$\Sigma$$ 为协方差矩阵。当分布中的元素相互独立时，该协方差矩阵为对角矩阵：$$ \Sigma=diag(\sigma_1^2, ..., \sigma_n^2)$$ 。

根据定义可知：

$$
\begin{cases}
\nabla_{\Sigma}|\Sigma| = |\Sigma|\Sigma^{-1} & (\Sigma = \Sigma^T)\\
\nabla_x x^TAx = 2Ax
\end{cases}
$$

这两个微分后面会用到。

### 1.2 高斯判别分析模型

假设预测量的先验概率呈伯努利分布，观察量关于每个预测量呈多元高斯分布：

$$
\begin{aligned}
p(y) &= \phi^y(1-\phi)^{(1-y)} \\
p(x|y=0) &= \frac{1}{(2\pi)^{d/2}|\Sigma|^{1/2}}
\exp\left(
-\frac{1}{2}(x-\mu_0)^T\Sigma^{-1}(x-\mu_0)
\right) \\
p(x|y=1) &= \frac{1}{(2\pi)^{d/2}|\Sigma|^{1/2}}
\exp\left(
-\frac{1}{2}(x-\mu_1)^T\Sigma^{-1}(x-\mu_1)
\right)
\end{aligned}
$$

于是我们可以写出它的对数似然估计，也就是我们的目标函数：

$$
\begin{aligned}
l(\phi,\mu_0,\mu_1,\Sigma) 
&= \log \prod_{i=1}^n p(x^{(i)},y^{(i)};\phi,\mu_0,\mu_1,\Sigma) \\
&= \log \prod_{i=1}^n p(x^{(i)}|y^{(i)};\mu_0,\mu_1,\Sigma)p(y^{(i)};\phi)
\end{aligned}
$$

可以上式分块：

$$
\begin{aligned}
l(\phi,\mu_0,\mu_1,\Sigma) = 
 & \sum_{y^{(i)}=0}\log(1-\phi) + \sum_{y^{(i)}=1}\log(\phi) \\
 &- \sum_{y^{(i)}=0}\frac{1}{2}(x-\mu_0)^T\Sigma^{-1}(x-\mu_0) \\
 &- \sum_{y^{(i)}=1}\frac{1}{2}(x-\mu_1)^T\Sigma^{-1}(x-\mu_1) \\
 &+ \frac{n}{2} \log |\Sigma^{-1}|- \frac{nd}{2} \log (2\pi) 
 \end{aligned}
$$

将其对于每个参数求偏微分，可得取对数最大似然估计时各参数的值为：

$$
\begin{aligned}
\phi &= \frac{1}{n}\sum_{i=1}^n 1\{y^{(i)}=1\} \\
\mu_0 &= \frac{\sum_{i=1}^n 1\{y^{(i)}=0\}x^{(i)}}{\sum_{i=1}^n 1\{y^{(i)}=0\}} \\ 
\mu_1 &= \frac{\sum_{i=1}^n 1\{y^{(i)}=1\}x^{(i)}}{\sum_{i=1}^n 1\{y^{(i)}=1\}} \\ 
\Sigma &= \frac{1}{n} \sum_{i=1}^n (x^{(i)} - \mu_{y^{(i)}})(x^{(i)} - \mu_{y^{(i)}})^T
\end{aligned}
$$



### 1.3 GDA与逻辑回归

根据贝叶斯原理，可以得到预测量随观察量的概率分布：

$$
\begin{aligned}
p(y=1|x;\phi,\mu_0,\mu_1,\Sigma) &= \frac{p(x|y=1)p(y=1)}{p(x)} \\
&= \frac{1}{1 + \frac{p(x|y=0)p(y=0)}{p(x|y=1)p(y=1)}} \\ 
&= \frac{1}{1 + \exp\left(
-(\Sigma^{-1}(\mu_1-\mu_0))^Tx
-\frac{1}{2}(\mu_0^T\Sigma^{-1}\mu_0-\mu_1^T\Sigma^{-1}\mu_1)
+\log(\frac{1-\phi}{\phi})
\right)}
\end{aligned}
$$

也就是说，通过合并参数，可将GDA化简成逻辑回归。因此，逻辑回归更通用。但如果数据符合GDA的限制条件时，使用GDA预测会更精确。

### 1.4 代码实现

以下代码先计算了含有两个分类的GDA模型中的必要参数，然后将其转换为逻辑回归进行进行预测：

```python
def fit(self, x, y):
    """Fit a GDA model to training set given by x and y by updating
    self.theta (dim + 1, 1).

    Args:
        x: Training example inputs. Shape (n_examples, dim).
        y: Training example labels. Shape (n_examples,).
    """
    d = x.shape[1]
    n = y.shape[0]
    n1 = np.sum(y)
    n0 = n - n1
    y = y.reshape((n, 1))
    # Find phi, mu_0, mu_1, and sigma
    phi = n1 / n
    mu_0 = np.dot(x.T, 1-y) / n0
    mu_1 = np.dot(x.T, y) / n1
    mu = np.dot(1-y, mu_0.T) + np.dot(y, mu_1.T)
    sigma = np.dot((x-mu).T, x-mu) / n
    # Write theta in terms of the parameters
    sigma_i = np.linalg.inv(sigma)
    theta0 = (np.log(phi / (1-phi))
              + np.dot(np.dot(mu_0.T, sigma_i), mu_0) / 2
              - np.dot(np.dot(mu_1.T, sigma_i), mu_1) / 2)
    theta_ex = np.dot(sigma_i, mu_1 - mu_0)
    theta = np.zeros((d+1, 1))
    theta[0][0] = theta0
    theta[1:, :] = theta_ex
    self.theta = theta
```

