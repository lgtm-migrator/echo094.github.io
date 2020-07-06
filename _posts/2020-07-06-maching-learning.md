---
layout: post
title:  "机器学习笔记-GLM"
date:   2020-07-06 00:00:00 +0000
tags: ML
---



旁听了一下CS229，受到了很大的打击，自己还是太菜了……

教案里面是按照自下而上的顺序讲的，先介绍了与之相关的内容，最后才介绍了GLM，于是第一次看完的时候感觉戛然而止、不知所云。在查了一些资料后，终于明白了一些。

资料：

[【机器学习】广义线性模型 - 知乎](https://zhuanlan.zhihu.com/p/88655099)

[广义线性模型(GLM)从人话到鬼话连篇 - 知乎](https://zhuanlan.zhihu.com/p/110268967)



## 1 广义线性模型

广义线性模型是一个预测模型，通过观察变量x预测变量y（**response variable**）。在这里，有三个假设条件。符合这三个条件的就属于广义线性模型：

1. 预测变量的分布必须属于指数分布家族，即：

   $$
   y | x;\theta \sim ExponentialFamily(\eta)
   $$

   在这里，变量 $$x$$ 和 $$\theta$$ 共同构成了自然参数 $$\eta$$ 。指数分布的形式如下：

   $$
   p(y;\eta) = b(y) \exp{(\eta^TT(y) - a(\eta))}
   $$

   其中，$$T(y)$$ 是充分统计量（**sufficient statistic**），通常有 $$T(y)=y$$，后面直接使用这个假设。

2. 我们的目标是预测 $$T(y)$$ 的期望值：

   $$
   h(x) = E(T(y)|x)
   $$

   这一函数也被称为响应函数（**response function**）： $$g(\eta)$$ ，其反函数被称为关联函数（**link function**）：$$ g^{-1}(\eta)$$ 。

3. 自然参数与观察变量是线性关系：$$\eta=\theta^Tx$$ 。

线性回归，logistic回归，以及softmax回归都属于广义线性模型。



## 2 GLM的实例



### 2.1 线性回归

在线性回归中，我们的预测量是测量值与预测值的误差e，这个误差可以假设服从高斯分布：

$$
\begin{aligned}
p(e^{(i)})
&= \frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{(e^{(i)})^2}{2\sigma^2}\right) \\
p(y^{(i)}|x^{(i)};\theta)
&= \frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{(y^{(i)}-\theta^Tx^{(i)})^2}{2\sigma^2}\right)
\end{aligned}
$$

由公式可知误差越小，概率越大。

在这里，我们令方差为1，均值 $$\mu = \theta^Tx$$ ，即可将误差e的分布转化为指数分布家族的形式：

$$
p(y;\mu) =
\frac{1}{\sqrt{2\pi}}\exp\left(-\frac{1}{2}y^2\right)
\exp\left(\mu y-\frac{1}{2}\mu^2\right)
$$

因此，自然参数 $$ \eta = \mu = \theta^Tx$$ ，于是可以得到线性回归的目标函数：

$$
h(x) = E(T(y)|x) = \mu = \theta^Tx
$$

然后，我们写出它的似然估计：

$$
L(\theta) = \prod_{i=1}^n p(y^{(i)}|x^{(i)};\theta)
$$

将其取对数可得：

$$
l(\theta) = n\log\frac{1}{\sqrt{2\pi}\sigma}
- \frac{1}{2\sigma^2} \sum_{i=1}^n (y^{(i)}-\theta^Tx^{(i)})^2
$$

另一方面，线性回归的代价函数为：

$$
J(\theta) = \frac{1}{2m}\sum_{i=1}^m(h_{\theta}(x^{(i)})-y^{(i)})^2
$$

因此，求预测函数的最小值就相当于求它的最大似然估计，也就是令误差最小。



### 2.2 logistic回归

逻辑回归的概率服从伯努利分布。我们先将伯努利分布改写成指数分布家族的形式：

$$
\begin{aligned}
p(y;\phi) &= (\phi)^y \cdot (1-\phi)^{(1-y)} \\
&= 1 \cdot \exp\left(\log(\frac{\phi}{1-\phi})y + \log(1-\phi)\right)
\end{aligned}
$$

于是，自然参数可表示为：

$$
\eta = \theta^Tx = \log(\frac{\phi}{1-\phi}) \Rightarrow
\phi = \frac{1}{1+e^{-\eta}}
$$

在这里，$$\phi$$ 就是sigmoid函数，于是可以得到logistic回归的目标函数：

$$
h(x) = E(T(y)|x) = \phi = \frac{1}{1+e^{-\theta^Tx}}
$$

然后，我们写出它的似然估计：

$$
\begin{aligned}
L(\theta) &= \prod_{i=1}^n p(y;\phi) \\
&= \prod_{i=1}^n (h_{\theta}(x^{(i)}))^{y^{(i)}} \cdot (1-h_{\theta}(x^{(i)}))^{1-y^{(i)}}
\end{aligned}
$$

将其取对数可得：

$$
l(\theta) =  \sum_{i=1}^n y^{(i)}\log h_{\theta}(x^{(i)}) + (1-y^{(i)})\log(1-h_{\theta}(x^{(i)}))
$$

因此，优化过程就是求上式的最大似然估计，由于在计算时习惯让损失函数越小越好，代价函数可以写为：

$$
J(\theta) = - \frac{1}{m} \sum_{i=1}^m [y^{(i)}\log (h_{\theta}(x^{(i)})) + (1-y^{(i)})\log(1-h_{\theta}(x^{(i)}))]
$$

### 2.3 softmax回归

与逻辑回归相似，只是预测值变成了多个分类。
