一、本文动机

1、在实时响应任务中，模型需要有很高的准确率和严格的响应时间限制。而通常高准确率的模型都具有更大的深度、更多的参数，以及更高的计算复杂度，使得这些模型在预测时会非常耗时，从而不能应用在对响应时间要求很高的系统中。

2、通常有两种方法可以用于在保证适当性能的同时降低运行时复杂度

（1）使用分解和压缩（factorizing or compressing）来直接简化计算

（2）使用teacher-student策略，预先训练一个复杂度比较高的teacher network，然后使用该网络来辅助训练一个简单的light network，最终使用light network进行预测。

3、本文在teacher-student的基础上，提出了一个改进的系统，能进一步提高整体系统的性能



二、网络结构与特点

1、网络结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/figure1.png)

损失函数为

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/equation1.png)

且其中hint loss只对light net进行更新，不对booster net产生任何影响。

2、参数共享

在RocketLaunching中，light net和booster net会共享一些低层的网络参数，该方案使得light net可以直接使用booster net学习到的信息。

3、协同训练

在常见的teacher-student中，会先预训练一个teacher net，然后对student net进行辅助训练；而RocketLaunching中，light net和booster net是同时进行协同训练的，这种训练方式有两个好处：

（1）整体的训练时间减少，从而使该框架可以应用在模型结构经常变化的系统中

（2）在light net的整个训练过程中都会受到booster net的指导，从而使得light net不仅可以从目标和本身输出之间的差距中进行学习，还可从booster net所提供的学习路径中进行学习。（复杂的网络所学到的知识不仅可以表现在其最终输出，而且也表现在整个训练过程中。）

3、hint loss

通过对hint loss进行优化，从而使得booster net的知识可以向light net进行转移。

本文中考虑了三种hint loss：

（1）MSE of final softmax：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/equation_lmse.png)

该loss函数对logits的梯度为：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/equation2.png)

由上式可以看出，当logits非常小时，pi(x)会非常接近零，从而导致gradient vanishing，使得该损失函数不能有效的进行学习（即使是light net的输出和booster net的输出之间区别非常大）

（2）MSE of logits

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/equation_lmimic.png)

该loss函数对logits的梯度为：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/equation3.png)

该损失可以直接减少logits之间的区别，从而避免了L~mse~中出现的gradient vanishing问题

（3）knowledge distillation

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/equation_lkd.png)

当T比较大时，该loss对logits的梯度为：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/equation4.png)

事实上，该loss也容易产生gradient vanishing问题。

另外，该loss中使用了temperature参数T，对输出的class probability进行“软化”（soften），从而使得该loss更加关注比较小的logits。Hinton建议中等大小的T参数比较合适，因为可以把非常小的logits忽略掉（该logits可能是噪声）。但是本文的工作中发现略大一些的T效果更好，会对所有的logits进行优化，可能的原因是非常小的logits中依然会包含booster net所获取的有用信息。

4、Gradient Block

当同时使用交叉熵和hint loss作为目标函数对booster net进行更新时，会使得booster net的输出受到light net的严重影响，从而妨碍了booster net直接从任务中进行学习。

另一方面，由于light net的学习能力有限，会使得booster net的性能下降，而light net会在训练过程中从booster net传输的知识中进行学习，所以会进一步降低light net的性能。

为了解决该问题，本文中提出gradient block技术，使得booster net不去对hint loss进行优化，也就是说，在hint loss反向传播时，把booster net独有的参数看作是常数，不进行更新。从而使得booster net不受light net的影响，有更高的自由度，能够直接从ground truth中进行学习，达到最优的性能。



三、实验

1、Experiments on CIFAR-10

基础效果：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/table1.png)

对不同hint loss的探索：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/table3.png)

RocketLaunching中不同组成部分的效果：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/table2.png)

在共享的网络层设置不同的层数：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/figure4.png)

特征可视化：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/figure5.png)

从该图中可以看出，light net和booster net在低层所生成的feature map是非常相似的。

2、Experiments on SVHN and CIFAR-100

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/table4.png)

3、Experiments on real Advertisement Dataset

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/RocketLaunching/table5.png)

















