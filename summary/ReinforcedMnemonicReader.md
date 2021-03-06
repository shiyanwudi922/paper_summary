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

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/figure1.png)



三、Feature-rich encoder

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/figure_encoder.png)

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



四、Iterative Aligner：由多个可重复的计算单元顺序连接组成。其中每一个计算单元都对文本中的每一个词的表示进行更新，后一个计算单元以前一个计算单元的更新后的文本词表示和encoder输出的问题词表示作为输入，以更新后的文本词表示作为输出。具体的结构和流程如下。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/figure_iterator.png)

1、SFU(Semantic Fusion Unit)

（1）SFU接收一个输入向量r和一个fusion vectors集合{f1, f2, … , fk}，然手生成输出向量o

（2）具体的处理方式类似于highway network：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/figure_sfu.jpg)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/equation4.png)

2、Interactive Aligning：接收前一个计算单元的文本词向量和encoder输出的问题词向量，输出查询相关的文本词表示(query-aware context representation)

（1）计算coattention matrix，实际上就是问题词向量和文本词向量的点积矩阵：
$$
B^t = Q'^TC' ∈ R^{n×m}, 其中B^t_{ij} = q^T_i ·  cˇ^{t-1}_j
$$
（2）计算attended query vectors，实际上就是用问题词向量表示文本中的每个词：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/equation2.png)

即先对coattention matrix的每一列进行softmax得到权重向量，再计算和问题词向量的加权和，从而得到使用问题词向量对每一个文本词的表示

（3）针对每一个文本词，使用SFU得到其对应的query-aware representation：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/equation3.png)

即针对文本中的每一个词，分别提取上一步计算的attended query vector和前一个计算单元输出的表示向量，做一些处理后，输入到SFU单元中，得到该词的进一步的表示向量。

3、Self Aligning：接收Interactive Aligning的输出作为输入，输出自相关的文本词表示(self-aware context representation)，整体的计算方式和Interactive Aligning相同。

（1）计算self-coattention matrix，即计算Interactive Aligning输出的文本表示矩阵的自身的点积矩阵，注意为了避免词向量的自我对齐，需要把该矩阵的对角线元素设置为0

（2）计算attended context vectors，即使用文本词向量做自我表示

（3）针对每一个文本词，使用SFU得到其对应的self-aware representation，只不过把输入修改为query-aware representation和attended context vector。

4、Aggregating：最后，使用BiLSTM对self-aware representation进行处理，得到最终的fully-aware context representation，可以输入到下一个计算单元，或者输入到Pointer中。



五、Memory-based Answer Pointer：由多个计算单元顺序连接而成，通过维护一个记忆向量，逐渐优化最终的结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/fiture_answer_pointer.png)

1、初始化记忆向量，一般初始化为对于问题的概括向量，本文中，初始化为encoder中最后一个词的hidden state

2、计算起始词的概率分布：通过记忆向量和fully-aware context repersentation来合成attention向量，作为起始词的概率分布

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/equation7.png)

3、更新记忆向量：使用SFU，把当前的记忆向量和文本向量在起始词概率分布上的加权和作为输入，输出新的记忆向量

4、计算终止词的概率分布：通过记忆向量和fully-aware context repersentation来合成attention向量，作为终止词的概率分布

5、如果当前计算单元为最后一个，则结束，当前计算单元的起始词和终止词的概率分布作为最终的输出；否则，再次更新记忆向量，使用SFU，把当前的记忆向量和文本向量在起始词概率分布上的加权和作为输入，输出新的记忆向量，该记忆向量作为下一个计算单元的输入。



六、训练目标函数

1、基于边界检测的有监督学习（最大似然目标函数）：基于真实的样本答案计算负对数似然损失函数

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/equation9.png)

该损失函数等价于直接对EM指标进行优化，但是当答案边界比较模糊或者答案很长时，不太适用

2、基于强化学习的目标函数：为了解决最大似然目标函数的问题，本文在强化学习的基础上，加入了以F1-score作为reward的目标函数

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/equation10.png)

即对于每一个样本，计算其预测答案和真实答案之间的F1-score，再计算所有样本上的聚合值

3、为了使训练更稳定，防止模型在训练后期覆盖掉前期的训练效果，本文把两种损失函数结合起来作为最终的损失函数

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/equation11.png)

该损失函数等价于同时优化EM和F1-score这两个指标。



七、实验分析

1、在TriviaQA数据集上的实验结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/table2.png)

2、在SQuAD数据集上的实验结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/table3.png)

3、Ablation的结果

（1）模型中不同组成部分的重要性试验

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/table4.png)

（2）Aligner和Answer Pointer的跳数(hop)的试验（当调整其中一个跳数时，另一个跳数默认为2）

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/table5.png)

4、可视化：验证了该模型可以通过把备选答案信息融入到memory向两种，从而逐渐定位到正确答案

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ReinforcedMnemonicReader/figure2.png)

