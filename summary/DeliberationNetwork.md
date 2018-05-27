一、引言

1、传统的encoder-decoder框架在生成序列时，每一个词只能使用之前已生成的词的信息，而不能使用未来的还未生成的词。

2、为了在生成每一个词的时候能使用句子的完整信息，本文使用deliberation network，包含两个decoder。第一个decoder按照标准的encoder-decoder进行操作，第二个decoder在此基础上，使用完整的句子信息进一步提炼生成序列。



二、网络结构和计算流程

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/figure1.png)

1、encoder和decoder1

由encoder对输入序列进行处理，得到对应的隐状态序列；

decoder1在生成每一个词时，先通过Badanau attention计算上下文向量，然后结合前一个状态和生成词计算状态向量，最后通过仿射变换和采样生成当前时刻的词。其中上下文向量计算方式为：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/equation1.png)

2、decoder2

decoder2会计算输入序列的上下文，decoder1对应的上下文，然后结合前一个状态和生成词计算状态向量，最后通过仿射变换和采样生成当前时刻的词。其中decoderq1对应的上下文计算方式为：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/equation2.png)



三、算法：在训练的过程中，如果对目标函数直接进行反向传播，则需要在decoder1对应的所有可能的生成序列上进行遍历，计算非常复杂；本文采用了蒙特卡洛采样对目标函数及其梯度进行无偏估计。具体计算如下：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/equation_infer_1.jpg)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/equation_infer_2.jpg)



四、实验

1、神经机器翻译：浅模型

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/table1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/table2.png)

2、神经机器翻译：深模型

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/table4.png)

3、文本摘要

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DeliberationNetwork/table5.png)

