![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SSD/SSD.png)

一、摘要

1、SSD把bounding box的输出空间进行离散化，变成在feature map上每一个位置的以不同的纵横比和尺度形成的默认box的集合。

2、SSD结合了不同网络层上的具有不同分辨率的feature map的预测结果，很自然地处理了不同尺寸实体的问题。

3、不需要单独的region proposal以及相关的特征提取过程，极大地提高了速度；使用了足够多的box，并且允许一个实体被多个box预测，提高了准确率。

二、简介

1、由于删除了bounding box proposal以及相关的特征提取过程，所以极大地提高了速度

2、和其他的删除proposal的方法相比，主要的改进为：

（1）使用小的卷积核进行类别和位置的预测；

（2）对同一个位置的不同的纵横比，使用独立的预测器；

（3）对网络中多层的feature map都进行预测，从而处理多尺度的问题

三、模型

1、在基础网络上添加卷积层，其中，每一层卷积输出的feature map逐渐减小，从而使得模型能够进行多尺度的预测

2、在某一层feature map的基础上，使用卷积进行预测：

设feature map的size为m*n*p，其中p为输出通道；feature map上每一个位置会预测b个bounding box，则有：

用于预测的卷积层的size为：p*k*k*[b*(4+21)]

输出的预测结果size为：m*n*[b*(4+21)]，对应于feature map上每一个位置、每一个bounding box的预测结果：4个坐标值，21个类别值

四、训练

1、匹配策略：先把每一个ground truth匹配到IoU最大的default box上；再把每一个default box匹配到IoU>0.5的ground truth上；其余的default box标记为背景。

2、损失函数：对于匹配到的default box，其坐标和类别同时参与计算损失函数；对于没有匹配到的default box，只有其类别参与计算损失函数；另外，对坐标的回归目标是其偏移量

3、default box的位置、尺度、纵横比：使用不同层的feature map，其中不同层的feature map其scale也不同；对于某一层的feature map，定义了default box的多个中心点和多个纵横比之后，就可以确定具体的default box。

4、负样本采样：由于负样本的比例过高，所以不使用所有的负样本，而是根据负样本的置信度进行排序，使用置信度最高的一部分，使得负样本和正样本的比例为3:1.

5、增加数据增强处理。

五、模型分析

1、数据增强：使用更多的数据增强技术可以提高性能

2、使用更多的box形状：当增加使用的box的纵横比时，性能进一步提升

3、使用多个层的feature map用于预测，可以提高性能

总的来说，SSD使用多个层的feature map，其中每个feature map对应的default box使用的scale都不同，从而对应了不同的分辨率；另外，在某个feature map上的每个位置，使用不同的纵横比形成不同的default box；这样，不同的scale结合不同的纵横比，可以应对不同的实体大小和形状的问题，使得SSD的性能非常好（即使输入图片的分辨率很低）。

<https://zhuanlan.zhihu.com/p/24954433>