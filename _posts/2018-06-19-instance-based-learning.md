---
layout: post
title: "基于实例的学习" 
---

# 1、基于实例的学习 vs 参数学习

在参数学习中，无论是密度估计、分类还是回归，我们都假设了一个在输入空间上有效的模型。于是，我们可以把估计概率密度函数、判别式函数或回归函数问题都归结为估计少量参数值。它的缺点是，假设并非总是成立的，并且不成立时可能会导致很大的误差。

在基于实例的学习（instance-based learning）中，我们只假定相似的输入具有相似的输出。这是一种合理的假设：无论是概率密度函数、判别式函数还是回归函数都是平滑、缓慢地变化的，相似的实例意味着相似的事物。

这样，我们的算法需要使用合适的距离度量，从训练集中找出相近的实例，并且基于它们来进行推断和计算，也即在基于实例的学习中，我们假定所有实例都可以表示为欧式空间中的点。

在参数学习方法中，所有训练实例都影响最终的全局估计；而在基于实例的学习方法中，一般情况下，每次估计时采用局部模型，即只受邻近实例的影响。

基于实例的学习所做的一般是先把训练实例存放在一个查找表中备用。这意味着所有训练实例都要存放，而存放所有训练实例需要$O(N)$存储量。此外，给定一个输入，应当找出相似的训练实例，而找出它们需要$O(N)$计算量。这种方法也称惰性（lazy）学习算法，因为不像急切（eager）的参数学习，给定训练集时，它们并不计算模型，而是将计算推迟到给定一个检验实例时才进行。对于参数学习方法，模型都比较简单，一般具有$O(d)$或$O(d^2)$量级个参数，并且一旦从训练集中计算出这些参数，我们保存模型并且计算输出时也不再需要训练集，计算量也为$O(d)$或$O(d^2)$。通常，$N$比$d$（或$d^2$）大得多，这种存储量和计算量的增加是基于实例的学习方法的缺点。


# 2、基于实例的密度估计

与通常的密度估计一样，我们假设样本$\mathcal{X} = \\{x^{(i)}\\}_{i=1}^N$独立地从一个未知的概率密度$p(\cdot)$中抽取的，$\widehat{p}(\cdot)$是$p(\cdot)$的估计。我们从单变量情况开始，即$x^{(i)}$是标量，稍后我们将推广到多维的情况。

基于实例的累积分布函数$F(x)$的估计是小于或等于$x$的样本所占的比例：

$$\widehat{F}(x) = \frac{\#\{x^{(i)} \leq x\}}{N}$$

其中$$\#\{x^{(i)} \leq x\}$$表示$x^{(i)}$小于或等于$x$的训练样本数。

类似地，基于实例的密度函数估计可以表示为：

$$\widehat{p}(x) = \frac{1}{h}\left(\frac{\#\{x^{(i)} \leq x+h\} - \#\{x^{(i)} \leq x\}}{N}\right)$$

其中$h$是区间长度，并且假定落入该区间的实例$x^{(i)}$是“足够接近”的。

## 2.1 直方图估计

最古老、最流行的方法是直方图估计（hisogram estimator）。在直方图中，输入空间被划分为称作“箱”（bin）的相等区间。给定原点$x_0$和箱宽$h$，箱区间为$$[x_0+mh, x_0+(m+1)h]$$（$m$是整数），密度估计由下式给出：

$$\widehat{p}(x) = \frac{\#\{x^{(i)}\text{ in the same bin as }x\}}{Nh}$$

直方图估计的优点是一旦计算和存放了箱估计，我们就不再需要保留训练集了。但如果没有实例落入箱中，则估计为0，且在箱边界处不连续。

## 2.2 质朴估计

质朴估计法（naive estimator）使得我们不必设置原点（但此时我们需要保留整个训练集）。它定义为：

$$\widehat{p}(x) = \frac{\#\{x-h/2 < x^{(i)} \leq x+h/2\}}{Nh}$$

容易看出，它等于$x$落在宽度为$h$的箱中心的直方图估计。

该估计还可以表示为：

$$\widehat{p}(x) = \frac{1}{Nh}\sum_{i=1}^N w(\frac{x-x^{(i)}}{h})$$

其中权重函数定义为：

$$w(u) = \begin{cases} 1 & -1/2<u\leq 1/2 \\
			0 & otherwise\end{cases}$$

这就好像对每个$x^{(i)}$都有一个围绕它的大小为$h$的影响区域，并且对落入该区域的$x$产生影响，而最终估计恰为影响区域包含$x$的$x^{(i)}$的影响之和。该估计不是连续函数，并在$x^{(i)}\pm h/2$处有跳跃。

## 2.3 核估计

为了得到光滑的估计，我们使用一个光滑的权重函数，称作核函数（kernel function）。最流行的是高斯核：

$$K(u) = \frac{1}{\sqrt{2\pi}}\exp(-\frac{u^2}{2})$$

核估计（kernel estimator）定义为：

$$\widehat{p}(x) = \frac{1}{Nh}\sum_{i=1}^N K(\frac{x-x^{(i)}}{h})$$

此时所有的$x^{(i)}$都对$x$上的估计有影响，并且其影响随着$\mid x^{(i)}-x\mid$的增加而平滑地减小。

当$h$很小时，每个训练实例只在其附近小区域内产生较大影响，而在较远的点上影响非常小。当$h$较大时，有更多的核重叠，我们得到较光滑的估计。如果$K(\cdot)$处处非负并且积分为1，即如果它是合法的密度函数，则$\widehat{p}(\cdot)$也是。

核估计的一个问题是窗口宽度在整个输入空间上是固定的。不过已经有些文献提出各种自适应方法将$h$看作$x$附近密度的函数。

## 2.4 $k$-最近邻估计

对于每个$x$，我们定义

$$d_1(x) \leq d_2(x) \leq \cdots \leq d_N(x)$$

为从$x$到样本中的点按递增序排列的距离：$$d_1(x)$$是最近样本的距离，$$d_2(x)$$是次近样本的距离，如此下去。

$k$-最近邻（$k$-nearest neighbor, k-nn）密度估计为：

$$\widehat{p}(x) = \frac{1}{2Nd_k(x)}$$

这就像$$h=2d_k(x)$$的质朴估计，不同之处是我们不是固定$h$并检查多少样本落入箱中，而是固定落入箱中的观测数$k$，并计算箱的大小。密度高的地方箱较小，而密度低的地方箱较大。

k-nn密度估计不是一阶连续的，它的导数在所有的$$\frac{1}{2}(\widetilde{x}^{(i)} + \widetilde{x}^{(i+k)})$$上不具有连续性，其中$$\widetilde{x}^{(i)}$$是样本的升序统计量。k-nn密度估计得到的不是严格的概率密度函数，因为它的积分是$\infty$，而不是1。

为了得到更光滑的估计和严格的概率密度函数，我们可以使用其影响随距离增加而减小的核函数，并采用自适应光滑参数$$h=d_k(x)$$：

$$\widehat{p}(x) = \frac{1}{Nd_k(x)}\sum_{i=1}^N K(\frac{x-x^{(i)}}{d_k(x)})$$

通常，$K(\cdot)$取高斯核。

## 2.5 从标量到多维数据的推广

给定$d$维观测样本$$D = \{\boldsymbol{x}^{(i)}\}_{i=1}^N$$，多元核密度估计为：

$$\widehat{p}(\boldsymbol{x}) = \frac{1}{Nh^d}\sum_{i=1}^N K(\frac{\boldsymbol{x}-\boldsymbol{x}^{(i)}}{h})$$

其中核函数应满足必要条件

$$\int_{\mathbb{R}^d}K(\boldsymbol{x})d\boldsymbol{x} = 1$$

一个最常用的候选是多元高斯核

$$K(\boldsymbol{u}) = (\frac{1}{\sqrt{2\pi}})^d \exp(-\frac{\|\boldsymbol{u}\|^2}{2})$$

然而，由于维灾难（curse of dimensionality），在高维空间使用基于实例的学习方法需要非常小心：假设$\boldsymbol{x}$是8维的，每个维使用10个箱的直方图，则也有$10^8$个箱。除非我们有大量数据，否则大部分箱为空，并且那里的估计为0。在高维空间，“近邻”概念的使用也要非常小心，比如使用欧氏距离意味着所有维上都具有相等的尺度，如果输入具有不同的尺度，则应当将它们规范化，使其具有相同的方差。

此外，我们还要考虑不同维之间的相关性，于是可以使用以下核函数：

$$K(\boldsymbol{u}) = \frac{1}{2\pi^{d/2}|\boldsymbol{S}|^{1/2}} \exp(-\frac{1}{2}\boldsymbol{u}^T\boldsymbol{S}^{-1}\boldsymbol{u})$$

相当于使用马氏距离而非欧氏距离。其中$\boldsymbol{S}$是样本的协方差矩阵估计，可以由$\boldsymbol{x}$附近的实例来进行计算，比如由最近的$k$个实例来计算。计算出来的$\boldsymbol{S}$可能是奇异的，可以通过PCA来消除奇异。

如果输入是离散的，我们可以使用汉明距离（Hamming distance），它对不匹配的属性计数：

$$HD(\boldsymbol{x},\boldsymbol{x}^{(i)}) = \sum_{j=1}^d 1(x_j\neq x_j^{(i)})$$


# 3、基于实例的分类

当用于分类时，我们使用基于实例的方法来估计类条件密度$$p(\boldsymbol{x}\mid C_j)$$。

类条件密度的核估计由下式给出：

$$\widehat{p}(\boldsymbol{x}\mid C_j) = \frac{1}{N_jh^d}\sum_{i=1}^N K(\frac{\boldsymbol{x}-\boldsymbol{x}^{(i)}}{h})r_j^{(i)}$$

其中如果$$y^{(i)}=C_j$$，则$$r_j^{(i)}$$为1，否则为0。$$N_j = \sum_i r_j^{(i)}$$表示属于第$j$个类的实例数。

先验密度的最大似然估计是$$\widehat{p}(C_j) = N_j/N$$，于是判别式函数可以表示为：

$$\begin{align}g_j(\boldsymbol{x}) & = \widehat{p}(\boldsymbol{x}\mid C_j)\widehat{p}(C_j) \\
					& = \frac{1}{Nh^d}\sum_{i=1}^N K(\frac{\boldsymbol{x}-\boldsymbol{x}^{(i)}}{h})r_j^{(i)}
\end{align}$$

公共因子$\frac{1}{Nh^d}$可以忽略。可以看出，每个实例都只为它的类投票，而对其他类没有影响；投票的权重由核函数$K(\cdot)$给定，通常赋予更近的实例更高的权重。

对于k-nn分类，我们有

$$\widehat{p}(\boldsymbol{x}\mid C_j) = \frac{k_j}{N_jV_k(\boldsymbol{x})}$$

其中$k_j$为k个最近邻中属于第$j$个类的数量，而$$V_k(\boldsymbol{x})$$是中心在$\boldsymbol{x}$，半径为$$d_k(\boldsymbol{x})$$的$d$维超球体积。

于是易得

$$\widehat{p}(C_j \mid \boldsymbol{x}) = \frac{\widehat{p}(\boldsymbol{x}\mid C_j)\widehat{p}(C_j)}{\sum_{\widetilde{j}} \widehat{p}(\boldsymbol{x}\mid C_{\widetilde{j}})\widehat{p}(C_{\widetilde{j}})} = \frac{k_j}{k}$$

k-nn分类器（k-nn classifier）将输入指派到输入的k个最近邻中具有最多实例的类。所有的近邻都有相同的投票权。如果出现平均，可以通过随机机制或者加权投票机制来打破。

k-nn分类器的一种特殊情况是最近邻分类（nearest neighbor classifier），其中$k=1$，输入被指派到最近的实例所在的类。



# 4、基于实例的回归

在回归中，给定训练集$$D = \{(\boldsymbol{x}^{(i)},y^{(i)})\}_{i=1}^N$$，其中$y^{(i)}\in \mathbb{R}$，我们假定

$$y^{(i)} = g(\boldsymbol{x}^{(i)}) + \varepsilon$$

在基于实例的方法中，我们只假定相近的$\boldsymbol{x}$具有相近的$g(\boldsymbol{x})$值。给定$\boldsymbol{x}$，我们的方法是找到$\boldsymbol{x}$的邻域，并求邻域中$y$的平均值，得到$\widehat{g}(\boldsymbol{x})$ （在有噪声的应用中，我们可以使用中位数而非均值）。基于实例的回归估计器又称平滑器（smoother），而该估计称作平滑（smoothing）。

与核估计一样，我们可以使用赋予较远的点较小权重的核函数，从而得到核平滑器（kernel smoother）：

$$\widehat{g}(\boldsymbol{x}) = \frac{\sum_iK(\frac{\boldsymbol{x}-\boldsymbol{x}^{(i)}}{h})y^{(i)}}{\sum_i K(\frac{\boldsymbol{x}-\boldsymbol{x}^{(i)}}{h})}$$

通常使用高斯核$K(\cdot)$。替换固定$h$，我们可以固定近邻数$k$，使得估计自适应$\boldsymbol{x}$周围的密度，从而得到k-nn平滑器（k-nn smoother）。







