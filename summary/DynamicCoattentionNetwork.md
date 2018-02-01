一、摘要：本文提出的DCN包涵两个主要部分

1、对问题和文档的依赖表示（co-dependent representation）进行融合，从而更加关注相关的部分

2、使用动态指向解码器（dynamic pointing decoder）在潜在的答案范围上进行迭代，从而得到正确的答案范围。<u>**该迭代过程使得模型可以从错误答案对应的局部极值上逐步进行修正。**</u>

二、DCN：dynamic coattention network

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/figure1.png)

1、文档和问题编码

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/doc_que_enc.jpg)

（1）初始词向量：使用GloVe在Common Crawl corpus上进行预训练，得到初始的词向量，同时把实验中用到的词典限制在Common Crawl corpus中的词，并且在实验中对词向量不再进行微调（防止过拟合）。

（2）文档编码：对文档的词向量序列，使用LSTMenc进行处理，得到文档编码矩阵D

（3）问题编码：对问题的词向量序列，使用相同的LSTMenc进行处理，先得到中间问题表示Q'，再进行一次非线性投射，得到最终的问题编码矩阵Q

2、coattention编码

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/figure2.png)

（1）就算affinity matrix，其实就是文档编码矩阵和问题编码矩阵的乘积，其中每个元素对应一个文档词和一个问题词的点积。再对affinity matrix分别按照行和列进行softmax归一化，得到attention矩阵AQ和AD

（2）对文档信息和问题信息进行融合，即先通过attention矩阵AQ使用文档来表示问题，再通过attention矩阵AD使用问题来表示文档，得到coattention context

（3）对coattention context再使用BiLSTM进行处理，得到最终的coattention encoding

3、dynamic pointing decoder：迭代计算answer span的起始和终止位置

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/figure3.png)

（1）通过LSTM对decoder中的状态序列进行处理。在每一个时刻，把前一次计算的answer span的起始和终止向量进行连接，作为LSTM的输入，前一次的hidden state作为反馈输入，计算当前时刻的hidden state。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/equation5)

（2）用两个结构相同，参数不同的HMN（highway maxout network）分别对coattention encoding中的每个词，计算其作为answer span的起始位置和终止位置的分数， 从而得到当前这轮迭代answer span的起始和终止位置。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/equation8)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/equation6-7)

（3）HMN的网络结构和计算方式

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/figure4.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/equation9-12.jpg)

其中，p是每一个maxout的pooling size。另外，在第一个maxout layer和最后一个maxout layer之间构造了highway connection。

（4）该网络结构使得模型可以通过迭代计算，从初始的错误答案所对应的局部极值中逐渐进行矫正，得到正确的答案，即具有自我修复能力

三、实验

1、结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/table1.png)

table1

2、case及解释

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DynamicCoattentionNetworksForQuestionAnswering/figure5)

（1）case1：开始时，模型预测的起始点是错的，但是在第三轮迭代时已经矫正到正确的位置；同理，对于错误的结束点，模型也可以做到正确矫正。

（2）case2：开始时，模型预测的起始点和结束点都是错的，但是依然可以做到正确较真。

（3）case3：说明了一种情况，模型有时很难在不同的局部极值之间进行选择，所以会在不同的起始点和结束点组合上来回交替。

3、本模型对于长文本不会有明显的性能下降