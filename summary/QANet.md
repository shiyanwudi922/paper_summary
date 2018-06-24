1、Input Embedding Layer

（1）每个词的embedding由该词的word embedding和character embedding组成

（2）word embedding：由预训练的GloVe向量组成，维度为300维，且在模型训练时保持固定，oov的词映射为<UNK>，并且使用可训练且随机初始化的embedding

（3）character embedding：每一个字符对应一个200维的向量，可训练；先把每个词填充或者截断为16个字符，然后把这16个字符的向量连接成一个200*16的矩阵，对每一行取max pooling，得到一个200维的字符级向量。

（4）将每个单词的word embedding和character embedding连接起来，在使用两层的highway network进行处理，得到每个单词最终的embedding



2、embedding encoder layer

（1）一个encoder是由三种基本的单元堆叠而成：[convolution-layer * # + self-attention-layer + feed-forward-layer]

（2）convolution-layer：使用depthwise separable convolution，优点是memory-efficient且有better generalization。kernel size是7，filter个数是128，卷积层的个数为4。输入为500维的词向量，由卷积映射为128维。这里的卷积是如何操作的没有说的很清楚，猜测是对于一个kernel size内的词，进行连接得到7*500的一维向量，然后使用kernel矩阵128 * 7 * 500维，对该一维向量转换后，得到128维的一维向量，作为卷积的输出。

（3）self-attention-layer：使用multi-head attention，head数为8

（4）在所有的子模块（convolution/attention/ffn）上都使用residual连接



3、context-query attention layer

（1）用C（n\*d）和Q（m\*d）分别表示文档和问题向量集合，先计算相似度矩阵S（n\*m），按行进行归一化得到S'(n\*m)，按列进行归一化得到S''(n\*m)，然后进行interaction，方法是先用文档表示问题，再用问题表示文档，表达式为B=S'\*S''^T^\*C

（2）向量相似度计算，使用trilinear函数f(q, c) = W~0~[q, c, q⊙c]