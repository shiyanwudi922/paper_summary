一、摘要：本文介绍了FusionNet，从三个角度扩展了attention的方法

1、提出了一个新的概念“history of word”：

对于一个词，其从词级别的embedding表示到高级的语义表示的组合，称为该词的history-of-word，描述了一个词的最完整的信息。

2、提出了一种改进的attention scoring function：

在以history-of-word作为输入的基础上，在scoring function上加入了对称性和非线性，从而减少计算代价并且提升性能。

3、提出了一种新的fully-aware multi-level attention mechanism：

fully-aware表示使用了词的完整信息，即history-of-word，来计算attention的权重；multi-level表示在词的多个级别的信息上进行聚合，把问题的信息聚合到文本上。

二、机器阅读理解的常用框架及本文中的扩展：给定两个向量集合A和B，使用B的信息去对A中的每一个向量进行修改和提升，也就是融合过程（fusion process），把B融合到A。目前在MRC中主要的工作集中于如何设计融合过程。

1、常用的机器阅读理解框架包括三个部分

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/figure2.png)

（1）Input Vectors：在文本和问题中每一个词的embedding

（2）Integration components：对input vector进行处理，通常使用rnn（figure2中的矩形部分）

（3）Fusion process：进行信息融合（一般是从问题到文本）（figure2中的箭头（1）（2）（2’）（3）（3’））

2、目前主要使用的fusion process主要有三种

（1）word-level fusion：把问题中的词信息直接提供到文本中。但是当文本中的词具有不同的语义时，这种方法作用不大。有一些word-level fusion是不使用attention的，例如在文本词的embedding上加入一个二值特征，说明该词在问题中是否出现过。

（2）high-level fusion：把问题中的语义信息融合到文本。可以帮助发现正确的答案，但是精确度不如词信息，可能导致模型不关注细节。

（2’）high-level fusion（alternative）：把high-level的问题信息融合到word-level的文本中

（3）self-boosted fusion：当文本比较长时，其中的距离比较大的子文本之间会互相依赖，所以使用自提升来对文本进行自我融合。由于文本通常含有过多的信息，所以通常把自提升放在问题融合之后，从而更关注文本中和问题相关的部分。

（3’）self-boosted fusion（alternative）：在问题的条件上进行自提升的fusion，例如coattention。这样就可以在问题信息融合之前进行自提升融合

3、fully-aware attention on history of word

（1）history-of-word：我们在阅读文本时，每一个词都会从低级概念逐渐地转换成更加抽象的高级概念，这个过程在人类的脑海中形成了每个词的历史，人类经常去使用history-of-word的概念，但是会忽略它的重要性。

在一段文本中，第i个词的history-of-word定义为：对该词所生成的所有的表示向量的连接，包括：word embedding、multiple intermediate and output hidden vectors in RNN、以及其他网络层中对应的表示向量。

（2）fully-aware attention：当把attention应用于把文本B中的内容信息融合到文本A中时，对于A中的每一个词，都和B中的所有的词计算attention分数、权重，并用B中的所有的词的加权和作为A中该词的表示。

原始的attention在计算分数时，使用的是A和B中的原始词，当使用fully-aware attention时，将该原始词改为对应的history-of-word。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/fully-aware-attention.png)

（3）improved scoring function

原始的attention scoring function：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/original-scoring.png)

两个大的参数矩阵直接交互会使得神经网络难以训练

加入对称性的attention scoring function：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/symmetric-scoring.png)

加入了对称性，参数矩阵只有一个（D为对角矩阵），一方面使得模型更容易训练，另一方面对称性使得模型可以对不相似的向量赋予更高的分数，即使模型具有在不相似的向量之间进行对齐的能力

加入对称性和非线性的attention scoring function：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/symmetric-nonlinear-scrong.png)

加入了非线性之后，可以在history-of-word的不同部分之间提供更丰富的交互

三、Fully-Aware Fusion Network

1、端到端框架

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/figure4.png)

具体流程

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/end-to-end-explanation.jpg)

2、应用于机器阅读理解

使用attention对问题向量序列进行合成，作为rnn的初始状态；在每一个时刻，先根据前一时刻的hidden state和文本向量序列计算attention权重，作为当前时刻输出词的概率分布，然后计算文本向量序列的加权和作为当前时刻rnn的输入，计算其输出；重复该过程，得到起始词和终止词的概率分布。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/comprehension-explanation.jpg)

四、实验

1、主要结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/table2&3&4.png)

2、attention函数的对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/table5.png)

3、history-of-word的效果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/FusionNet/table6.png)