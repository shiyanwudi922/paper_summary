![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-FCN/figure1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-FCN/figure2.png)

一、摘要

1、使用了全卷积网络，避免了成千上万次的单个的region在网络中传输，节省了时间，并且使得几乎所有的计算都可以在整个图像上进行共享；

2、提出了位置敏感的分数图（position-sensitive score map）和位置敏感pooling层（position-sensitive pooling lay），来解决在图像分类中要求有位移不变性，而在实体检测中要求位移可变性的问题；

二、简介

1、流行的用于实体检测的深度网络通常由RoI pooling层分成两个子网

（1）共享的全卷积网络，和具体的RoI无关

（2）RoI相关的子网，不进行计算共享，每一个RoI进行单独传输

2、（1）目前在图像分类中效果最好的网络都是全卷积网络（GoogleNet、ResNet），但是如果直接把这些网络用于实体检测，会导致所有的网络层都是共享的卷积层，而RoI相关的子网部分没有隐含层，从而效果会很差，和其在分类中表现出来的性能不符

（2）为了解决这个问题，ResNet中有意的在卷积层之间插入了RoI pooling层，形成了更深的RoI相关的子网络，提高了准确率，但是降低了速度（因为每个RoI都要单独传输）

3、（1）在ResNet中有意的插入RoI pooling层，是由于在分类网络中需要提高网络的位移不变性，而在实体检测网络中，需要提高位移可变性。

（2）更深的卷积层对位移更不敏感，从而具有更强的位移不变性，而插入RoI层后，使得之后的网络对单个的RoI进行操作，是位移可变的，从而打破了原来的位移不变性，但是牺牲了训练和测试的效率

（3）本文中提出的R-FCN有两个主要的优点：

a、使用全卷积网络，一方面减少单个region在网络中的传输，另一方面使得在整个图像上计算的特征能在所有region上进行共享（计算共享）

b、使用position-sensitive score map 和 position-sensitive RoI pooling相结合，使得模型具有位移可变性

三、方法

1、简述

（1）使用两阶段检测：region proposal（RPN） + region classification and regression

（2）position-sensitive score map：最后一个卷积层会输出k²(C+1)个feature map；同时每一个region proposal会切分成k×k个网格，每一个网格代表了RoI中的一个位置（上左、上中等）；每一个网格会对应C+1个feature map，对应C个类别和1个背景

（3）position sensitive RoI pooling：把每一个RoI分成k×k个网格，每一个网格对应C+1个score map，则该网格会在其对应的score map上的对应区域做pooling（average、max等等）

（4）pooling之后，每个RoI会得到k²组分数，其中每一组包含对C+1个类别的打分，将这k²组分数进行组合（取平均），得到最终对于该RoI属于每个类别的打分，进行softmax之后，可用于训练时生成交叉熵损失或者测试时进行推断。

（5）bounding box回归：使用对RoI分类完全相同的方法进行回归，最后一个卷积层输出4k²个score map，其余的方法和分类完全相同。（这里为了简单，使用了类别未知的回归，所以最后一个卷积层输出4k²个score map，如果使用特定类别的回归，则需要输出4k²C个score map）

2、训练

（1）由RPN产生proposal，由R-FCN进行分类和回归

（2）采用faster R-CNN的方法在RPN和R-FCN之间进行交替训练

（3）训练R-FCN时使用RPN产生的proposal，根据IoU>=0.5进行打标

（4）训练R-FCN时使用OHEM (online hard example mining)，即：假设RPN产生N个proposal，先计算每一个proposal的损失（分类+回归），在根据损失进行排序，选择损失最大的B个proposal进行反向传播

3、推断：RPN产生RoI，R-FCN进行分类和回归，NMS

4、visualization：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-FCN/figure3.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-FCN/figure4.png)