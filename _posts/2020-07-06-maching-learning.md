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

广义线性模型是一个预测模型，属于判别模型（**discriminative** model）。通过观察变量x预测变量y（**response variable**）。在这里，有三个假设条件。符合这三个条件的就属于广义线性模型：

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
   h(x) = E(T(y)|x) = \frac{\partial}{\partial\eta}a(\eta)
   $$

   这一函数也被称为响应函数（**response function**）： $$g(\eta)$$ ，其反函数被称为关联函数（**link function**）：$$ g^{-1}(\eta)$$ 。

3. 自然参数与观察变量是线性关系：$$\eta=\theta^Tx$$ 。

另外，预测变量的方差可表示为：

$$
Var(Y;\eta) = E(Y^2;\eta) - (E(Y;\eta))^2 = \frac{\partial^2}{\partial\eta^2}a(\eta)
$$

正常情况下，GLM的代价函数采用下述定义（可删去常数项）：

$$
J(\theta) = -\frac{1}{n}\log L(\theta) = -\frac{1}{n}\log \prod_{i=1}^n p(y^{(i)}|x^{(i)}, \theta)
$$

因此其微分方程也有固定的形式：

$$
\nabla_{\theta}J = -\frac{1}{n}\sum_{i=1}^n x^{(i)}(y^{(i)} - a^{'}(\eta))
$$



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
J(\theta) = \frac{1}{2n}\sum_{i=1}^n(h_{\theta}(x^{(i)})-y^{(i)})^2
$$

因此，求预测函数的最小值就相当于求它的最大似然估计，也就是令误差最小。

**解析法**

将输入矩阵X定义为(n, d+1)的形式，将结果y定义为(n, 1)的形式，那么系数可以直接用最小二乘公式计算出来：

$$
X^T X\theta = X^Ty
$$



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
J(\theta) = - \frac{1}{n} \sum_{i=1}^n [y^{(i)}\log (h_{\theta}(x^{(i)})) + (1-y^{(i)})\log(1-h_{\theta}(x^{(i)}))]
$$

**牛顿法**

在求解逻辑回归问题时，也可以使用牛顿法。在梯度下降法中，我们使用代价函数的一阶导数逼近其最小值。而在牛顿法中，我们通过寻找它的一阶导数的零点获得最优解。这两种方法本质上是一样的，但各有优缺点：牛顿法不需要考虑学习率这一参数，并且迭代次数相对更少，但是在每一次迭代中需要计算逆矩阵。

牛顿法的迭代公式为：

$$
\begin{aligned}
\theta 
&:= \theta - \frac{l^{'}(\theta)}{l^{''}(\theta)} \\
&:= \theta - H^{-1}\nabla_{\theta}l(\theta)
\end{aligned}
$$

其一阶微分为：

$$
\nabla_{\theta}l(\theta) = -\frac{1}{n}X^T(y-h)
$$

其二阶微分为：

$$
H = (-\frac{1}{n}) \cdot (-X^T \cdot diag(h(1-h)) \cdot X)
$$

其中，X的尺寸为(n, d + 1)，对角矩阵的尺寸为(n, n)，n为数据组数，d为特征数。



### 2.3 softmax回归

与逻辑回归相似，只是预测值变成了多个分类。



### 2.4 Poisson回归

先将泊松分布的概率函数转化为指数分布家族的形式：

$$
\begin{aligned}
p(y;\lambda) 
&= \frac{e^{-\lambda}\lambda^y}{y!} \\
&= \frac{1}{y!}\exp(\log(\lambda) \cdot y - \lambda)
\end{aligned}
$$

其中，

$$
\eta = \log\lambda \Rightarrow
\lambda = e^{\eta}
$$

模型的目标函数可以表示为：

$$
h_{\theta}(x) = E(T(y)|x) = \lambda = e^{\eta} = e^{\theta^Tx}
$$

该模型的对数似然估计为：

$$
\begin{aligned}
l(\theta) &= \log  \prod_{i=1}^n p(y^{(i)};\lambda) \\
&= \sum_{i=1}^n -h(x^{(i)}) + y^{(i)}\log h(x^{(i)}) - \log (y^{(i)}!)
\end{aligned}
$$

假设只有一组样本，上式的微分方程为：

$$
\frac{\partial }{\partial  \theta_j} l(\theta) 
= h^{'}(x)(\frac{y}{h(x)} - 1)
= (y-h(x))x_j
$$

由此可以得到反向传播函数（这里是最大化）：

$$
\theta := \theta + \alpha(y^{(i)}-h(x^{(i)}))x^{(i)}
$$

根据模型的对数似然估计，将代价函数设定为（常数项可以省略）：

$$
J(\theta) = -\frac{1}{n}\log L(\theta) 
= \frac{1}{n} \sum_{i=1}^n \left( 
h(x^{(i)}) - y^{(i)}\log h(x^{(i)}) + \log (y^{(i)}!)
\right)
$$


