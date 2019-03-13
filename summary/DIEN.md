一、动机：之前的ctr预估模型，不管是dnn类型还是rnn类型，都存在以下两个问题

1、直接使用用户行为，或者用户行为经过rnn处理后，来表示用户的兴趣。但是用户行为实际上只是用户兴趣的载体，是一种隐式的体现，所以这种表示方法不够直接有效；

2、用户的兴趣包含很多种，而且是在动态变化的，而之前的模型往往对这种用户兴趣的动态特性建模不够充分，容易受到兴趣漂移（interest drifting）的影响。

本文通过提出两个模块：Interest Extractor Layer和Interest Evolving Layer来解决这两个问题。



二、模型结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DIEN/figure1.jpg)

DIEN的整体框架是在embedding+dnn的基础上，增加了Interest Extractor Layer和Interest Evolving Layer

1、Interest Extractor Layer：一方面使用gru来对用户行为进行处理，提取用户行为之间的依赖关系；另一方面加入auxiliary loss来提升效果，auxiliary loss也是主要的创新点。

(1)、使用auxiliary loss的原因：
a、gru的hidden state只能获取用户历史行为之间的依赖关系，而用户的行为仅仅是用户兴趣的一种隐式的体现，所以只获取了用户行为之间依赖关系的hidden state所包含的直接和用于兴趣相关的信息并不多，难以直接有效地用于表示用户的兴趣；
b、用户最终的点击行为是由历史行为中最后一个时刻的兴趣决定的，那么以最终点击行为作为label只能对历史行为的最后一个时刻的兴趣进行有效的监督学习，而之前时刻的兴趣无法得到有效的学习。

(2)、使用auxiliary loss的优点：
a、对hidden state使用后一个时刻的点击行为进行监督，而后一时刻的点击行为直接来源于当前的兴趣，所以会使当前的hidden state中包含和用户兴趣更直接相关的信息；
b、每一时刻的hidden state都用后一时刻的点击行为进行监督，从而所有时刻的hidden state都能进行有效的监督学习；
c、减少了gru在对长序列建模时，反向传播训练的难度；
d、使embedding向量包含更多的语义信息。

2、Interest Evolving Layer：进一步使用gru对用户兴趣的演进过程进行建模，同时使用attention对gru中的update gate进行加权，提升效果，AUGRU的使用是主要的创新点。

在兴趣建模中，attention和gru的结合有三种主要的方式：
（1）、AIGRU -- attention score作用在gru的输入上，直接相乘。问题：对于不相关的兴趣，理论上可以将其对应的attention score将为0，但是就算这样，仍然会改变gru的hidden state，从而对兴趣演进过程产生影响；
（2）、AGRU -- attention score作用在gru的update gate上，直接替换。问题：虽然可以避免1中出现的问题，但是由于直接使用标量替换了向量，失去了update gate中的维度信息；
（3）、AUGRU -- attention score作用在gru的update gate上，直接相乘。既避免了不相关兴趣对hidden state的改变，降低了interest drifting的影响，又保留了update gate的维度信息，使得相关兴趣的演进更加平滑。

对于某一个candidate item，AUGRU可有有效地增强用户行为序列中相关兴趣的影响，降低无关兴趣的效果。



三、实验：数据集包括两个公共数据集和一个工业数据集，对比算法包括BaseModel、W&D、PNN、DIN、使用attention的两层gru。

1、和其他算法对比，说明整体模型的效果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DIEN/table2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DIEN/table3.png)

2、和不同结构的模型对比，说明auxiliary loss和AUGRU的效果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DIEN/figure2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DIEN/table4.png)

3、对AUGRU的hidden state进行可视化，说明对用户兴趣演进建模的效果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DIEN/figure3.png)

4、展示线上a/b test的对比结果，说明线上效果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DIEN/table5.png)

线上效果表现真的非常强悍……