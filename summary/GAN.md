![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/GAN/GAN_arch.jpeg)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/GAN/GAN_V.png)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/GAN/GAN_D.jpeg)

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/GAN/GAN_G.jpeg)

三、优化算法

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/GAN/GAN_Algo.png)

四、优化算法的收敛性证明

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/GAN/GAN_Converge.png)

1、重新关注V(G,D)的示意图

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/GAN/GAN_V.png)

2、在优化算法中的每一轮迭代中，分两步进行，第一步会在给定G的条件下，对D进行优化。

这一步事实上是在给定G的条件下，根据D来求得V(G,D)的上确界V(G,D*)，如下图黑线所示

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/GAN/GAN_Supp.jpeg)

3、第二步是在固定D*的条件下，根据G对V(G,D)进行一次梯度下降优化，即对上确界进行一次优化

4、已知定理：当取得最优值时，凸函数上确界的次微分包含该凸函数的次微分。

所以根据G对V(G,D*)进行了梯度下降优化的同时，也相当于对V(G,D)进行了梯度下降优化

5、V(G,D*)是G的凸函数，且有唯一的全局最优解，所以只要G的更新步长足够小，则最终一定能收敛到P~data~



参考：

1、<https://medium.com/@samramasinghe/generative-adversarial-networks-a-theoretical-walk-through-5889d5a8f2bb>

2、<https://zhuanlan.zhihu.com/p/29837245>

3、<https://zhuanlan.zhihu.com/p/27012520>