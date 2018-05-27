一、在机器翻译中的Human Parity

1、本文中重新定义了机器翻译中的Human Parity的概念

如果双语学家判断由人生成的备选翻译和由机器产生的被选翻译是等价的，则该机器达到人类水平。

对于一个测试数据集，如果由机器翻译系统产生的结果所获得的人类打分结果和人类翻译结果所获得的人类打分结果之间的统计差别不是很大，则说明该机器翻译系统达到人类水平。

打分方法：翻译结果在多大程度上准确的表达了原文本的语义信息

2、本文中使用了四种方法来使翻译系统达到人类水平

（1）使用了翻译问题的对偶性，使得系统可以同时从source-to-target和target-to-source模型中进行学习，主要使用的技术为对偶学习(Dual Learning)和联合训练(Joint Training)

（2）对exposure bias问题（在序列生成过程中，之前步骤中产生的误差会在之后的步骤中被放大）进行进一步处理，主要使用的技术为推断网络（Deliberation Networks）和一致性正则化（agreement regularization）。

（3）数据选择与过滤

（4）系统融合



二、对偶学习：本文中使用了两种形式的对偶学习

1、对偶无监督学习：使用了单语数据进行训练，具体的方法参考对偶学习论文

2、对偶有监督学习：使用双语数据进行训练。核心思想是使得主模型和对偶模型的联合概率具有一致性，p(x,y) = p(x)p(y|x) = p(y)p(x|y)，从而在训练时引入一个额外的损失函数：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/equation5.png)

其中p(x)和p(y)分别是两个语言模型的打分结果。



三、联合训练：参考joint training论文



四、推断网络：参考deliberation network论文



五、一致性正则化

（1）以source-to-target模型为例，常规的序列生成方式是从左到右进行生成，如果出现exposure bias，会导致从左到右生成的结果在从右到左的模型中的打分会比较低。所以对于source-to-target模型，训练L2R模型时，使用R2L模型进行辅助训练，在目标函数中加入两个KL散度作为正则项：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/equation9.png)

通过限制L2R产生的数据分布和R2L产生的数据分布尽可能相同，从而抑制exposure bias问题。

该目标函数对L2R模型参数的梯度为：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/equation10.png)

其中，第一部分为双语训练数据的似然函数；第二部分为R2L模型产生的“违训练数据”的似然函数；第三部分为L2R模型自己产生的“违训练数据”的加权似然函数，对L2R和R2L模型的不一致的样本进行惩罚。在训练L2R模型时，R2L模型充当了辅助系统，在第二部分中提供了违训练数据，在第三部分中计算权重。

（2）用同样的方法可以再训练R2L模型。

（3）训练好的L2R模型可以为R2L模型提供更好的违训练数据和更准确的权重，从而提升R2L模型；而之后R2L模型又可以进一步用于提升L2R模型。以这种方法L2R和R2L可以互相提升，该过程进行到验证集上的性能不再上升为止。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/figure2.png)

（4）由此，一共有四个模型：S2T-L2R、S2T-R2L、T2S-L2R、T2S-R2L，这四个模型可以通过一个统一联合训练框架，使用一致性正则化进行训练。训练时，T2S-L2R模型产生加权违训练数据，从而训练两个S2T模型；训练之后的S2T-L2R模型再生成加权违训练数据，从而训练两个T2S模型。四个模型互相提升，知道达到收敛。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/algorithm1.png)



综合总结：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/equation_infer_1.jpg)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/equation_infer_2.jpg)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/equation_infer_3.jpg)



六、数据选择和过滤

1、在机器翻译中，有两个重要的问题：选择任务相关的数据，去除噪声数据。

2、本文中使用的方法：训练一个统一双语神经机器翻译系统，即可以把不同的语言映射到同一个向量空间，从而使用单个模型进行不同方向的翻译。本文训练该模型时，使用已知的高质量且在相关领域中的数据进行训练，可以同时达到选择相关数据和去除噪声的目的。

3、对于双语数据中的一个样本，把源句子S和目标句子T分别通过encoder进行处理，从而得到对应的句子向量表示（隐状态向量和）：Ssv，Tsv.

4、计算每一个语句对的相似度
![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AchievingHumanParity/equation14.png)

设置一个阈值，删除所有的相似度小于阈值的语句对，从而对数据进行选择和过滤。



七、系统融合与重排序



