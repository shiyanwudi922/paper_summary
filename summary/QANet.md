一、摘要

之前的机器阅读理解模型基本上都是基于RNN的，而RNN的序列化性质会导致模型训练和推断速度非常慢，无法并行化，所以本文提出了一个新的模型结构，具有以下特点：

（1）数据处理完全抛弃了rnn，只使用前馈网络结构，包括卷积、self-attention和feed forward network，一方面，由卷积对局部交互进行建模，由self-attention对全局交互进行建模，另一方面，模型不具有序列化性质，可以并行化。所以该模型可以同时兼顾准确率和速度这两方面的性能；

（2）由于训练速度加快，所以模型可以在更多的数据上进行训练。本文中提出了一种新的数据增强的方法，使用双向机器翻译模型对原始的阅读理解数据进行增强，提高了训练数据的质量和多样性（quality、diversity），进一步提高了模型准确率。

二、模型结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/figure1.png)

1、Input Embedding Layer

（1）每个词的embedding由该词的word embedding和character embedding组成

（2）word embedding：由预训练的GloVe向量组成，维度为300维，且在模型训练时保持固定，oov的词映射为<UNK>，并且使用可训练且随机初始化的embedding

（3）character embedding：每一个字符对应一个200维的向量，可训练；先把每个词填充或者截断为16个字符，然后把这16个字符的向量连接成一个200*16的矩阵，对每一行取max pooling，得到一个200维的字符级向量。

（4）将每个单词的word embedding和character embedding连接起来，在使用两层的highway network进行处理，得到每个单词最终的embedding



2、embedding encoder layer

（1）一个encoder是由三种基本的单元堆叠而成：[convolution-layer * # + self-attention-layer + feed-forward-layer]

（2）convolution-layer：使用depthwise separable convolution，优点是memory-efficient且有better generalization。kernel size是7，filter个数是128，卷积层的个数为4。输入为500维的词向量，由卷积映射为128维。这里的卷积是如何操作的没有说的很清楚，猜测是对于一个kernel size内的词，进行连接得到7*500的一维向量，然后使用kernel矩阵128 * 7 * 500维，对该一维向量转换后，得到128维的一维向量，作为卷积的输出。

（3）self-attention-layer：使用multi-head attention，head数为8

（4）在所有的子模块（convolution/attention/ffn）中，都有layer normalizaiton，以及residual连接

（5）这里encoder block的个数为1



3、context-query attention layer

（1）用C（n\*d）和Q（m\*d）分别表示文档和问题向量集合，先计算相似度矩阵S（n\*m），按行进行归一化得到S'(n\*m)，按列进行归一化得到S''(n\*m)，然后进行interaction，方法用问题表示文档，得到attention矩阵A，表达式为A = S'\*Q；以及先用文档表示问题，再用问题表示文档，得到attention矩阵B，表达式为B=S'\*S''^T^*C。C、A和B一起做为下一层的输入。

（2）向量相似度计算，使用trilinear函数f(q, c) = W~0~[q, c, q⊙c]



4、model encoder layer

（1）本层的输入为C、A和B，在每一个位置上的输入为[c, a, c⊙a, c⊙b]

（2）本层encoder的框架和embedding encoder layer相同，只是其中卷积层个数为2，同时encoder block的个数为7，且在其中每3个副本之间进行权重共享



5、output layer

（1）生成答案起始和终止位置的概率分布：考虑共享权重，前面的model encoder有三个不同的层，其输出分别设为M~0~、M~1~、M~2~，先将M~0~和M~1~、M~0~和M~2~进行连接，在分别进行线性映射和softmax，得到起点和重点位置的概率分布，表达式为：

p^1^ = softmax(W~1~[M~0~;M~1~])，p^2^ = softmax(W~2~[M~0~;M~2~])

以起点和终点的概率乘积作为一个答案的分数，在推断时选择分数最高的答案作为最终答案。

（2）损失函数：使用训练集上的平均交叉熵做为目标函数

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/figure_obj.png)



三、data augmentation by backtranslation：本文中使用了对称的神经机器翻译模型进行data augment，具体的做法如下

（1）原训练数据中的一个样本为一个三元组（d, q, a），其中d为文档，q为问题，a为对应的答案。在做data augment时，把该三元组转化为另外一个三元组（d', q, a'），其中q保持不变，从而扩展训练数据集。

（2）document paraphrasing：对于文档d，先分解成对应的句子序列，对于其中每一个句子，先使用source-target模型翻译成k个其他语言的句子（k为beam width），再把这k个其他语言的句子用target-source模型翻译成k^2^个原语言的句子，作为对应句子的paraphrase。从文档d的每一个句子对应的paraphrase集合中进行采样得到d'，作为对应的document paraphrase。

（3）answer extraction：对文档d进行paraphrase之后，d‘中可能不再包含原来的答案a，所以要进行新的answer extraction。设原文档为d，包含答案a的句子为s，对应的paraphrase文档为d'，句子为s'，计算答案a的起点/终点词和s'中每一个词的character-level 2-gram score，选择分数最高的备选答案作为新的答案a'。



四、实验

1、SQuAD

（1）准确率

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/table2.png)

（2）加速

和RNN之间的对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/table3.png)

和BiDAF之间的对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/table4.png)

（2）ablation study和分析

下表同时说明了原模型中卷积、self-attention结构的作用，数据增强的作用，以及在训练时对原始数据和增强的数据使用不同的采样比所产生的作用

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/table5.png)

（3）鲁棒性

将模型在adversarial SQuAD数据集上进行测试，从而说明其鲁棒性

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/table6.png)



2、TriviaQA

（1）准确率

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/table7.png)

（2）速度

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/QANet/table8.png)

