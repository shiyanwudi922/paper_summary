一、摘要

1、深度学习包含两个阶段：模型初始化，模型微调（训练）

2、本文的主要亮点在于对网络第一层的初始化，提出了两个模型FNN(Factorization machine supported neural network)和SNN(sampling-based neural network)



二、模型结构

1、FNN

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FNN/figure1.png)

在FNN中，主要的思路是先训练一个FM模型，从而每一个特征都有一个系数以及一个用于交叉的向量表示，把该系数和向量进行连接，作为该特征的embedding。

FNN的第一层就是所有特征的embedding的连接，再连接上FM模型中的偏倚参数。

之后使用多层全连接进行传输

2、SNN

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FNN/figure2.png)

SNN和FNN之间的区别主要是第一层的结构和初始化方法不同。

SNN中使用两种方法对第一层进行初始化：sampling-based Boltzmann machine和sampling-based denoting auto-encoder。

3、正则化

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FNN/equation11.png)

这里需要注意一点，W0是embedding矩阵，直接使用equation11作为正则化时，会使得对于每一个batch，都对所有的embedding进行正则化，但是其中有很多embedding对当前batch的损失函数并没有贡献，另外，也会降低训练速度。可调整为只对对当前损失函数有贡献的embedding进行正则化



三、实验

1、性能对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FNN/table1.png)

2、超参数选择：本文对相关的超参数进行选择，包括

number of training epochs ==> early stopping

learning rate：1，0.1，0.01，0.001，0.0001

negtive unit sampling of SNN-RBM and SNN-DAE：1，2，4

activation function：linear，sigmoid，tanh

3、模型结构选择

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FNN/figure3.png)

实验结果表明，diamond（钻石）结构的效果最好

4、正则化对比：本文对L2-norm和dropout进行了对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FNN/figure4.png)

实验结果表明，dropout的效果要优于L2-norm。

另外，FNN和SNN对于dropout的敏感度是不同的，从而导致他们最优的dropout rate不相同



四、现在来看FNN显得比较简单，但是该方法奠定了很多之后的DNN相关方法的基础。