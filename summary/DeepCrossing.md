一、摘要

为了解决手动提取组合特征代价比较高的问题，本文提出了Deep Crossing模型，用于自动进行特征组合。模型的输入是单独特征的集合，可以是稀疏或者密集的，交叉特征由网络进行隐式提取。网络由embedding layer，stacking layer，a cascade of Residual Units组成。

二、模型结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossing/figure1.png)

1、embedding layer

本文中的embedding和常见的不太一样，主要在于把embedding看作对one-hot向量进行全连接传输，全连接的权重就是对应的embedding向量，本文中在全连接的基础上增加了偏倚向量和relu激励函数

2、stacking layer

stacking layer实际上就是把embedding进行连接

3、Residual Layer

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossing/figure2.png)

在全连接时使用了residual connection进行。

4、scoring layer

scoring layer就是最终的输出层，从而可以计算损失，本文使用的是logloss

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossing/equation1.png)

三、实验

1、performance on a pair of text inputs

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossing/table3.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossing/table4.png)

2、Beyond Text Input

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossing/figure5.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossing/figure6.png)

3、Comparison with production models

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossing/table5.png)

