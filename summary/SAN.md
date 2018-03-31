一、摘要

1、本文提出的stochastic answer network在answer network部分具有两个特点：

（1）使用了多步推断，模拟人类的逐步理解过程，在每一步都生成一个答案，把所有生成的答案取均值作为最终的结果

（2）在训练时，在对所有推断步的结果取均值之前，先使用了dropout，随机删除某个推断步的结果，从而促使每一个推断步都产生正确的结果，提高模型的鲁棒性；在预测时，不再使用dropout。

2、在效果上，动态多步推断要优于固定多步推断，固定多步推断要优于单步推断。



二、模型结构：SAN由四个部分组成，设问题次序列为Q，文本词序列为P

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/figure2.png)

1、lexicon encoding layer

（1）对于Q和P，先预训练得到他们的GloVe向量，300维

（2）对于P中的每个词，加入三种类型的语言特征：9维的POS embedding，8维的NER embedding，3维的binary exact match特征（分别表示P中的每个词是否匹配到Q中某个词的original、lowercase、lemma形式）

（3）对于P的每个词，使用attention计算使用Q对该词的表示，280维：首先使用单层神经网络对P和Q中的每个词的300维GloVe向量进行转换，变成280维的向量，然后通过280维的向量使用点积来计算attention分数，最后使用Q的280维向量结合attention分数来表示P中的词，得到P的280维向量表示

（4）将以上得到的特征进行连接，得到Q的300维向量表示，P的600维向量表示

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/lexicon_attention_score.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/lexicon_aligned_question_emb.png)

其中，g()表示单层神经网络ReLU(Wx)

（5）分别使用两个双层前馈神经网络对Q和P进行处理

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/lexicon_FFN.png)

得到最终的问题和文本的向量表示

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/lexicon_Q.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/lexicon_P.png)

其中，d为128

2、contextual encoding layer

（1）用两个双层BiLSTM分别处理问题和文本

（2）之后再使用maxout layer来减少参数

（3）最后使用layer normalization进行处理

最终的问题和文本向量表示

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/contextual_Q.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/contextual_P.png)

3、Memory Generation Layer

（1）mutual attention：再次使用attention，用问题表示文本。先使用单层神经网络对问题和文本进行处理，处理之后的向量计算attention矩阵，然后对attention矩阵使用dropout，再通过attention矩阵来用问题表示文本（注意该问题向量是单层神经网络处理之前的向量），最后把该表示向量和单程神经网络之前的文本向量进行连接得到文本表示

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/equation1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/equation2.png)

（2）self-attention：使用self-attention，用上一步得到的文本进行自我表示

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/equation3.png)

（3）把前面两部得到的文本表示进行连接，再通过BiLSTM进行处理，得到最终的memory

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/equation4.png)

4、Answer Module

（1）使用rnn来维护状态变量。使用固定长度的rnn来维护一个状态变量，rnn的初始状态是问题向量的加权和（attention），每一步的输入是Badanau attention计算得到的memory的加权和。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/memory_s0_attention.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/memory_s0.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/memory_badanau_attention.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/memory_input.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/memory_state.png)

（2）在rnn的每一步，使用hidden state生成一个起始词分布和一个终止词分布

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/equation5&6.png)

所有的起始词分布求均值，所有的终止词分布求均值，得到最终的答案分布

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/SAN/equation7&8.png)

（3）训练时，在上一步的取均值之前，先使用一次dropout，随机丢弃某些时刻的预测结果，用剩余的求均值；在预测时，不再使用dropout

（4）answer module的优点：一方面使用多步推断，模拟人类的理解过程，且同时使用所有时刻的输出，取均值，提升模型效果；另一方面，训练时在layer级别上使用dropout，进一步提高了模型的鲁棒性。

三、实验结果与分析

