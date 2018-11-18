一、BERT的关键点

1、使用了比之前其他模型更多的数据：BooksCorpus(800M words)和English Wikipedia(2500M words)。

2、使用两种新的预训练任务：Masked Language Model（负责使模型具有双向性，也是最重要的任务），以及Next Sentence Prediction（用于语句关系推理）

3、在输入中使用三种Embedding：WordPiece Embedding、Segment Embedding、Position Embedding

4、模型使用双向Transformer（或者称为Transformer Encoder，单向的Transformer称为Transformer Decoder）：单向或者双向是由传输过程中每个词使用的是其单个方向的上下文还是两边全部的上下文，由self-attention中的mask来控制



二、简介

1、语言模型预训练对于改进sentence-level和token-level的nlp任务都是很有效的

2、预训练的语言表示有两种使用方法：feature-based（ELMo），把预训练的语言表示作为下游任务的特征；fine-tuning（GPT），预训练任务和下游任务的模型基本相同，先通过预训练来学习通用的参数，然后在训练下游任务时添加少量的特定参数，进行微调即可

3、目前的框架严重的限制了预训练语言表示的能力（尤其是fine-tuning方法），主要的额约束是由于标准的语言模型都是单向的，从而限制了预训练过程中可以选择的框架

4、本文中对fine-tuning方法进行了改进，提出了BERT预训练模型，通过引入MLM预训练任务来解决之前模型的单向问题

5、本文的主要贡献为：

（1）验证了双向预训练对于语言表示的重要性

（2）预训练表示使得我们不需要再花非常大的精力来设计复杂的任务相关的模型

（3）BERT的双向特性是其取得成功的最重要的特性



三、相关工作

1、feature-based 方法

在feature-based方法中，预训练的word/sentence/paragraph embedding都会被当作下游NLP系统的组成部分，一般作为下游模型的特征；

ELMo从不同的维度对word embedding进行了扩展，提出从语言模型中提取上下文相关的特征。

2、fine-tuning方法

在fine-tuning方法中，预训练和微调的模型是几乎相同的，先使用预训练来学习大量通用的参数，然后再微调任务相关的少量参数；该方法的优点是在微调过程中只需要从头学习很少的参数。

OpenAI GPT使用该方法在很多GLUE任务上都取得了SOTA的结果

3、使用有监督数据进行迁移学习

除了使用无监督预训练，在NLI和MT等任务中，有监督的预训练也会起到有效的作用



四、BERT介绍

1、模型框架

BERT中使用多层双向Transformer Encoder作为主要结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/transformer_encoder.png)

本文中使用两种size的BERT：

BERT~BASE~：L=12, H=768, A=12, Total Parameters=110M

BERT~LARGET~：L=24, H=1024, A=16, Total Parameters=340M

其中，L表示层数，H表示hidden size，A表示self-attention中head的数量

通常在文献中，双向Transformer被称为Transformer Encoder，单向Transformer被称为Transformer Decoder.

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/figure1.png)

2、输入表示

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/figure2.png)

对于输入的每一个词，通过对其三种embedding求和得到词表示

（1）WordPiece Embedding

（2）Segment Embedding：每一条输入序列包含两个语句，BERT中一方面使用分隔符[SEP]进行分割，另一方面使用两个不同的embedding来对这两个句子进行区分

（3）Positional Embedding

另外，输入序列的头部插入一个分类token([CLS])，作为一个特殊的分类embedding，其对应的最后一层的隐状态作为整个序列的表示被用于分类任务中。

3、预训练任务

（1）Masked Language Model

在改预训练任务中，会在输入序列中随机掩盖一些词，然后使用这些词的最后的隐向量来预测这些词，本文中掩盖概率为15%。

MLM虽然可以取得双向的效果，但是有两个问题：

一是由于[MASK]字符在之后的fine-tuning中不会出现，所以会在pre-training和fine-tuning之间造成一定的不匹配；该问题通过以下做法来缓解：在选取15%的掩盖词之后，以80%的概率替换为[MASK]，以10%的概率替换为随机词，以10%的概率不变

二是由于只会预测15%的词，所以需要比传统语言模型更多的迭代次数，但是实践说明该方法在效果上的提升是值得多花一些训练代价的。

（2）Next Sentence Prediction

为了使模型能够理解语句关系，BERT中加入了一个额外的预训练任务，在每一个输入序列中包含两个语句，A和B，采样时以B以50%的概率取A之后的语句，50%的概率随机取语句进行连接。序列头部的[CLS]对应的label表示是否B是否是A的下一句。

4、预训练过程

（1）BERT使用BooksCorpus（800M words）和English Wikipedia（2500M words）进行预训练

（2）在生成输入序列时，选取两个语句A和B，B以50%的概率选择A之后的句子，使用[CLS]和[SEP]进行连接；进行WordPiece分词，然后再随机选择15%的词进行mask。

（3）训练中的loss是平均MLM似然和平均NSP似然的和

5、微调过程

根据不同的下游任务类型，对预训练模型进行轻微调整，然后使用对应的下游任务数据进行训练即可



五、实验

1、GLUE数据集：本文另外一个关键点是引入了GLUE中的多个任务进行实验

（1）在预训练模型结构的基础上，不同任务的修改方式

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/figure3.png)

（2）GLUE结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/table1.png)

2、SQuAD结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/table2.png)

3、NER结果：CoNLL-2003

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/table3.png)

4、SWAG数据集

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/table4.png)



六、Ablation

1、预训练任务的效果对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/table5.png)

由MLM引入的双向性是BERT最重要的改进

2、模型大小的效果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/table6.png)

本文是第一次验证了即使对于规模非常小的任务，使用非常大的模型也是可以带来很大提升的，前提是需要有充分的预训练。

3、训练迭代次数的影响

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/figure4.png)

BERT需要大量的预训练从而使微调能够达到很好的效果；

MLM模型比LTR模型收敛得更慢，但是其准确率从一开始就比LRT模型要高

4、BERT用在feature-based方法中

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/BERT/table7.png)

BERT对于feature-based方法也是很有效的