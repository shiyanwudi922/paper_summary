一、摘要

1、在机器翻译中，一方面平行语料的获取非常困难，另一方面单语数据容易获得且有助于提升翻译的质量。

2、本文中提出一种新的方法，在MLE训练的基础上加入了联合EM训练，从而联合训练source-to-target和target-to-source模型，能够更好的利用单语数据。

3、训练使用迭代的方法，每一轮迭代中，两个模型分别用于翻译单语数据，同时用翻译概率对翻译结果进行加权，然后把加权的翻译结果作为违双语数据和原有的双语数据结合，用于训练另一个模型。通过加权，可以有效的过滤质量比较差的违双语数据。



二、训练流程

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/JointTraining/figure1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/JointTraining/algorithm1.png)

1、使用迭代的方式进行训练，每一轮迭代中，都分别对source-to-target模型和target-to-source模型进行训练。

2、初始时，先使用原有的双语数据训练初始source-to-target和target-to-source模型

3、每一轮迭代中，先使用target-to-source模型对target语料库进行翻译，提取top-k的翻译结果，并使用归一化的翻译概率进行加权，得到违双语数据；对原有的双语数据，设置权重为1；将违双语数据和原双语数据结合起来，训练source-to-target模型；同时使用相同的方法训练target-to-source模型。



三、训练目标

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/JointTraining/equation_infer.jpg)



四、实验

1、数据集：LDC，NIST OpenMT，WMT

2、Chinese-English翻译结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/JointTraining/table1.png)

3、English-to-German翻译结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/JointTraining/table2.png)

4、Effect of Joint Training

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/JointTraining/table3.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/JointTraining/figure2.png)





