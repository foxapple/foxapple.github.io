---
layout: post
title: 使用 SmartLearner 来学习英语
---
# 使用 SmartLearner 来学习英语
----

最近有了学习英语的想法，很多学习攻略都讲了这样的方法

> 
> 通过阅读文章来学习新的单词。
>

我也觉得这个方法很有道理，但是有一个问题就是学习梯度如何控制？如果一篇文章太难了，那么我们很难有兴趣去读下去， 如果一篇文章太简单了，学习的效率又会很低，而且文章中新学到的单词只出现了一次，很难符合记忆曲线的要求。找到合适的文章就是一个很麻烦的事情。

有了上述的想法，我简单的分析了一下我的需求。

1. 文章不能太难，也不能太简单
2. 文章中应该重复出现正在学习的单词

根据这个需求，文章中最好应该是由正在学习的单词来组成的，同时难词应该越少越好。

简单的分一下需要干的活：
1. 找文章
2. 挑文章

刚开始我还想得写个爬虫，每天来爬去新闻，但是后来发现根本不用那么麻烦，[NewsAPI](https://newsapi.org/) 提供了非常方便的接口，对开发者还是免费的。
然后就是挑文章这个事情了，比较麻烦的就是难词的判断，我先找了 3000 basic word list，再加上要学习的 word list， 这个 list 之外的单词就会被认为是 hard word。当然这样会有很高的误判率，因为新闻中有很多人名，地名，这些不应该算做 hard word，通过使用 CoreNLP（我真是调包小能手）分析一下文章的词性就好了，CoreNLP 可以分析出这个词是不是特殊名词，但是这样运行速度太慢，只要是名词，就可以认为不是 hard word。

这样我就完成了 basic 的需求，一键就能看到系统『精心』挑选的新闻，里面全是我需要学的单词，而且还没有什么不常用的复杂词。

然而，我还是没有兴趣去读，我不禁陷入了沉思。

[SmartLearner](https://github.com/foxapple/SmartLearner)

以上.