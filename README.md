#! https://zhuanlan.zhihu.com/p/707452354

# TWO WAY RANGING（TWR）详解：TWR的各种分类，优势，误差分析（附公式推导）

## TWO WAY RANGING(TWR)的分类

根据**测距方式**和**计算方式**，严格意义上Two way ranging(TWR)可以被分为四类，分别为：

- Single-Sided TWR(SS-TWR)

- Symmetric Double-Sided TWR(SDS-TWR)
- Alternative Double-Sided TWR(AltDS-TWR)
- Asymmetric Double-Sided TWR(ADS-TWR)

**根据发送包的数量的不同，可以分为single side和double side**

![SS-TWR and DS-TWR](https://pic4.zhimg.com/80/v2-e85e590bea91dd79c978314a8b4a9390.png)

如图所示，在single-sided TWR中，设备A发*Poll*请求，设备B收到Poll请求后经过$T_{reply1}$后发*Response*,只在设备A处经历了一个round

在double-sided TWR中，设备A发*Poll*请求，设备B收到Poll请求后经过$T_{reply1}$后发*Response*,在设备A收到*Response*后经过$T_{reply2}$后再发*Final ACK*，设备B收到*Final ACK*后结束DS-TWR，在设备A和设备B处分别经历了一个round

**根据左右两边的replay time是否相同，可以分为symmetric和asymmetric**

如果$t_{replyB}$和$t_{replyA}$相同，那么就是symmetric

如果$t_{replyB}$和$t_{replyA}$不同，那么就是asymmetric

![Asymmetric TWR](https://pic4.zhimg.com/80/v2-2565da1525a4b7939d65ba5ee5d20e91.png)

如图所示，便是asymmetric的一种极端情况，其中$t_{replyA}$为0。这种情况主要是为了在实现AltDS-TWR和SDS-TWR的表现的同时，尽量减少TWR的测距时间

**根据在SDS-TWR中采用的算法的区别，可以分为traditional和alternative，也就是传统的SDS-TWR和AltDS-TWR在接下来会详细说明**

## TWO WAY RANGING(TWR)的误差分析

![The Principle of Two Way Ranging](https://pic4.zhimg.com/80/v2-80cf81b6c24cab7105f960c9a3789034.png)

影响TWR测量的因素主要有以下三点：

1.Clock Shift

2.Antenna Delay

3.Mutipath Effect

其中，clock drift是其中的主导因素

因为两边的设备都有各自独立且未经同步的时钟，因此两边的设备都将会被time reference上的不完美所影响

尽管clock drift很大程度上依赖于所选择的timing reference，但是IEEE 802.15.4a standard提供的clock drift会限制在±$20$ ppm



### Single-Sided TWR(SS-TWR)

SS-TWR的过程如图1所示，SS-TWR对于飞行时间的估计可以由下式得到：
$$
T_f=\frac{1}{2}(R_a-D_b) \tag{1}
$$
假设设备A的时钟偏移为$e_a$，设备B的时钟偏移为$e_b$，那么我们有
$$
\hat{R}_a=(1+e_a)R_a \\\tag{2}
\hat{D}_b=(1+e_b)D_B \\
\hat{T}_f=\frac{1}{2}(\hat{R}_a-\hat{D}_b)
$$
误差也就是
$$
\hat{T}_f-T_f=\frac{1}{2}(\hat{R}_a-\hat{D}_b)-\frac{1}{2}(R_a-D_b)=\frac{1}{2}(e_a R_a-e_b D_b) \tag{3}
$$
因为
$$
R_a=2T_f+D_b\tag{4}
$$
所以
$$
\hat{T}_f-T_f=\frac{1}{2}(e_a R_a-e_b D_b)=e_aT_f+\frac{1}{2}(e_a-e_b) D_b\tag{5}
$$
通常而言，飞行时间$T_f$是几十到100纳秒，然而$D_b$是毫秒的数量级（reply delay是由即将发送的包的长度和测量电路，控制逻辑共同决定），因此不难看出在这个误差中，$D_b$占据了主导因素

假设reply delay可以低至1 ms，在IEEE 802.15.4a协议中$(e_a-e_b)$最差的情况下是±$ 40$ ppm

只考虑reply delay的影响，在上面方程中的飞行时间的误差至少是20ns
$$
\frac{1}{2}(e_a-e_b) D_b=\frac{1}{2}*40*10^{-6}*1*10^{-3}=20*10^{-9}s
$$
20ns的一个时间误差对应的距离是
$$
d=20*10^{-9}*3*10^{8}=6m
$$
显然对于一个室内的Real Time Localization System(RTLS)来说，这个误差是不可接受的

大部分的RTLS要求测距误差在±$10$ cm以内，对应的时间误差也就是±$\frac{1}{3}$ ns

在当前假设下，我们需要对时间误差的结果提高至少60倍。但是reply delay几乎不可能继续减少，而且想要实现更准确的timing reference，例如1ppm的timing reference在许多moblie systems中也不现实（因为需要expensive, power-hungry的Temperature Compensated Crystal Oscillators(TCXOs)）。

### Symmetic Double-Sided TWR（SDS-TWR）

因此，为了提高测距的准确性，我们引入了另一次的报文交换，也就是DS-TWR这种测距方式，如图2所示



和SS-TWR相似，我们可以得到以下方程
$$
R_a=2T_f+D_b \\
R_b=2T_f+D_a\tag{6}
$$
将两个方程的左右两边相加，我们可以得到
$$
T_f=\frac{1}{4}(R_a+R_b-D_a-D_b)\tag{7}
$$
假设设备A的时钟偏移为$e_a$，设备B的时钟偏移为$e_b$，那么我们有
$$
\hat{R}_a=(1+e_a)R_a \\
\hat{D}_b=(1+e_b)D_b \\
\hat{R}_b=(1+e_b)R_b \\
\hat{D}_a=(1+e_a)D_a \\\tag{8}
$$
在考虑时钟偏移的情况下，我们可以得到
$$
\hat{T}_f=\frac{1}{4}(\hat{R}_a+\hat{R}_b-\hat{D}_a-\hat{D}_b)\tag{9}
$$
飞行时间的测量误差为：
$$
\hat{T}_f-T_f=\frac{1}{4}(\hat{R}_a+\hat{R}_b-\hat{D}_a-\hat{D}_b)-\frac{1}{4}(R_a+R_b-D_a-D_b) \\
=\frac{1}{4}e_a(R_a-D_a)+\frac{1}{4}e_b(R_b-D_b) \\
=\frac{1}{2}(e_a+e_b)T_f+\frac{1}{4}(e_a-e_b)(D_b-D_a)\tag{10}
$$
很显然，当$D_a和D_b$两个reply delay相等的时候，测量误差将要被最小化，在这种情况下右手边的第二项将为0，因此这就是Symmetric Double Side Two Way Ranging（SDS-TWR）

但是在实际中，我们很难让$D_a$和$D_b$对称（完全相同），因为当包被接收时，reply delay便开始计时，这个时间点随机发生相对于时钟的clock edges。以Decawave的DW 1000 IC为例，它使用125MHz的时钟，所以在$D_a$和$D_b$之间的时间差将在0-8ns内均匀分布

和之前分析一样，根据IEEE 802.15.4a，$(e_a-e_b)$和$(e_a+e_b)$的值均应该在$40$ ppm之间，因此右手边第二项时间的测量误差的最大值为
$$
\frac{1}{4}(e_a-e_b)(D_b-D_a)=\frac{1}{4}*40*10^{-6}*8*10^{-9}=80*10^{-15}s=80fs
$$
对应的测距误差为
$$
80*10^{-15}*3*10^{8}=24*10^{-6}m=24um
$$
假设$T_f$是$x$ ms,右手边第一项的值
$$
\frac{1}{2}(e_a+e_b)T_f=\frac{1}{2}*40*10^{-6}*x*10^{-6}=20x*10^{-12}=20x \ ps
$$
对应的测距误差为
$$
20x*10^{-12}*3*10^{8}=6x*10^{-3}m=6x\ mm
$$
所以，从这个误差计算的过程中，我们可以清晰的看到SDS-TWR把由于时钟漂移造成的误差控制在了可接受的范围内以及DS-TWR相比SS-TWR的优势所在

### Alternative Double-Sided TWR（AltDS-TWR）

但是在很多实际应用中，需要相同的reply delay是很难做到的，这意味着三个包不得不以准确的计时器发送，但是在未经协调的网络中，包与包的碰撞很容易发生，使得这整个过程也不得不重新进行，进一步的阻塞整个网络。同时，由于要求两次reply delay的时间相同，这也要求整个TWR报文交换的过程需要比之前需要的时间更长

因此，如果我们可以放松对于timing的要求，那么整个网络的吞吐量就很容易增加。

为了解决equal reply delay的需求，另一种TWR的处理方式被引入，它就是Alternative Double-Sided TWR

AltDs-TWR和SDS-TWR发送包的pattern和process是完全相同的，不同的是最终对于飞行时间估计方式

因此(6)式依然成立，不同与(7)式对于方程左右两边相加，AltDS-TWR将(6)式里面的两个方程左右相乘，

我们可以得到
$$
R_aR_b=(2T_f+D_b)(2T_f+D_a)\tag{11}
$$
我们把右边与$T_f$无关的项移至左边，可以得到
$$
R_aR_b-D_aD_b=2T_f(2T_f+D_a+D_b)\tag{12}
$$
因为
$$
2T_f+D_a+D_b=R_a+D_b=R_b+D_a\tag{13}
$$
所以式（12）还可以被写为
$$
R_aR_b-D_aD_b=2T_f(2T_f+D_a+D_b)\\=2T_f(R_a+D_a)\\=2T_f(R_b+D_b)\\=T_f(R_a+R_b+D_a+D_b)\tag{14}
$$
因此，对于$T_f$的估计为：
$$
T_f=\frac{R_aR_b-D_aD_b}{R_a+R_b+D_a+D_b}
$$
假设设备A的时钟偏移为$e_a$，设备B的时钟偏移为$e_b$，那么我们有
$$
\hat{R}_a=(1+e_a)R_a \\
\hat{D}_b=(1+e_b)D_b \\
\hat{R}_b=(1+e_b)R_b \\
\hat{D}_a=(1+e_a)D_a \\\tag{15}
$$
在考虑时钟偏移的情况下，我们可以得到
$$
\hat{T}_f=\frac{\hat{R}_a\hat{R}_b-\hat{D}_a\hat{D}_b}{2(\hat{R}_a+\hat{D}_a)}\\
=\frac{\hat{R}_a\hat{R}_b-\hat{D}_a\hat{D}_b}{2(\hat{R}_b+\hat{D}_b)}\\
=\frac{\hat{R}_a\hat{R}_b-\hat{D}_a\hat{D}_b}{\hat{R}_a+\hat{R}_b+\hat{D}_a+\hat{D}_b}\tag{16}
$$
(16)式也可以写为
$$
\hat{T}_f=\frac{e_ae_b(R_aR_b-D_aD_b)}{2e_a(R_a+D_a)}=k_bT_f\\
=\frac{e_ae_b(R_aR_b-D_aD_b)}{2e_b(R_b+D_b)}=k_aT_f\\\tag{17}
$$
飞行时间的测量误差为：
$$
\hat{T_f}-T_f=e_bT_f=e_aT_f\tag{18}
$$
从（18）式，我们可以看出在AltDS-TWR这种方法中，飞行时间的测量误差不在与reply delay有关

这也是AltDS-TWR相比于SDS-TWR的优势所在之一：

**AltDS-TWR比SDS-TWR更加鲁棒，传统的对称的SDS-TWR这种方法需要两侧的reply delay相同，然而AltDS-TWR可以在任何情况下都实现一个好的表现。**

同时我们也可以看出
$$
Error=e_bT_f=e_aT_f\tag{19}
$$
**最终的测量误差实际上只与其中的某个设备有关**，**我们也可以利用这个事实。如果某个设备具有更好的表现力，我们便可以利用这个设备进行计算，从而实现更准确的测量。**

## 误差对比

| Method    | error                                                   | ranging requirment |
| --------- | ------------------------------------------------------- | ------------------ |
| SS-TWR    | $e_aT_f+\frac{1}{2}(e_a-e_b)D_b$                        |                    |
| DS-TWR    | $\frac{1}{2}(e_a+e_b)T_f+\frac{1}{4}(e_a-e_b)(D_b-D_a)$ | $D_a=D_b$          |
| AltDS-TWR | $e_bT_f$  OR   $e_aT_f$                                 |                    |



 参考文献

[1] Neirynck, D., Luk, E., & McLaughlin, M. (2016, October). An alternative double-sided two-way ranging method. In *2016 13th workshop on positioning, navigation and communications (WPNC)* (pp. 1-4). IEEE.

[2] Lian Sang, C., Adams, M., Hörmann, T., Hesse, M., Porrmann, M., & Rückert, U. (2019). Numerical and experimental evaluation of error estimation for two-way ranging methods. *Sensors*, *19*(3), 616.
