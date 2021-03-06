一、NMT的三个天生的缺点

1、训练和推断的速度非常慢（训练：数据并行和模型并行；推断：量化推断）

2、难以有效地处理稀有词（分段方法）

3、有时会难以对输入句子的所有的词都进行合适地翻译，即难以完整地“覆盖”输入（解码时加入length normalization和coverage penalty）



二、模型架构：encoder、decoder、attention

1、总体架构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Google'sNeuralMachineTranslationSystem--BridgingTheGapBetweenHumanAndMachineTranslation/figure1.png)

2、RNN的残差链接（Residual Connections）

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Google'sNeuralMachineTranslationSystem--BridgingTheGapBetweenHumanAndMachineTranslation/figure2.png)

该残差链接用在encoder和decoder的第三层以上的网络，大大改善了反向过程中的梯度传播过程，使得可以训练更深的encoder和decoder网络。

3、encoder第一层使用双向rnn

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Google'sNeuralMachineTranslationSystem--BridgingTheGapBetweenHumanAndMachineTranslation/figure3.png)

4、模型并行化

（1）数据并行：同时训练n个模型的副本，在n个副本之间进行参数共享，每个副本对参数进行异步更新；

（2）模型并行：将网络的不同层跑在不同的gpu上；对softmax层也进行分解，使不同的输出符号子集跑在不同的gpu上；

（3）限制：模型并行化对模型架构带来了一些限制，例如：不能在所有的encoder层使用双向网络；不能用decoder的顶层输出进行attention计算，只能用最底层。



三、分段方法：在对oov的词进行翻译时，通常有两种方法，一是copy，基于外部对齐模型的attention，或者负载的特定目标的指向网络；二是子词单元（sub-word units），使用字符、混合词/字符，或者其他子词。

1、本文中最成功的方法是使用子词单元，以数据驱动的方式建立子词模型

2、针对在翻译中需要把某些稀有词直接从输入copy到输出的情况，本文中在源语言和目标语言上使用共享的子词模型来解决。保证了在源语言和目标语言中，相同的字符串以相同的方式被分段，更容易进行copy。

3、子词模型（wordpiece）在字符模型的灵活性和单词模型的有效性之间取得了一个折衷，能够更有效的处理“无限字典”的问题，从而获得更高的BLEU。

4、另外一个解决oov问题的办法是使用混合单词/字符模型：首先像传统模型一样维护一个词典，然后在碰到oov词的时候，将其分解成对应的字符进行处理。



四、训练指标

1、最大似然目标函数

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Google'sNeuralMachineTranslationSystem--BridgingTheGapBetweenHumanAndMachineTranslation/equation7.png)

只使用最大似然目标函数，无法反应任务奖励函数（task reward function），会导致模型对于解码过程中出现的错误不具有鲁棒性，因为这些错误在训练过程中没有出现过。

2、本文中，在使用最大似然预训练的模型的基础上，使用task reward进行微调，可以进一步提升性能。

3、强化目标函数

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Google'sNeuralMachineTranslationSystem--BridgingTheGapBetweenHumanAndMachineTranslation/equation8.png)

在测试时，在单句上直接使用BLEU分数会带来一些不必要的结果（因为BLEU是针对语料库提出的指标），所以本文进行了一些修改，提出了GLEU指标（召回率和准确率的最小值），在语料库级别上具有和BLEU类似的性能，而且在单句上不具有BLEU的缺点。

4、混合目标函数

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Google'sNeuralMachineTranslationSystem--BridgingTheGapBetweenHumanAndMachineTranslation/equation9.png)

5、通常会先用最大似然目标函数进行训练，直到收敛；再进一步使用混合目标函数进行微调，直到验证集上的BLEU分数不再变化。



五、量化模型和量化推断：推断时的计算密集性阻碍了模型在生产环境中的大规模部署，所以本文提出了使用低精度算法的量化推断。

1、训练阶段

（1）变量剪切：对lstm中的accumulator(cell state和x)的值剪切到[-delta, delta]，对softmax的logits的值剪切到[-gamma, gamma]，并使用高精度的整数表示。

（2）浮点操作改为不同精度的整数操作：在训练阶段，不会对浮点操作修改为整数操作。

2、decoding阶段

（1）变量剪切：对lstm中的accumulator(cell state和x)的值剪切到[-delta, delta]，对softmax的logits的值剪切到[-gamma, gamma]，并使用高精度的整数表示。

（2）浮点操作改为不同精度的整数操作：对lstm中的浮点矩阵相乘，修改为一定精度的整数操作；对softmax中的浮点矩阵相乘，也修改为一定精度的整数操作

3、这个方法在效率和准确率之间达到了一个很好的平衡：计算代价比较高的操作（矩阵相乘）变成了整数操作，提高了效率；对误差敏感的accumulator的值使用高精度的整数表示，准确率非常高且对量化误差具有鲁棒性。

4、实验证明本文方法的准确率和效率：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Google'sNeuralMachineTranslationSystem--BridgingTheGapBetweenHumanAndMachineTranslation/table1.png)



六、解码

1、在原始的beam search上增加了两个改进

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/Google'sNeuralMachineTranslationSystem--BridgingTheGapBetweenHumanAndMachineTranslation/equation14.png)

（1）length normalization：在原始beam search中，decoding的每一步，都会在序列的对数概率上加一个负数，从而导致模型倾向于选择长度较短的输出序列，所以需要加入length normalization来抑制这种情况；

（2）coverage penalty：防止模型出现“不能对输入序列的每一个词都进行翻译”的情况；

2、为了提高decoding效率，进行了剪枝

（1）减少beamsize的值；

（2）在decoding的每一步，生成新的token时，只保留分数最高的beamsize个token；

（3）仍然在decoding的每一步，只保留分数最高的beamsize个序列，其余的序列从候补集中删除；

3、length normalization和coverage penalty对于只使用ML目标函数训练的模型比较有效；而对先使用ML预训练，再使用RL进行微调的模型不是特别有效，主要是由于在RL微调的过程中，模型已经注意了完整的源句子，不会产生欠翻译或过翻译的问题（under-translate, over-translate）。