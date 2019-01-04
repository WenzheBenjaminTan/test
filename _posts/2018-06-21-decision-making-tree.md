---
layout: post
title: "决策树" 
---

# 1、引言

决策树是一种实现分治策略的层次数据结构，它是一种有效的“判别式模型”，可以用于分类和回归。

决策树由一些内部决策节点（decision node）和终端树叶节点（leaf node）组成。每个决策节点$m$实现一个具有离散输出的测试函数$$f_m(\boldsymbol{x})$$，用于标记分支。给定一个输入在决策节点应用测试，并根据测试输出确定下一个分支。这一过程从根节点开始，并递归地重复，直至到达一个树叶节点。这时，树叶节点的值形成输出。

每个决策节点的$$f_m(\boldsymbol{x})$$定义了一个输入空间的判别式，不同的决策树方法对$$f_m(\boldsymbol{x})$$假设不同的模型。每一个树叶节点有一个输出标签：对于分类，该标签是类代码；而对于回归，该标签是一个数值。一个树叶节点定义了输入空间的一个局部区域，落入该区域的输入实例具有相同的输出。区域的边界被从树根到该树叶的路径上的内部决策节点中的判别式定义。

决策树的层次安排使得输入实例的区域可以快速确定。比如说，如果决策是二元的，则在最好情况下每个决策可以去掉一半的备选区域。假设总共有$b$个区域，则在最好情况下可以平均通过$$\log_2b$$次决策找到正确的区域。决策树的另一个优点是可解释性：可以很容易地把决策树转换称一组容易理解的IF-THEN规则。

我们将从一个决策节点只使用一个输入变量的单变量树开始，介绍如何为分类和回归构造这样的树。然后，我们将这种方法推广到多变量树。

# 2、单变量树

在单变量树（univariate tree）中，每个内部决策节点的测试只使用一个输入维$$x_i$$。如果输入维$$x_i$$是非数值（有序）的，取$n$个可能的值之一，则决策节点检查$$x_i$$的值，并取相应的分支，实现一个$n$路划分。如果输入维$$x_i$$是数值（有序）的，则决策节点的测试是比较：

$$f_m(\boldsymbol{x}): x_i \geq w_{m0}$$

其中$$w_{m0}$$是适当选择的阈值。该决策节点将将输入空间一分为二：$$L_m = \{\boldsymbol{x}\mid x_i \geq w_{m0}\}$$ 和 $$R_m = \{\boldsymbol{x}\mid x_i < w_{m0}\}$$。称作二元划分（binary split）。

构造给定训练样本的树的过程叫作树学习。对于给定的训练集，存在许多对它进行无错编码的树，而为了简单起见，我们感兴趣的是寻找其中最小的树。这里树的大小用树中的节点数和决策节点判别式模型的复杂性来度量。寻找最小树问题是NP-完全的，我们必须使用基于启发式的局部搜索方法，在合理的时间内得到合理的树。

我们提供的树学习算法是贪心算法，从包含全部训练数据的根集合开始，每一步都选择最佳划分。依赖于所选取的属性是数值的还是非数值的，每次将数据划分成两个或$n$个子集，然后使用对应的子集递归地进行划分，直到不需要再划分。此时，创建一个树叶节点并标记它。

## 2.1 分类树

在用于分类的树，即分类树（classification tree）中，划分的优劣用不纯性量度（impurity measure）来定量分析。如果划分后对应的实例都属于相同的类，则称该划分是纯的。

对于节点$m$，令$$N_m$$为到达节点$m$的训练实例数。对于根节点，$$N_m$$等于$N$。设$$N_m$$个实例中$$N_m^i$$个属于$C_i$类，则$$\sum_iN_m^i = N_m$$。如果一个实例到达节点$m$，则它属于$$C_i$$类的概率估计为

$$\widehat{p}(C_i\mid \boldsymbol{x},m) \equiv p_m^i = \frac{N_m^i}{N_m}$$

如果对于所有的$i$，$$p_m^i$$为0或1，则节点$m$对应的划分是纯的。如果划分是纯的，则我们不需要进一步划分，并可以添加一个树叶节点，用$$p_m^i$$为1的类标记。

一种度量不纯性的可能函数是熵函数

$$H_m = -\sum_{i=1}^Kp_m^i \log_2p_m^i$$

其中定义$$0\log 0 \equiv 0$$。在信息论中，熵是对一个实例的类代码进行编码所需要的最少位数。对于二类问题，如果$$p^1=1$$，$$p^0=0$$，则所有的实例都属于$$C_1$$类，我们什么都不需要发送，熵为0。如果$$p^1=p^0=0.5$$，则我们需要发送一位数来通告两种情况之一，即熵为1。在这两个极端之间，我们可以设计编码，更可能的类用较短的编码，更不可能的类用较长的编码，可以让平均下来每个信息使用不足一位。当存在$K > 2$时，形同的讨论成立，并且当$p^i=1/K$时熵最大，为$\log_2 K$。

如果节点$m$对应的划分是不纯的，则应该进一步划分实例，降低不纯度。对于数值属性，可能存在多个划分位置。在这些可能的划分中，我们寻找最小化划分后不纯度的划分。当然，这是一个局部最优，不能保证找到最小的决策树。

设在节点$m$，$$N_m$$个实例中$$N_{mj}$$个取分支$j$，即在这些实例在测试$$f_m(\boldsymbol{x})$$时返回的输出为$j$。对于具有$n$个值的非数值属性，有$n$个可能输出；而对数值属性，只有两个可能输出（即$n=2$）。在两种情况下，都满足$$\sum_{j=1}^nN_{mj}=N_m$$。再设$$N_{mj}$$个实例中有$$N_{mj}^i$$个属于$$C_i$$类，则容易得到：$$\sum_{i=1}^KN_{mj}^i=N_{mj}$$，$$\sum_{j=1}^nN_{mj}^i = N_m^i$$。

于是，给定节点$m$、测试返回输出$j$，$$C_i$$类的概率估计为：

$$\widehat{p}(C_i\mid \boldsymbol{x},m,j) \equiv p_{mj}^i = \frac{N_{mj}^i}{N_{mj}}$$

则划分后的总不纯度为：

$$\mathcal{I}_m = -\sum_{j=1}^n\frac{N_{mj}}{N_m}\sum_{i=1}^Kp_{mj}^i\log p_{mj}^i$$

对于数值属性，为了得到$$p_{mj}^i$$，我们还需要知道节点$m$的$$w_{m0}$$。在$$N_m$$个数据点之间存在$$N_m-1$$个可能的$$w_{m0}$$：我们不需要考虑所有（无限个）可能的点；例如，我们只需要考虑两个样本点之间的中值就够了。还有一点要注意，最佳划分总是在属于两个不同类的两个相邻样本点之间。这样我们检查每一个，并取最高纯度作为该节点对应的纯度。对于非数值属性，不需要这种迭代。

对于所有非数值属性和数值属性，对于数值属性的所有可能划分位置，我们计算不纯度，并选取具有最小熵的划分方式。于是，对于所有不纯的分支，树学习递归地、平行地进行，直到所有分支都是纯的。


## 2.2 回归树

回归树（regression tree）可以用几乎与分类树完全相同的方法构造，唯一的不同是适合分类的不纯性度量用适合回归的不纯性度量取代。

设$b_m(\boldsymbol{x})=1$表示实例$\boldsymbol{x}$到达节点$m$，否则为0。在回归树中，划分的好坏用估计值的均方误差来度量。令$$g_m$$为节点$m$的估计值，则均方误差表示为

$$E_m = \frac{1}{N_m}\sum_t(g_m-y^{(t)})^2b_m(\boldsymbol{x}^{(t)})$$

其中$$N_m = \sum_tb_m(\boldsymbol{x}^{(t)})$$。节点的估计值我们一般使用到达该节点的所有实例的输出均值（如果噪声太大用中值）

$$g_m = \frac{\sum_tb_m(\boldsymbol{x}^{(t)})y^{(t)}}{\sum_tb_m(\boldsymbol{x}^{(t)})}$$

于是均方误差对应于节点上的方差。如果在一个节点上，误差是可以接受的，即$$E_m < \theta_r$$，则创建一个树叶节点，存放$$g_m$$值。如果误差不能接受，则对到达节点$m$的实例进一步划分，使得诸分支的误差和最小。与分类一样，在每个节点上，我们寻找最小误差的属性（以及数值属性的划分阈值），然后递归地进行上述过程。

# 3、剪枝

通常，如果到达一个节点的训练实例数小于训练集的某个百分比（如5%），则无论是否不纯或是否有错误，该节点都不进一步分裂。其基于的思想是：基于较少实例的决策树导致较大方差，从而导致较大泛化误差。在树构造之前停止构造称作树的预剪枝（prepruning）。

得到较小树的另一种做法是后剪枝（postpruning）。前面我们看到，树的学习过程中很贪心，在每一步，我们做出一个决策（即产生一个决策节点）并继续进行，绝不回溯尝试其他可能的选择。而后剪枝则可以，我们可以试图找出并剪掉不必要的子树。

在后剪枝中，我们让树完全增长直到所有树叶都是纯的并具有零训练误差，然后我们找出导致过拟合的子树并剪掉它们。最开始时，我们从被标记的数据集中保留一个剪枝集（puning set），在训练阶段不使用。对于每棵子树，我们用一个被该子树覆盖的训练实例标记的树叶节点替换它。如果该树叶在剪枝集上的性能不比该子树差，则剪掉该子树并保留树叶节点，因为子树附加的复杂性是不必要的；否则保留子树。

预剪枝与后剪枝相比，预剪枝较快，但是后剪枝通常导致更准确的树。

# 4、多变量树

在构造单变量树时，只使用一个输入维来划分。而在构造多变量树（multiariate tree）时，在每个决策节点都可以使用所有的输入维，因此更加一般。

当所有输入都是数值属性时，二元线性多变量节点定义为

$$f_m(\boldsymbol{x}): \boldsymbol{w}^T\boldsymbol{x}+w_{m0} > 0$$

还可以使用非线性多变量节点

$$f_m(\boldsymbol{x}): \boldsymbol{x}^T\boldsymbol{W}_m\boldsymbol{x}+\boldsymbol{w}^T\boldsymbol{x}+w_{m0} > 0$$

还有一种可能是使用球形节点

$$f_m(\boldsymbol{x}): \|\boldsymbol{x} - \boldsymbol{c}_m\| \leq \alpha_m$$

其中$$\boldsymbol{c}_m$$是圆心，$$\alpha_m$$是半径。



