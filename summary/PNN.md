一、摘要

本文在embedding+MLP的基础上，提出了product layer，用于处理交叉特征，包括inner-product和outer-product两种形式。



二、网络结构

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/figure1.png)

1、总体结构

自顶向下计算

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation3.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation5.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation6.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation7.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation8.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation9.png)

该网络结构的主要亮点在于，对特征进行embedding之后，先分成两路分别计算，然后再合并。

2、z路

z向量有两种理解方式：一种是对所有特征的embedding进行连接，然后经过一个全连接网络生成lz；另一种是把z看作特征embedding形成的二维矩阵，lz中的每一个元素都是该二维矩阵和一个系数矩阵的点积，即有D1个系数矩阵。

3、p路：p路就是本文中提出的product-layer，p向量是一个N*N的矩阵，其中每一个元素都是对应两个特征embedding的积，通过取内积或者外积，有对应的两种不同的形式

（1）内积：p向量中的元素是两个特征embedding的内积

此时，如果直接先算两两的内积，得到p矩阵，再计算p矩阵和一个系数矩阵的点积，得到lp向量的一个元素，从而进一步得到lp向量，计算复杂度比较高，所以提出相应的简化算法。

假设全连接传输时，lp中每一个元素对应的系数矩阵是某个参数向量的外积

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation_w.png)

那么lp中该元素的计算方式为

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation11.png)

即，对lp中的每一个元素，设置一个N维的参数向量，将该向量作为权重，求所有特征的embedding向量的加权和，再求该和向量的模，即为该lp的元素，从而进一步得到lp向量。

通过该方法，计算复杂度大大减少。

（2）外积：p向量总的元素是两个特征embedding的外积

此时，p向量的维度变成MN\*MN，直接计算的话，复杂度太高，所以使用一种简化算法：element-wise superposition（按位叠加），即把所有的外积都按位相加，得到一个M\*M的矩阵来代替原来的MN\*MN矩阵。

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/equation14.png)

即把所有特征的embedding向量相加，得到的和向量再求外积即可。

该计算方法也可以极大的减少计算量。

4、之后，把p向量和z向量相加，再经过多层全连接网络进行传输，即为整体的PNN网络结构



三、实验

1、性能对比

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/table1.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/table2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/table3.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/figure2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/figure3.png)

2、超参数

embedding layer size：2，10， 50， 100。本文中使用10

network depth：

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/figure4.png)

activation function

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/PNN/figure5.png)



四、本文中提出的使用product-layer来提取交叉特征，以及对于不同的product形式的简化计算，思路比较新颖。

