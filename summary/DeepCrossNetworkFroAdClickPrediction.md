一、摘要

1、在预测模型中，特征工程通常是成功的关键

2、常用模型的问题

（1）需要进行手动特征工程和穷尽搜索

（2）使用DNN时，虽然可以自动学习特征交互，但是只能进行隐式的特征交叉，并且不能学习到所有类型的交叉特征。同时，训练效率比较低。

3、本文提出的deep & cross network的优点

（1）、cross network的每一层都会进行显示的特征交叉

（2）、cross network的每一层都会使交叉特征的阶数增加，所以最终交叉特征的阶数取决于cross network的层数

（3）、cross network包含了对应阶数的所有可能的交叉特征，且它们的系数都不相同

（4）、cross network的参数数量和输入特征维数成线性关系，参数量级比较小，训练效率比较高

（5）、cross network不需要手动特征工程和穷尽搜索



二、网络结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/figure1.png)

1、Embedding and Stacking Layer

先对离散特征进行embedding，对连续特征进行归一化（在本文中，连续特征使用log变换进行归一化）；然后将其进行连接（本文中连接操作成为stacking）

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/equation2.png)

2、Cross Network

把Embedding and Stacking Layer的输出x~0~作为Cross Network的原始输入，在Cross Network的每一层，执行以下操作：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/equation3.png)

其中，x~l+1~是第l+1层的输出，x~l~是第l层的输出，x~0~是Cross Network的输入，w~l~和b~l~是对应的参数。

（1）当第l+1层接收到第l层的输出后，通过求x~0~和它的外积，从而提高交叉特征的阶数，也就是交叉特征的阶数会随着层数的增加而增加。一个l层的cross network的交叉特征的最大阶数为l+1，且该网络包含了从一阶到l+1阶的所有交叉特征。

（2）在cross network的每一层中，w~l~和b~l~都是列向量，维数和输入向量的维数相同，所以cross network的参数数量为2\*d\*L~c~（其中，d为向量维数，L~c~为层数）。其时间和空间复杂度都和输入向量维度成线性关系，所以相比于传统的dnn，cross network只增加了很少的复杂度

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/figure2.png)

3、Deep Network

该网络就是通常使用的全连接前馈神经网络

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/equation4.png)

4、Combination Layer

该层吧cross network和deep network的输出向量进行连接后，输入到sigmoid函数，作为网络的最终输出

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/equation5.png)

损失函数即为常见的逻辑损失

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/equation6.png)



三、Cross Network分析

本文从三个角度对cross network进行分析：polynomial approximation, generalization to FMs, efficient projection

1、Polynomial Approximation

（1）、任意一个符合特定平滑性假设的函数，都可以用多项式以任意的准确率进行近似

（2）、cross network可以对同阶多项式进行近似，且高效、表达能力强，更容易扩展到真实数据集

（3）、所以cross network也可以对一个任意的函数进行很好的近似：cross network -> polynomial of same degree -> any function

（4）通过O(d)个参数，cross network包含了同阶多项式中所有的交叉项，且它们的系数各不相同

2、Generalization of FMs

（1）cross network继承了FM算法中参数共享的思路，并且扩展到了深层结构中：

（2）参数共享：不仅可以使模型的效率更高，而且可以泛化到从未出现过的交叉特征，且对噪声的鲁棒性更好；

（3）扩展到深层结构：FM只能表示二阶交叉项，是一种单层的结构，cross network扩展到了多层和高阶的交叉项，该阶数由cross network的层数决定

3、Efficient Projection

（1）在每一个cross layer中，首先建立了d^2个交叉项，然后通过一种高效的方法再将其投射回原始的维度，该方法的代价和d成线性关系。而通常直接的投射方法和d成立方关系。



四、实验结果

1、实现细节
本文中，实数特征使用log变换进行归一化，分类特征进行embedding的维数为 6\*（分类特征的取值个数）^1/4^

2、不同模型的性能对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/table1.png)

3、DCN和DNN的性能对比

（1）达到相同的logloss所需要的参数量

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/table2.png)

（2）参数量相同时能达到的logloss

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/table3.png)

（3）随着cross layer数量的增加，logloss的变化

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeepCrossNetworkForAdClickPrediction/figure3.png)

