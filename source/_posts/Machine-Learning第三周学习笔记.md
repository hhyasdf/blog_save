---
title: Machine Learning第三周学习笔记
date: 2017-02-10 12:31:22
categories: Machine Learning
tags: 
	- Machine Learning
	- Algorithm
mathjax: true
---

本周的课程的内容大概为： 监督学习里的分类问题的概念和性质、逻辑回归模型和用正则化解决线性回归和逻辑回归中的过拟合问题

#### Classification and Representation ####

当我们尝试解决分类问题时，首先想到的肯定是尝试用线性回归的方案，但这显然是不奏效的。分类问题和回归问题差不多，只不过输出y的取值为一些有限的离散值。我们首先（以及之后一段时间）考虑二类问题，也就是y的值只能等于0或1（之后我们会将其发展到多类问题）。比如说垃圾邮件问题，我们根据邮件的特征 $x^i$ 得到 $y^i$ （$y^i \in (0,1)$，0也本称为负类，1也被称为正类），即是否为垃圾邮件。

当我们用线性回归算法解决分类问题时，会表现得很不好，因为大于1和小于0的值会没有意义。为了解决这个问题我们采取的方法是通过将 $\theta^Tx $ 插入逻辑函数，改变假设函数 $h_\theta(x)$ 的形式来满足 $ 0 \leq h_\theta(x) \leq 1$ 。

在新的假设函数 $h_\theta(x)$ 中我们会用到“Sigmoid Function”，也被叫做“Logistic Function”：

$ h_\theta(x) = g(\theta^Tx) \\\\ z=\theta^Tx \\\\ g(z)=\frac{1}{1+e^{-z}}$   

sigmoid function的图像如下图：

![sigmoid-function](/images/machine-learning/sigmoid-function.png)

$g(\theta^Tx)$ 的值域覆盖了（0,1）之间的值，使其适用于将任意值函数转换为更适合分类的函数。 $h_\theta(x)$ 的结果即为输出为1的概率。比如说，当 $h_\theta(x)=0.7$ 时，我们视为我们输出为1的概率为70%。类似的，$1-h_\theta(x)$ 即为输出为0的概率：

$ h_\theta(x) = P(y=i \mid x;\theta) = 1-P(y=0 \mid x;\theta)   \\\\ P(y=0 \mid x;\theta) + P(y=0 \mid x;\theta) = 1$ 

为了得到我们的分类{1,0}，我们可以将假设函数的输出转化成以下形式：

$h_\theta(x) \geq 0.5 \rightarrow y=1 \\\\ h_\theta(x) \le 0.5 \rightarrow y = 0$  

而我们的逻辑函数 $g(z)$ 具有以下性质：

$g(z) \geq 0.5 \quad when \quad z \geq 0$ 

将 $z$ 替换成 $\theta^Tx$ 之后，根据以上推导，我们可以得到以下结论：

$\theta^Tx \geq 0 \rightarrow y = 1 \\\\ \theta^Tx \leq 0 \rightarrow y = 0 $ 

由此我们可以看到，$\theta^Tx = 0$ 是一个比较特殊的状态，我们将其称之为决策边界（decision boundary）

![decision-boundary-example](/images/machine-learning/decision-boundary-example.png)

####  Logistic Regression Model ####

如果我们将线性回归中的代价函数同样用于逻辑回归的话，我们会得到一个波动较大，具有许多局部最低点的代价函数，换句话说，它不会是个凸函数。

因此，在逻辑回归中我们用到的代价函数如下：

$J(\theta) = \frac{1}{m}\sum\limits_{i=1}^{m} Cost(h_\theta(x^i,y^i))  \\\\ Cost(h_\theta(x),y) = -log(h_\theta(x))  \quad if \quad y=1  \\\\ Cost(h_\theta(x),y) = -log(h_\theta(x))  \quad if \quad y=1 \\\\ Cost(h_\theta(x),y) = -log(1-h_\theta(x))  \quad if \quad y=0 $ 

当 $y=1$ 时，我们以 $J(\theta) $ 为纵坐标， $h_\theta(x)$ 为横坐标绘制可得下图：

![logistic-cost-function-y=1](/images/machine-learning/logistic-cost-function-y=1.png)

类似的，当 $y=0$ 时我们可以得到下图：

![logistic-cost-function-y=0](/images/machine-learning/logistic-cost-function-y=0.png)

$Cost(h_\theta(x),y)=0 \quad if \quad h_\theta(x) =y \\\\ Cost(h_\theta(x),y) \rightarrow \infty \quad if \quad y = 0 \quad and \quad h_\theta(x) \rightarrow 1 \\\\ Cost(h_\theta(x),y) \rightarrow \infty \quad if \quad y = 1 \quad and \quad h_\theta(x) \rightarrow 0$

如果输出y等于0，当我们的假设函数输出也为0时，代价为0，此时当假设输出接近1时，代价趋于无穷大；如果输出y等于1，当我们的假设函数输出也为1时，代价为0，此时当假设输出接近0时，代价趋于无穷大。注意，这样我们就得到了对于逻辑回归为凸函数的代价函数 $J(\theta)$。

#### Simplified Cost Function and Gradient Descent ####

通过某种方式，我们可以将代价函数的两种情况简化为一种：

$Cost(h_\theta(x),y)=-y \log(h_\theta(x) - (1-y)\log(1-h_\theta(x)))$ 

当我们将代价函数展开后可得：

$J(\theta)=-\frac{1}{m}\sum\limits_{i=1}^{m}[ y^i \log(h_\theta(x^i)) + (1-y^i)\log(1-h_\theta(x^i))]$ 

向量化实现后：

$ h=g(X \theta) \\\\ J(\theta)=\frac{1}{m}(-y^T\log(h)-(1-y)^T\log(1-h))$

##### Gradient Descent  #####

一般的梯度下降形式为：

$Reapeat : \lbrace\quad \\\\  \ \ \ \  \theta_j:=\theta_j-\alpha \frac \delta {\delta\theta_j}J(\theta) \quad \\\\ \rbrace $

将代价函数带入并求偏导，我们可以得到：

$Reapeat : \lbrace\quad \\\\  \ \ \ \  \theta_j:=\theta_j-\frac\alpha{m}\sum\limits_{i=1}^m(h_\theta(x^i)-y^i)x_j^i\quad \\\\ \rbrace $

注意到这和我们在线性回归中使用的算法相同，我们仍然必须同时更新 $\theta$ 中的所有值。

当然，我们在实际运用机器学习算法时可以用到更高级的算法（如："Conjugate gradient", "BFGS", and "L-BFGS"）来优化我们的下降过程，用已有的库是很好的选择。

####  Multiclass Classification: One-vs-all  ####

现在我们将从二类问题推广到多类问题。也就是 $y^i \in (0,1,...,n)$。

因为 $y^i \in (0,1,...,n)$，我们将我们的问题分为 n+1 个二类问题，在每一个问题中，我们预测输出y在某一个类中的可能性：

$y \in {0,1...n} \\\\ h_\theta^0(x)=P(y=0 \mid x;\theta) \\\\ h_\theta^1(x)=P(y=1 \mid x;\theta) \\\\ ...\\\\ h_\theta^n(x)=P(y=n \mid x;\theta) \\\\ prediction= \max_i( h_\theta ^{(i)}(x) )$

每次我们选择其中一个类，并将其他所有类看成另外一个类，然后不断地重复二类问题中用到的算法，然后找到输出（概率）最高的假设为我们的预测。下图即三类问题的实际应用：

![one-vs-all](/images/machine-learning/one-vs-all.png)

####  Solving the Problem of Overfitting  ####

考虑从x∈R预测y的问题。下面的最左边的图显示了将 $y =θ_0+θ_1x$ 拟合到数据集的结果。 

![under-and-overfitting](/images/machine-learning/under-and-overfitting.png)

我们看到数据并不真的位于直线上，因此拟合度不是很好。但是，如果我们加一个特征量 $x^2$，然后拟合 $y = \theta_0 + \theta_1x + \theta_2x^2$，然后我们的模型变得更好地拟合了数据，我们可能会天真地以为，当我们模型的阶数变得更高后，会拟合得更好，然而，阶数过高也会有风险：上图中最右边即为5阶函数拟合的图形，我们可以看到尽管曲线完美地经过了每一个点，但它却不是一个很好的预测模型。左侧的图即为欠拟合（underfitting）实例，右侧的图即为过拟合（overfitting）实例。

欠拟合（underfitting）或高偏差（high bias）是我们的假设函数 h 的形式不能很好地映射数据的趋势的情况，它通常是由一个太简单或太少功能的函数引起的。 另一个极端，过拟合（overfitting）或高方差（high variance）是由适合可用数据的假设函数引起的，但不能很好地推广以预测新数据。它通常是由一个复杂的函数引起的，它产生了大量与数据无关的不必要的曲线和角度。

此术语适用于线性和逻辑回归。 解决过度拟合问题有两个主要选择：

1.Reduce the number of features:

* Manually select which features to keep.
* Use a model selection algorithm (studied later in the course.

2.Regularization:

* Keep all the features, but reduce the magnitude of parameters $\theta_j$
* Regularization works well when we have a lot of slightly useful features.

#### Regularization ####

##### Cost Function #####

如果我们的假设函数过拟合，我们可以增加它们的成本来减少我们的函数中的一些项所携带的权重。

比如我们想让下面的式子更加接近平方：

$\theta_0 + \theta_1x + \theta_2x^2 + \theta_3x^3 + \theta_4x^4$

那么我们就会想要降低 $\theta_3x^3$ 和 $\theta_4x^4$ 项的权重。不需要去除这些项或改变我们假设函数的形式，我们可以修改我们的代价函数：

$min_\theta\ \dfrac{1}{2m}\sum\limits_{i=1}^m (h_\theta(x^{(i)}) - y^{(i)})^2 + 1000\cdot\theta_3^2 + 1000\cdot\theta_4^2$

可以看到，我们在代价函数后面加了两个项来增加 $\theta_3$ 和 $\theta_4$ 的成本。现在，为了使代价函数接近0，我们必须将 $\theta_3$、$\theta_4$ 的值减小到接近0。这将大大减小我们的假设函数中的 $θ_3x^3$ 和 $θ_4x^4$ 的值。因此，我们可以看到新的假设（由粉红色曲线描绘）看起来像一个二次函数，但由于超小项 $θ_3x^3$ 和 $θ_4x^4$ 可以更好地拟合数据。

![regularized-costfuction](/images/machine-learning/regularized-costfuction.png)

我们还可以将我们所有的 $ \theta$ 参数放在一个求和中：

$min_\theta\ \dfrac{1}{2m}\ \left[ \sum\limits_{i=1}^m (h_\theta(x^{(i)}) - y^{(i)})^2 + \lambda\ \sum\limits_{j=1}^n \theta_j^2 \right]$ 

式中的 $\lambda$ 即为 正则化参数（regularization parameter）。它决定了θ参数的成本是多少。

使用上述成本函数与额外的求和，我们可以平滑我们的假设函数的输出，以减少过拟合。如果λ选择太大，它可能平滑函数太多，并导致欠拟合。如果λ= 0或太小则会导致其他项的权重大大减小。

##### Regularized Linear Regression #####

我们可以将正则化运用到线性回归和逻辑回归。 首先讲线性回归。

* ### Gradient Descent

对于梯度下降。我们将 $\theta_0$ 和其他模型参数分开，因为我们不想惩罚 $\theta_0$ 

$Repeat \lbrace \\\\  \ \ \ \ \theta_0:=\theta_0-\alpha\frac{1}{m}\sum\limits_{i=1}^{m}(h_\theta(x^i-y^i)x_0^i)  \\\\ \ \ \ \ \theta_j:=\theta_j-\alpha \left[ \frac{1}{m}\sum\limits_{i=1}^{m}(h_\theta(x^i-y^i)x_0^i) \right]  \quad j \in {1,2...n}\\\\   \rbrace$

我们正则化的过程体现在 $\frac{\lambda}{m}\theta_j$ 项中。通过一些操作，我们的更新规则也可以表示为：

$\theta_j := \theta_j(1 - \alpha\frac{\lambda}{m}) - \alpha\frac{1}{m}\sum\limits_{i=1}^m(h_\theta(x^{(i)}) - y^{(i)})x_j^{(i)}$

上式中第一项的系数 $1 - \alpha\frac{\lambda}{m}$ 始终小于1.直观地，你可以看到它在每次更新时将θj的值减少一些量。注意，第二项和以前的版本完全相同。

* ### Normal Equation

现在我们将正则化运用到正规方程里来。

要正则化正规方程，等式依然和以前一样，除了我们需要在其中加入一项：

$\theta=(X^TX+\lambda L)^{-1}X^T y  \\\\  where \quad L= \begin{bmatrix} 0 & & & & \newline & 1 & & & \newline & & 1 & & \newline & & & \ddots & \newline & & & & 1 \newline\end{bmatrix}$

L是一个矩阵，在左上角为0，在对角线上为1，其他地方为0。它应该具有维度（n + 1）×（n + 1）。 直观地，这是单位矩阵（尽管我们不包括 $x_0$）乘以单个实数λ。

回想一下，如果 $ m \leq n$，则 $X^TX$ 是不可逆的。 然而，当我们添加项 $λ⋅L$ 时，则 $X^TX +λ⋅L$变为可逆的。

##### Regularized Logistic Regression #####

我们可以用与我们正则化线性回归类似的方式处理逻辑回归，来避免过度拟合。 下图显示了由粉色线显示的正则化函数如何比由蓝色线表示的非正则化函数更不可能过拟合：

![regularized-logistic-regression](/images/machine-learning/regularized-logistic-regression.png)

* ### Cost Function

回想，我们原本的逻辑回归成本函数为：

$J(\theta) = - \frac{1}{m} \sum\limits_{i=1}^m \large[ y^{(i)}\ \log (h_\theta (x^{(i)})) + (1 - y^{(i)})\ \log (1 - h_\theta(x^{(i)})) \large]$

我们可以通过在最后加一项来将其正则化：

$J(\theta) = - \frac{1}{m} \sum\limits_{i=1}^m \large[ y^{(i)}\ \log (h_\theta (x^{(i)})) + (1 - y^{(i)})\ \log (1 - h_\theta(x^{(i)}))\large] + \frac{\lambda}{2m}\sum_{j=1}^n \theta_j^2$ 

第二个叠加，$\sum_{j=1}^n \theta_j^2$ 意味着明确排除偏差项 $\theta_0$ ，我们可以注意到索引从1到n，当下降时，我们只需不断运行：

![regularized-logistic-regression-descent](/images/machine-learning/regularized-logistic-regression-descent.png)

