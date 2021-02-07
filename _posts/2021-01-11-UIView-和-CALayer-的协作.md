---
title: 'UIView 和 CALayer 的协作'
layout: post
comments: true
---

在几年前，我浏览过由猫神翻译的一篇Objc中国的文章。讲的是UIView和CALayer的协作关系，讲的非常好；但是一段时间过去后，发现自己慢慢的又遗忘了，所以打算自己写一下，并且跑一些Demo来加深自己的理解。

## UIView 和 CALayer 的关系

iOS面试特别喜欢问这个问题，一般回答都是UIView负责事件的交互，CALayer负责绘制；再深入一点，UIView是CALayer的代理，我们基于代理里面的函数 **action** 展开讲讲。 

```swift
func action(for: CALayer, forKey: String) -> CAAction?
```

CAAction在我印象中没有接触过，查看了下文档是用于响应 layer 属性的变化。

> An interface that allows objects to respond to actions triggered by a CALayer change.

CALayer默认是有隐式动画，隐约的感觉到有一丝丝的关系🤔

## CALayer 隐式动画 和 UIView 的关系

CALayer大部分属性都是有隐性动画的，这也是我认为 iOS 动画开发体验比 android 好。回到上面的议题，UIView 和 CALayer是一一对应的，你修改了UIView 的属性，其实是设置到CALayer上，但是我们发现一个问题：在UIView 上修改属性，并没有动画效果，但是在animation block中又有了动画效果。

是不是CAAction搞的鬼，如果返回nil默认没有动画，如果有就执行相关的动画，让我们继续思考🤔

继续观察上面提到的**CAAction** ，在 CALayer  找到如下函数

```swift
func action(forKey event: String) -> CAAction?
```

> 1. If the layer has a delegate that implements the action(for:forKey:) method, the layer calls that method. The delegate must do one of the following:
>    1. Return the action object for the given key.
>    2. Return the NSNull object if it does not handle the action.
>
> 2. The layer looks in the layer’s actions dictionary for a matching key/action pair.
>
> 3. The layer looks in the style dictionary for an actions dictionary for a matching key/action pair.
>
> 4. The layer calls the defaultAction(forKey:) class method to look for any class-defined actions.
>
> If any of the above steps returns an instance of NSNull, it is converted to nil before continuing.
>

大概就是通过：代理（UIView） --> layer actions --> style actions --> class-defined actions. 获取对应的action.

如果任何一层返回NSNull，则不再向下寻找。

验证下：

```swift
//自定义Layer
class DLayer: CALayer {
	override func action(for layer: CALayer, forKey event: String) -> CAAction? {
        let action = super.action(for: layer, forKey: event)
    		//查看action
        return action
  }
}
//自定义view 并且自定Layer
class DView: UIView {
    override class var layerClass: AnyClass{
        return DLayer.self
    }
}

//测试代码
let view = DView(frame: CGRect.zero)
self.view.addSubview(view)

view.frame = CGRect(x: 0, y: 0, width: 10, height: 10) ------ 1

UIView.animate(withDuration: 5) {
  view.frame = CGRect(x: 0, y: 0, width: 20, height: 20) ------ 2
}
```

其中1处，打印action 为 **<null>** ,  2处打印为 **_UIViewAdditiveAnimationAction** 应该是个实现了CAAction协议的内部类，验证了我们上面的猜想。

继续打印 CALayer  添加动画的函数

```swift
override func add(_ anim: CAAnimation, forKey key: String?) {
  print(key!)
  print(anim.debugDescription)
  super.add(anim, forKey: key)
}

//
position
<CABasicAnimation:0x600001d9cf20; toValue = NSPoint: {0, 0}; additive = 1; delegate = <UIViewAnimationState: 0x7f9122c084b0>; fillMode = both; timingFunction = easeInEaseOut; duration = 5; fromValue = NSPoint: {-10, -10}; keyPath = position>
bounds.origin
<CABasicAnimation:0x600001d9d2c0; toValue = NSPoint: {0, 0}; additive = 1; fromValue = NSPoint: {0, 0}; keyPath = bounds.origin; delegate = <UIViewAnimationState: 0x7f9122c084b0>; fillMode = both; timingFunction = easeInEaseOut; duration = 5>
bounds.size
<CABasicAnimation:0x600001d9d400; toValue = NSSize: {0, 0}; additive = 1; fromValue = NSSize: {-20, -20}; keyPath = bounds.size; delegate = <UIViewAnimationState: 0x7f9122c084b0>; fillMode = both; timingFunction = easeInEaseOut; duration = 5>
```

是不是感觉非常熟悉，这就是我们平常自定义动画的方式。我们在修改frame时候，默认系统添加了3个animation，分别是 position、bounds.origin、bounds.size。

## 总结

通过上面的思考，我们对 Layer 和 View 关系有了进一步的了解，特别是 CA Animation这一块。

1. UIView 持有 CALayer， 并且是 CALayer的代理
2. 如果是通过UIView修改属性造成Layer属性变化时，Layer会通过4个方式来获取CAAction。
3. 获取action后，添加对应CALayer CAAnimation



