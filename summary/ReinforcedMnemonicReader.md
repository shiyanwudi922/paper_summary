一、常用的"encoder-interaction-pointer"结构的问题和本文提出的"reinforced mnemonic reader"的应对方法

1、encoder部分

EIP框架：一般只使用了词级别和字符级别的embedding来表示文本和问题此序列，却少了语法和语言相关的信息。

RMR解决方法：在已有的词级别和字符级别的embedding基础上，加入了语法和语言特征，包括exact match binary特征、POS特征、NER特征和问题类别特征。

2、interaction部分

EIP框架：使用单次attention和coattention难以获取在文本的不同部分之间存在的长上下文依赖

RMR解决方法：在文本和问题之间进行迭代对齐，通过多次迭代，从而有效地把相关的语义信息融合到上下文的每一个词中，获得fully-aware的上下文表示

3、pointer部分

EIP框架：在复杂的MC任务中，需要进行多句推断，且可能出现多个备选答案。而单跳("one-hop")形式的预测难以完全理解问题，可能会产生错误的答案。

RMR解决方法：使用一种memory-based的方法，使得模型可以逐渐增加相关的阅读知识，同时持续地优化预测结果。

4、目标函数

EIP框架：使用边界检测法(boundary detecting method)提取目标函数（即直接使用标准答案边界的位置和预测结果之间的交叉熵），从而训练模型。该方法对标准答案的位置进行训练，等价于直接优化EM(exact match)指标。但是该优化策略难以处理答案边界比较模糊或者答案比较长的情况。

RMR解决方法：引入新的目标函数，把最大似然交叉熵和强化学习的rewards结合起来，能够同时优化EM和F1-score这两个指标。



二、模型概述：本文提出的模型由三个部分组成——feature-rich encoder、iterative aligner、memory-based answer pointer



三、Feature-rich encoder

1、Hybrid Embedding

（1）词级别的embedding：文本和问题的词序列先映射成GloVe词向量序列（在训练过程中不再继续训练）

（2）字符级别的embedding：文本和问题中的每一个词，使用双向 LSTM处理其字符序列，把两个方向上最后时刻的hidden state连接起来作为词的字符级别embedding

（3）把每个词的词级别embedding和字符级别embedding连接起来，作为最终词的embedding

2、additional features

（1）exact matching(EM)：对于文本中的每一个词，使用一个二值特征，表示该词是否出现在问题中；同时用相同的方法处理问题中的每个词

（2）POS和NER的embedding：为POS和NER构造embedding矩阵，对于文本和问题中的每一个词，获取其对应的POS和NER的embedding

（3）问题类型编码：对语料库中出现次数最高的9类问题类型进行embedding编码，对于每一个问题，先获取其问题类型的embedding，使用前馈神经网络进行映射，把结果加入到对应的问题embedding矩阵中

3、encoding

（1）把以上词级别embedding、字符级别embedding、EM特征、POS embedding、NER embedding、问题类型编码全部连接起来，得到对应的文本特征矩阵和问题特征矩阵

（2）使用BiLSTM分别处理文本和问题，得到最中的encoder编码：文本特征矩阵C'和问题特征矩阵Q'