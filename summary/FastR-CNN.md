![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FastR-CNN/figure1.png)
一、优点

1、加速

（1）训练阶段

a、对整个图像进行CNN传输，从而在该图像上的多个region proposal之间进行特征共享，减少了CNN传输的次数和参数的数量，从而使前向传输和反向传播的效率大大提升。

b、使用多任务损失函数，可以进行整体的单阶段训练（同时训练softmax分类和bounding box回归），而不需要再像之前进行多阶段训练（依次训练softmax分类、SVM、bounding box回归）

（2）测试阶段

a、对大型的全连接层参数矩阵进行奇异值分解，变成两个小的矩阵，减少了参与计算的参数个数，提升了速度

2、空间

（1）不需要多阶段训练，对于分类和回归，没有提取并存储特征的开销

3、效果

（1）使用多任务损失函数进行训练，通过特征共享来互相影响，从而提升效果

（2）允许对卷积层的参数进行微调，从而进一步提升效果（但是不一定需要对所有的卷基层都进行微调）

二、SPPnet不对SPP layer之前的卷积进行微调的原因

1、在SPPnet训练时，在一个minibatch内，每一个region proposal来源于不同的图像，并进行独立的CNN传输，导致SPP layer的输入数非常多，从而参数非常多，如果对SPP layer之前的层进行微调，计算效率会非常低。另外，当region proposal的receptive field非常大（接近于原始图像）时，该问题会更加严重；

2、在fast R-CNN训练时，在一个minibatch内，多个region proposal来自于同一个图像，从而做到了feature map在region proposal之间的共享，当对SPP layer之前的层进行微调时，很多参数是共享的，计算效率大大提升。

三、对设计决策的评价

1、使用多任务的损失函数进行训练，可以通过共享特征进行互相影响，从而提升检测效果

2、尺度不变性：测试时使用单一尺度的输入还是多尺度的输入。

单一尺度的输入，速度快，但是效果不好；多尺度的输入，速度慢，但是效果很好（因为可以提供尺度不变性）。需要在速度和效果之间取合适的折中。

对于比较深的模型来说，其学习能力较强，能够从单尺度的输入中学习到尺度不变性（使用多尺度学习对效果的提升不明显，但是速度下降很大），所以一般使用比较深的模型+单尺度输入即可达到最优的折中。

3、当模型容量足够的时候，增加训练数据量可以有更好的效果

4、分类时使用softmax还是SVM。使用softmax会在类别之间引入竞争，从而提升效果，另外在训练时不需要分阶段训练，所以使用softmax会好一些。

5、使用更多的proposal不一定会提升效果