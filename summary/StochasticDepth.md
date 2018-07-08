一、摘要、介绍与背景

1、层数非常多的网络通常具有很多问题，例如：gradient vanish、diminishing feature reuse、训练时间非常长等。

2、本文中提出一种stochastic depth的训练方法，使用了一种看似矛盾的配置：在训练时使用比较浅的网络，但是在测试时使用深层网络

3、做法是：从非常深的网络开始，在训练中，对于每一个mini-batch，随机地丢弃一些网络层，用恒等变换来代替。从而使得在训练时，网络层数的期望比较小，但是测试时，网络层数比较大。

4、梯度消逝的一些处理方法：更精细的初始化、隐层监督（辅助损失函数，参考论文Learning Longer-term Dependencies in RNNs with Auxiliary Losses，https://arxiv.org/abs/1803.00144）、以及batch normalization

5、diminishing feature reuse的一些处理方法：在网络层之间加入直接链接

6、在实验中，stochastic depth进一步减小了训练时间和测试误差。

训练时间减少主要是因为在训练时随机删除了网络层，所以训练时的网络平均深度减小，从而导致训练时间减小；

测试误差减少主要是因为，一方面在训练时随机删除网络层，所以前向和后向传播的长度也减小了，增强了梯度，使得网络能够得到更好的训练；另一方面，使用stochastic depth进行训练会产生隐式的ensemble效果，在测试时相当于将训练时每一轮不同的网络结构进行了ensemble。

7、类似于dropout，stochastic depth也可以起到正则化的作用，且在batch normalization的同时依然能起到作用。

8、dropout和stochastic depth对比

相同点：都是通过对网络中的某些部分进行丢弃，产生隐式ensemble的效果，从而达到正则化的目的

不同点：（1）dropout对某一层中的某些节点进行丢弃，改变的是网络的宽度；而stochastic depth对某些层直接丢弃，改变的是网络的深度；（2）dropout基本不改变训练时间，而stochastic depth极大的减少了训练时间；（3）dropout在使用batch normalization时基本失效，而stochastic depth在使用batch normalization时依然有效。

二、Stochastic Depth方法与分析

1、ResBlock

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/equation1.png)

2、Stochastic Depth

实现方式：对于第l个ResBlock，设b~l~为伯努利随机变量，代表该ResBlock是否被跳过，其概率参数为p~l~，则在第l个ResBlock的转换函数上乘以b~l~ 即可，该ResBlock传输函数变成

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/equation2.png)

3、survival probability

p~l~被成为survival probability，代表第l个ResBlock的转换函数被保留的概率。p~l~成为模型训练中新的超参数，有两种设置方式：

（1）所有的p~l~都设置为相同的概率值；

（2）把p~l~设置为l的平滑函数，例如线性衰减函数

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/equation4.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure2.png)

其中p~0~=1，p~L~为最后一个ResBlock被保留的概率

4、网络深度期望

由以上实现方式，训练时网络的期望深度为

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/equation_net_depth.png)

例如，当选择线性衰减函数，且p~L~=0.5时，网络深度期望为（3L-1)/4 

5、训练时间

由于在训练时，stochastic depth减少了平均网络深度，省去了许多前向-后向计算过程，从而极大的减少了训练时间。

例如，选择线性衰减函数，且p~L~=0.5时，理论上训练时间可以减少25%

6、Implicit Model Ensemble

除了加速训练之外，stochastic depth能进一步提高测试准确率。其中一个重要的原因是stochastic depth可以看作是隐式地训练了对多个ResNet进行了ensemble。

对于一个有L个ResBlock的网络来说，由于每个ResBlock都可能激活或跳过，所以一共有2^L^个可能的子网络结构，训练时，每次迭代都选择一个可能的结构进行训练，测试时，将所有子网络进行加权求和。

7、Stochastic depth during testing

由于训练时使用p~l~对每一层进行选择，所以在测试时需要进行对应的修改，测试时的每个ResBlock传输函数为

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/equation5.png)

三、实验结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/table1.png)

1、CIFAR-10 和 CIFAR-100

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure3.png)

2、SVHN

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure4.png)

3、训练时间对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/table2.png)

4、训练1202层ResNet

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure4.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure5.png)

5、ImageNet

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure6.png)

四、分析实验

1、改善反向传播中的梯度效果：缓解了梯度消逝的问题

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure7.png)

图中两次梯度下降是由于两次对学习率进行调整。同时可以看出，尤其是在调节学习率之后，stochastic depth对gradient vanish的抑制作用很明显

2、stochastic depth对超参数（概率p~L~）的鲁棒性

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/StochasticDepth/figure8.png)

具体结论参考论文