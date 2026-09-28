---
title: Perfect—Secrecy
published: 2026-09-28
description: '完美保密笔记：从香农安全到完美安全，证明两者等价，并给出完美不可区分性的定义与等价证明。'
image: ''
tags: [Cryptography,Perfect secrecy,Shannon secrecy,Perfect Indistinguishability]
category: '专题讲义'
draft: false
lang: ''
---

## What is Perfect？

回忆上节课可证明安全的一般框架，安全性通常由三部分构成：安全定义、计算困难假设和归约证明。这个框架主要刻画的是计算安全，即敌手计算能力有限，因此可以把安全性建立在某些问题“计算上不可行”的假设之上。

本节要讲的完美保密（perfect secrecy）则是一类更强的安全概念。安全定义通常有两个组成部分：安全保证（security guarantee）和威胁模型（threat model）。完美保密要求即使敌手拥有无限计算能力，也能满足安全保证。由于敌手拥有无限算力，因此一个计算困难的假设是毫无意义的。（敌手总能算出来！）

但完美保密并不是“没有假设”。它不依赖计算困难假设，却依赖信息论与概率模型假设：消息空间、密钥空间、密文空间及其概率分布；密钥通常要求均匀随机（例如Gen生成的密钥不是随机的，比如低位总是0，这样有效的密钥空间就变小了。）；密钥与消息独立等。

因此，在可证明安全的三要素中，完美保密并不是直接不需要“假设”，不需要的是计算困难假设和归约到困难问题的步骤，它用直接的概率论证明替代归约。这只能在困难假设上说是“无条件的”。

（其实在这里也可以看出，安全性由：安全定义、**数学假设**和归约证明是很贴切的，数学假设包含了计算困难假设和概率假设，这是一个更普遍的模型）

## 香农安全（Shannon secrecy）

现在的威胁模型是一个无限算力的敌手，那我们来考虑需要有怎么样的安全保证：

（实际上，上节课已经讲过了，我们这里再来复习一下）我们有下面几个自然的尝试：

### Attempt 1：密钥完全安全

在会话中，密钥是完全安全的，不会被敌手知道。

这显然不够。例如加密算法

$$
Enc(sk,m)=m
$$

把明文直接“加密”成明文。此时密钥 sk 确实完全安全，敌手不知道 sk，但明文完全泄露。所以密钥安全是安全的必要条件，但不是充分条件：安全可以推出密钥安全，但密钥安全不能推出安全。

### Attempt 2：明文完全安全

在会话中，明文是完全安全的，不会被敌手知道。

这是信息论意义上的绝对安全，即敌手不知道任何有关明文的信息。

这个想法太好了，以至于不可能是真的。实际上，我们总是可以得到明文长度的一个估计，如果密钥和密文都是 128 bit，那么 (sk,c) 的总信息量最多 256 bit。如果明文的信息量超过这个上界，比如一个均匀随机的 257 bit 消息，那么根据信息论，任何确定算法都不可能从 (sk,c) 恢复出明文——因为解密函数Dec(sk,c)的输出空间受限于输入的信息量。

所以“明文完全安全”在信息论上是不可能的，除非明文本身信息量很小。否则我们不能要求敌手对明文没有任何信息，因为敌手至少知道明文空间、分布、长度等。

### Attempt 3：敌手不能获得“额外”信息

在会话中，敌手不能获得关于明文的“额外”信息。

敌手事先可能知道一些关于明文的信息，比如明文的语言、长度、可能的内容范围等。这个尝试要求的是：看到密文后，敌手对明文的认知不能比看密文前更多。

现在我们把第三个尝试用数学语言表示出来。

:::important[Definition]
香农安全（Shannon, 1949）：

对于一个加密方案$\Pi=(Gen，Enc，Dec，\mathbb{M}，\mathbb{K})$ ，如果

$$
\forall t\in \mathbb{M},c\in \mathbb{C},Pr_{m\leftarrow D}[m=t]=Pr_{m\leftarrow D,sk\leftarrow Gen}[m=t|Enc(sk,m)=c]
$$

（也可以写成 $\forall m\in \mathbb{M},c\in \mathbb{C},Pr[M = m | C = c] = Pr[M = m]$）

我们说 $\Pi$ 对分布 D 是香农安全的。如果 $\Pi$ 对任意一个分布 D 均有上式成立，则称$\Pi$满足香农安全。
:::

攻击者在不看到密文时，对于明文空间中的字符有一个概率分布D，在看到密文后，攻击者对明文空间的概率分布不变，则称对这个分布有香农安全。如果对于任何分布D都成立，则称这个加密方案是香农安全的。

:::note
注1：香农安全并不是完整的“安全定义”，它是一个安全保证，和无限算力的敌手这个威胁模型合起来才是完整的安全定义。这在后面的IND-CPA安全也有体现，IND是计算下的不可区分性，是安全保证，而CPA是威胁模型。

注2：这里有个要求$Pr[Enc(sk,M)=c]>0$ 。如果$Pr[Enc(sk,M)=c]=0$ ，此时条件概率$Pr[M=m|Enc(sk,M)=c]$无定义。
:::

## 完美安全（Perfect secrecy）

香农安全的定义虽然很好得表现了“敌手不能获得关于明文的“额外”信息”的想法，但在验证一个具体方案时并不方便。原因有以下两点：

1）它要求对**所有**明文分布 D 成立。分布有无穷多个，我们不可能逐一验证。

2）它涉及条件概率。每次验证都要计算后验分布，还要处理 $Pr[C=c]=0$ 之类的边界情况。

因此一个自然的想法是：能不能找到一个**不依赖明文分布 D**、只涉及加密方案本身的等价条件？如果可以，验证就会简单得多。

下面我们直接给出一个完美安全的定义

:::important[Definition]
完美安全 ：

对于一个加密方案 $\Pi$ = (Gen，Enc，Dec，M，K) ，如果

$$
\forall m_1,m_2\in M,c\in C,Pr_{sk\leftarrow Gen}[Enc(sk,m_1)=c]=Pr_{sk\leftarrow Gen}[Enc(sk,m_2)=c]
$$

我们说这个加密方案$\Pi$是**完美安全**的。
:::

这里 sk 由 Gen 随机生成，敌手知道 Gen的分布但不知道本次抽到的具体密钥。敌手观察到的是密文的具体取值 c。如果对所有 m1,m2，同一个 c 在两种明文下的出现概率相同，那么看到 c 就无法区分明文是 m1 还是 m2。

也就是说，对于任何消息 m，密文的分布始终相同（如果不相同，那么观察到 c 后，敌手对明文的信念会改变，即获得了额外信息，这就不满足香农安全），就说明只看密文无法改变对明文先验的分布，即无法得到明文的额外信息。这从直觉上这很好地满足了香农安全的要求。

## 完美安全与香农安全的等价性

下面我们来严格证明香农安全和完美安全在数学上是完全等价的：

:::important[Theorem]
加密方案$\Pi$是香农安全的当且仅当它是完美安全的。
:::

**Proof**
### 法一
对于任何加密方案，$\mathbb{M}$上的任何分布，对于Pr[M = m]>0的任何m$\in \mathbb{M}$，以及任何c $\in \mathbb{C}$，我们有

$$
\begin{aligned}
Pr[C = c | M = m] &= Pr[Enc(sk,M) = c | M = m]\\
&= Pr[Enc(sk,m) = c | M = m]\quad \quad \quad (1) \\
&= Pr[Enc(sk,m) = c]
\end{aligned}
$$

由贝叶斯公式，对于Pr[C = c] > 0的任意 c$\in \mathbb{C}$，有

$$
Pr[M = m | C = c]Pr[C = c]= Pr[C = c | M = m]Pr[M = m]\quad \quad (2)
$$

##### PP $\Rightarrow$ SP

由完美安全定义 $\forall m,m' \in \in \mathbb{M},c\in \mathbb{C}$，有

$$
Pr[Enc(sk,m) = c]=Pr[Enc(sk,m') = c]
$$

固定$\mathbb{M}$上的分布M，明文m$\in \mathbb{M}$，密文c$\in \mathbb{C}$且Pr[C = c] > 0。

如果$Pr[M = m] = 0$，那么

$$
Pr[M = m | C = c] = Pr[M = m]=0
$$

下面证明$Pr[M = m] > 0$的情况。

对于 c$\in \mathbb{C}$，$p_c:= Pr[Enc(sk,m) = c]$ 。

由等式(1)可得

$$
Pr[C = c | M = m] =p_c\quad,\quad Pr[Enc(sk,m') = c]=Pr[C = c | M = m' ]
$$

再由完美安全定义$Pr[Enc(sk,m) = c]=Pr[Enc(sk,m') = c]$，可得 $\forall m'\in \mathbb{M}$，

$$
Pr[C = c | M = m' ] = p_c
$$

由全概率公式

$$
\begin{aligned}
Pr[C = c] &=\sum_{m'\in \mathbb{M}} Pr[C = c | M = m' ] · Pr[M = m' ] \\&=\sum_{m'\in \mathbb{M}} \quad p_c · Pr[M = m' ] \\
&= p_c \sum_{m'\in \mathbb{M}} \ Pr[M = m' ]\\
&= p_c= Pr[C = c | M = m]
\end{aligned}
$$

其中求和范围为 $Pr[M = m'] > 0$ 时的 $m'$ ，和式的值显然为1。$p_c:= Pr[Enc(sk,m) = c]$与 $m'$ 无关，可以提出来。

由式（2）即有

$$
Pr[M = m | C = c]= Pr[M = m]
$$

##### SP $\Rightarrow$ PP

由于香农安全对任意分布都成立，故不妨取$\mathbb{M}$上的均匀分布。

由香农安全定义 $\forall m\in \mathbb{M}$，有

$$
Pr[M = m | C = c] = Pr[M = m]
$$

再由等式(2)，可知

$$
Pr[C = c | M = m] = Pr[C = c]
$$

由等式(1)，所以

$$
Pr[Enc(sk,m) = c] = Pr[C = c | M = m]= Pr[C = c]
$$

再由等式(1)，$\forall m' \in \mathbb{M}$ ，也有

$Pr[C = c]=Pr[C = c | M = m'] =  Pr[Enc(sk,m') = c]$

下面证明$Pr[C = c]$与m的选取无关即可

由全概率公式

$$
Pr[C = c]=\sum_{m' \in \mathbb{M}}Pr[C=c|M=m']\cdot Pr[M=m']
$$

在均匀分布下有

$$
Pr[M=m']=\frac{1}{|\mathbb{M}|}
$$

所以

$$
Pr[C = c]=\frac{1}{|\mathbb{M}|}\sum_{m' \in \mathbb{M}}Pr[C=c|M=m']
$$

$\sum_{m' \in M}Pr[C=c|M=m']$相当于对$\mathbb{M}$中所有可能的m做遍历然后求和，因此相对于c是一个固定的常值，所以$Pr[C=c]$也是一个常值，所以$Pr[C = c]$与m的选取无关，即

$$
Pr[Enc(sk,m) = c]=Pr[C = c]=Pr[Enc(sk,m') = c]
$$

:::note
注：这里要求 $Pr[M=m]>0$，否则不能用贝叶斯公式。均匀分布下自动满足该条件。
:::

### 法二

下面我们从概率矩阵的视角来更好地理解完美安全：

定义 $(m_i,c_j):=Pr[Enc(sk,m_i)=c_j]$ ，即 $m_i$被加密成 $c_j$ 的概率。因此可以得到下面这样一个矩阵

$$
A:=\begin{bmatrix}
(m_1, c_1) & (m_1, c_2) & \cdots & (m_1, c_j) & \cdots & (m_1, c_p) \\
(m_2, c_1) & (m_2, c_2) & \cdots & (m_2, c_j) & \cdots & (m_2, c_p) \\
\vdots     & \vdots     &        & \vdots     &        & \vdots     \\
(m_i, c_1) & (m_i, c_2) & \cdots & (m_i, c_j) & \cdots & (m_i, c_p) \\
\vdots     & \vdots     &        & \vdots     &        & \vdots     \\
(m_q, c_1) & (m_q, c_2) & \cdots & (m_q, c_j) & \cdots & (m_q, c_p)
\end{bmatrix}
$$

由(1)可知

$$
Pr[C = c_j | M = m_i] =Pr[Enc(sk,m_i) = c_j]=A_{ij}
$$

由于对每个固定明文 $m_i$，加密结果必然落在密文空间中，所以有

$$
\sum_{j=1}^pA_{ij}=1 , \forall i
$$

即**每一行**的元素之和为 1。

但**每一列**的元素之和不一定为 1，因为不同明文可能以不同概率产生同一个密文。

完美安全要求

>$\forall m,m'\in M,c\in C,Pr[Enc(sk,m)=c]=Pr[Enc(sk,m')=c]$

即

$$
A_{1j}=A_{2j}=\dots=A_{qj} ,\forall j
$$

也就是说，矩阵的每一列中的元素相等。

##### PP $\Rightarrow$ SP

如果一个加密算法是完美安全的，那么它的矩阵就有如下形式

$$
A:=\begin{bmatrix}
a_{1} & a_{2} & \cdots  & a_{p} \\
a_{1} & a_{2} & \cdots  & a_{p} \\
\vdots & \vdots &         & \vdots \\
a_{1} & a_{2} & \cdots  & a_{p} \\
\end{bmatrix}
$$

其中 $a_j\geq0$，且

$$
\sum_{j=1}^pa_{j}=1
$$

由全概率公式

$$
\begin{aligned}
Pr[C=c_j]&=\sum_{i=1}^q  Pr[C=c_j|M=m_i] \cdot  Pr[M=m_i]\\
&=\sum_{i=1}^q  A_{ij} \cdot  Pr[M=m_i]\\
&=a_j\sum_{i=1}^q Pr[M=m_i]=a_j
\end{aligned}
$$

即

$$
Pr[C=c_j]=a_j=A_{ij} =Pr[Enc(sk,m_i) = c_j]=Pr[C = c_j | M = m_i]
$$

由(2)得$Pr[M = m_i]=Pr[ M = m_i| C = c_j],\forall i,j$

##### SP $\Rightarrow$ PP

如果一个加密算法是香农安全的，$\forall i,j$ 有

$$
Pr[M = m_i]=Pr[ M = m_i| C = c_j]
$$

由(2)得，$\forall i,j$ 有

$$
Pr[ C = c_j]=Pr[C = c_j| M = m_i]=A_{ij}
$$

$Pr[ C = c_j]$与 $i$ 无关，所以 $A_{ij}$ 也与 $i$ 无关，即

$$
A_{1j}=A_{2j}=\cdots=A_{qj}
$$

即满足完美安全。

## 完美不可区分性（Perfect Indistinguishability）

我们来看看完美安全的定义

>$\forall m_0,m_1\in M,c\in C,Pr_{sk\leftarrow Gen}[Enc(sk,m_0)=c]=Pr_{sk\leftarrow Gen}[Enc(sk,m_1)=c]$

敌手看到c的一个概率分布，由c的概率分布，不能区分c究竟是由m1还是m2加密而来的。自然地，完美就有以下的想法：是不是只要说明一个加密算法有某种“不可区分”明文的性质，就能满足完美安全呢？

我们先给出一个试验：

设 $\Pi$ = (Gen，Enc，Dec)是一个消息空间为$\mathbb{M}$的加密方案，A是敌手。基于A和 $\Pi$，我们定义一个试验 $PrivK^{eav}_{A,\Pi}$ 如下:

对抗性不可分辨实验(The adversarial indistinguishability experiment)   $PrivK^{eav}_{A,\Pi}$

1. 敌手A输出一对消息 $m_0,m_1\in \mathbb{M}$
2. 使用Gen生成密钥 sk，并随机选择 $b\in${0，1}。计算密文$c^*\leftarrow Enc(sk,m_b)$并将其提供给A。我们将 $c^*$ 称为挑战密文（challenge ciphertext）。
3. 敌手A输出一位 b‘。
4. 如果b‘ = b，我们记 $PrivK^{eav}_{A,\Pi}$ = 1（在这种情况下，我们说A成功了），否则为0。

A通过输出随机猜测以1/2的概率成功是显而易见的。完美不可区分性要求任何A都无法做得更好。

:::important[Definition]
完美不可区分性

对于消息空间$\mathbb{M}$上的加密方案 $\Pi$= (Gen，Enc，Dec)。如果对于**任意**的、**算力不受限制**的敌手A，始终有

$$
Pr[PrivK^{eav}_{A,\Pi}= 1]=\frac{1}{2}
$$

则称$\Pi$是完美不可区分（perfectly indistinguishable）的。
:::

:::note
注1：同样的，完美不可区分性是针对无限算力的敌手而言的。之后我们常用的是安全保证是计算不可区分性，是对有限算力的敌手而言的，IND-CPA安全中的IND指的就是这个。

注2：这和后面的EAV安全是有区别的，由于敌手是无限算力的，所以完美不可区分性的要求更为严格，相较于EAV安全允许1/2相差一个negl(n)，完美不可区分性要求敌手获胜的概率严格为1/2。
:::

:::important[Theorem]
加密方案$\Pi$是完美安全的当且仅当它是完美不可区分的。
:::

**Proof**

##### PP $\Rightarrow$ PI

$\Pi$是完美安全的，所以 $\forall c\in C$，有

$$
Pr[Enc(sk,m_0)=c]=Pr[Enc(sk,m_1)=c]=:p_c
$$

所以

$$
Pr[c^*=c|b=0]=Pr[Enc(sk,m_b)=c|b=0]=Pr[Enc(sk,m_0)=c]=p_c
$$

同理有 $p_c=Pr[c^*=c|b=1]$

由全概率公式

$$
Pr[c^*=c]=Pr[c^*=c|b=0]Pr[b=0]+Pr[c^*=c|b=1]Pr[b=1]=p_c
$$

由贝叶斯公式

$$
Pr[c^*=c|b=0]Pr[b=0]=Pr[b=0|c^*=c]Pr[c^*=c]
$$

所以 $Pr[b=0|c^*=c]=Pr[b=0]=\frac{1}{2}$

同理有$Pr[b=1|c^*=c]=Pr[b=1]=\frac{1}{2}$

所以敌手A在得知密文后，无论采取什么方式，输出的b'的概率仍然是 $\frac{1}{2}$

即

$$
Pr[b'=b]=\frac{1}{2}
$$

即

$$
Pr[PrivK^{eav}_{A,\Pi}= 1]=\frac{1}{2}
$$

##### PI $\Rightarrow$ PP

$\Pi$是完美不可区分的，所以

$$
Pr[PrivK^{eav}_{A,\Pi}= 1]=Pr[b'=b]=\frac{1}{2}
$$

要证：$\forall m_0,m_1\in \mathbb{M}, \forall c \in \mathbb{C}$，有

$$
Pr[Enc(sk,m_0)=c]=Pr[Enc(sk,m_1)=c]
$$

反证法：假设 $\exists m'_0,m'_1\in \mathbb{M} ,c' \in \mathbb{C}$，使得

$$
p_0:=Pr[Enc(sk,m'_0)=c']\neq Pr[Enc(sk,m'_1)=c']=: p_1
$$

由于A是无限算力的，所以A总能找到这样的一组 $m'_0,m'_1,c'$

因此可以构造这样的一个敌手A：

1. 敌手A输出一对 $m_0,m_1$
2. 收到挑战密文 $c^*$ 后，检查 $c^*$ 是否等于 $c'$。如果$c^*=c'$，则输出 $b'=0$；否则输出$b'=1$

由全概率公式

$$
\begin{aligned}
Pr[b'=b]&=Pr[b'=0|b=0]Pr[b=0]+Pr[b'=1|b=1]Pr[b=1] \\
&=\frac{1}{2}Pr[c^*=c'|b=0]+\frac{1}{2}Pr[c^*\neq c'|b=1]\\
&=\frac{1}{2}Pr[Enc(sk,m_0)=c']+\frac{1}{2}(1-Pr[c^* = c'|b=1])\\
&=\frac{1}{2}p_0+\frac{1}{2}(1-p_1)\\
&=\frac{1}{2}+\frac{1}{2}(p_0-p_1)
\end{aligned}
$$

因为 $p_0 \neq p_1$，所以 $p_0 - p_1 \neq 0$

所以

$$
Pr[b'=b]=\frac{1}{2}+\frac{1}{2}(p_0-p_1) \neq \frac{1}{2}
$$

与$\Pi$是完美不可区分的矛盾！

因此不存在这样的一组$m'_0,m'_1,c'$，即$\Pi$是完美安全的。
