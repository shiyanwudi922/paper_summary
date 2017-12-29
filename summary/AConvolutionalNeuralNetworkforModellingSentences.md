![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AConvolutionalNeuralNetworkforModellingSentences/figure3.png)

一、摘要

1、使用了跨度为m的一维宽卷积，可以提取长度小于等于m的n-gram特征

2、使用动态k-max pooling

（1）定义：一方面对于线性序列，提取其最大的k个值组成的子序列，而不是仅仅返回最大值；另一方面，k的取值可以定义为其他参数的函数，进行动态选取。

（2）优点：对卷积生成的n-gram特征进行选择和组合，其特征范围是全局的（global），不仅考虑了特征是否出现，而且维持了其出现的顺序和相对位置，通过改变分辨率忽略了其绝对位置，通过动态地选择k能够更加平滑地提取高阶和长范围的特征。

3、通过卷积和k-max pooling的结合，对于每一个句子都引入了特征图（feature graph），可以显示地提取短/长范围的关系。通过feature graph，可以说明在高层上的小的filter可以获取输入句子中距离比较远的非连续短语之间的语法和语义联系。

二、简介

1、常用的神经语句模型

（1）NBoW（neural bag-of-words）

做法：将句子中的每一个词映射成向量，再进行合成，得到句子的向量表示，最后通过全连接层对该特征进行分类。

（2）RecNN（recursive neural network）

做法：使用了解析树，对于树中的每一个节点，其左右孩子所代表的的上下文通过一个层进行联合，该层的权重在树中的所有节点之间共享，在顶层节点上计算的层提供了句子的表示。

特例：RNN（recurrent neural network），一般用于语言模型，最后一个词对应的网络状态可以作为句子的表示

（3）卷积神经网络

做法：将句子中的词映射为向量，组成语句矩阵，通过一维卷积和k-max pooling进行特征提取

2、max-tdnn：窄卷积 + max pooling+全连接层

（1）优点：

（a）对句子中词的顺序敏感

（b）不依赖于外部的语言特征

（c）除了在窄卷积中考虑比较少的边缘词以外，该模型对于句子中的词所产生的信号赋予了相同的重要性

（2）缺点

（a）使用窄卷积，使得特征提取器的跨度限制为卷积核的宽度m：提升m或者增加卷积层可以提高该跨度，但是会进一步忽略边缘词，并且提高对输入句子长度的要求

（b）使用max pooling，仅提取最大值，如能区分相关的特征是仅出现一次还是多次，并且忽略了特征的顺序和相对位置

三、模型组成

1、宽卷积

2、k-max pooling

3、dynamic k-max pooling

4、非线性函数

5、多feature map

6、折叠

四、语句模型的性质

1、n-gram的顺序

句子模型的一个基本性质是对句子中出现的词的顺序具有敏感性，包括：

（1）当一个相关的n-gram出现时能够有效地识别出来

（2）除了识别以外，还要考虑该n-gram出现的次数、顺序和相对位置

本文中提出的网络通过以下方法实现这两个要求：

（1）使用跨度为m的一维宽卷积，可以发现长度小于等于m的n-gram

（2）使用k-max pooling，对卷积层提取的n-gram特征进行选择和组合，从而考虑了n-gram特征的次数、顺序和相对位置，并且对n-gram具有绝对位置不变性

2、特征图（induced feature graph）

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/AConvolutionalNeuralNetworkforModellingSentences/figure3.png)

（1）由卷积层和pooling层引出

（2）有向无环图

（3）去除折叠层后，语句矩阵的每一行都会引入一个子图，并在根节点上进行结合

（4）每一个子图的结构都不相同，反映了该子图识别的特征关系

（5）折叠层的效果实在根节点之前把低层的子图进行结合

（6）DCNN的特征图比较特殊的原因是其pooling操作的范围是全局的，使得高阶特征在输入中对应的词的范围变化非常大，具体的变化范围由对应的子图反应出来