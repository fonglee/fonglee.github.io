---
layout: post
title: 一种实现离散卡尔曼滤波器的方法
excerpt: "一篇翻译自Kristian Lauszus的 A practical approach to Kalman filter and how to implement it的文章，关于离散卡尔曼滤波器的具体实现"
modified: 2016-04-06
tags: [算法]
comments: true
image:
  feature: sample-image-4.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Overview</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->


#前言
本篇文章翻译自Kristian Lauszus的
[A practical approach to Kalman filter and how to implement it](http://blog.tkjelectronics.dk/2012/09/a-practical-approach-to-kalman-filter-and-how-to-implement-it/)。  
我对卡尔曼滤波器如何工作的一直非常有兴趣，并且在我的平衡机器人中也使用了卡尔曼滤波。但是我从未解释过它是怎么工作的，其实我从来没有好好坐下来，拿出一张纸和一张笔，来推导它，所以我并不知道它是怎么工作的。  
直到我在我的原始代码中发现了一个错误，我才发现理解卡尔曼滤波器是多么重要，我后文中将会提到这个错误。  
实际上，在2011年的12月份，当时我还在大学读书，我写了一篇关于卡尔曼滤波器的硕士课程论文。但是我仅仅是使用卡尔曼滤波器来估计一个叠加了高斯白噪声的DC信号的真实电压。我的课程论文可以在这个[文件](http://www.tkjelectronics.dk/uploads/Kalman_SRP.zip)中找到，它是丹麦语，但是你可以使用谷歌翻译来翻译它。对于这篇论文如果你有什么疑问，可以在下面评论。  
好了，让我们回到本文的主题。首先，我先声明下，正如我前面所说，我从来没有好好坐下来，对在加速度计和陀螺仪中应用的卡尔曼滤波器算法具体推导过。虽然这些并不是很难，但是我必须承认，我还没有研究过它实际怎么工作的以及这背后深厚的理论。我觉得大多数人和我一样，只对如何实现这个滤波器更感兴趣。  
在我们开始之前，我们必须有一些关于矩阵运算的基础知识，比如矩阵乘法和矩阵转置等等。如果你还不了解，请浏览下面的网页。

- [Matrix product ](https://en.wikipedia.org/wiki/Matrix_multiplication#Matrix_product_.28two_matrices.29)
- [Multiply matrix](http://www.mathwarehouse.com/algebra/matrix/multiply-matrix.php)
- [Transpose](https://en.wikipedia.org/wiki/Transpose)
- [Covariance matrix](https://en.wikipedia.org/wiki/Covariance_matrix)

可能有些人还不知道卡尔曼滤波器具体是什么，下面我简单的解释一下。它是一种使用一系列随着时间的推移而变化的观测值，在本文中，即加速度计。这些观测值包含噪声，将带来观测误差。卡尔曼滤波器将根据当前和之前的状态来试图估计系统的状态，这将比单独使用测量值更加精确。  
在本文中，问题主要在于机器人来回的移动，导致加速度计在测量重力加速度时，会带来很大的噪声。陀螺仪的问题主要是它的漂移，就像一个旋转的轮子，当转速降低的时候，角速度将减慢。  
综上，你可以知道，在短期内只能相信陀螺仪，长期内，你只能相信加速度计。  
实际上这里有一个很简单的方法来解决这个问题，就是互补滤波器，它就是对加速度计进行低通滤波，对陀螺仪进行高通滤波。虽然它没有卡尔曼滤波器精确，但是有人已经使用微调的互补滤波器实现了平衡机器人。互补滤波器和卡尔曼滤波器之间的比较在[这篇博客](http://robottini.altervista.org/kalman-filter-vs-complementary-filter)中有详细的论述。  
卡尔曼滤波器是基于测量值来产生一个统计最优的系统状态。为了得到最优解，它需要知道输入滤波器的测量值的噪声，还有系统本身的噪声及过程噪声。而且这些噪声必须是高斯白噪声，幸运的是大多数随机噪声都有这个特性。  
如果你想知道更多关于卡尔曼滤波器的理论，你可以参考如下链接：

- [Kalman filter](https://en.wikipedia.org/wiki/Kalman_filter)
- [Kalman introduction](http://www.cs.unc.edu/~welch/media/pdf/kalman_intro.pdf)
- [Kalman courses](http://academic.csuohio.edu/simond/courses/eec644/kalman.pdf)

#系统状态

本文的后面部分可能会令你感到困惑，但是我想如果你擅长数学，顺着本文的思路拿起笔和一张纸来推导，那么这些对你来说将不会很困难。如果你和我一样，没有一个计算矩阵的计算器或者程序，那么我推荐一个在线的计算器[Wolfram Alpha](http://www.wolframalpha.com/),我本文所有的公式都是通过它计算的。  
我将使用[维基百科](https://zh.wikipedia.org/wiki/%E5%8D%A1%E5%B0%94%E6%9B%BC%E6%BB%A4%E6%B3%A2)中的符号，但是如果一个矩阵是常量，并且不随着时间的变化而变化，我将不在后面写上$k$下标，比如$F_k$可以简写成$F$。  
同时我将解释下本文用到的符号。  
首先，我说明一个表示先前状态的符号标记：  

$$
\hat x_{k-1|k-1}
$$

它是一个根据之前的状态和它之前的状态估计而估计得出的先前状态。  
下一个是先验状态：

$$\hat x_{k|k-1}$$

这里先验的意思是这个状态是根据之前的系统状态估计得出的时刻k的状态。  
最后一个叫做后验状态：

$$\hat x_{k|k}$$

是一个加入了观察量的在时刻k的估计。  
问题在于系统状态本身是隐藏的，只能通过观测变量$z_k$来观察，这也称作[隐马尔可夫模型](https://zh.wikipedia.org/wiki/%E9%9A%90%E9%A9%AC%E5%B0%94%E5%8F%AF%E5%A4%AB%E6%A8%A1%E5%9E%8B)。这就表示状态将依赖于k时刻的状态和所有先前的状态，这也意味着在卡尔曼滤波器稳定之前你不可以相信状态的估计，可以看一看前面我提交作业中的图表。  
$\hat x$中ˆ代表这是一个状态估计。不像单独的$x$表示真实值-这是一个我们尝试去估计的量。  
因此在时刻k的状态记做：

$$x_k$$

在时刻k的系统状态，计算公式如下:

$$x_k=Fx_{k-1}+Bu_k+w_k$$


其中$x_k$是给定的状态矩阵：

$$
        \begin{bmatrix}
        \theta\\
        \dot\theta_b\\
        \end{bmatrix}_k
$$

正如上文所示，滤波器的输出将是角度$\theta$和基于加速度计和陀螺仪的偏置$\dot\theta_b$。偏置是陀螺仪已漂移的量。这表示陀螺仪测量值通过减去这个偏置将得到真实的速率。  
接下来是$F$矩阵，这是一个施加于先前状态$x_{k-1}$的状态转移模型。  
$F$被定义为：

$$
  F= 
  \begin{bmatrix}
  1 & -\Delta t\\
  0 & 1\\
  \end{bmatrix}
$$

$-\Delta t$可能看起来令人困惑，但是稍后将解释它的意义。  
接下来是控制输入$u_k$，在这里它是陀螺仪在k时刻测得的每秒的角度，这个也叫做速率$\dot \theta$。我们改写状态方程为：

$$x_k = Fx_{k-1} + B\dot\theta_k + w_k$$

下面是$B$矩阵,被称为控制输入模型，它被定义为：

$$B = 
\begin{bmatrix} \Delta{t}\\0 \\ \end{bmatrix}
$$

它的意义在于当把速率$\dot\theta$乘以时间$\Delta t$，你将得到角度$\theta$，因为不能根据速率计算偏置，所有我们将矩阵底部设置为0。  
$w_k$是一个在k时刻均值为0，协方差为$Q$的高斯白噪声:

$$w_k \thicksim N(0,Q_k)$$

$Q_k$是过程噪声协方差矩阵，在这里表示估计加速度和偏置的协方差矩阵。我们假设偏置估计和加速度是互相独立的，因此它就等于加速度的估计方差和偏置的方差。  
最终的矩阵定义如下所示：

$$
  Q_k = 
  \begin{bmatrix}
  Q_\theta & 0\\
  0 & Q_{\dot\theta_b}\\
  \end{bmatrix} \Delta t
$$

上式中，$Q_k$协方差矩阵取决于当期时刻k，因此加速度方差$Q_\theta$和偏置的方差$Q_{\theta_b}$乘以了时间$\Delta t$。  
这表示更新上次的状态之后，随着时间的推移，过程噪声将增大。比如陀螺仪可能漂移。  
为了使卡尔曼滤波器工作，你将需要知道这些常数。  
注意如果你设置一个比较大的值，在状态的估计中将出现更多的噪声。比如估计的角度开始漂移，你必须增加$Q_{\dot\theta_b}$的值。如果估计的速度变慢，你可以更加信任角度值，你应该减少$Q_\theta$的值，让它更加灵敏。
测量值$z_k$  
首先我们看下在真实的状态$x_k$下的测量值。公示如下：

$$z_k = Hx_k + v_k$$

$z_k$可以通过当前的状态$x_k$乘以H矩阵加上测量噪声$v_k$。  
$H$叫做观测模型，是用来把真实的状态映射到观测空间中。真实的状态是不能被观测到的。因为测量值只有加速度计的测量值，所以$H$定义如下：

$$H = [1 ~ 0]$$

测量值的噪声是均值为0，方差为$R$的高斯白噪声：

$$v_k \thicksim N (0, R) $$

但是因为$R$不是一个矩阵，所以测量噪声就等于测量的方差，因为同一个变量的协方差等于方差。想获得更多信息，请查看该[网页](https://zh.wikipedia.org/wiki/%E5%8D%8F%E6%96%B9%E5%B7%AE%E7%9F%A9%E9%98%B5)。  
现在我们可以定义$R$为：

$$R = E[v_k ~ v_k^T] = var(v_k)$$

更多的关于协方差的信息请查看[维基百科](https://zh.wikipedia.org/wiki/%E5%8D%8F%E6%96%B9%E5%B7%AE%E7%9F%A9%E9%98%B5)和我提交的[作业](http://www.tkjelectronics.dk/uploads/Kalman_SRP.zip)。  
我们这里假设测量噪声都是一样的，并且不依赖于时间$k$:

$$var(v_k) = var(v)$$

注意如果你把测量噪声方差$var(v)$设置的太高，滤波器将反应较慢，因为它会更加不相信新的测量值。但是如果设置过小，该值可能过冲，并且带来更多的噪声，因为更加相信加速度的测量值。  
因此你必须找到过程噪声方差$Q_\theta$，$Q_{\dot\theta_b}$和测量噪声$var(v)$的测量方差。有很多方法去发现这个值，但是这个超出了本文的范畴。

#卡尔曼滤波器方程
现在回到需要用到的方程，我们将使用它来估计在时刻k的系统的真实状态$\hat x_k$。一些聪明的家伙想出了下面的方程来估计系统的状态。方程可以写的更紧凑，但我更喜欢让他们分开，因为这样更容易实现和理解不同的步骤。

##预测
在前两个方程中，我们将尝试预测当前状态，和在时刻k的误差协方差矩阵。首先滤波器将基于所有的先前状态和陀螺仪的测量值尝试估计当前状态：

$$
\hat x_{k\vert k-1} = F\hat x_{k\vert k-1} + B\dot\theta_k
$$

这也是为什么它被称为一个控制输入，因为我们使用它作为一个额外的输入来估计在时刻k的状态，它被称为先验状态$\hat x_{k\vert k-1}$，正如在本文前面部分所述。 接下来是，我们将基于之前的误差协方差矩阵$P_{k-1\vert k-1}$尝试估计先验误差协方差矩阵$P_{k\vert k-1}$，其定义为：

$$P_{k\vert k-1} = FP_{k-1\vert k-1}F^T + Q_k$$。 


该矩阵用于估计我们的对估计状态的当前值的信任程度。越小我们越信任当前估计的状态。上述公式的原理其实很容易理解，很明显的,当我们更新上次状态的估计时，误差协方差会增加，因此我们用状态转换模型$F$乘以误差协方差矩阵同时乘以$F^T$并加上在时刻k的进程噪声$Q_k$。  
误差协方差矩阵$P$，在这里是一个2×2矩阵：

$$
  P = 
  \begin{bmatrix}
  P_{00} & P_{01}\\
  P_{10} & P_{11}\\
  \end{bmatrix}
$$

##更新
首选我们需要做的是计算测量值$z_k$和先验状态$x_{k|k-1}$之间的误差，这也被称为新息：

$$\tilde y = z_k - H\hat x_{k|k-1}$$

观测模型$H$用于将先验状态$x_{k\vert k-1}$映射到来自于加速度计的测量值的观测空间，因此该新息不是矩阵。

$$\tilde y_k = [\tilde y]_k$$

接下来，我们需要做的是计算新息方差：

$$S_k = HP_{k|k-1}H^T + R$$

它的作用是，试图预测我们应该对基于先验误差协方差矩阵$P_{k|k-1}$和测量协方差矩阵$R$的测量有多大的信任度。观测模型$H$被用来映射先验误差协方差矩阵$P_{k|k-1}$到观测空间。  
测量噪音越大，$S$值也越大，这意味着我们不应该过多信任输入的测量值。  
在这里，$S$不是一个矩阵，并且写为：

$$S_k = [S]_k$$

下一步骤是计算卡尔曼增益。卡尔曼增益用于指出我们应该有多相信新息，并且被定义为：

$$K_k = P_{k|k-1}H^TS^{-1}_k$$

你可以看到，如果我们不那么相信新息，新息的协方差$S$会很高，如果我们更相信状态估计，则误差协方差矩阵$P$会很小，因此卡尔曼增益会很小。反过来如果我们更相信新息但不信任的当前状态估计，卡尔曼增益会很大。  
如果你把从更深层面上思考，你可以看到，观测模型$H$的转置用于将误差协方差矩阵$P$的状态映射观测空间。然后，我们通过乘以新息协方差$S$的逆来和误差协方差矩阵比较。  
这是有意义的，因为我们将使用观测模型$H$，从状态误差协方差中提取数据，并与当前的新息方差的估计相比较。  
需要注意的是，如果你不知道刚开始的状态，你可以像如下所示这样设定误差协方差矩阵：

$$
  P = 
  \begin{bmatrix}
  L& 0\\
  0 & L\\
  \end{bmatrix}
$$

其中，$L$代表了一个很大的数。  
对于我的平衡机器人，我知道起始角度，我通过校准找到了陀螺仪的偏置，所以我假设，状态在启动时是已知的，如下所示是我的误差协方差矩阵的初始化：

$$
  P = 
  \begin{bmatrix}
  0& 0\\
  0 & 0\\
  \end{bmatrix}
$$

你可以看看我的[校准例程](https://github.com/TKJElectronics/BalancingRobotArduino/blob/master/BalancingRobotArduino.ino#L448-L454)以获取更多信息。  
在这种情况下，卡尔曼增益是一个2×1的矩阵：

$$
  K = 
  \begin{bmatrix}
  K_0\\
  K_1\\
  \end{bmatrix}
$$

现在我们可以更新当前状态的后验估计：

$$\hat x_{k|k} = \hat x_{k|k-1} + K_k\tilde y_k$$

这是通过将先验状态$\hat x_{k|k-1}$加上卡尔曼增益和新息$\tilde y_k$的乘积  
注意，新息$\tilde y_k$是测量值$z_k$和估计的先验状态$\hat x_{k|k-1}$的偏差，所以新息可以是正的也可以是负的。  
这个简化公示在这里可以理解为，我们通过之前的状态和陀螺仪测量值和从加速度得到的观测值来简单地纠正估计的先验状态$\hat x_{k|k-1}$。
我们需要做的最后一件事是更新后验误差协方差矩阵：

$$P_{k|k} = (I - K_kH)P_{k|k-1}$$

其中，$I$被称为单位矩阵，其定义为：

$$
  I = 
  \begin{bmatrix}
  1& 0\\
  0 & 1\\
  \end{bmatrix}
$$

这个滤波器做的就是，基本上在根据我们修正估计的多少来校正自己的误差协方差矩阵。这是有意义的，因为我们基于先验误差协方差矩阵$P_{k\vert k-1}$和新息的协方差$S_k$修正状态。

#滤波器的实现

在本节中，我将使用上面的公式用一个简单的C++代码实现卡尔曼滤波器，它可以用于平衡机器人，[四轴飞行器](http://blog.tkjelectronics.dk/2012/03/quadcopters-how-to-get-started/)和其他你需要计算角度，偏置或速率的应用。  
如果需要代码，可以在GitHub上找到：
https://github.com/TKJElectronics/KalmanFilter。
我会简单地在每一个步骤的顶部写方程，然后在那之后简化它们，我会写如何使用c语言实现。

##步骤1：

$$
\begin{align}
\dot x_{k|k-1} &= F\hat x_{k-1|k-1}+B\dot\theta_k \\
  \begin{bmatrix}
  \theta\\
  \dot\theta_b\\
  \end{bmatrix}_{k|k-1} 
  &= 
  \begin{bmatrix}
  1 & -\Delta{t}\\
  0 & 1\\
  \end{bmatrix}
   \begin{bmatrix}
  \theta\\
  \dot\theta_b\\
  \end{bmatrix}_{k-1|k-1}+\begin{bmatrix}
  \Delta t\\
  0\\
  \end{bmatrix}\dot\theta_k \\
  &=
  \begin{bmatrix}
  \theta + \Delta t (\dot\theta - \dot\theta_b)\\
  \dot\theta_b\\
  \end{bmatrix}
  \end{align}
$$

你会发现角度的先验估计等于之前的状态估计加上经过偏置补偿的角速率乘以时间。  
因为我们不能直接测量先验偏置估计的偏置，因此这里就让它等于前一个时刻的。  

C语言实现如下所示：

~~~c 
rate = newRate - bias;
angle += dt * rate;
~~~
注意，我计算的是没有偏置的速率，因此它可以被被用户方便的使用。
Wolfram Alpha的链接如下：
[Eq. 1.1](http://www.wolframalpha.com/input/?i=%7B%7B1%2C-t%7D%2C%7B0%2C1%7D%7D*%7B%7Btheta%7D%2C%7Bb%7D%7D%2B%7B%7Bt%7D%2C%7B0%7D%7D*r) 

##步骤2：

$$
\begin{align}
P_{k|k-1} &= FP_{k-1|k-1}F^T + Q_k \\
  \begin{bmatrix}
  P_{00} & P_{01}\\
  P_{10} & P_{11}\\
  \end{bmatrix}_{k|k-1} 
  &=
  \begin{bmatrix}
  1 & -\Delta t\\
  0 & 1\\
  \end{bmatrix}
  \begin{bmatrix}
  P_{00} & P_{01}\\
  P_{10} & P_{11}\\
  \end{bmatrix}_{k-1|k-1} 
  \begin{bmatrix}
  1 &0 \\
  -\Delta t & 1\\
  \end{bmatrix}
  +
  \begin{bmatrix}
  Q_\theta& 0\\
  0 & Q_{\dot\theta_b}\\
  \end{bmatrix} \Delta t \\
  &=
  \begin{bmatrix}
  P_{00} + \Delta t(\Delta t P_{11} - P_{01} - P_{10} + Q_\theta) & P_{01} - \Delta tP_{11} \\
  P_{10} - \Delta t P_{11}&P_{11} + Q_{\dot\theta_b}\Delta t \\
  \end{bmatrix} 
  \end{align}
$$

上述方程用C语言可以这样写：

~~~c
P[0][0] += dt * (dt*P[1][1] - P[0][1] - P[1][0] + Q_angle);
P[0][1] -= dt * P[1][1];
P[1][0] -= dt * P[1][1];
P[1][1] += Q_gyroBias * dt;
~~~

注意，这就是在我原来使用的代码中有错误的部分。

##步骤3:

$$
\begin{align}
\tilde y &= z_k - H\hat x_{k|k-1} \\
&=z_k - \begin{bmatrix} 1 & 0 \\ \end{bmatrix}
\begin{bmatrix}
\theta \\
\dot\theta_b
\end{bmatrix}_{k|k-1} \\
&= z_k - \theta_{k|k-1}
\end{align}
$$

新息用c语言计算的代码如下：

~~~c
y = newAngle - angle;
~~~

##步骤4：

$$
\begin{align}
S_k &= HP_{k|k-1}H^T + R \\
&= \begin{bmatrix} 1 & 0\\ \end{bmatrix} 
\begin{bmatrix} 
P_{00} & P_{01} \\ 
P_{10} & P_{11} \\
\end{bmatrix}_{k|k-1}
\begin{bmatrix}
1 \\
0 \\
\end{bmatrix} + R \\
&= {P_{00}}_{k|k-1} + var(v)
\end{align}
$$

用c语言表示非常简单：

~~~c
S = P[0][0] + R_measure;
~~~

##步骤5：

$$\begin{align}
K_k & = P_{k|k-1}H^TS^{-1}_k \\
\begin{bmatrix}
K_0 \\
K_1 \\
\end{bmatrix}_k 
& = \begin{bmatrix}
P_{00} & P_{01} \\
P_{10} & P_{11} \\
\end{bmatrix}_{k|k-1}
\begin{bmatrix}
1 \\
0 \\
\end{bmatrix}
S^{-1}_k \\
& = 
\frac {
\begin{bmatrix}
P_{00} \\
P_{10} \\
\end{bmatrix}_{k|k-1} }
{S_k}
\end{align}
$$

请注意，在其他情况下$S$可能是一个矩阵，你不能只是简单地用$P$除以$S$。相反，您要计算矩阵的逆。关于如何计算它,请参考[网页](http://mathworld.wolfram.com/MatrixInverse.html)。  
C实现如下所示：

~~~c
K[0] = P[0][0] / S;
K[1] = P[1][0] / S;
~~~

##步骤6：

$$
\begin{align}
\hat x_{k|k} &= \hat x_{k|k-1} + K_k\tilde y_k \\
\begin{bmatrix}
\theta \\
\dot\theta_b \\
\end{bmatrix}_{k|k}
&=\begin{bmatrix}
\theta \\
\dot\theta_b \\
\end{bmatrix}_{k|k-1} + 
\begin{bmatrix}
K_0 \\
K_1 \\
\end{bmatrix}_k \tilde y_k \\ &=
\begin{bmatrix}
\theta \\
\dot\theta_b \\
\end{bmatrix}_{k|k-1} + 
\begin{bmatrix}
K_0\tilde y \\
K_1\tilde y \\
\end{bmatrix}_k
\end{align}
$$

方程式相当短，C语言实现如下所示：

~~~c
angle += K[0] * y;
bias += K[1] * y;
~~~

##步骤7：

$$
\begin{align}
\hat x_{k|k}  &= \hat x_{k|k-1} + K_k\tilde y_k \\
\begin{bmatrix}
P_{00} & P_{01} \\
P_{10} & P_{11} \\
\end{bmatrix} 
  &= \left(
\begin{bmatrix}
1 & 0 \\
0 & 1 \\
\end{bmatrix} -
\begin{bmatrix}
K_0 \\
K_1 \\
\end{bmatrix}_k
\begin{bmatrix}
1 & 0 \\
\end{bmatrix} \right)
\begin{bmatrix}
P_{00} & P_{01} \\
P_{10} & P_{11} \\
\end{bmatrix}_{k|k-1} \\
 &=
\begin{bmatrix}
P_{00}  & P_{01} \\
P_{10} & P_{11} \\
\end{bmatrix}_{k|k-1} -
\begin{bmatrix}
K_0P_{00} & K_0P_{01} \\
K_1P_{00} & K_1P_{01} \\
\end{bmatrix}
\end{align}
$$

这里，我们再次减少误差协方差矩阵，因为状态的估计误差已经降低。  
C代码如下所示：

~~~c
float P00_temp = P[0][0];
float P01_temp = P[0][1];
P[0][0] -= K[0] * P00_temp;
P[0][1] -= K[0] * P01_temp;
P[1][0] -= K[1] * P00_temp;
P[1][1] -= K[1] * P01_temp;
~~~

我发现下面的方差完全适用于大多数的IMU：

~~~c
float Q_angle = 0.001;
float Q_gyroBias = 0.003;
float R_measure = 0.03;
~~~

这里强调一点，非常重要的，如果你启动时需要使用输出，那么就需要在启动时设置目标角度。欲了解更多信息，请参阅我的平衡机器人[校准程序](https://github.com/TKJElectronics/BalancingRobotArduino/blob/master/BalancingRobotArduino.ino#L448-L454)。  
我写了一个可用于任何支持浮点运算的微控制器的库。源代码可以在[GitHub](https://github.com/TKJElectronics/KalmanFilter)上找到。    
如果你想看关于卡尔曼滤波的视频，我推荐这个[视频](http://www.youtube.com/watch?v=FkCT_LV9Syk)。  
请注意，如果你需要表示3D方向的角度，您不能使用该库，因为欧拉角有奇点的问题，你需要使用四元数来实现它，这又是一个很长的话题。  
以上就是我所知道的，我希望给你带来帮助。如果你有任何疑问，在下面随意发表评论-如果你需要写方程式，它也支持LaTeX语法。  
如果您发现任何错误，请告诉我。

