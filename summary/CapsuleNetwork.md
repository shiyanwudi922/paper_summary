一、CapsuleNet的特点

1、提出了Capsule的概念，同时对特征进行分层，一个Capsule就是某一层中的一组神经元组成的向量，代表了该层的某一个实体的特征（例如低层的Capsule可能表示眼睛、鼻子、嘴巴，高层的Capsule可能表示人脸）。CapsuleNet不仅对输入数据中的局部特征进行了建模（Capsule向量的模小于1，表示其代表的实体在输入数据中出现的概率，Capsule的方向表示了该实体的位置、亮度、姿势等其他特征），而且对于不同层次的特征之间的层次相对位置关系进行了建模（例如低层代表眼睛、鼻子、嘴巴的Capsule和高层代表人脸的Capsule之间的关系）。低层Capsule在转化成prediction vector时进行了仿射变换，该变换矩阵就代表了低层Capsule和高层Capsule之间的层次关系。

2、Capsule保证了输入数据中局部特征的“不变性”。即当输入数据中的局部特征发生变化时，对应的Capsule向量的模不变，表示该实体出现的概率不变，但是Capsule向量的方向会发生等价的变化。基于这种不变性，如果输入数据中的同一个实体有不同的实例（例如一个人脸，有不同的角度），对于CNN来说，需要每一个实例都有足够多的样本（每一个角度的人脸都需要比较多的样本），但是CapsuleNet不需要特别多的样本。

3、提出了Dynamic Routing的算法，通过routing-by-agreement的方式来确定低层Capsule如何把信息发送到高层的Capsule（发送到某些高层Capsule的信息多，另外一些高层Capsule的信息少），以及高层的Capsule主要从哪些低层Capsule中接收信息



二、从低层Capsule向量到高层Capsule向量的计算

设第l层某个Capusle向量为ui，第l＋1层某个Capsule向量为vj，则计算过程为

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/equation2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/equation1.png)

其中，Wij表示了低层Capsule和高层Capsule之间的相互关系，是一个系数矩阵，通过反向传播训练得到。

cij是prediction vector的加权系数，通过dynamic routing计算得到。dynamic routing的计算过程为：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/procedure1.png)



三、边缘损失

本文把CapsuleNet用在MNIST数据上，最后一个Capsule层有10个Capsule，分别代表结果的10个类别，每一个类别都可以计算一个对应的损失函数：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/equation4.png)

其中，若样本的真实类别为c，则Tc＝1，m+ = 0.9，m- ＝ 0.1。

一个样本的损失是10个类别的边缘损失之和



四、CapsuleNet的结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/figure2.png)

其中，需要注意的是，PrimaryCaps层一共具有6 * 6 * 8 * 32个特征，包含了6 * 6 * 32个8维的Capsule，DigitCaps层比较简单，具有10个16维的Capsule，该结构从仿射矩阵Wij的维度为8 * 16也可推断出来。

另外，在训练过程中，在DigitCaps层之后增加了三个全连接层对输入图像进行重建，使用最后一层的输出和输入图像之间的误差平方和作为重建损失，同时放缩系数为0.0005，加入到总的损失函数中。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/figure3.png)



五、实验

1、MNIST

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/table1.png)

2、一个Capsule中的每一维表示了什么

Capsule中的每一维从不同的角度表示了其所代表的实体的特征，例如，数字的宽度、厚度；某些数字特有的特征，例如，2的尾长，6的头长、圆圈；组合特征、局部特征等等。

实验：在计算出DigitCapsule之后，对其向量进行一些破坏，然后观察最终生成的结果，如下

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/figure4.png)

3、CapsuleNet对数据的仿射变换具有鲁棒性

实验：在MNIST数据上训练模型，在affNIST数据上测试，affNIST中每一个样本都是在MNIST基础上进行一个随机的仿射变换得到的。

结果为，在MNIST测试集上准确率为99.23%的CapsNet，在affNIST测试集上的准确率为79%；在在MNIST测试集上准确率为99.22%的CNN，在affNIST测试集上的准确率为66%.

说明CapsNet具有更好的鲁棒性。



六、能够分割高度覆盖的数字

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/CapsuleNet/figure5.png)



参考：https://medium.com/ai%C2%B3-theory-practice-business/understanding-hintons-capsule-networks-part-i-intuition-b4b559d1159b