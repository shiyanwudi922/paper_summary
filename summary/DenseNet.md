一、DenseNet的主要优点

1、比传统的卷积网络的参数更少，参数利用率更高（parameter efficiency），因为不需要去重复学习冗余的featuremap：传统的卷积网络中，每一层接收前一层的输出，在保留必要信息的基础上，增加新的信息，然后传输到下一层，所以每一层需要对之前的信息进行一定程度的保持，相当于学习了很多冗余信息；而DenseNet在每一层显式地区分了需要保留的信息和需要新加入的信息，不需要对信息进行重复学习。

2、网络中信息和梯度的流动方式得到了改进，增强了信息的传播，使得网络更容易训练。同时每一层都可以直接连接到损失函数的梯度和原始的输入信号，从而起到了隐式的深层监督效果（implicit deep superision）。另外，密集连接也在一定程度上起到了正则化的效果。

3、缓解了gradient vanishing的问题

4、增强了特征复用（feature reuse）



二、DenseNet

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/figure2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/table1.png)

1、ResNet

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/equation1.png)

ResNet的优点是梯度可以直接通过恒等链接从后面的网络层传播到前面的网络层

2、Dense connectivity

将网络中的每一层直接连接到之后的每一层

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/equation2.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/figure1.png)

3、Composite function

每一个卷积层的处理是一个复合函数Hl(x)，包含：BN、ReLU、Conv

4、Pooling layer

（1）在equation_2中，要求不同层输出的feature-map的size相同，才可以进行连接。但是，在卷积网络中，一个重要的操作是down-sampling，该操作是会改变feature-map的size的。

（2）所以，为了使用down-sampling，本文把整个网络分割成多个密集连接的块（dense block）。在相邻的dense block之间的网络层成为过渡层(transition layer)。

（3）在dense block内，层与层之间是密集连接的；dense block之间通过transition layer连接；transition layer进行1✖️1卷积和2✖️2pooling。

5、Growth rate

（1）在DenseNet中，使每一个卷积层输出相同数量的feature-map，设为k，即为网络的growth rate。第l层的输入feature-map数为k0+k(l-1)

（2）由于DenseNet通过增加shotcut连接，显示地区分了每一个网络层需要新加入的信息和需要保留的信息，每一层只需要负责增加新的信息，所以可以使用非常窄的网络层，即k的取值可以相对较小

（3）可以把某一层之前所有层产生的feature-map作为网络的全局状态（global state of the network），那么growth rate也就规定了每一层可以像全局状态中新增多少信息

6、Bottleneck layers

（1）虽然每一个网络层只产生k个feature-map，但是当网络层比较多时，密集连接会使得某些层的输入feature-map非常多，从而导致参数量过多

（2）借鉴ResNet中bottleneck的思想，在Hl(x)中增加bottleneck层，先使用1✖️1的卷积进行降维，在用3✖️3的卷积进行特征提取

7、Compression

为了进一步提高模型的紧凑性，本文在transition layer中进一步减少feature-map的数量，即对于m个输入feature-map，transition layer输出floor(θm)个feature-map



三、实验

1、在CIFAR和SVHN上的分类结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/table2.png)

（1）除了准确率更高之外，DenseNet的参数效率更高(和1001层的pre-activation ResNet相比，准确率相差无几，但是只使用了10%的参数量)。

（2）非常高的参数利用率带来的另一个效果是DenseNet更不容易过拟合

2、在ImageNet上的分类结果

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/table3.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/figure3.png)



四、讨论

1、Model compactness

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/figure3.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/figure4.png)

上图说明了DenseNet可以使用更少的参数达到相同的性能，所以参数的效率更高

2、Implicit Deep Supervision

DenseNet中的每一层都通过shorter connection从损失函数中接受到了额外的监督，即DenseNet具有deep supervision的效果，所以可以有效地提高网络的准确率

3、DenseNet和stochastic depth的关系

在stochastic depth中，ResNet中的网络层可以随机丢弃，从而在周围的网络层之间产生直接连接，所以stochastic depth产生了和DenseNet非常相似的连接形式：在任意两个pooling layer之间，任意两层都以一定的概率被直接连接。

4、Feature Reuse

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DenseNet/figure5.png)

（1）在同一个dense block中，前面的网络层产生的特征可以被后面的层直接使用

（2）一个dense block中的所有的网络层所产生的特征都会被其之后的transition layer直接使用，也说明了信息在DenseNet中可以更直接的从第一层传输到最后一层

（3）实验证明，transition layer更容易产生冗余特征，从而证明了在transition layer进行压缩的正确性

（4）在网络的后面几层仍然会产生一些更高阶的特征