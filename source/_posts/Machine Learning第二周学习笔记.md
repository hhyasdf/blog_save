---
title: Machine Learning第二周学习笔记
date: 2017-02-07 18:11:08
categories: Machine Learning
tags: 
	- Machine Learning
	- Algorithm
mathjax: true
---

本周内容主要是线性回归的扩展问题（多特征量线性回归）以及对应的梯度下降算法和技巧、正规方程的应用，最后还有octave的基本操作。

#### Multiple Features####

之前我们讲的例子都是一个特征量（比如说预测房价就只用了房屋面积一个特征量），但实际上我们在现实应用中用到的特征量一般都是多个的。先讲一下标记的方法：

>$ x^i_j $ = value of feature $ j $ in the $ i^{th} $ training example
>$ x ^i $ = the column vector of all the feature inputs of the $ i^{th} $ training example
>$ m $ = the number of training examples
>$ n $ = $\left|x^{th}\right| $;(the number of features) 

多变量的线性回归假设函数表达式如下：

$ h(x) = \theta_0 + \theta_1x_1+\theta_2x_2+\theta_3x_3+...+\theta_nx_n $

如果在式中加入 $x_0 = 1 $ 那么上式可以转化成矩阵相乘的形式（其中 $ x_0 = 1 $）：

$h(x) = \left[\begin{matrix} \theta_0\qquad\theta_1\qquad...\qquad\theta_n \end{matrix}\right] \left[\begin{matrix} x_0 \\\\ x_1 \\\\ ... \\\\ x_n\end{matrix}\right]$

<!-- more -->

这是对于一个训练样本的假设函数 $ h()$ 的向量化（vectorization）。

而如果我们将训练集整体（存储在矩阵 $ X ​$ 中）进行运算：

$ X=\left[\begin{matrix} x^1_0 \quad x^1_1 \quad ...  \quad x^1_n\\\\ x^2_0 \quad x^2_1 \quad ...  \quad x^2_n \\\\ ... \\\\ x^m_0 \quad x^m_1 \quad ...  \quad x^m_n \end{matrix}\right] \qquad \theta = \left[\begin{matrix} \theta_0 \\\\ \theta_1 \\\\ ... \\\\ \theta_n\end{matrix}\right] $ 

这样你就可以直接用这两个矩阵进行运算：

$ h_\theta (X) = X\theta $

#### Gradient Descent For Multiple Variables####

对于多特征量的线性回归问题，讲道理和单特征量的线性回归问题是差不多的，只需要加入 $x_0 = 1 $ 递归下降的过程公式就可以写成以下形式：

repeat until convergence: $\qquad \theta_j  :=   \theta_j - \alpha\frac1 m\sum_{i=1}^m(h_\theta(x^i)-y^i)x_j^i ,\quad for \ j = 0 , 1, 2, ... , n $ 

下图将单特征量的情况和多特征量的情况进行了对比：

![gradient-descent-for-multiple-variables](/images/machine-learning/gradient-descent-for-multiple-variables.png)

#### Gradient Descent in Practice I - Feature Scaling####

这一节以及之后的一节介绍的是多特征量能提高梯度下降效率的两个小技巧，本节讲的是特征缩放（feature scaling）。很容易得到，当我们用多个特征量来进行机器学习时，很可能会碰到特征量之间的取值范围的大小差距很大（比如说 $0.1-1$ 和 $10^3-10^6$），如果直接将特征量代入进行梯度下降，很容易会降低下降效率。这是因为θ将在小范围上快速下降，并且在大范围上缓慢下降，因此当变量非常不均匀时，θ将无效地振荡到最佳值。

此时我们采用特征缩放和均值归一化来提高下降效率。 特征缩放包括将输入值除以输入变量的范围（即，最大值减去最小值），得到刚刚为1的新范围。平均归一化涉及从该输入变量的值中减去输入变量的平均值，输入变量导致输入变量的新平均值刚好为零。 要实施这两种技术，请按照以下公式调整输入值：

$ x_i := \frac{x_i - \mu_i}{s_i} $

其中，$ \mu_i $是所有该类特征量的平均值，$ s_i $是特征量的取值范围大小$ \left( {max - min} \right) $或标准偏差（注意：除以范围或除以标准偏差，会得到不同的结果。）

一般我们会将其缩放成范围 $ -1 <= x <= 1 $ 或 $ -0.5 <= x <= 0.5 $，当然这不是硬性要求，我们只是想加速下降的过程，只需要使所有变量都大致在同一范围即可。

#### Gradient Descent in Practice II - Learning Rate####

首先介绍调试梯度下降的方法：

1. 以迭代次数为x轴、以代价函数 $ J(\theta) $的值为y轴绘制图像（推荐）。如果我们能观察到每次迭代代价函数的值都会减小，并且减小的速率越来越小，则下降算法工作正常，如果$ J(\theta) $增加，那么就应该减小 $\alpha$ 值；如果$ J(\theta) $减小速率太慢，则应该增大 $\alpha$ 值。

以下分别为正常工作和异常工作中绘制的图像：

![gradient-descent-work-right](/images/machine-learning/gradient-descent-work-right.png)

![gradient-descent-work-wrong](/images/machine-learning/gradient-descent-work-wrong.png)

2. 自动收敛测试。 如果在一次迭代中$ J(\theta) $减小小于E，则声明收敛，其中E是诸如10-3的一些小值。 然而，在实践中很难选择这个阈值。

#### Features and Polynomial Regression####

我们可以用两种不同的方式改进所采用的特征量和假设函数的形式：

1.我们可以将多个特征量整合成一个来找到更加符合的模型。例如，我们可以通过取 $x_1⋅x_2$将$x_1$和$x_2$组合成新的要素$x_3$。

2.多项式回归。如果直线不能很好地拟合数据，那么我们的假设函数可以不需要是线性的。我们可以通过使它是二次，立方或平方根函数（或任何其他形式）来改变我们的假设函数的行为或曲线。例如，如果我们的假设函数是$h_\theta (x)=\theta_0+\theta_1 x_1$ 那么我们可以基于$x_1$创建附加特征，以获得二次函数$h_\theta(x)=\theta_0+\theta_1 x_1+\theta_2 x_1^2 $或三次函数$h_\theta(x)= \theta_0+\theta_1 x_1+\theta_2 x^2_1+\theta_3 x^3_1$。（在三次方函数中）我们可以创建新的特征量$$x_2 = x_1^2$$和$$x_3=x_1^3$$来进行梯度下降。当然我们也可以将假设函数变为平方根形式：$h_\theta(x)=\theta_0+\theta_1 x_1+\theta_2\sqrt{x_1} $。

当我们使用多项式回归时我们需要注意，此时我们的特征量的取值范围可能差距会变得很大，这时候特征缩放就很重要了。

#### Normal Equation####

梯度下降给出了使代价函数$ J(\theta) $最小化的一种方式。让我们讨论这样做的第二种方式，这次显式地执行最小化，而不诉诸迭代算法。 在“正规方程（normal equation）”方法中，我们将通过显式地取其相对于$\theta_j$的导数并将它们设置为零来最小化$ J(\theta) $。 这允许我们找到最佳的$\theta$而不迭代。 正规方程公式如下：

$ \theta=(X^TX) ^{-1}X^Ty$

![normal-equation-example](/images/machine-learning/normal-equation-example.png)

在正规方程中我们不需要进行特征缩放。

以下是递归下降和正规方程的优缺点比较：

| Gradient Descent           |             Normal Equation              |
| -------------------------- | :--------------------------------------: |
| Need to choose alpha       |         No need to choose alpha          |
| Needs many iterations      |            No need to iterate            |
| O ($kn^2$)                 | O ($n^3$), need to calculate inverse of $X^TX$ |
| Works well when n is large |         Slow if n is very large          |

对于正规方程，它的计算复杂度是$O(n^3)$。因此，如果我们有大量的特征量要进行计算，那么正规方程会很慢。在实践中，如果特征量在10000个以下，那么建议用正规方程计算假设函数。

#### Octave Basic####

这一小节介绍了octave的基本操作和向量化的思想。向量化思想即将一般的在编程中需要用（多个）循环来计算的操作改成用（库）矩阵计算，从而简化了代码和计算步骤，优化了计算性能。
