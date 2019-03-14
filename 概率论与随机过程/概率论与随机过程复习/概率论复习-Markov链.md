# 概率论复习-Markov链

> Markov过程按状态空间$I$和参数集$T$进行分类。
>
> $I和T均离散，即马尔科夫链$
>
> $I离散、但T连续，即为纯不连续的马尔科夫过程（间断性Markov过程）$
>
> $I和T均连续型Markov过程$

## Markov链

对状态空间为$I$的随机过程，$i_0,i_1,...i_{n+1}\in I$，参数集为T，$\left\{ \xi _ { n } , n \in T \right\} $，有：
$$
P \left\{ \xi _ { n + 1 } = i _ { n + 1 } | \xi _ { 0 } = i _ { 0 } , \cdots \xi _ { n } = i _ { n } \right\} = P \left\{ \xi _ { n + 1 } = i _ { n + 1 } | \xi _ { n } = i _ { n } \right\}
$$
则称$\left\{ \xi _ { n } \right\} $为Markov链，上式表达的性质称为Markov性。

## 转移概率&随机矩阵

称条件概率$p _ { i j } ( n ) = P \left\{ \xi _ { n + 1 } = j | \xi _ { n } = i \right\} $为Markov链$\left\{ \xi _ { n } , n \in T \right\} $的一步转移概率，其中$i , j \in I $

> 下一个转移状态不仅与上一个时刻的状态有关，且与当前的参数n有关，但如果$p_{ij}(n)$不依赖于n，则Markov链具有平稳的转移概率，即转移概率具有平稳性（齐次Markov链）

### 一步转移概率矩阵（随机矩阵）

$$
P = \left[ \begin{array} { c c c c } { p_{11} } & { ... } & { p_{1n} } & { ... } \\ { p_{21} } & { ... } & { p_{2n} } & { ... } \\ { ... } & { ... } & { ... } & { ... } \end{array} \right]
$$

性质：

1. $p _ { i j } \geq 0 ( i , j \in I ) $
2. $\sum _ { j \in I } p _ { i j } = 1 ( i \in I ) $（从状态i出发转移到系统各个状态的概率之和为1）

### n步转移概率：

称条件概率$ p _ { i j } ^ { ( n ) } = P \left\{ \xi _ { m + n } = j | \xi _ { m } = i \right\} ( i , j \in I , n \geq 1 ) $为Markov链$ \left\{ \xi _ { n } , n \in T \right\}$的n步转移概率，并称$ \boldsymbol { P } ^ { ( n ) } = \left( p _ { i j } ^ { ( n ) } \right)$为Markov链的n步转移概率矩阵，其中：

1. $ p _ { i j } ^ { ( n ) } \geq 0$
2. $ \sum _ { j \in I } p _ { i j } ^ { ( n ) } = 1 $

$\boldsymbol { P } ^ { ( n ) }$为随机矩阵，当n=1时，$ \boldsymbol { P } ^ { ( 1 ) } = \boldsymbol { P }$  ,$p _ { i j } ^ { ( 0 ) } = \left\{ \begin{array} { l } { 1 i = j } \\ { 0 i \neq j } \end{array} \right.$

### C-K方程：

![image-20190104210630864](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104210630864.png)

> 意思就是n步转移概率可以通过所有的单步转移概率路径相加得到
>
> **n步转移概率矩阵是一步转移概率矩阵的n次幂**

### 初始概率概率&绝对概率

![image-20190104213015822](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104213015822.png)

### Markov链的性质：

![image-20190104213142363](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104213142363.png)

![image-20190104213318977](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104213318977.png)

![image-20190104213329215](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104213329215.png)

## Markov链的状态分类(互通性)

![image-20190104214239536](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104214239536.png)

### 定理：

![image-20190104215120460](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104215120460.png)

## 状态的分类

![image-20190104220923104](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104220923104.png)

> 其中G.C.D的意思是最大公约数

### 引理：

![image-20190104221117291](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104221117291.png)

<font color=#FF0033>重要例题：</font>

![image-20190104221755858](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104221755858.png)

设$ I = \{1,2,3,4\} $，转移概率如图，显然状态2和状态3有相同的周期d=2，但是从状态3出发经过两步必然返回到3，而状态2则不然。

#### <font color=#FF0033>重要定义</font>

![image-20190104223202432](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104223202432.png)

### 常返&非常返

![image-20190104223745091](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190104223745091.png)



### 母函数：

![image-20190105100522361](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190105100522361.png)

### 常返与非常返n步返回的概率和：

1. $状态i常返\Leftrightarrow \sum _ { n = 0 } ^ { \infty } p _ { i i } ^ { ( n ) } = \infty$

   > 常返的每次概率都为1($p_{ii}=1$)

2. $状态i非常返，\sum _ { n = 0 } ^ { \infty } p _ { i i } ^ { ( n ) } = \frac { 1 } { 1 - f _ { i i } }$

   > 能返回的概率和，因为只返回有限次。

### 返态的极限返回概率：

![image-20190105103217810](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190105103217810.png)

> 若j是零常返或非常返态，则$ \lim _ { n \rightarrow \infty } p _ { i j } ^ { ( n ) } = 0$

### <font color=#FF0033>证明/判别->常返性：</font>

1. 状态j常返$\Leftrightarrow f _ { j j } = 1 \Leftrightarrow \sum _ { n = 0 } ^ { \infty } f _ { j j } ^ { ( n ) } = 1 \Leftrightarrow \sum _ { n = 0 } ^ { \infty } p _ { j j } ^ { ( n ) } = \infty$
2. 状态j正常返$ \Leftrightarrow f _ { j j } = 1 , \mu _ { j } < \infty \Leftrightarrow \sum _ { n = 0 } ^ { \infty } p _ { jj } ^ { ( n ) } = \infty , \lim _ { n \rightarrow \infty } p _ { j j } ^ { ( n ) } > 0$
3. 状态j零常返$ \Leftrightarrow f _ { j j } = 1 , \mu _ { j } = \infty \Leftrightarrow \sum _ { n = 0 } ^ { \infty } p _ { j j } ^ { ( n ) } = \infty , \lim _ { n \rightarrow \infty } p _ { j j } ^ { ( n ) } = 0$
4. 状态j非常返$ \begin{array} { c } { \Leftrightarrow f _ { j j } < 1 \Leftrightarrow \sum _ { n = 0 } ^ { \infty } f _ { j j } ^ { ( n ) } < 1 \Leftrightarrow \sum _ { n = 0 } ^ { \infty } p _ { j j } ^ { ( n ) } < \infty } \\ { \Rightarrow \lim _ { n \rightarrow \infty } p _ { j j } ^ { ( n ) } = 0 } \end{array}$

<font color=#ff0033>重要结论：有限状态的Markov链至少有一个常返态</font>

### <font color=#ff0033>互通与状态分类的关系</font>

1. 若i常返，且$ i\rightarrow j$,则$ j \rightarrow i$
2. 若$ i \leftrightarrow j$，则状态i，j有下列情况
   - 同为常返或同为非常返；若为常返，则同为正常返或同为零常返；
   - i,j有相同的周期

## 状态空间的分解：

![image-20190105110205704](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190105110205704.png)

> 两个子集中的元素均不可达

![image-20190105110315792](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190105110315792.png)

### <font color=#FF0033>重要分解</font>

![image-20190105110330205](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190105110330205.png)



### Markov链遍历性：

![image-20190107151751469](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190107151751469.png)

![image-20190107151859732](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190107151859732.png)

## Markov链的平稳分布

![image-20190107152012117](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190107152012117.png)

> 解释：$\pi_j$ 的值等于所有与它相连的点概率分布的加权（i到j的概率）和

![image-20190107152256389](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190107152256389.png)

![image-20190107152548668](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190107152548668.png)

![image-20190107152736948](/Users/haohao/Documents/概率论与随机过程/概率论与随机过程复习/assets/image-20190107152736948.png)

