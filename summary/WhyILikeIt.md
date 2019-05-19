一、摘要

1、本文提出了一个新颖的multi-task模型，用于在推荐中同时进行打分预测和推荐解释

2、打分预测部分使用的是概率矩阵分解（probabilistice matrix factorization）

3、推荐解释部分使用的是Seq2Seq模型，生成评论式的解释

4、对Seq2Seq模型的训练使用了GAN和强化学习的思想

5、训练过程中，将Seq2Seq模型中encoder生成的文本特征和矩阵分解模型中生成的隐向量特征进行互相正则化，从而将打分预测和推荐解释结合起来

6、本文的亮点在于对矩阵分阶、Seq2Seq、GAN、强化学习这些算法思想的结合。



二、模型结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/WhyILikeIt/figure1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/WhyILikeIt/figure2.jpg)

具体的解释：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/WhyILikeIt/explanation.jpg)

使用强化学习对generator进行训练时，将generator看作是agent（智能体），当前状态是目前已生成的序列，采取的行为是生成序列中的下一个词，得到的reward是discriminator对当前序列的打分。

使用MC进行discriminator对子序列的打分进行近似：从一个子序列开始，重复地进行采样生成，直到生成K个完整的序列，使用discriminator对这K个序列进行打分，在求平均，从而得到子序列的打分。

优化算法：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/WhyILikeIt/algorithm.jpg)



三、实验

1、Rating Prediction

使用了五个数据集，五个对照算法，对照指标使用SSE

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/WhyILikeIt/table2.jpg)

2、Explanation Quality 

使用相同的数据集，五个对照算法，对照指标使用perplexity和tf-idf similarity

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/WhyILikeIt/table3.jpg)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/WhyILikeIt/table4.jpg)

3、Performance of the Discriminator 

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/WhyILikeIt/figure3.jpg)