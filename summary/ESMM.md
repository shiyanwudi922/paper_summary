一、摘要

1、本文工作主要针对CVR建模

2、传统CVR建模中，由于使用在点击数据中训练的模型对全部的展示数据经预测，容易产生SSB（sample selection bias）问题；由于点击数据的量级通常比较效，又容易产生DS（data sparsity）问题

3、本文提出的ESMM，通过使用用户行为的序列化模式（impression -> click -> conversion），直接在全部的特征空间中进行CVR建模，并且在CTR和CVR之间使用了迁移学习，同时解决了上述的两个问题

二、传统CVR建模中的问题

1、SSB：sample selection bias

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ESMM/figure1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ESMM/SSB_understanding.jpg)

2、DS：data sparsity

点击事件的稀疏性导致了CVR训练数据的极度稀疏

三、ESMM模型

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ESMM/figure2.png)

ESMM模型使用了用户行为的序列模式

1、在完整特征空间中进行建模

（1）在完整特征空间中分别对CTR和CTCVR进行建模，同时把CVR作为其中的中间变量，从而使CVR能够直接在完整特征空间中建模，解决SSB问题

（2）将DIVISION（除法）建模方式替换为相乘，从而解决数据不稳定问题

（3）在训练时，序列标注y->z被拆分为两部分：y和y&z，从而充分利用了点击和转化之间的序列依赖关系

2、特征迁移学习

在CVR和CTR之间共享特征的embedding，通过迁移学习，使得CVR网络能够从非点击数据总进行学习，从而解决DS问题

四、实验

1、开放数据集

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ESMM/table2.jpg)

2、产品数据集

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/ESMM/figure3.jpg)