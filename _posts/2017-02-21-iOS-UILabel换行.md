---
title: "iOS-UILabel换行"
layout: post
comments: true
---
## 有中/英文的时候，自动换行没有满足要求

其实Label有个单独的属性来处理换行的问题。默认应该最末... 处理
但是如果我们用autolayout + numberOfLines=0 来处理的话。NSLineBreakByTruncatingTail、NSLineBreakByTruncatingHead,NSLineBreakByTruncatingTail 都是没有效果的。
NSLineBreakByWordWrapping = 0,     	// Wrap at word boundaries, default 单词边界
NSLineBreakByCharWrapping,		// Wrap at character boundaries 字符边界
```
@property(nonatomic)        NSLineBreakMode    lineBreakMode;   // default is NSLineBreakByTruncatingTail. used for single and multiple lines of text
```

如果把Label 设置成NSLineBreakByWordWrapping
![屏幕快照 2017-02-21 15.42.12]({{ site.url }}/assets/media/14876611088840/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-21%2015.42.12.png)

如果把Label 设置成NSLineBreakByCharWrapping
![屏幕快照 2017-02-21 15.42.22]({{ site.url }}/assets/media/14876611088840/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-21%2015.42.22.png)




