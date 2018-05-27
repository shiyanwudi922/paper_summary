一、摘要：

1、本文提出了一种新的sequence to sequence框架，完全基于cnn。

2、优点1: 在训练时，所有元素上的计算操作可以完全并行化，从而有效利用gpu的有点；另外，由于每一个输入元素需要经历的非线性计算数量是固定的（为卷积层的个数，在rnn中，每个元素需要经历的非线性计算数量和输入序列长度以及当前元素的位置相关），所以优化过程更容易。

3、优点2:使用了GLU作为非线性单元，并使用了residual connection，使梯度传播更容易。

4、优点3:在每一个decoder层都使用了一个独立的attention模块，提高模型的表达能力。

二、模型结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/figure1.png)

1、position embeddings

对于输入序列，除了做word embedding之外，还对每一个元素的位置做了位置编码，从而显示地表示模型当前处理的输入序列的位置。word embedding和position embedding相加的和作为最后的表示序列，输入到模型中。

对于输出序列，做相同的处理。

2、卷积层结构

encoder和decoder的每一层都是用相同的结构。

其中每一层进行如下的一维卷积：

（1）一维卷积：使用大小为2d*kd的系数矩阵，对于输入序列中以目标元素为中心的连续k个元素的连接向量（维数为kd的一维向量），进行一维卷积（相乘），得到维数为2d的一维向量，加上偏倚，从而得到单个输入元素的卷积结果。对于每个输入元素进行相同的处理，得到输入元素的一维卷积结果（每一个元素都用维数为2d的向量表示）。

（2）GLU：gated linear unit

对于一个维数为2d的向量，使用其中的一半作为sigmoid函数的输入，产生门限单元，去对另外一半进行筛选，得到GLU的输出

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/equation_GLU.png)

（3）residual connection

GLU的结果再加上卷积层的输入，作为卷积层的输出

3、Multi-step Attention：在decoder中的每一层，都会进行attention模块计算，其中每一个attention的计算方式为

（1）计算attention的query：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/equation_1.png)

（2）计算attention权重

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/attention_score.png)

（3）计算attention加权和

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/equation_2.png)

（4）上下文向量c和当前decoder层的输出的和，作为下一个decoder层的输入

4、归一化策略：本文中通过有效的权重初始化和对网络局部的归一化（放缩），从而保证网络的方差不会过分地变化。

（1）对residual模块的输入和输出进行放缩

（2）对attention产生的条件输入向量c进行放缩

（3）对encoder layer的梯度进行放缩

5、权重初始化：权重初始化的动机和归一化相同，都是维持激励的方差在前向和后向的过程中尽可能不变。

（1）embedding由均值为0，标准差为0.1的正态分布进行初始化

（2）根据某一层的输出是否要输入到GLU中，从而进行不同的初始化

（3）根据使用的dropout，结合是否输入到GLU，也进行不同的初始化

三、实验

1、recurrent vs. convolutional model

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/table1.png)

2、ensemble results

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/table2.png)

3、generation speed

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/table3.png)



4、position embedding

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/table4.png)

5、multi-step attention

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/table5.png)

6、kernel size and depth

kernel size：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/table7.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/table8.png)

depth：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/figure2.png)

7、abstract summarization

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ConvolutionalSequenceToSequenceLearning/table6.png)

