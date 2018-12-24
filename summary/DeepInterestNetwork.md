一、摘要

1、常见模型的做法： Embedding&MLP，先把大规模稀疏特征映射成低维embedding向量，然后转换成<font color=red size=5>固定长度</font>的用户兴趣表示向量，最后通过连接在输入到MLP中去学习特征之间的非线性关系。这种方法的瓶颈在于使用固定长度的向量来表示用户的兴趣，把用户多种类型的兴趣都压缩在一个向量中，从而对于不同的备选广告，用户兴趣的表示向量不变，使得模型难以从用户丰富的历史行为中有效提取用户多种多样的兴趣表示。

2、本文中提出的DIN模型中，使用了local activation unit，对于一个特定的备选广告，能够自适应地从用户历史行为中学习对应的兴趣表示。不同的广告对应着用户不同类型的兴趣，所以其对应的用户兴趣表示向量也不同，从而可以有效地利用用户兴趣的多样性，极大的提高了模型的表示能力。

3、本文中还提出了mini-batch aware regularizaiton和data adaptive activation function来进行辅助训练



二、介绍

1、用户兴趣是多种多样的，通常可以从用户的历史行为数据中获取。

2、在Embedding&MLP方法中，通过把用户行为的embedding向量转换成一个固定长度的向量，从而对用户所有的兴趣进行表示，即用户多种多样的兴趣被压缩到一个固定长度的向量中。

3、为了使该固定长度的向量有足够的表示能力，其维度需要被扩展的非常高。但是这会导致参数量增多，有过拟合的风险。另外，也会使得计算和存储的代价比较高。

4、另外，在对一个备选广告进行预测时，也不需要把用户的所有的兴趣都压缩在一个向量中，因为对于该广告，只有用户的部分兴趣会影响其行为（是否点击）

5、DIN通过引入local activation unit，对于某个备选广告，考虑了用户历史行为和该广告之间的相关性，通过soft-searching来计算并提取历史行为中和备选广告相关的部分，通过weighted sum pooling来获取用户的兴趣表示，该表示和备选广告相关，不同的广告对应不同的表示向量。在用户行为中，相关性高的行为具有更高的权重，从而支配着整个表示向量。

6、提出了mini-batch aware regularization：当目标函数中不含有正则项时，对于每一个mini-batch,优化算法只需要更新其中出现过的稀疏特征对应的参数；但是当目标中含有l2-regularization时，在每一个mini-batch中，需要对所有的 参数计算l2-norm，这个计算量过大，实际中很难实现。本文中提出的mini-batch aware regularization，在每一个mini-batch中，也只对出现过的特征对应的参数计算l2-norm，从而有效的减小了计算量

7、另外，设计了data adaptive activation function，对PReLU进行扩展，通过输入数据的分布自适应地调整整流点。和batch normalization的思路类似，都是使数据的分布和激励函数的分布尽量匹配，不同的是BN改变的是输入数据的分布，而本文中是对激励函数进行修改。



三、Deep Interest Network

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/figure2.png)

1、Feature Repersentation

特征类型

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/table1.png)

Multi-group categorical form: one-hot, multi-hot



2、Base Model(Embedding&MLP)

Embedding layer: one-hot — single embedding vector; multi-hot — list of embedding vectors

Pooling layer and concatenation layer: e~i~ =pooling(e~i1~,e~i2~,…e~ik~)

MLP: multi-layer perceptron

Loss: negative log-likelihood

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation2.png)



3、DIN结构

（1）在base model中，使用有限长度的用户向量表示难以表达出用户多种多样的兴趣；而增加向量维度会导致参数量大幅度增加，引起过拟合，同时增加了存储和计算开销。

（2）在DIN中，使用了local activation unit，不再把用户兴趣表示为一个固定不变的向量，而是根据不同的备选广告，考虑其与用户历史行为的相关性，自适应地计算用户兴趣表示

（3）local activation unit结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation3.png)

类似于使用additive的attention，但是有两点不同：

a、在计算attention score时，使用的也是两层神经网络，但是输入除了两个向量之外，还有这两个向量的外积向量，外积对于相关性建模提供了显式的知识

b、计算出score之后，不再使用softmax进行归一化，保留了用户兴趣的强度，相当于不同的广告能够激起用户兴趣的强度也不同



四、Training Techniques

1、Mini-batch Aware Regularization

当不使用正则化时，损失函数为loss = crossEnt，此时反向传播只会对样本中非零特征对应的embedding参数进行更新；但是没有正则化会导致过拟合严重。

当使用传统的正则化时，损失函数为loss = crossEnt + l2，此时不管样本中是否出现某个特征，反向传播都会对所有的参数进行更新，但是这样会导致计算代价过大，且没有出现的特征，其对应的参数事实上不用更新。

所以，本文中提出mini-batch aware regularization，在每一个batch中，只对出现过的特征对应的参数进行正则化，从而做到同时防止过拟合和减少计算量。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation4.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation5.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation6.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation7.png)



2、Data Adaptive Activation Function

PReLU函数：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation8.png)

PReLU函数的固定整流点为0，但是当每一层的数据分布不同时，该函数不再合适。

本文中提出了data adaptive activation function

Dice 函数：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation9.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/figure3.png)

Dice有两种理解方式：
（1）、可以看作是PReLu的一种扩展，其核心思想是根据输入分布来自适应地调整整流点
（2）、可以看作是batch normalization的一个变种：在BatchNorm中，是根据数据的分布，对数据进行平移和放缩，改变的是数据；在Dice中，根据数据的分布，对激励函数进行平移和放缩，最终可以起到类似的效果。两种方式的思路都是使得数据的分布和激励函数的分布尽量匹配。

另外，Dice其实是平移放缩之后的sigmoid函数，在切换控制时更平滑



五、实验

1、实验数据
本文使用amazon dataset、movielens dataset、alibaba dataset进行实验

2、指标：本文使用两种指标

（1）用户加权auc：传统的AUC衡量了排序的质量， user-weighted AUC衡量了intra-user的排序质量，在展示广告系统中，user-weighted AUC和在线性能的相关性更高

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation10.png)

（2）RelaImpr: relative improvement

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/equation11.png)

3、amazon dataset和movielens dataset的模型对比结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/table3.png)

4、不同的正则化技术对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/table4.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/figure4.png)

（1）当使用细粒度特征（goods_id）时，模型比较容易出现过拟合，在不同的正则化技术中，本文中提出的mini-batch aware regularization对应的 auc最高

（2）另外，当使用细粒度特征后，模型的AUC比不用该特征高得多，说明了细粒度特征包含更丰富的信息

5、alibba dataset的模型对比结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/table5.png)

（1）当只使用DIN，不用mini-batch aware regularization和data adaptive activation function时，模型依旧有很大提升

（2）加入mini-batch aware regularization和data adaptive activation function后，进一步带来了大幅度提升

6、在线A/B实验

A/B实验中，DIN提升了10.0% CTR 和 3.8% RPM

7、可视化

（1）对于某个备选广告，在用户行为列表中，相关性高的行为其权重高，相关性低的行为权重低

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/figure5.png)

（2）商品embedding向量聚类结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepInterestNetwork/figure6.png)


​			
​		
​	



