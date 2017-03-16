---
title: Machine Learning第五周学习笔记
date: 2017-03-16 14:33:08
categories: Machine Learning
tags: 
	- Machine Learning
	- Algorithm
mathjax: true
---

本周内容介绍了用神经网络算法学习的操作过程。

同样的，我们先做出一些声明：

* L = 神经网络中所有的层数
* $s_l$ = 第 $l$ 层中的激励单元数
* K = 输出单元（类）的数量

#### Cost Function ####

我们会开始讲一种在给定训练集下为神经网络拟合参数的学习算法，正如我们所讨论的大多数学习算法一样，我们先从代价函数开始讲起。

在神经网络中，我们可能会有很多输出节点。我们将 $h_\Theta(x)_k$ 表示为第 $k$ 个输出的假设。我们的代价函数将是在逻辑回归中使用的代价函数的一个泛化。回想一下，正则化逻辑回归的成本函数是：

$J(\theta) = - \frac{1}{m} \sum_{i=1}^m [ y^{(i)}\ \log (h_\theta (x^{(i)})) + (1 - y^{(i)})\ \log (1 - h_\theta(x^{(i)}))] + \frac{\lambda}{2m}\sum_{j=1}^n \theta_j^2$

对于神经网络，它变得略微有些复杂：

$\begin{gather} J(\Theta) = - \frac{1}{m} \sum_{i=1}^m \sum_{k=1}^K \left[y^{(i)}_k \log ((h_\Theta (x^{(i)}))_k) + (1 - y^{(i)}_k)\log (1 - (h_\Theta(x^{(i)}))_k)\right] + \frac{\lambda}{2m}\sum_{l=1}^{L-1} \sum_{i=1}^{s_l} \sum_{j=1}^{s_{l+1}} ( \Theta_{j,i}^{(l)})^2\end{gather}$

我们增加了几个嵌套求和把所有假设的成本相加来表示整个模型的成本。在正则化部分我们也同样需要考虑每层的权重矩阵中的每一个模型参数。当然，不包括偏置单元。



<!-- more -->

#### Backpropagation Algorithm ####

反向传播算法（Backpropagation Algorithm）是我们在神经网络中为了最小化成本函数需要用到的算法。与在逻辑回归和线性回归中的梯度下降算法一样，我们的目标是计算：

$\min_\Theta J(\Theta)$

也就是说我们需要找到一最优的模型参数 $\Theta$ 的集合来使我们的成本函数最小化。首先让我们来看看用来计算 $J(\Theta)$ 的偏导数的方程：

$\dfrac{\partial}{\partial \Theta_{i,j}^{(l)}}J(\Theta)$

为了计算这个偏导项，我们用以下算法：

![neural-networks-backpropagation-algorithm](/images/machine-learning/neural-networks-backpropagation-algorithm.png)

##### **Back propagation Algorithm** #####

首先给定训练集：$\lbrace (x^{(1)}, y^{(1)}) \cdots (x^{(m)}, y^{(m)})\rbrace$

* 对于所有的 $(l,i,j)$ 将 $\Delta^{(l)}_{i,j}$ 初始为 0

从 t = 1 到 10 遍历训练集：

1. Set $a^{(1)} := x^{(t)}$ 
2. 用正向传播（forward propagation）计算 $a^{(l)} \text{  for  l=2,3,...,L}$

![neural-networks-forward-propagation](/images/machine-learning/neural-networks-forward-propagation.png)

3. 用 $y^{(t)}$ 计算 $\delta^{(L)} = a^{(L)} - y^{(t)}$ 

   其中 L 是总共的层数，$a^{(L)}$ 是最后一层的所有激励单元的输出向量。所以我们对于最后一层的“误差值”就是最后一层的实际计算结果和 $y$ 中正确输出之间的差异。要获得在最后一层之前层的 $\delta$ 值，我们可以从左到右用一个方程（$.*$ 表示矩阵对应元素之间的两两相乘）：

4. 用 $\delta^{(l)} = ((\Theta^{(l)})^T \delta^{(l+1)})\ .\*\ a^{(l)}\ .\*\ (1 - a^{(l)})$  计算  $\delta^{(L-1)},\delta^{(L-2)},...,\delta^{2}$ 

   层 $l$ 的 $\delta$ 值通过将下一层中的 $\delta$ 值乘以层 $l$ 的 $\theta$ 矩阵来计算。然后我们用称为 g' 或 g-prime 的函数（就是激励函数 g 对于 $z^{(l)}$ 的导数）进行元素乘法。g-prime 导数项也可以写成：

   $g'(z^{(l)}) = a^{(l)}\ .*\ (1 - a^{(l)})$

5. $\Delta^{(l)}_{i,j} := \Delta^{(l)}_{i,j} + a_j^{(l)} \delta_i^{(l+1)}$ 或用向量化形式，$\Delta^{(l)} := \Delta^{(l)} + \delta^{(l+1)}(a^{(l)})^T$ 

因此，我们不断更新我们的 $\Delta$ 矩阵。最后得到：

* $D^{(l)}_{i,j} := \dfrac{1}{m}\left(\Delta^{(l)}_{i,j} + \lambda\Theta^{(l)}_{i,j}\right)$ , if j ≠ 0.
* $D^{(l)}_{i,j} := \dfrac{1}{m}\Delta^{(l)}_{i,j}$ , if j = 0.

其中：

$\frac \partial {\partial \Theta_{ij}^{(l)}} J(\Theta) = D_{ij}^{(l)}$

#### Backpropagation Intuition ####

回忆神经网络的成本函数。如果我们考虑只有一个类（伯努利分类）的情况（k = 1）并且不考虑正则化，那么成本为：

$cost(t) =y^{(t)} \ \log (h_\Theta (x^{(t)})) + (1 - y^{(t)})\ \log (1 - h_\Theta(x^{(t)}))$

直观地看，$\delta_j^{(l)}$ 是 $a^{(l)}_j$ 的“误差”。正式点来说，$\delta$  实际上是成本函数的导数：

$\delta_j^{(l)} = \dfrac{\partial}{\partial z_j^{(l)}} cost(t)$

回想我们的导数是与成本函数相切的线的斜率，因此，斜率越陡，结果越不准确。让我们考虑下面的神经网络，看看如何计算一些 $\delta^{(l)}_j$ ：

![neural-networks-backpropagation-algorithm-delta-compute](/images/machine-learning/neural-networks-backpropagation-algorithm-delta-compute.png)

从上图中我们可以看出，为了计算 $\delta_2^{(2)}$ 我们将权重矩阵 $\Theta_{12}^{(2)}$ 和 $\Theta_{22}^{(2)}$ 和它们在每条边右边对应的 $\delta$ 值相乘。因此我们得到 $\delta^{(2)}_2=\Theta^{(2)}_{12}\*\delta^{(3)}_{1}+\Theta^{(2)}_{22}\*\delta^{(3)}_2$.为了计算每一个 $\delta^{(l)}_j$ ，我们可以从图像的右边开始算起。将每条边视为 $\Theta_{ij}$ ，从左到右，要计算 $\delta^{(l)}_j$ ，只需要计算 $a^{(l)}_j$ 的每个权重与它对应的 $\delta$ 乘积之和。另外一个例子是：$\delta_2^{(3)} = \Theta_{12}^{(3)} \* \delta_1^{(4)}$ 。

#### Implementation Note: Unrolling Parameters ####

为了某些优化函数和计算方便的需要，我们可以将矩阵展开为向量的形式，并且也可以很方便地将对应向量还原。octave中对应的函数为 thetaVector( ) 和 reshape( )。

#### Gradient Checking ####

为了验证模型是否正确地下降，我们可以通过计算出代价函数的偏导来验证梯度的正确性。对于一元函数：

$\dfrac{\partial}{\partial\Theta}J(\Theta) \approx \dfrac{J(\Theta + \epsilon) - J(\Theta - \epsilon)}{2\epsilon}$

对于多元函数，其偏导为：

$\dfrac{\partial}{\partial\Theta_j}J(\Theta) \approx \dfrac{J(\Theta_1, \dots, \Theta_j + \epsilon, \dots, \Theta_n) - J(\Theta_1, \dots, \Theta_j - \epsilon, \dots, \Theta_n)}{2\epsilon}$

我们一般令 ${\epsilon}$ 为 $10^{-4}$ 。这样我们可以计算出近似的导数来观察反向传播算法是否正常工作。正常情况下  gradApprox ≈ deltaVector。

要注意的是，一旦我们完成一次验证后，我们就不需要再计算近似导数了，因为效率太低。

#### Random Initialization ####

为了达到良好的下降效果，我们需要一个**随机**初始权重矩阵。值得注意的是，如果 $\Theta = 0$ 或类似情况，则每一列的权重值将会保持相等。

#### Putting It Together ####

First, pick a network architecture; choose the layout of your neural network, including how many hidden units in each layer and how many layers in total you want to have.

* Number of input units = dimension of features $x^{(i)}$
* Number of output units = number of classes
* Number of hidden units per layer = usually more the better (must balance with cost of computation as it increases with more hidden units)
* Defaults: 1 hidden layer. If you have more than 1 hidden layer, then it is recommended that you have the same number of units in every hidden layer.

##### **Training a Neural Network** #####

1. Randomly initialize the weights
2. Implement forward propagation to get $h_\Theta(x^{(i)})$ for any $x^{(i)}$
3. Implement the cost function
4. Implement backpropagation to compute partial derivatives
5. Use gradient checking to confirm that your backpropagation works. Then disable gradient checking.
6. Use gradient descent or a built-in optimization function to minimize the cost function with the weights in theta.