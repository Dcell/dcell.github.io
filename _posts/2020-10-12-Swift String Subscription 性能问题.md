---
layout: post
comments: true
---

最近刷Leetcode非常的郁闷，一题有关字符串的算法，用Swift写实现一直都是超时。然后换成Java写又没问题，我一度怀疑都是Swift的字典不是用Hash索引的。
尴尬的是最后才发现是Swift Subscription获取字节性能比较差。
具体可以参考老外这篇文章[String Subscription and Performance Problems in Swift](https://programmer.help/blogs/string-subscription-and-performance-problems-in-swift.html)
* 依次遍历字符/索引。避免“跳”到第n个位置。
* 如果需要“随机”访问所有字符，请首先将字符串转换为数组。
* 使用字符串的UTF-16视图可能比使用字符视图更快。