---
layout: post
title:  "UIView的initWithFrame和initWithCoder"
date:   2017-02-20 14:41:25 +0800
categories: jekyll update
---
1. ## initWithFrame: - It is recommended that you implement this method. You can also implement custom initialization methods in addition to, or instead of, this method.

2. ## initWithCoder: - Implement this method if you load your view from an Interface Builder nib file and your view requires custom initialization.


-------
# UIView和XIB
1. 首先自定义下UIView
2. 再创建一个Xib文件
3. Xib中随便拖拽几个UIView
4. 设置File owner为自定义UIView
5. 连接一个View到自定义UIView
6. 添加到ViewController中测试

![屏幕快照 2017-02-14 11.17.13]({{ site.url }}/assets/media/14870388549569/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-14%2011.17.13.png)
![屏幕快照 2017-02-14 11.30.23]({{ site.url }}/assets/media/14870388549569/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-14%2011.30.23.png)
![屏幕快照 2017-02-14 11.31.32]({{ site.url }}/assets/media/14870388549569/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-14%2011.31.32.png)



```
如果在finishInit 中不执行 
Bundle.main.loadNibNamed("CustomView", owner: self, options: nil)
将报错。因为我们已经将一个UIView 和 自定义View连线，执行load以后默认就会初始化View
```


![屏幕快照 2017-02-14 11.34.38]({{ site.url }}/assets/media/14870388549569/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-14%2011.34.38.png)


```利用@IBDesignable 在Storeboard中直接显示自定义View
```
1. 在自定义View Class 添加 @IBDesignable
2. 在storyboard ->ViewController 拖拽一个UIView ， class 改成我们自定义View


```
可惜storyboard 报错了，IB Designables: Failed to update auto layout status: The agent raised a "NSInternalInconsistencyException" exception: Could not load NIB in bundle: 'NSBundle </Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/Library/Xcode/Overlays> (loaded)' with name 'CustomView'
```

```查看错误信息应该表示Bundle 无法加载NIB
通过在网上查找资料，问题其实是这样的。

When loading the nib, we're relying on the fact that passing bundle: nil defaults to your app's mainBundle at run time.
我们修改代码，将Bundle.main.loadNibNamed("CustomView", owner: self, options: nil) 改为
Bundle(for: CustomView.self).loadNibNamed("CustomView", owner: self, options: nil)
再查看下，结果显示正确
```
![屏幕快照 2017-02-14 11.57.24]({{ site.url }}/assets/media/14870388549569/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-14%2011.57.24.png)


