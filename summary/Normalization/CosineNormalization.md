一、简介

1、传统的多层神经网络使用点积的结果作为激励函数的输入 ==> 点积的结果是无界的 ==> 对应神经元的方差比较大 ==> 对输入分布的变化敏感 ==> 泛化能力差；使ICS(internal covariate shift)的情况更加恶化，降低训练速度

2、本文工作的简要概括

（1）CN比BN、WN、LN有更低的测试误差

（2）centered cosine normalizaiton(pearson correlation coefficient)可以进一步减小测试误差

（3）CN比其他的normalization的稳定性更好

（4）CN具有和其他normalization相似的加速训练能力

二、Cosine Normalization

1、Cosine Similarity

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Normalization/CosineNormalization/equation4.png)

Pearson Correlation Coefficient

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Normalization/CosineNormalization/equation13.png)

使用Cosine Similarity或者Pearson Correlation Coefficient直接代替神经网络中的点积计算，可以减小神经元的方差。

2、如果使用ReLU作为激活函数，那么normalization的结果不需要进行再放缩和再平移，从而减少了需要学习的参数或者需要预设的超参数；但是如果使用其他的激活函数，例如sigmoid、tanh、softmax，那么normalization的结果需要进行再放缩和再平移，从而充分利用激活函数的非线性性质。

3、在全联接网络中，隐层所有的单元对应的输入向量都相同（前一层的输出向量），从而它们对应的输入norm也都相同；在卷积网络中，输入向量来源于不同的receptive field，所以会有不同的输入norm。

4、为了避免分母为0的情况，在权值向量和输入向量中加入了偏移，即多加了一个非零的维度：w = [w1, w2...wi]，x = [x1, x2...xi] 变成 w = [w0, w1, w2...wi]，x = [x0, x1, x2...xi]

5、cosine normalization不依赖于batch或者mini-batch中样本的统计量，所以模型可以使用batch gradient decent或者SGD进行训练；另外，cosine normalization在训练和推断的前向传播过程中的计算方式是相同的。

三、讨论

1、和weight normalization的对比

（1）CN在WN的基础上，通过进一步除以输入的幅度，把激励函数的输入限制在一个更小的范围内，从而减小了神经元的输入方差

（2）CN使得模型对于不同的输入幅度具有更好的鲁棒性，即当输入的幅度发生变化时，网络输出的概率分布不变。例如：对于WN，当输入为x时，假设输出层softmax的输入为[1,2]，则输出概率分布为[0.2689,0.7311]，当输入为10x时，输出层softmax的输入为[10,20]，则输出概率分布变为[0,1]，即WN对于输入的幅度不具有鲁棒性；而对于CN，当输入的幅度变化时，输出的概率分布是不变的。

（3）在反向传播时，对于WN，输入幅度的变化会引起参数梯度的变化，而CN通过对输入进行归一化避免了这种影响。

2、和layer normalization的对比：使用皮尔逊相关系数

（1）LN只对输入进行了限制，而CN对输入和权重都进行了限制，所以CN对于输入和权重的平移和放缩都具有鲁棒性

（2）LN在激励函数之前、点积之后计算均值和标准差；CN在点积之前计算均值和标准

（3）在CNN中，LN在整个网络层的范围上计算均值和标准差，而CN在receptive field上计算：当计算均值和标准差的范围上的所有的神经元都有相似的贡献或者功能时，normalization的作用会比较明显。所以当LN应用于CNN时，该条件是不满足的，但是CN满足该条件。

四、实验结果

1、MNIST

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Normalization/CosineNormalization/figure2&table2.png)

2、20NEWA GROUP

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Normalization/CosineNormalization/figure3&table3.png)

3、CIFAR-10

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Normalization/CosineNormalization/figure4&table4.png)

4、CIFAR-100

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Normalization/CosineNormalization/figure5&table5.png)

5、SVHN

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Normalization/CosineNormalization/figure6&table6.png)