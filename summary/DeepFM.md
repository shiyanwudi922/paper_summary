一、摘要
1、已有的很多模型存在两个问题：一是会对低阶特征和高阶特征之间有所倾斜，只着重考虑其中一种；二是需要专家级别的特征工程。
2、DeepFM通过两种做法来解决这两个问题：一是把FM算法考虑低阶特征的能力和深度学习进行特征学习的能力结合起来，同时考虑低阶和高阶特征；二是在wide和deep组成部分之间共享embedding特征，从而不需要对embedding进行与训练，并且在不需要特征工程的同时，减小了输入的维度，提高训练效率。



二、模型结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepFM/figure1.png)

令(X , y) 为模型的原始训练数据，X为包含m个field的输入向量，通常会涉及到一个user-item对；令(x, y)为对特征进行1-of-k编码之后的数据，x = [x~field1~,x~field2~,...,x~fieldj~,...,x~fieldm~] ，x~fieldj~是第j个field的向量表示，所以x是高维稀疏向量。

对于特征i，标量wi表示其一阶重要性，隐向量（embedding）Vi表示该特征和其他特征进行交叉时的重要性（在FM部分中表示二阶特征，在deep部分中表示高阶特征）

1、FM部分

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepFM/figure2.png)

FM中同时包含一阶和二阶特征。

一阶特征为1-of-k向量的加权和。

二阶特征为每个field的embedding向量的两两之间的点积。

FM输出为Addition单元和Inner Product单元的和：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepFM/equation2.png)

2、Deep部分

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepFM/figure3.png)

1-of-k向量进行embedding之后，再输入到全连接的DNN中。

3、embedding特征共享

另外，FM部分和deep部分共享相同的embedding特征，会有两个优点：一是同事从原始特征里学习低阶和高阶特征；二是不需要特征工程

这种特征embedding的共享策略，通过低阶和高阶特征交互以反向传播的形式影响了特征表示，从而更准确的对特征进行建模。

4、和其他DNN模型的关系

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepFM/figure5.png)

（1）FNN：使用FM对embedding进行预训练，然后和DNN进行连接

（2）PNN：在embedding层和DNN时间加入一个乘积层，根据乘积的形式可以分为三种，IPNN(inner product)，OPNN（outer product），PNN*(both inner and outer product)。

（3）wide&deep：同时学习低阶和高阶特征，但是需要复杂的特征工程

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepFM/table1.png)



三、实验

1、数据集：criteo dataset(http://labs.criteo.com/downloads/2014-kaggle-display- advertising-challenge-dataset/ )

以及公司数据集

2、实验结果

（1）效率对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepFM/figure6.png)

（2）效果对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepFM/table2.png)

学习特征交互会极大提高模型性能；

同时学习低阶和高阶特征极大提高模型性能；

共享embedding特征也会提高模型性能。

（3）超参数

激励函数：relu和tanh比sigmoid更适合deep模型

dropout、每层的神经元个数、隐层个数需要调整到合适的值。

网络形状：本文中，‘constant’结构比increasing、decreasing、diamond的效果都要好



四、结论：本文提出的DeepFM不需要进行预训练，同时学习低阶和高阶特征，并且在wide和deep之间使用embedding特征共享，从而不需要进行特征工程。









