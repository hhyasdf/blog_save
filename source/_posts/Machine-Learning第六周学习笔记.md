---
title: Machine-Learning第六周学习笔记
date: 2017-03-18 13:55:58
categories: Machine Learning
tags: 
	- Machine Learning
	- Algorithm
mathjax: true
---

本周课程讲的主要是在实际运用机器学习时所要注意的地方，通常会遇到的状况以及解决方案等等。

很多人在处理机器学习问题时，大多数时间都是“凭感觉”的，也就是说，当他们遇到诸如过拟合、欠拟合等模型表现不佳的情况时，他们通常靠直觉决定下一步该做什么。当然这通常会导致大量时间的浪费。在机器学习中，我们也有一系列方法来决定，为了优化模型，接下来你是该收集更多数据还是增加特征量数量等等。

Once we have done some trouble shooting for errors in our predictions by:

* Getting more training examples
* Trying smaller sets of features
* Trying additional features
* Trying polynomial features
* Increasing or decreasing λ

We can move on to evaluate our new hypothesis.

#### Evaluating a Hypothesis ####

要判断模型的表现，我们就需要一个合适的方法得到具体的模型的表现，只有这样我们才能理性地判断模型是否有效。

然而，较小的训练误差并不能表示模型一定有效，因为我们的模型最后是用来泛化未知数据的，然而，当我们找到最佳的模型参数时，模型已经处于对训练集高度优化的状态，但这种状态并不足以说明模型预测未知数据的表现。

所以通常，我们将所有数据分为训练集（training set）和测试集（test set），并用训练集训练模型，用测试集衡量模型的表现。当然，如果是有序数据集，最好随机选取两集合内的元素。

<!-- more -->

对于不同的问题，我们也有不同的衡量方式。对于线性回归：

$J_{test}(\Theta) = \dfrac{1}{2m_{test}} \sum_{i=1}^{m_{test}}(h_\Theta(x^{(i)}_{test}) - y^{(i)}_{test})^2$

对于分类问题，我们提取分类错误率：

$err(h_\Theta(x),y) = \begin{matrix} 1 & \mbox{if } h_\Theta(x) \geq 0.5\ and\ y = 0\ or\ h_\Theta(x) < 0.5\ and\ y = 1\newline 0 & \mbox otherwise \end{matrix} \\\\ \text{Test Error} = \dfrac{1}{m_{test}} \sum^{m_{test}}_{i=1} err(h_\Theta(x^{(i)}_{test}), y^{(i)}_{test})$

这表示了错误分类所占的比例。

#### Model Selection and Train/Validation/Test Sets ####

假如你想要确定对于某组数据最合适的多项式次数是几次、怎样选用正确的特征来构造学习算法或者假如你需要正确选择学习算法中的正则化参数λ，你应该怎样做呢？这些问题我们称之为模型选择问题。

对于这种问题，我们通常需要把数据分为三组：训练集、（交叉）验证集（cross validation set）和测试集。

这样，当我们有多个不同阶数（或正则化参数等等不同特性）的模型时（通常是按一定规律变化的，比如阶数递增），我们就可以用CV集判断每个模型的效果，并选择其中最好的一个，以此作为较好的假设。

当然，因为与之前评估假设同样的理由，我们也需要一个单独的测试集来评估模型的泛化表现。

一种对数据集的分法：

* Training set: 60%
* Cross validation set: 20%
* Test set: 20%

这样我们就可以出于不同的目的来分别计算每个模型在每个数据集里的“误差量（error values）”：

1. Optimize the parameters in Θ using the training set for each polynomial degree.
2. Find the polynomial degree d with the least error using the cross validation set.
3. Estimate the generalization error using the test set with $J_{test}(\Theta^{(d)})$, (d = theta from polynomial with lower error)

#### Diagnosing Bias vs. Variance ####

首先我们需要了解欠拟合、过拟合和偏倚、方差之间的关系。通常，高偏倚对应着欠拟合，高方差对应着过拟合。我们在处理机器学习问题中，需要判断预测中的问题原因是高偏倚还是高方差，以此来判断接下来该做什么。

让我们以线性回归中的阶数（d）选择来看看如何判断高偏倚或高方差：

![diagnosing-bias-vs-variance](/images/machine-learning/diagnosing-bias-vs-variance.png)

随着我们的阶数 d 不断增加，训练误差不断减小，而CV误差则是先减少后增加。由此我们可以得出结论：

* 高偏倚（欠拟合）时：$J_{train}(\Theta)$ 和 $J_{CV}(\Theta)$ 都很大
* 高方差（过拟合）时：$J_{train}(\Theta)$ 很小，而 $J_{CV}(\Theta)$ 很大并远远大于 $J_{train}(\Theta)$ 

#### Regularization and Bias/Variance ####

然后让我们来看看怎样自动选择正则化参数 $\lambda $。首先我们以4阶函数为例，画出不同 $\lambda$ 下的表现：

![regularization-and-bias⁄variance](/images/machine-learning/regularization-and-bias⁄variance.png)

从上图中，我们可以看到，随着 $\lambda$ 增加，我们的拟合变得更加刚性。另一方面，当 $\lambda$ 趋近于 0，模型表现为过拟合。所以，我们要如何选择一个“刚刚好”的正则化参数呢？我们采用以下方法：

1. Create a list of lambdas (i.e. λ∈{0,0.01,0.02,0.04,0.08,0.16,0.32,0.64,1.28,2.56,5.12,10.24});
2. Create a set of models with different degrees or any other variants.
3. terate through the λs and for each λ go through all the models to learn some Θ.
4. Compute the cross validation error using the learned Θ (computed with λ) on the $J_{CV}(\Theta)$ **without** regularization or λ = 0.
5. Select the best combo that produces the lowest error on the cross validation set.
6. Using the best combo Θ and λ, apply it on $J_{test}(\Theta)$ to see if it has a good generalization of the problem.

#### Learning Curves ####

学习曲线（Learning Curves）是以样本数量为横轴，以误差值为纵轴的图像。它是一种很好的判断一个学习算法是否处于方差、偏倚问题或二者都有的工具。

我们可以很简单地知道，当训练集的数据量很小（只有一两个）时，我们可以很容易地得到 0 误差，因为我们可以很快地找到经过所有给定点的曲线。而后，随着训练集的变大，模型的误差会变大，并且当数据量增大到一定程度时，误差增大的趋势会变得平缓。

现在让我们看看高偏倚和高方差时学习曲线的表现：

![learning-curves-high-bias](/images/machine-learning/learning-curves-high-bias.png)

当模型有高偏倚的问题时，随着训练集的增大，$J_{train}(\Theta)$会不断增大，而$J_{CV}(\Theta)$会不断减小，并到达一定程度时，$J_{train}(\Theta) \approx J_{CV}(\Theta)$ 。此时，获得更多的训练样本个并不会起到帮助。

![learning-curves-high-variance](/images/machine-learning/learning-curves-high-variance.png)

当模型处于高方差的状态时，$J_{train}(\Theta)$ 也会随着训练样本的增多不断增加。 $J_{CV}(\Theta)$ 会持续下降而不是趋于平缓。$J_{train}(\Theta) < J_{CV}(\Theta)$ ，但两者之间的差距会很大。此时，获得更多的训练样本是有帮助的。

#### Deciding What to Do Next Revisited ####

以下是不同的问题和对应解决办法：

* **Getting more training examples:** Fixes high variance
* **Trying smaller sets of features:** Fixes high variance
* **Adding features:** Fixes high bias
* **Adding polynomial features:** Fixes high bias
* **Decreasing λ:** Fixes high bias
* **Increasing λ:** Fixes high variance.

##### **Diagnosing Neural Networks** #####

* 一个具有较少层数和激励单元的神经网络容易表现得欠拟合。但它是容易计算的。
* 一个大型的具有更多参数的神经网络容易变得过拟合，并且计算代价比较大。这种情况下，可以使用正则化来解决过拟合

默认使用一个隐藏层是一个很好的开始。你可以用CV集来测试不层数的神经网络来从中获得一个较好的层数。

##### Model Complexity Effects: #####

* 低阶（低复杂度）的模型有较大的偏倚和低方差，这种情况下，模型一致性差
* 高阶（高复杂度）的模型能很好地拟合训练集的数据，但不能很好地拟合测试集，这种情况下，模型在训练集上具有低偏倚和高方差
* 实际运用中，我们需要选择一个处于两者之间的情况，在这种情况中，模型具有很好的泛化能力并且能较好地拟合训练集。

#### Machine Learning System Design ####

接下来我们将谈到机器学习系统的设计。我们将以垃圾邮件分类器为例。

当我们有一堆邮件作为数据集时，我们可以为每一封邮件构建一个向量，向量中的每一个条目代表一个单词，并通常包括10000到50000个条目。我们通过查找我们的数据集中最常用到的单词来收集。如果在该邮件中找到了这个单词，我们将其对应的条目置为 1。如果没有找到则置为 0。一旦我们准备好所有的向量，我们就可以开始训练我们的算法，并用其进行分类。

![system-design-spam-classifier](/images/machine-learning/system-design-spam-classifier.png)

那么我们应该怎样提高分类器的准确性呢？

* Collect lots of data (for example "honeypot" project but doesn't always work)
* Develop sophisticated features (for example: using email header data in spam emails)
* Develop algorithms to process your input in different ways (recognizing misspellings in spam).

我们很难给出哪个选择是最有帮助的。

##### Error Analysis #####

我们所建议的解决机器学习的方法途径是：

* Start with a simple algorithm, implement it quickly, and test it early on your cross validation data.
* Plot learning curves to decide if more data, more features, etc. are likely to help.
* Manually examine the errors on examples in the cross validation set and try to spot a trend where most of the errors were made.

例如，假设我们有500封电子邮件，我们的算法错误分类了100个电子邮件，我们可以手动分析100个电子邮件，并根据它们是什么类型的电子邮件将它们分类。然后，我们可以尝试提出新的线索和功能，将帮助我们正确地分类这100个电子邮件。因此，如果我们的错误分类电子邮件大多是木马邮件，我们可以从中找到一些特殊的规律，并添加到我们的模型中。我们还可以根据错误率来决定用什么办法识别每个单词：

![system-design-spam-classifier-word-analyze](/images/machine-learning/system-design-spam-classifier-word-analyze.png)

将错误结果作为单个数值是非常重要的。否则，很难评估算法的性能。例如，如果我们使用词干提取，这是将具有不同形式（fail/failing/failed）的同一词当作一个词（fail）的过程，并且得到3％的错误率而不是5％，那么我们应该将它添加到我们的模型。然而，如果我们试图区分大写字母和小写字母，并得到3.2％的错误率，而不是3％，那么我们应该避免使用这个新的功能。因此，我们应该尝试新的东西，得到一个量化的错误率，并根据我们的结果决定是否要保留新的功能。

##### Error metrics for Skewed Classes  #####

之前我们提到，采用一个合适的方式度量误差是对于我们改进算法的很重要的一件事情。然而，在分类问题中，存在这一种很特殊的情况，我们需要用与之前定义的不同的方式来度量误差，它叫做“偏斜类（skewed classes）”。

让我们考虑之前讨论的癌症预测的问题。我们假设我们得到了一个不错的模型，并且经过训练，它的准确率达到了99%，这意味着只有1%的错误。而随后我们发现，训练集中只有0.5%的患者真正得了癌症，这样看来，1%的概率就显得不怎么好了。因为如果我们的程序改为一直预测 y = 0，那么错误率将只有 0.5%，但我们并不能说 y = 0 是一个很好的机器学习算法，或者说我们提升了它的性能。

因此，如果你有一个偏斜类，用分类精确度并不能很好地衡量算法的效果，因为你可能会获得一个很高的精确度和一个很低的错误率，但我们并不知道我们是否真的优化了算法。

这个时候，我们采用一种不同的衡量方式：叫做查准率（precision）和召回率（recall）。

![skewed-classes](/images/machine-learning/skewed-classes.png)

查准率和召回率都越高越好。这样，即便我们有一个非常偏斜的类，算法也不能够"欺骗"我们。仅仅通过预测y总是等于0或者y总是等于1它没有办法得到高的查准率和高的召回率。因此我们能够更肯定，拥有高查准率或者高召回率的模型，是一个好的分类模型，给予我们一种更直接的方法来评估模型的好与坏。这是一种更好的评估学习算法的标准。

在许多应用中，我们试图平衡查准率和召回率来获得更好的模型。我们可以通过调整逻辑回归算法中的临界值（h > 临界值 输出 1）来调整查准率和召回率。当我们增大临界值时，说明这种情况下，我们需要有相当的把握才能做出决定，而这样会表现为较高的查准率和较低的召回率。反之，降低临界值的话，我们会得到一个具有较低查准率和较高召回率的模型。

对于大多数模型，我们需要通过改变临界值来权衡查准率和召回率。当然我们也可以通过 F值（也叫做 F1 值）来结合查准率和召回率自动选择模型参数：

$F = 2\frac{RP}{R+P} \quad\text{P = precision, R=recall}$

这样我们就可以通过CV集来测试不同的模型，从而自动选择一个较好的临界值。