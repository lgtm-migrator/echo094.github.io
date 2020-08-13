---
layout: post
title:  "机器学习笔记-GMM与EM"
date:   2020-08-12 00:00:00 +0000
tags: ML
---


最大期望算法（Expectation-Maximization *algorithm*, **EM**）可以用于对包含隐含变量或缺失数据的概率模型进行参数估计。

在无监督学习中，有一组训练集为 $$ \{ x^{(1)}, ..., x^{(n)}\}$$ ，当这个训练集中的样本来自于多个分布$$ \{ z^{(1)}, ..., z^{(k)}\}$$ 时，可以将其概率表示为联合分布： $$ p(x^{(i)}, z^{(i)}) = p(x^{(i)} \vert z^{(i)})p(z^{(i)}) $$ 。在这里，$$ z^{(i)}$$ 就是上面的标签数据或隐含变量，因此可以使用EM算法计算参数。

当这些分布是高斯分布时，这个分布模型就是高斯混合模型（Gaussian Mixture Model, **GMM**）。

令概率 $$ p(z^{(i)} = j) = \phi_j$$ ，令高斯分布为 $$ x^{(i)} \vert z^{(i)} = j \sim N(\mu_j,\Sigma_j)$$ 。

该模型的最大似然估计为：

$$
\begin{aligned}
l(\phi,\mu,\Sigma) 
&= \sum_{i=1}^n \log p(x^{(i)};\phi,\mu,\Sigma) \\
&= \sum_{i=1}^n \log \sum_{z^{(i)}=1}^k p(x^{(i)}\vert z^{(i)};\mu,\Sigma)p(z^{(i)};\phi)
\end{aligned}
$$

当标签$$ z^{(i)}$$ 给出时，为有监督EM，当标签$$ z^{(i)}$$未知时，属于半监督EM，而当标签$$ z^{(i)}$$部分已知时，为半监督EM。



## 1 Supervised EM

在这种情况下，令已知的标签为 $$ \tilde{z}^{(i)} $$ 。最大似然估计式可写为：

$$
l(\phi,\mu,\Sigma) 
= \sum_{i=1}^n \log p(x^{(i)}\vert z^{(i)};\mu,\Sigma) + \sum_{i=1}^n \log p(z^{(i)};\phi)
$$

于是，在这种情况下，没有E-step这个步骤。M-step的计算公式为：

$$
\begin{aligned}
\phi_j &:= \frac{1}{\tilde{n}} \sum_{i=1}^{\tilde{n}} 1\{\tilde{z}^{(i)}=j\} \\
\mu_j &:= \frac{\sum_{i=1}^{\tilde{n}} 1\{\tilde{z}^{(i)}=j\}\tilde{x}^{(i)}}{\sum_{i=1}^{\tilde{n}} 1\{\tilde{z}^{(i)}=j\}}  \\
\Sigma_j &:= \frac{\sum_{i=1}^{\tilde{n}} 1\{\tilde{z}^{(i)}=j\}(\tilde{x}^{(i)}-\mu_j)(\tilde{x}^{(i)}-\mu_j)^T}{\sum_{i=1}^{\tilde{n}} 1\{\tilde{z}^{(i)}=j\}}
\end{aligned}
$$



## 2 Unsupervised EM

在这种情况下，每个样本都有可能是任何一个标签，由于最大似然估计存在下述性质：

$$
\begin{aligned}
\log p(x;\phi) &= \log \sum_z p(x,z;\phi) \\
&= \log \sum_z Q(z) \frac{p(x,z;\phi)}{Q(z)} \\
&\ge  \sum_z Q(z) \log \frac{p(x,z;\phi)}{Q(z)}
\end{aligned}
$$

当 $$ Q(z) = p(z \vert x; \phi) $$ 时，上式中的等式成立。因此，首先在E-step中使用该性质得到最大似然估计在当前参数下的最大期望：

$$
l(\phi) = \sum_{i=1}^n \sum_{j=1}^k w_j^{(i)} \log \frac{p(x^{(i)},z^{(i)};\phi)}{w_j^{(i)}}
$$

其中：

$$
\begin{aligned}
w_j^{(i)} :&= p(z^{(i)} = j | x^{(i)};\phi,\mu,\Sigma) \\
&= \frac{N(x^{(i)}|\mu_j,\Sigma_j) \phi_j}{\sum_{j = 1}^k N(x^{(i)}|\mu_j,\Sigma_j) \phi_j}
\end{aligned}
$$

然后在M-step中更新参数：

$$
\begin{aligned}
\phi_j &:= \frac{1}{n} \sum_{i=1}^{n} w_j^{(i)} \\
\mu_j &:= \frac{\sum_{i=1}^n w_j^{(i)}x^{(i)}}{\sum_{i=1}^n w_j^{(i)}}  \\
\Sigma_j &:= \frac{\sum_{i=1}^n w_j^{(i)}(x^{(i)}-\mu_j)(x^{(i)}-\mu_j)^T}{\sum_{i=1}^n w_j^{(i)}} 
\end{aligned}
$$



## 3 Semi-supervised EM

这种情况结合了上述两种情况，似然估计为：

$$
l_{semi-sup}(\phi) = l_{unsup}(\phi) + \alpha l_{sup}(\phi)
$$

E-step的计算与无监督情况相同：

$$
w_j^{(i)} := p(z^{(i)}=j|x^{(i)};\phi,\mu,\Sigma)
$$

M-step的更新公式是上述两种情况的叠加：

$$
\begin{aligned}
\phi_l &:= \frac{\sum_{i=1}^n w_l^{(i)} + \alpha \sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}}{n + \alpha \tilde{n}} \\
\mu_l &:= \frac
{\sum_{i=1}^n  w_l^{(i)}x^{(i)} + \alpha\sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}\tilde{x}^{(i)}}
{\sum_{i=1}^n  w_l^{(i)} + \alpha\sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}}  \\
\Sigma_l &:= \frac{
\sum_{i=1}^n w_l^{(i)}(x^{(i)}-\mu_l)(x^{(i)}-\mu_l)^T
+\alpha\sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}
(\tilde{x}^{(i)}-\mu_l)(\tilde{x}^{(i)}-\mu_l)^T
}
{\sum_{i=1}^n w_l^{(i)}+\alpha\sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}}
\end{aligned}
$$

推导过程如下：

$$
\begin{aligned}
l(\phi) &= 
\sum_{i=1}^n
\left(
\sum_{z^{(j)}}Q_j^{(i)}(z^{(i)})\log\frac{p(z^{(i)},x^{(i)};\phi)}{Q_j^{(i)}(z^{(i)})}
\right)
+ \alpha
\left(
\sum_{i=1}^{\tilde{n}}\log p(\tilde{z}^{(i)},\tilde{x}^{(i)};\phi)
\right) \\
&=\sum_{i=1}^n
\left(
\sum_{j=1}^k w_j^{(i)}\log\frac{p(x^{(i)}|z^{(i)}=j;\mu,\Sigma)p(z^{(i)}=j;\phi)}{w_j^{(i)}}
\right)
+\alpha
\left(
\sum_{i=1}^{\tilde{n}}\log p(\tilde{x}^{(i)}|\tilde{z}^{(i)};\mu,\Sigma)p(\tilde{z}^{(i)};\phi)
\right)
\end{aligned}
$$

其中：

$$
p(x^{(i)}|z^{(i)}=j;\mu,\Sigma) = 
\frac{1}{(2\pi)^{d/2}|\Sigma_j|^{1/2}}\exp
\left(
-\frac{1}{2}(x^{(i)}-\mu_j)^T\Sigma_j^{-1}(x^{(i)}-\mu_j)
\right)
$$



对似然估计求 $$\mu_l$$ 的偏导：

$$
\begin{aligned}
\nabla_{\mu_l} l 
&= \nabla_{\mu_l} 
\left(
\sum_{i=1}^n -w_l^{(i)}
\frac{1}{2}(x^{(i)}-\mu_l)^T\Sigma_l^{-1}(x^{(i)}-\mu_l)
+\alpha\sum_{\tilde{z}^{(i)}=l}
-\frac{1}{2}(\tilde{x}^{(i)}-\mu_l)^T\Sigma_l^{-1}(\tilde{x}^{(i)}-\mu_l)
\right) \\
&= \sum_{i=1}^n w_l^{(i)}(\Sigma_l^{-1}x^{(i)}-\Sigma_l^{-1}\mu_l)
+\alpha\sum_{\tilde{z}^{(i)}=l}(\Sigma_l^{-1}\tilde{x}^{(i)}-\Sigma_l^{-1}\mu_l) \\
&= 0
\end{aligned}
$$

得到 $$\mu_l$$ 为：

$$
\mu_l = \frac
{\sum_{i=1}^n  w_l^{(i)}x^{(i)} + \alpha\sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}\tilde{x}^{(i)}}
{\sum_{i=1}^n  w_l^{(i)} + \alpha\sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}}
$$



整理似然估计方程，得到含有 $$\Sigma_l$$ 的部分：

$$
\begin{aligned}
l(\Sigma_l) 
&= \sum_{i=1}^n
w_l^{(i)}\log(p(x^{(i)}|z^{(i)}=l;\mu,\Sigma)
+\alpha \sum_{\tilde{z}^{(i)}=l}
\log p(\tilde{x}^{(i)}|\tilde{z}^{(i)};\mu,\Sigma) 
+ C_1 \\
&= \sum_{i=1}^n
-\frac{1}{2}w_l^{(i)}((x^{(i)}-\mu_l)^T\Sigma_l^{-1}(x^{(i)}-\mu_l)-\log|\Sigma_j^{-1}|) \\
&+\alpha \sum_{\tilde{z}^{(i)}=l}
-\frac{1}{2}((\tilde{x}^{(i)}-\mu_l)^T\Sigma_l^{-1}(\tilde{x}^{(i)}-\mu_l)-\log|\Sigma_j^{-1}|)
+ C_2
\end{aligned}
$$

对似然估计求 $$\Sigma_l$$ 的偏导：

$$
\begin{aligned}
\nabla_{\Sigma_l^{-1}} f &= 
\sum_{i=1}^n
-\frac{1}{2}w_l^{(i)}((x^{(i)}-\mu_l)(x^{(i)}-\mu_l)^T-\Sigma_j)
+\alpha \sum_{\tilde{z}^{(i)}=l}
-\frac{1}{2}((\tilde{x}^{(i)}-\mu_l)(\tilde{x}^{(i)}-\mu_l)^T-\Sigma_j) \\
&= 0
\end{aligned}
$$

得到 $$\Sigma_l$$ 为：

$$
\Sigma_l = \frac{
\sum_{i=1}^n w_l^{(i)}(x^{(i)}-\mu_l)(x^{(i)}-\mu_l)^T
+\alpha\sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}
(\tilde{x}^{(i)}-\mu_l)(\tilde{x}^{(i)}-\mu_l)^T
}
{\sum_{i=1}^n w_l^{(i)}+\alpha\sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}}
$$



使用Lagrangian数乘法求解 $$\phi_l$$： 

$$
\mathcal{L}(\phi) = l(\phi) + \beta(\sum_{j=1}^k\phi_j - 1)
$$

对似上式求 $$\phi_l$$ 的偏导：

$$
\begin{aligned}
\nabla_{\phi_l} \mathcal{L}
&= \nabla_{\phi_l} \left(
\sum_{i=1}^n w_l^{(i)}\log(\phi_l)
+ \alpha\sum_{\tilde{z}^{(i)}=l}\log(\phi_l)
\right)
+ \beta \\
&= \sum_{i=1}^n \frac{w_l^{(i)}}{\phi_l}
+ \alpha \sum_{i=1}^{\tilde{n}}\frac{1\{\tilde{z}^{(i)}=l\}}{\phi_l}
+ \beta \\
&= 0
\end{aligned}
$$

因此可以得到：

$$
\phi_l = \frac{\sum_{i=1}^n w_l^{(i)} + \alpha \sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}}{-\beta}
$$

由于存在限制条件：

$$
\sum_{l=1}^k \phi_l = (\sum_{i=1}^n \sum_{l=1}^k w_l^{(i)} 
+ \alpha \sum_{i=1}^{\tilde{n}} \sum_{l=1}^k 1\{\tilde{z}^{(i)}=l\})/(-\beta) = 1
$$

得到$$\beta$$ 为：

$$
-\beta = n + \alpha \tilde{n}
$$

最终，得到 $$\phi_l$$ 为：

$$
\phi_l = \frac{\sum_{i=1}^n w_l^{(i)} + \alpha \sum_{i=1}^{\tilde{n}}1\{\tilde{z}^{(i)}=l\}}{n + \alpha \tilde{n}}
$$


