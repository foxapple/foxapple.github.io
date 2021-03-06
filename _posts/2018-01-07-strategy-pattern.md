﻿---
layout: post
title: 设计模式——策略模式
---
# 设计模式——策略模式
---

在这周主要学习了计算机基本原理，但是感觉和 C 语言和计算机基本原理没有太大的关系。为了准备 1 月 20 号的考试，我现在正在好好地准备 C 语言。

好了回到主题——设计模式上，在上周我就已经看了好多章了，但是很显然我的速度是跟不上我看的速度，所以想把所有看完的都写上来必然是不现实的。所以只能写一小部分。

### 策略模式

策略模式使用场景是有多个算法的情况，并且这些算法会经常的变化。这时候为了使得程序的复用性，我们使用策略模式。

策略模式的基本组成可以参照下图:

![](https://foxapple.github.io/images/2018-01-07/strategy.PNG)

首先有个 Strategy 抽象类，Strategy 应该有个 Compute() 函数，提供这些算法的输入是什么，应该输出什么样的值，下面三个具体的策略实现类，要重写父类的 Compute() 方法。通过不同的算法来计算，算法也好，方式也好，算出的来的值可以一样，也可以不一样。

可以看到 Context 类和 Strategy 抽象类是聚合关系，说明 Context 类中包含多个 Strategy 类。一般，Context 类里要有一个 Strategy 成员，然后 Context 有两个函数，一个是 Setting()，另一个就是 Compute()，前者用来设定是哪一种用来被调用，是对 Strategy Compute() 的一个包装。

这样策略模式的结构就基本写好了，要用的地方，只需要接触 Context 类，而不用接触 Strategy 类了，但是使用 Context 类的人必须要知道有哪些策略，才能来 Setting()，这算是策略模式的一个缺点。

现在来看策略模式，其实就是 Strategy 抽象类(简单策略模式) + 简单工厂方法。

> 策略模式是一种定义一系列算法的方法，从概念上看，这些算法都是完成相同的工作，只是实现不同。他可以通过相同的方式调用不同的算法，减少了不同算法类之间的耦合。『DPE』
>
> 策略模式的 Strategy 类为 Context 定义了一系列的可供复用的算法，而且这些算法的公共功能可以很容易的知道。『DP』
>
> 策略模式相交于简单策略模式的优点在于，如何使用具体实现的职责，交给了 Context 对象来实现了，最大化减轻了客户端的职责。『DPE』
>
> 策略模式还有一个优点就是方便了单元测试，现在每个算法都有了自己的类，很容易通过 Strategy 类来进行接口单元测试。『DPE』

有时候，真的感觉策略模式就是一种编程宗教，哈哈哈。因为这些编程范式并没有通过严密的科学证明，都是大家的美感所导致的。而且都充满了『DP』『DPE』的字眼，仿佛就像圣经一样。

以上。

























