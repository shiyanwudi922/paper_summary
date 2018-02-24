四、Iterative Aligner：由多个可重复的计算单元顺序连接组成。其中每一个计算单元都对文本中的每一个词的表示进行更新，后一个计算单元以前一个计算单元的更新后的文本词表示和encoder输出的问题词表示作为输入，以更新后的文本词表示作为输出。具体的结构和流程如下。

1、SFU(Semantic Fusion Unit)

（1）SFU接收一个输入向量r和一个fusion vectors集合{f1, f2, … , fk}，然手生成输出向量o

（2）具体的处理方式类似于highway network：

figure_sfu

equation4

2、Interactive Aligning：接收前一个计算单元的文本词向量和encoder输出的问题词向量，输出查询相关的文本词表示(query-aware context representation)

（1）计算coattention matrix，实际上就是问题词向量和文本词向量的点积矩阵：
$$
B^t = Q'^TC' ∈ R^{n×m}, 其中B^t_{ij} = q^T_i ·  cˇ^{t-1}_j
$$
（2）计算attended query vectors，实际上就是用问题词向量表示文本中的每个词：

equation2

即先对coattention matrix的每一列进行softmax得到权重向量，再计算和问题词向量的加权和，从而得到使用问题词向量对每一个文本词的表示

（3）针对每一个文本词，使用SFU得到其对应的query-aware representation：

equation3

即针对文本中的每一个词，分别提取上一步计算的attended query vector和前一个计算单元输出的表示向量，做一些处理后，输入到SFU单元中，得到该词的进一步的表示向量。

3、Self Aligning：接收Interactive Aligning的输出作为输入，输出自相关的文本词表示(self-aware context representation)，整体的计算方式和Interactive Aligning相同。

（1）计算self-coattention matrix，即计算Interactive Aligning输出的文本表示矩阵的自身的点积矩阵，注意为了避免词向量的自我对齐，需要把该矩阵的对角线元素设置为0

（2）计算attended context vectors，即使用文本词向量做自我表示

（3）针对每一个文本词，使用SFU得到其对应的self-aware representation，只不过把输入修改为query-aware representation和attended context vector。

4、Aggregating：最后，使用BiLSTM对self-aware representation进行处理，得到最终的fully-aware context representation，可以输入到下一个计算单元，或者输入到Pointer中。



五、Memory-based Answer Pointer：由多个计算单元顺序连接而成，通过维护一个记忆向量，逐渐优化最终的结果

1、初始化记忆向量，一般初始化为对于问题的概括向量，本文中，初始化为encoder中最后一个词的hidden state

2、计算起始词的概率分布：通过记忆向量和fully-aware context repersentation来合成attention向量，作为起始词的概率分布

equation7

3、更新记忆向量：使用SFU，把当前的记忆向量和文本向量在起始词概率分布上的加权和作为输入，输出新的记忆向量

4、计算终止词的概率分布：通过记忆向量和fully-aware context repersentation来合成attention向量，作为终止词的概率分布

5、如果当前计算单元为最后一个，则结束，当前计算单元的起始词和终止词的概率分布作为最终的输出；否则，再次更新记忆向量，使用SFU，把当前的记忆向量和文本向量在起始词概率分布上的加权和作为输入，输出新的记忆向量，该记忆向量作为下一个计算单元的输入。



六、训练目标函数

1、基于边界检测的有监督学习（最大似然目标函数）：基于真实的样本答案计算负对数似然损失函数

equation9

该损失函数等价于直接对EM指标进行优化，但是当答案边界比较模糊或者答案很长时，不太适用

2、基于强化学习的目标函数：为了解决最大似然目标函数的问题，本文在强化学习的基础上，加入了以F1-score作为reward的目标函数

equation10

即对于每一个样本，计算其预测答案和真实答案之间的F1-score，再计算所有样本上的聚合值

3、为了使训练更稳定，防止模型在训练后期覆盖掉前期的训练效果，本文把两种损失函数结合起来作为最终的损失函数

equation11

该损失函数等价于同时优化EM和F1-score这两个指标。

