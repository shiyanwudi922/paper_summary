一、摘要

1、FM的问题：对所有的特征交互使用相同的权重进行建模，但是并非所有的特征交互对任务的作用效果都相同。

2、本文提出的方法：对不同特征交互的重要性使用attention neural network进行区分。



二、介绍

1、在监督学习中，当特征包含大规模离散特征时，需要着重考虑特征之间的交互

2、常见的方法是直接使用特征之间的乘积作为交互特征，然后对每一个特征都学习一个权重，但是存在两个问题，一是需要大量的人工特征工程，二是泛化能力比较弱。比如，如果某个交叉特征在训练集中没有出现过，则该交叉特征对应的权重无法进行学习，在预测时，如果预测样本中出现了该交叉特征，那么该交叉特征无法起到作用

3、FM用于解决泛化性问题，通过对每一个特征学习一个embedding向量，以向量的内积作为交叉特征的权重，FM可以对任何交叉特征的权重进行学习。当测试样本中出现训练集中没有的交叉特征时，其子特征对应的embedding向量可以通过和训练集中其他特征的交互进行学习，从而对该交叉特征的权重进行建模。

4、但是FM的问题在于，对所有的交叉特征使用相同的权重进行建模，不能区分不同交互特征的重要性。

5、本文中提出了AFM，通过attention neural network对交叉特征的重要性进行建模，不仅取得了更好的性能，同时增加了模型的可解释性和透明度。

6、另外，FM用相同的方式对所有的特征交互进行建模：首先，对于某一个特征，其涉及到的所有的交互特征都会共享其特征向量；其次，所有交互特征的权重都是1（该权重不是xixj），但是实际上并非所有的特征对预测都是有用的，有一些可能是噪声。



三、Attetional Factorization Machines

1、model

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/figure1.png)

其中，input layer、embedding layer和FM中相同，对非零特征进行embedding，然后和对应的特征值相乘。

2、pair-wise interaction layer

设输入特征向量为x，其中非零特征集合为X，embedding layer的输出为一个embedding向量集合E={vixi}~i∈X~，pair-wise interaction layer的输入为E，对E中的向量进行两两按位相乘，输出所有的向量对两两按位相乘后的结果向量集合

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/equation2.png)

f~PI~(E)是一个向量集合，其中每一个向量是两个不同特征的embedding按位相乘的结果。

3、attention-based pooling layer

Attention-based pooling layer以f~PI~(E)为输入，对于其中每一个向量，先用一个attention network计算其相应的attention score，并进行softmax得到attention weight，该attention network是一个两层神经网络

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/equation5.png)

即使用additive attention。

之后，使用attention weight对f~PI~(E)中的向量进行加权求和

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/equation4.png)

最终，f~Att~(f~PI~(E))是一个向量

4、总体形式

对f~Att~(f~PI~(E))再进行一次单层传输，从而得到模型的输出，模型的总体计算形式为

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/equation6.png)

5、训练

（1）目标函数：本文处理的是回归问题，所以使用平方损失作为目标函数；可针对具体问题修改为合适的目标函数

（2）过拟合：由于之前说明过并非所有的特征交互都是有用的，所以在pair-wise interaction layer使用dropout；另外，对attention network中的权重矩阵w使用l2 norm。



四、实验

1、超参数的影响：主要是dropout和l2 norm

（1）dropout：先把l2 norm的系数设置为0，从而只有dropout对模型产生影响。另外把AFM中的attention部分删除，就变成了FM，从而进行实验。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/figure2.png)

通过设置合适的dropout比率，AFM和FM的性能都有很大提升。

本文中实现的FM比LibFM的性能要好，一方面由于LibFM使用基本的SGD，而本文中使用adagrad，可以根据参数的频率自适应地调整学习率；另一方面LibFM使用的是l2 norm，没有dropout有效。

相比FM和LibFM，AFM的性能有大幅度的提升，即使不使用dropout，AFM在出现overfitting的基础上依然比FM和LibFM的最优性能要好，从而证明了attention network学习特征交互权重的效果。

（2）l2 norm

在dropout设置为最优值的基础上（下图中λ = 0的情况），使用l2 norm可以进一步提升性能

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/figure3.png)



2、Attention Network的影响

（1）attention factor（attention network中隐含层节点数）

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/figure4.png)

随着attention factor的变化，网络的性能相对比较稳定。

当attention factor为1时，attention network就降级成了一个线性回归模型，但是在这种模型容量极大受限的条件下，AFM的性能仍然远超过FM，从而进一步证明了AFM基于交叉向量计算其重要性分数的合理性。

（2）收敛速度和拟合、泛化能力

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/figure5.png)

从图中可以看出，AFM的收敛速度更快，同时其拟合能力和泛化能力都更强。

（3）可解释性

首先不使用attention network，固定交叉特征的权重，训练网络；之后固定feature embedding，训练attention network之后，性能提升了3%，进一步证明了attention network的有效性。

之后，随机选择三个样本，提取其交叉特征的attention score和interaction score

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/table1.png)

其中，表中每一个元素包含两个数字，第一个数字表示attention score（attention weight），第二个数字表示interaction score（xixj）。

可以看出，FM对所有的交叉特征赋予相同的权重，AFM对不同的交叉特征赋予不同的权重。



3、不同模型间的对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AFM/table2.png)

（1）AFM以最少的参数取得了最好的性能

（2）HOFM提取更高阶的特征，使用了翻倍的参数量，但是只取得了轻微的效果提升，从而引出了一个可能的研究方向——设计更有效的方法来提取高阶特征

（3）DeepCross的性能最差，说明了更深的模型不一定都是有用的，通常会受到过拟合的影响，且更难以优化