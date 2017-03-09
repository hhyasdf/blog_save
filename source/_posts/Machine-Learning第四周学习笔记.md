---
title: Machine Learning第四周学习笔记
date: 2017-03-03 18:11:08
categories: Machine Learning
tags: 
	- Machine Learning
	- Algorithm
mathjax: true
---

这周的内容中简要介绍了监督学习中的神经网络算法的概念和表示。

#### Why is Neural Networks ####

之前我们学习了线性回归、多项式回归和逻辑回归，我们可以在多项式中包含所有的特征量并通过提高多项式的次数来获得复杂度更高以及拟合得更好的模型。但，当所要计算的特征量很大时，多项式的项数会变得特别多。会出现运算量过大的问题。因此，对于特征量个数很大的问题，用多项式拟合并不是一个好做法。

而神经网络算法（Neural Networks）在解决非线性分类问题上，被证明是一种好得多的算法。即使你的输入特征空间、或输入的特征维数n很大也能轻松搞定。

神经网络算法是一种古老的模拟大脑作用效果的算法。大量生物学方面的发现让人们提出了一个假设：“大脑用一种方法处理所有的事情”，而神经网络算法正是给予于这个假设。

#### Model Representation of Neural Networks    ####

同大脑神经网络一样，神经网络算法中也有类似“神经元”的计算单元，每一个单元有类似于“树突”的输入和类似于“轴突”的输出。在我们的模型中，输入是特征量$x_1,...,x_n $ （$x_0$ 为偏置单元（bias unit），始终等于1） ，而输出是假设函数的结果。在神经网络中，我们使用与分类问题同样的逻辑函数 $\frac{1}{1 + e^{-\theta^Tx}}$ ,我们有时也把它叫做 sigmoid (logistic) **activation** function。$\theta$ 模型参数有时也被叫做权重（weight）。

一个简单的表示如下：

$ \begin{bmatrix}x_0 \newline x_1 \newline x_2 \newline \end{bmatrix}\rightarrow\begin{bmatrix}\ \ \ \newline \end{bmatrix}\rightarrow h_\theta(x)$

<!-- more -->

上图中，$x_1$、$x_2$表示输入节点，$x_0$为偏置单元始终等于1，而该层也叫做“输入层（input layer）”，也就是 layer 1。而最后的假设函数 $h_\theta(x)$ 给出输出，所以该层也叫做“输出层（output layer）”。在输出层和输入层之间的所有层成为“隐藏层（hidden layer）”

而后我们来用 $a_0^2 ...a_n^2$ 来标记隐藏层，并且我们把它们叫做“激励单元（activation units）”。

$a_i^j = \text{"activation" of unit i in layer j} \\\\  \Theta^j = \text{matrix of weights controlling function mapping from layer j to layer j+1}$

如果我们将隐藏层画出来，它将是下面这样：

$\begin{bmatrix}x_0 \newline x_1 \newline x_2 \newline x_3\end{bmatrix}\rightarrow\begin{bmatrix}a_1^{(2)} \newline a_2^{(2)} \newline a_3^{(2)} \newline \end{bmatrix}\rightarrow h_\theta(x)$

每个“激励单元”的值：

$a_1^2=g(\Theta_{10}^1x_0+\Theta_{11}^1x_1+\Theta_{12}^1x_2+\Theta_{13}^1x_3) \\\\ a_2^2=g(\Theta_{20}^1x_0+\Theta_{21}^1x_1+\Theta_{22}^1x_2+\Theta_{23}^1x_3) \\\\ a_3^2=g(\Theta_{30}^1x_0+\Theta_{31}^1x_1+\Theta_{32}^1x_2+\Theta_{33}^1x_3) \\\\ h_\theta(x)=a_1^3=g(\Theta_{10}^2a_0^2+\Theta_{11}^2a_1^2+\Theta_{12}^2a_2^2+\Theta_{13}^2a_3^2)$

这意味着在此例中我们用一个 $3 \times 4$ 的矩阵来计算来计算我们的激励单元。我们将每行参数应用到我们的输入，以获得一个激活节点的值。而（假设）输出则是将激励节点与另一个包含第二层节点的权重的参数向量 $\Theta^2$ 相乘的结果应用于逻辑函数得到。

$\Theta^j$ 表示每层节点的权重矩阵。每个矩阵的大小取决于：如果 $j$ 层中有 $s_{j}$ 个节点，而在 $j+1$ 层中有 $s_{j+1}$ 个节点，那么 $\Theta^j$ 的大小即为 $s_{j+1} \times (s_j + 1)$。（有偏置单元的情况）下图描述了我们的模型表示：

![neural-networks-model-representation](/images/machine-learning/neural-networks-model-representation.png)

而后，我们将对上述函数进行向量化实现。我们定义一个新的变量 $z_k^j$ ，它包含了 $g$ 函数中的参数。在之前的例子中，如果我们用变量 z 表示所有参数会得到：

$a_1^2=g(z_1^2) \\\\ a_2^2=g(z_2^2) \\\\ a_3^2=g(z_3^2)$

换句话说，对于第 2 层的第 $k$ 个节点，变量 z 等于：

$z_k^2=\Theta_{k,0}^1x_0+\Theta_{k,1}^1x_1 +...+\Theta_{k,n}^1x_n$

x 和 $z^j$ 的向量表示：

$\begin{align}x = \begin{bmatrix}x_0 \newline x_1 \newline\cdots \newline x_n\end{bmatrix} &z^{(j)} = \begin{bmatrix}z_1^{(j)} \newline z_2^{(j)} \newline\cdots \newline z_n^{(j)}\end{bmatrix}\end{align}$

令 $x= a^1$ ，我们可以将等式重写为：

$z^{(j)} = \Theta^{(j-1)}a^{(j-1)}$

我们将大小为 $s_j \times (n+1)$ （$s_j$是激励节点的数量）的矩阵 $\Theta^{j-1} $ 与维度为 $n+1$ 的向量 $a^{j-1}$ 相乘。得到维度为 $s_j$ 的向量 $z^j$ 。这样我们可以得到表示 $j$ 层激励节点的向量：

$a^{(j)} = g(z^{(j)})$

其中，将逻辑函数运用于 $z^j$ 向量中的每一个元素。当计算完 $a^j$ 之后，我们可以加上一个等于 1 的偏置单元 $a_0^j$。为了计算最后的假设，让我们首先计算另一个 z 向量：

$z^{(j+1)} = \Theta^{(j)}a^{(j)}$

我们将 $\Theta^{j-1} $ 的下一个向量 $\Theta^j$ 与我们刚刚计算得到的所用激励节点相乘得到了最后的 z 向量。因为最后的权重矩阵 $\Theta^j$ 只有一行，所以我们的结果是一个单独的数。这样我们就得到了最后的结果：

$h_\Theta(x) = a^{(j+1)} = g(z^{(j+1)})$

注意，在这最后一步，在层 j 和层 j + 1 之间，我们做的事情与在逻辑回归中做的完全相同。 将所有这些中间层添加到神经网络中允许我们更优雅地产生有趣的和更复杂的非线性假设。



#### Examples and Intuitions ####

我们会通过一个具体的例子来了解神经网络是如何进行计算的。

考虑下面的问题，我们有二进制的输入特征 $x_1 $、$x_2$ ，要么取 1，要么取 0 ，所以 $x_1$、$x_2$ 只能有两种取值。在左边我们只画出了两个正样本和两个负样本，但你可以认为这是一个更复杂的学习问题的简化版本，在这个复杂问题中，我们可能在右上角有一堆正样本，在左下角有一堆用圆圈表示的负样本（如右图）。我们想要学习一种非线性的决策边界来区分正负样本。

![neural-networks-model-representation-example](/images/machine-learning/neural-networks-model-representation-example.png)

观察左边这个简单的例子。由图像的特点，我们可以联想到 XNOR 运算。我们关心的是能否找到一个神经网络模型来拟合这种训练集。

为了能拟合 XNOR 运算，我们先讲解一个稍微简单的神经网络，它拟合了 AND 运算。

依然令 $x_1$、$x_2$ 为二进制输入。我们函数的图像如下：

$\begin{align}\begin{bmatrix}x_0 \newline x_1 \newline x_2\end{bmatrix} \rightarrow\begin{bmatrix}g(z^{(2)})\end{bmatrix} \rightarrow h_\Theta(x)\end{align}$

其中，偏置单位 $x_0$ 始终为 1 。

我们将第一个权重矩阵设置为：

$\Theta^{(1)} =\begin{bmatrix}-30 & 20 & 20\end{bmatrix}$

这样，我们会得到只有当 $x_1$、$x_2$ 都为 1 输出才会为 1 的函数。换句话说：

$h_\Theta(x)=g(-30+20x_1+20x_2) \\\\ \\\\ x_1 =0 \quad \text{and} \quad x_2=0 \quad \text{then} g(-30) \approx 0 \\\\ x_1 =0 \quad \text{and} \quad x_2=1 \quad \text{then} g(-10) \approx 0 \\\\ x_1 =1 \quad \text{and} \quad x_2=0 \quad \text{then} g(-10) \approx 0 \\\\ x_1 =1 \quad \text{and} \quad x_2=1 \quad \text{then} g(10) \approx 1$

因此，我们使用一个小的神经网络而不是使用一个实际的 AND gate 来构建了计算机的一个基本运算。神经网络也可以用于模拟其他所有的逻辑门。以下是逻辑运算符“OR”的示例：

![neural-networks-model-representation-example-OR](/images/machine-learning/neural-networks-model-representation-example-OR.png)

其中 $g(z)$ 如下：

![neural-networks-model-representation-example-OR-gz](/images/machine-learning/neural-networks-model-representation-example-OR-gz.png)

同样，对于 AND，NOR 和 OR 运算，我们只需要改变 $Θ^{(1)}$ 矩阵的取值：

$\begin{align}AND:\newline\Theta^{(1)} &=\begin{bmatrix}-30 & 20 & 20\end{bmatrix} \newline NOR:\newline\Theta^{(1)} &= \begin{bmatrix}10 & -20 & -20\end{bmatrix} \newline OR:\newline\Theta^{(1)} &= \begin{bmatrix}-10 & 20 & 20\end{bmatrix} \newline\end{align}$

而后我们可以将它们放在一起得到 XNOR 逻辑操作（当 $x_1$ 和 $x_2$ 都为 0 或 1 时输出 1）：

$\begin{align}\begin{bmatrix}x_0 \newline x_1 \newline x_2\end{bmatrix} \rightarrow\begin{bmatrix}a_1^{(2)} \newline a_2^{(2)} \end{bmatrix} \rightarrow\begin{bmatrix}a^{(3)}\end{bmatrix} \rightarrow h_\Theta(x)\end{align}$

对于第一层和第二层之间的转换，我们用 $\Theta^1$ 矩阵将 AND 和 NOR 运算结合起来：

$\Theta^{(1)} =\begin{bmatrix}-30 & 20 & 20 \newline 10 & -20 & -20\end{bmatrix}$

对于第二层和第三层，我们用 $\Theta^2$ 矩阵来实现 OR 操作：

$\Theta^{(2)} =\begin{bmatrix}-10 & 20 & 20\end{bmatrix}$

让我们将所有的节点都列出来：

$\begin{align}& a^{(2)} = g(\Theta^{(1)} \cdot x) \newline& a^{(3)} = g(\Theta^{(2)} \cdot a^{(2)}) \newline& h_\Theta(x) = a^{(3)}\end{align}$

这样我们就通过两个隐藏层实现了 XNOR 操作。

#### Multiclass Classification ####

为了将数据分成多个类，我们需要让我们的假设函数返回一个值向量。假设我们将要区分四个类，我们将会用下面的例子来了解分类是如何完成的。该算法将图像作为输入并相应地对其进行分类：

![neural-networks-model-representation-multiclass-classification](/images/machine-learning/neural-networks-model-representation-multiclass-classification.png)

我们将结果集合定义如下：

$y^i= \begin{bmatrix} 1  \newline  0 \newline 0 \newline 0  \end{bmatrix} , \begin{bmatrix} 0  \newline  1 \newline 0 \newline 0  \end{bmatrix} , \begin{bmatrix} 0  \newline  0 \newline 1 \newline 0  \end{bmatrix} , \begin{bmatrix} 0  \newline  0 \newline 0 \newline 1  \end{bmatrix}$

每个 $y^i$ 表示对应于汽车，行人，卡车或摩托车的不同图像。中间的每个隐藏层都为我们提供了一些形成最终假设的新的信息。如下：

$\begin{bmatrix}  x_0 \newline x_1 \newline x_2 \newline ... \newline x_n  \end{bmatrix} \rightarrow \begin{bmatrix}  a_0^2 \newline a_1^2 \newline a_2^2 \newline ...   \end{bmatrix} \rightarrow  \begin{bmatrix}  a_0^3 \newline a_1^3 \newline a_2^3 \newline ...   \end{bmatrix} \rightarrow  ... \rightarrow \begin{bmatrix}  h_\Theta(x)_1 \newline h_\Theta(x)_2 \newline h_\Theta(x)_3 \newline h_\Theta(x)_4  \end{bmatrix} $

我们对一组输入的结果假设可能如下：

$h_\Theta(x) =\begin{bmatrix}0 \newline 0 \newline 1 \newline 0 \newline\end{bmatrix}$

在这种情况下，表示图片属于第三类，或 $h_\Theta(x)_3$，也就是摩托车。

