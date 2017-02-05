---
title: Machine Learning 第一周学习笔记
date: 2017-1-19
categories: Machine Learning
tag: Machine Learning， Algorithm
---

本周主要介绍了机器学习的概念和基本术语及算法



#### What is Machine Learning?####

&nbsp;&nbsp;&nbsp;&nbsp;Two definitions of Machine Learning are offered. Arthur Samuel described it as: "the field of study that gives computers the ability to learn without being explicitly programmed." This is an older, informal definition.

&nbsp;&nbsp;&nbsp;&nbsp;Tom Mitchell provides a more modern definition: "A computer program is said to learn from experience E with respect to some class of tasks T and performance measure P, if its performance at tasks in T, as measured by P, improves with experience E."

&nbsp;&nbsp;&nbsp;&nbsp;Example: playing checkers.

&nbsp;&nbsp;&nbsp;&nbsp;E = the experience of playing many games of checkers

&nbsp;&nbsp;&nbsp;&nbsp;T = the task of playing checkers.

&nbsp;&nbsp;&nbsp;&nbsp;P = the probability that the program will win the next game.

&nbsp;&nbsp;&nbsp;&nbsp;In general, any machine learning problem can be assigned to one of two broad classifications:

&nbsp;&nbsp;&nbsp;&nbsp;Supervised learning and Unsupervised learning.


#### Supervised learning ####

&nbsp;&nbsp;&nbsp;&nbsp;机器学习包括两种方式：监督学习（Supervised learning）和无监督学习（Unsupervised learning）

&nbsp;&nbsp;&nbsp;&nbsp;在监督学习中，我们得到的数据集是已知输出的，换而言之我们可以知道数据集中某一个输入所应该对应的输出，我们当然也会知道数据集中的数据有哪些类型等信息。

&nbsp;&nbsp;&nbsp;&nbsp;监督学习又可以分为两种问题：回归问题（regression problems）和分类问题（classification problems）。在回归问题中，数据的输出是连续的，我们试图在这些连续输出内预测结果，这意味着我们试图将输入变量映射到某些连续函数。而在分类问题中，数据的输出的离散集合是已知的，我们只需要在这个已知的集合中预测输出。换句话说，我们试图将输入变量映射到离散类别。

* Example 1:

Given data about the size of houses on the real estate market, try to predict their price. Price as a function of size is a continuous output, so this is a regression problem.

We could turn this example into a classification problem by instead making our output about whether the house "sells for more or less than the asking price." Here we are classifying the houses based on price into two discrete categories.

* Example 2:

(a) Regression - Given a picture of a person, we have to predict their age on the basis of the given picture

(b) Classification - Given a patient with a tumor, we have to predict whether the tumor is malignant or benign.


#### Unsupervised learning####

&nbsp;&nbsp;&nbsp;&nbsp;在无监督学习中，我们所能获得的关于已知数据集合的信息是非常少的，甚至我们也无法知道我们得到的结果应该是什么样。我们可以通过基于数据中的变量之间的关系对数据进行聚类来导出数据结构，对数据进行分类（没有预先的类型设置）。但我们不一定知道变量的影响。对于无监督学习，没有基于预测结果的反馈。

* Example:

Clustering: Take a collection of 1,000,000 different genes, and find a way to automatically group these genes into groups that are somehow similar or related by different variables, such as lifespan, location, roles, and so on.

Non-clustering: The "Cocktail Party Algorithm", allows you to find structure in a chaotic environment. (i.e. identifying individual voices and music from a mesh of sounds at a cocktail party).

#### Model Representation####

&nbsp;&nbsp;&nbsp;&nbsp;为了以后的学习，这里用 ｍ 表示训练集（training set），及我们用来学习的数据集合，用 x<sup>i</sup> 表示输入的特征，用 y<sup>i</sup> 表输出变量或目标变量（我们需要预测的目标）。而训练集中的一对（x<sup>i</sup>，y<sup>i</sup>）则为一个训练样本（ training example）。上标表示训练集中的某个。我们还将使用 X 表示输入值的空间，Y 表示输出值的空间。在该示例中，X = Y =ℝ。

&nbsp;&nbsp;&nbsp;&nbsp;对于监督学习，我们的目标是：根据训练集找到一个恰好的假设（hypothesis）h（是个函数）使得我们可以根据这个 h : X －> Y 来预测对应x的输出y（例如根据房屋大小预测价格）。我们可以用下图的模型表示。

![model-representation](/images/machine-learning/model-representation.png)

&nbsp;&nbsp;&nbsp;&nbsp;当我们所要预测的输出是连续的，我们称这是一个回归问题，而当ｙ只是一些少量的离散值时，我们称这是一个分类问题。

#### Cost Function####

&nbsp;&nbsp;&nbsp;&nbsp;由于我们需要找到一个最佳的假设（hypothesis），所以我们需要一个权衡假设函数的工具，这就是代价函数（cost function）。这需要使用来自x的输入和实际输出y的假设的所有结果的平均差（实际上是平均值的更好的版本）。

![cost-function](/images/machine-learning/cost-function.png)

&nbsp;&nbsp;&nbsp;&nbsp;对于不同的假设函数 J，我们会得到不同的代价函数值，因此，代价函数值就变成了模型参数（θ<sub>n</sub>）的函数（这里假设 h = θ<sub>0</sub> + θ<sub>1</sub>x）。单价函数也被称作“平方误差函数”或“均方误差”。平均值被减半（1/2），以便于计算梯度下降，因为平方函数的导数项将抵消1/2项。我们可以借助下图理解代价方程做了什么:

![cost-function-image](/images/machine-learning/cost-function-image.png)

&nbsp;&nbsp;&nbsp;&nbsp;如果我们需要得到的假设函数尽量准确（拟合），那么我们就需要代价方程的值尽量小（甚至是0）。

#### Gradient Descent####

&nbsp;&nbsp;&nbsp;&nbsp;梯度下降（Gradient Descent）是机器学习中一种简单又基础的得到最优假设函数（找到代价函数的最小值）的算法。在这里，我们需要理解梯度下降方法来确定模型参数（θ<sub>n</sub>）的值（依然假设 h = θ<sub>0</sub> + θ<sub>1</sub>x）。想象一下，我们基于假设函数 h 绘制出代价函数的图像（将θ<sub>0</sub>作为x轴，θ<sub>1</sub>作为y轴，假设函数的值作为z轴）：

![gradient-descent-cost-function-image](/images/machine-learning/gradient-descent-cost-function-image.png)

&nbsp;&nbsp;&nbsp;&nbsp;根据图像，很显然，如果我们找到了图像中的最低点，那么我们就成功地找到了对应的最优假设函数。现在，我们任意初始化我们的模型参数（即在图像上任意取一点），然后在最陡下降的方向上逐步降低成本函数，每个步骤的大小由参数α确定，其被称为学习速率。上图中每个“星”之间的距离表示由我们的参数α确定的步长。较小的α将导致较小的阶跃，较大的α导致较大的阶跃。通过代价函数 J（θ~0~，θ~1~）的偏导数来确定采取步骤的方向。根据图表上的起始位置，可以在不同的点结束。上图显示了两个不同的起点，分别位于两个不同的地方。（此时找到的是局部最优点）

&nbsp;&nbsp;&nbsp;&nbsp;梯度下降算法可以这样表示：

![gradient-descent--representation](/images/machine-learning/gradient-descent-representation.png)

&nbsp;&nbsp;&nbsp;&nbsp;需要注意的是，我们必须同时更新所有的模型参数θ<sub>n</sub>：

![gradient-descent-update-simultaneously](/images/machine-learning/gradient-descent-update-simultaneously.png)

&nbsp;&nbsp;&nbsp;&nbsp;现在，让我们来简单探讨一下梯度下降算法工作的过程和原理。首先我们先来研究只有一个模型参数的简单的情况（即 h = θ<sub>0</sub> + θ<sub>1</sub>x 中 θ<sub>0</sub> 等于0）：

![gradient-descent-intuition](/images/machine-learning/gradient-descent-intuition.png)

&nbsp;&nbsp;&nbsp;&nbsp;此时，微分项即为一元函数 J（θ<sub>1</sub>）的导数，此时，又因为 θ 减去 α 与倒数的乘积，所以， J（θ<sub>1</sub>）会不断减小（理论上）达到最小值（此时倒数为0）。而对于 α 我们则需要一个合适的大小，如果 α 过大的话，可能会出现不收敛的现象，如果 α 过小的话，又可能会出现无法在合理的时间内找到最优情况的问题（降低速度太慢）。而对于一个固定的合适的 α ，因为 θ 值在靠近最低点，导数也会随之变小，进而即使 α 为一个固定值，减小的幅度也会不断下降，进而达到收敛。

&nbsp;&nbsp;&nbsp;&nbsp;剩下的问题就是如何用代码表示梯度下降的过程了。我们以应用于线性回归问题（h = θ<sub>0</sub> + θ<sub>1</sub>x）举例。当具体应用于线性回归的情况下，可以导出梯度下降方程的新形式。 我们可以替换我们的实际成本函数和我们的实际假设函数，并将公式修改为（整个推导过程即 J（θ<sub>0</sub>，θ<sub>1</sub>...，θ<sub>n</sub>）分别对于 θ<sub>0</sub>，θ<sub>1</sub>...，θ<sub>n</sub> 求偏导）：

![gradient-descent-for-linear-regression](/images/machine-learning/gradient-descent-for-linear-regression.png)

&nbsp;&nbsp;&nbsp;&nbsp;这样我们就将 θ<sub>j</sub> 的两种情况分离为 θ<sub>0</sub> 和 θ<sub>1</sub> 的单独方程。

&nbsp;&nbsp;&nbsp;&nbsp;这一切的要点是，如果我们开始猜测我们的假设，然后重复应用这些梯度下降方程，我们的假设将变得越来越准确。在整个过程中放我们需要查看每个步骤的整个训练集中的每个训练样本，因此这种对原始成本函数 J（θ<sub>0</sub>，θ<sub>1</sub>...，θ<sub>n</sub>）的简单梯度下降算法被称为**批量梯度下降算法**。注意，虽然梯度下降通常易受局部最小值的影响，但我们在这里提出的用于线性回归的优化问题只有一个全局优化问题，没有其他局部优化问题，因此梯度下降总是收敛（假设学习速率α不太大）到全局最小值。在线性问题中，J是凸二次函数。

#### Linear Algebra Review####

&nbsp;&nbsp;&nbsp;&nbsp;这一小节主要介绍了线性代数中矩阵的基本性质和四则运算（略），主要学习到的思想就是，运用矩阵可以用矩阵中简单的计算步骤来计算大量的数据，简化了代码（有些库还对计算性能有一定优化）。
