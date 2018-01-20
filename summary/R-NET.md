一、引言：本文有三个贡献

1、使用gated attention-based recurrent network，从而得到question-aware passage representation，同时加入了额外的门单元，用于处理“在回答一个特定问题时，文本中的词具有不同的重要性”

2、使用self-matching，让文本进行自我表示，从而尽可能对汇聚整个文本中的信息

3、产生了state-of-the-art的结果



二、R-NET网络结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/figure1.png)

1、问题和文本编码

（1）词向量：对于问题和文本中的每一个词，使用两种编码方式。一是使用word-level的词向量，该向量使用GloVe进行预训练得到，且在模型训练的过程中保持固定；二是使用character-level的词向量，对每个词的字符序列，使用双向rnn进行处理，取最后一个时刻的hidden state连接得到字符级别的词向量。再将每个词的word-level向量和charater-level向量连接，得到最终每个词的向量。

（2）编码：获得问题和文本的词向量序列之后，分别使用双向rnn进行处理，对每一个时刻的hidden state进行连接，从而得到问题和文本的编码序列。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation1%262.png)

2、gated attention-based recurrent networks

（1）对于文本序列中的每一个词，都在问题序列上计算attention，并得到对应的问题序列的加权和上下文向量

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation4.png)

（2）加入门操作，对文本序列中的每一个词及其上下文向量的内容进行选择，从而处理“文本中只有部分信息和特定的问题相关，通过门操作来决定文本中的信息对于特定问题的重要性”

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation6.png)

（3）使用双向rnn对门限之后的向量序列进行处理（在equation3中显示的是单向rnn，但是figure1中显示的是双向），计算question-aware repersentation for passage

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation3.png)

3、self-matching attention

（1）以上得到的question-aware passage representation会有一个问题，包含的上下文信息比较少，所以本文中使用了self-matching attention

（2）对于question-aware passage representation中的每一个词，计算在自身序列上的attention，并得到对应的加权和上下文向量

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation8.png)

（3）加入门操作，方式和之前一样

（4）使用双向rnn对门限之后的向量序列进行处理，得到最终的文本向量序列

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation7.png)

4、输出层：输出层使用的是poiter network，实质上是一个基于attention且只有两个time step的rnn，每一个time step分别输出答案序列在原文中的的起始位置和终止位置。在每一个时刻上，其attention概率分布就是起始/终止位置的概率分布，本文中使用最大概率对应的位置作为当前时刻的输出位置。

（1）pointer network的初始状态：由问题向量序列通过attention合成

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation11.png)

其中Vr是参数向量

（2）在每个时刻，通过attention计算起始/终止位置

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation9.png)

（3）在每个时刻，使用attention计算文本向量的加权和，作为rnn的输入，计算当前时刻rnn的输出

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/equation10.png)



三、实验

1、本文中提出的R-NET在SQuAD和MS-MARCO上均达到了state-of-the-art的结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/table2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/R-NET/table3.png)

2、除了直接的实验结果，本文还列出了在研究过程中的一些探索，虽然最后没有成功，但是这些探索的方向值得学习：

（1）句子排序（Sentence Ranking）

（2）加入语法信息（Syntax Information）：词性标注，命名实体识别线性PCFG树标注、依赖标签等。

（3）多级推断（Multi-hop Inference）

（4）问题生成（Question Generation）

