一、摘要

1、当数据中存在大量稀疏特征时，为了有效地进行学习，需要着重考虑特征之间的交互

2、FM是一种常用的对二阶特征交互进行建模的方法，但是其建模方式是线性的，不能有效地提取真实数据中的非线性和复杂的结构；DNN可以用来学习这些非线性特征，但是其深层结构会使得其难以训练

3、本文提出了一种新的NFM模型，可以将FM对二阶特征进行线性建模的能力和神经网络对高阶特征进行非线性建模的能力进行无缝的结合。

4、FM算法的问题在于只能提取二阶特征，且只能进行线性建模

5、本文中提出Bilinear Interaction Pooling的做法，一方面从神经网络的角度来理解FM算法，另一方面相比于传统的直接对特征embedding向量进行连接或平均的操作，Bi-Interaction能够提取更多的特征交互信息，从而使之后的网络层能够学习到更有意义的信息



二、对特征交互进行建模

1、基于embedding的模型大致分为两类：

（1）基于FM的线性模型

（2）基于神经网络的非线性模型

2、FM算法的优缺点

（1）优点：泛化性——可以泛化到训练集中没有出现过的特征组合；通用型（generality）——FM能够模拟许多特定的分解模型（MF, SVD++等）

（2）缺点：表示能力不足，只能提取二阶特征，且只能进行线性建模。

3、DNN算法的优缺点

（1）优点：能够隐式地提取任意阶的组合特征

（2）缺点：DNN通常以特征embedding的连接作为输入，会使得低层只包含非常少的特征交互信息；DNN的训练比较困难（vanishing/exploding gradient, overfitting, degradation, ICS）。



三、Neural Factorization Machine

1、NFM模型

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/equation2.png)

在上式中，前两项是线性部分，第三项f(x)表示特征交互，是NMF的核心部分，由一个多层前馈网络组成，如下所示

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/figure2.png)

2、Embedding Layer

对每一个特征进行embedding，然后使用特征值对embedding向量进行放缩，得到一个embedding向量集合 Vx = {x1v1, x2v2, …, xnvn}。

这里使用输入特征值对embedding向量进行了放缩，而不是简单地进行embedding table lookup， 从而考虑了实数特征的影响。

3、Bi-Interaction Layer

（1）计算方式

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/equation3.png)

对Vx中每一个向量对，先进行element-wise multiplication，得到一个k维的向量（k是embedding的维度），然后把所有向量对的处理结果加起来，得到Bi-Interaction Layer的输出。

（2）简化计算

上式可以写成以下形式

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/equation4.png)

从而把计算时间复杂度从O(kn^2^)变成O(kn)，说明Bi-Interaction Pooling没有引入额外的模型参数，且可以在线性时间内进行计算。

另外，该简化过程和原始的FM的化简过程其实是相同的，区别只是在最后一步得到k维向量后，是否求和。

4、Hidden Layer

在Bi-Interaction pooling layer之后是一组全连接隐含层，用于学习高阶特征交互

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/equation5.png)

通过使用非线性的激励函数，从而可以学习非线性特征交互；另外可以自由选在全连接层的结构，tower，constant，diamond等。

5、Prediction Layer

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/equation6.png)

本文中用于回归，可相应地改为二分类、多分类等其他形式

6、最终形式

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/equation7.png)

7、NFM Generalizes FM

FM实际上是没有hidden layer的NFM，此时NFM的输出为

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/equation8.png)

可以看出当h为全1的常数向量时，就和FM等价。

本文第一次从神经网络的角度来理解FM，从而使得可以把神经网络中相关的技术用在FM上。

8、和wide_deep以及DeepCross之间的关系

（1）、当把NFM中的Bi-Interaction层换成对embedding直接连接，再对MLP使用tower结构（residual单元）之后，就变成了wide_deep(deepcross)。

（2）对embedding直接进行连接，所得到的向量不具有任何交互特征信息，从而使得这种模型必须完全依赖之后的隐含层来提取特征交互，但是这种模型通常很难训练。
而Bi-interaction层包含了二阶特征交互，从而使得之后的隐含层更容易提取高阶特征。

9、Learning

（1）本文使用mini-batch SGD进行训练

（2）对Bi-Interaction Layer的输出和隐含层的输出使用dropout

（3）对Bi-Interaction Layer的输出和隐含层的输出使用batch normalization



四、实验

1、Study of Bi-Interaction Pooling

(1) dropout用于提升泛化性

dropout和l2 norm的对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/figure3.png)

可以看出，dropout明显比l2 norm的效果好，原因可能是l2只是在每次更新时对参数的数值进行抑制，而dropout可以看作是很多子模型的集成。

是否进行dropout的对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/figure4.png)

可以看出，增加了dropout之后，训练集的误差上升了，但是验证集的误差下降了，说明dropout可以有效抑制过拟合。

（2）batch normalization加速训练

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/figure5.png)

加入batch normalization之后，模型收敛速度极快



2、Impact of Hidden Layer

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/figure6.png)

由图中可以看出，增加了隐含层并使用非线性激励函数之后，模型性能大幅度提升，从而说明了以非线性方式提取高阶特征交互的重要性。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/table2.png)

对NFM和普通的连接网络进行扩展隐含层后，得到结论：

（1）在Bi-interaction之后使用多个隐含层，并没有使得性能进一步提升。主要是因为Bi-interaction layer已经提取了二阶特征交互信息，所以之后使用简单的非线性函数就可以获取高阶特征交互；

（2）当把Bi-interaction替换成简单的连接之后，再增加隐含层会使得性能逐步提升，且其最优性能仍然没有超过NFM-1；

（3）由此证明了在低层使用包含更多信息的操作的必要性

预训练：使用FM预训练得到的embedding对网络进行初始化

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/figure7.png)

（1）、预训练使得NFM收敛的速度极快
（2）、最终的性能并没有提升，证明了NMF的鲁棒性，对参数的初始化不敏感



3、性能对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/table3.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/NFM/figure8.png)

（1）NFM以最少的模型参数达到了最好的效果

（2）FM和HOFM之间的差距说明了提取高阶特征交互的重要性；HOFM和NFM之间的差距说明了非线性的重要性

（3）DeepCross的性能并不突出，说明了更深层的学习并非是必须的。





























































