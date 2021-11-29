---
title: 模糊逻辑续篇——CombsMethod
author: 零度
tags:
  - Game
  - AI
date: 2021-11-30 01:05:48
categories: GameMaker
abbrlink: a6e9e1d4ad4a07ee
---





介绍Combs Method在解决模糊逻辑中规则爆炸增长的问题。

<!-- more -->

## 模糊逻辑的爆炸增长问题

在前一篇文章所叙述的模糊推理系统中，仅仅拥有三个模糊语言变量，其中两个是条件，一个是结果，每种语言变量有三个成员，就产生了九种规则。显然，规则数目会随着规模的增大呈现出指数级增长。因此我们需要一种方法来减少规则产生的数目。

让我们先看到之前的规则：

if （*p* and *q*） then *r*

这个规则相当于 if w then r，其中w是p和q的交集。这样的规则我们称之为intersection rule configuration (IRC)。这种规则的弊端很明显，一是当p或q改变的时候，w必然会随之改变，这样就会产生一种新的规则关系；二是w和r的关系取决于我们定义p和q的精确度，当前提条件数量增加的时候，它们的交叉关系会不可避免地爆炸增长。

假如能够把一个交集变成各自对r的关系，便能够极大的减少规则数量，使设定规则随着问题规模线性增长。

## 等价关系

这里给出以下等价关系：
$$
((p\cap q)\Rightarrow r)\Leftrightarrow ((p\Rightarrow r) \cup (q\Rightarrow r))
$$

$$
((p\cup q)\Rightarrow r)\Leftrightarrow ((p\Rightarrow r) \cap (q\Rightarrow r))
$$

$$
(p\Rightarrow (r\cap s))\Leftrightarrow ((p\Rightarrow r) \cap (p\Rightarrow s))
$$

$$
(p\Rightarrow (r\cup s))\Leftrightarrow ((p\Rightarrow r) \cup (p\Rightarrow s))
$$

本文就以最常见的第一种等价关系作出解释。

对于if p then r，它等价于not p or r。当p为真时，才会考虑r；当p为假时，表达式表现为真。为此我们可以列出真值表。

|  p   | not p |  r   | not p or r | if p then r |
| :--: | :---: | :--: | :--------: | :---------: |
|  T   |   F   |  T   |     T      |      T      |
|  T   |   F   |  F   |     F      |      F      |
|  F   |   T   |  T   |     T      |      T      |
|  F   |   T   |  F   |     T      |      T      |

对于四种等价关系，同样可以列出真值表证明。除了真值表之外，利用其等价于not p or r，还可以从数学上推导证明。对于两个或多个并集的规则集，我们称之为union rule configuration(URC)。

URC有一个潜在的问题在于直观上人们不能很好地判断最终的结果（但我感觉一直挺直观的，怕不是我哪里理解有问题。。。）

## URC如何作用于模糊推理系统

还是以前一篇文章的武器选择系统为例。

|          | 近距离      | 中等距离             | 远距离             |
| -------- | ----------- | -------------------- | ------------------ |
| 子弹少   | 不期望（0） | **期望（0.2）**      | **不期望（0.2）**  |
| 子弹中等 | 不期望（0） | **非常期望（0.67）** | **不期望（0.33）** |
| 子弹多   | 不期望（0） | 非常期望（0）        | 期望（0）          |

这是我们的IRC矩阵。如果变为URC，我们就要单独为一个模糊语言变量的成员设定相应的关系，因为if p then r。这里有两个条件模糊成员变量。

| Ammo     | AmmoLow then Undesirable | AmmoOk then Desirable     | AmmoMore then VeryDesirable |
| -------- | ------------------------ | ------------------------- | --------------------------- |
| Distance | Close then Undesirable   | Medium then VeryDesirable | Far then Undesirable        |

这个就是URC的矩阵。可以看出它比起IRC矩阵少了相当多的维度。至于后续的计算就是处理多个置信度的问题了。

有时候我们会遇到某个模糊变量成员过多的问题，这时我们就需要调整隶属函数来让每个模糊语言变量的成员数目一致。

作者在论文提到条件模糊语言变量和后果模糊语言变量应该是具有单调性的线性关系，但我觉得没必要。此时成员数目不一致的问题也得以解决。但是！此时会有一个比较严重的问题——当你在调整优化的时候，后果模糊语言变量去模糊化的结果可能不会是连续变化的。这在有些时候是不符合直觉的，也会出大问题。因为此时在URC矩阵中，后果的不同成员权重实际是不一致的。

比如上述的URC矩阵中，Desirable只有一处，而Undesirable却有三处。Desirable成员就完全不会受到Distance的影响。可能URC的难以直观理解就体现在这里。

但我总觉得哪里不对，觉得将IRC的模糊语言变量直接应用于URC有什么大问题。直到我看到了作者在论文中写的这句话

> Please understand that we are NOT advocating the transformation of existing rules from one format to another. Instead, we are suggesting that the problem space be modeled using a different rule configuration (URC) than the one normally used (IRC).

所以当我们采用URC时，还是得重新审视一下模糊语言变量。

## 带权重的URC

下面看一个作者论文中的例子。

模糊语言变量：

1. Temperature (COLD, COOL and WARM)

2. Rate of Temperature Change (DECREASING, STEADY and INCREASING)

3. Furnace Output (LOW, MODERATELY LOW, MODERATE, MODERATELY HIGH and HIGH)

规则矩阵：

| **Temp/Rate**  | **Cold**  | **Cool**  | **Warm** |
| :------------: | :-------: | :-------: | :------: |
| **Decreasing** |   High    | Mod. High | Moderate |
|   **Steady**   | Mod. High | Moderate  | Mod. Low |
| **Increasing** | Moderate  | Mod. Low  |   Low    |

我们可以看到，当温度处于Cold，同时温度变化速率处于Steady时，此时炉子仍不会处于最大功率；当温度处于相当高，同时温度变化速率也处于Steady时，炉子仍然不会过分降低功率。这些都是我们所不需要的。我们希望在Steady时，Cold下炉子输出为High，Warm下输出为Low，Cool下输出为Moderate。为此我们需要把这个矩阵做一个修改：

| **Temp/Rate**  |  **Cold**   |   **Cool**   |  **Warm**   |
| :------------: | :---------: | :----------: | :---------: |
| **Decreasing** |   High(1)   | Mod. High(4) | Moderate(7) |
|   **Steady**   |   High(2)   | Moderate(5)  |   Low(8)    |
| **Increasing** | Moderate(3) | Mod. Low(6)  |   Low(9)    |

修改后的矩阵也有一些问题，从规则2到规则3、规则2到规则5、规则5到规则8和规则7到规则8都经历了突变的过程。其次，在这个规则矩阵里规则1和规则2相同、规则8和规则9相同，意味着我们可以去除Steady以缩减规则数目。但是这时候如果温度变化速度刚好等于Steady的话，就没有任何规则会生效，同样会导致输出曲线的不平滑。

那如果是一个URC矩阵呢？

| **Temperature** | 1.0  |    Cold Then High    | 1.0  |  Cool Then Moderate  | 1.0  |    Warm Then Low    |
| :-------------: | :--: | :------------------: | :--: | :------------------: | :--: | :-----------------: |
|    **Rate**     | 1.0  | Decreasing Then High | 0.0  | Steady Then Moderate | 1.0  | Increasing Then Low |

这样，通过给规则增加一个权重，我们就能在Steady时，仅考虑温度的影响。比起URC要简单得多。

## 武器选择系统中URC与IRC的比较

回顾上一篇文章，我们用IRC规则得到了期望值，分别是最大值平均方法的60.42和中心法的62。这里用前文描述的URC也做了一个计算，最大值平均法得到的结果是56.94，中心法得到的结果是59.41。

而当我把Distance的Far then Undesirable改成Far then Desirable时，最大值平均法得到的结果是60.5，中心法得到的结果是62.34。这和IRC相比更为接近。当然我没有做大范围的测试，因为太晚了，要睡觉了。等我有空了再把之前写的模块更新一下发上来。

## 参考文献

[1] http://athena.ecs.csus.edu/~hellerm/EEE222/Atricles/Combs_Fuzzy_Logic/Combs_Rapid_Inference.htm （这是Combs的论文，挺简单的，也很详细）

[2] https://en.wikipedia.org/wiki/Combs_method