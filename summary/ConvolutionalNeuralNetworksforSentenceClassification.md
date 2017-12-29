![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalNeuralNetworksforSentenceClassification/figure1.png)

1、要点

（1）使用一个卷积层，但是具有多个尺寸的卷积核

（2）每一个卷积窗口在多个词向量的所有维度上进行卷积

（3）卷积层的输出是多个一维向量，即每个feature map都是一个一维向量，接下来会在整个一维向量上做 max pooling，组成全连接层的输入

（4）词向量可以被看做是特征提取器，把词的语义特征编码到其向量表示中