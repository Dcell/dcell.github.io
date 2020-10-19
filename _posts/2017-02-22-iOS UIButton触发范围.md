---
layout: post
title:  "iOS UIButton触发范围"
date:   2017-02-22 9:32:25 +0800
categories: jekyll update
---

在开发过程中我经常被测试提的一个Bug就是：一个带图片的按钮老是点击不中，体验非常差。
想想也是如果自己碰到这样的情况也挺恼火的，就想着如何解决
1. 把Button的frame设置大点 
2. 把Button的触发范围再大点

方案1嘛，简单，但是这样一来，公司的超级牛逼设计师的设计效果不就被破坏了嘛
方案2认为可行。


    
``` 
open func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? // recursively calls -pointInside:withEvent:. point is in the receiver's coordinate system

open func point(inside point: CGPoint, with event: UIEvent?) -> Bool // default returns YES if point is in bounds
```
    
官方文档的解释：
        This method traverses the view hierarchy by calling the point(inside:with:) method of each subview to determine which subview should receive a touch event. If point(inside:with:) returns true, then the subview’s hierarchy is similarly traversed until the frontmost view containing the specified point is found. If a view does not contain the point, its branch of the view hierarchy is ignored. You rarely need to call this method yourself, but you might override it to hide touch events from subviews.

This method ignores view objects that are hidden, that have disabled user interactions, or have an alpha level less than 0.01. This method does not take the view’s content into account when determining a hit. Thus, a view can still be returned even if the specified point is in a transparent portion of that view’s content.

Points that lie outside the receiver’s bounds are never reported as hits, even if they actually lie within one of the receiver’s subviews. This can occur if the current view’s clipsToBounds property is set to false and the affected subview extends beyond the view’s bounds.

如果我们复写 `func point(inside point: CGPoint, with event: UIEvent?) -> Bool` 函数，将判断条件的范围扩大的话，应该能解决这个问题。

-------
`CGRect`有2个函数：`func insetBy(dx: CGFloat, dy: CGFloat) -> CGRect` 和 `func contains(_ point: CGPoint) -> Bool`
这里稍微对`func insetBy(dx: CGFloat, dy: CGFloat) -> CGRect` 实践了下，很多网上的解释都有误。
    A rectangle. The origin value is offset in the x-axis by the distance specified by the dx parameter and in the y-axis by the distance specified by the dy parameter, and its size adjusted by (2*dx,2*dy), relative to the source rectangle. If dx and dy are positive values, then the rectangle’s size is decreased. If dx and dy are negative values, the rectangle’s size is increased.
    矩形。原点值是在x轴偏移dx和y轴偏移dy,及其大小调整(2 * dx,2 *dy),如果dx和dy > 0,那么矩形的大小变小，否则相反。

-------
实践下

```
        let view1 = UIView(frame: CGRect(x: 100, y: 100, width: 100, height: 100))
        view1.backgroundColor = UIColor.red
        self.view.addSubview(view1)
        
        let view2Rect = view1.frame.insetBy(dx: 10, dy: 10)
        let view2 = UIView(frame: view2Rect)
        view2.backgroundColor = UIColor.green
        self.view.addSubview(view2)
```
-------
```
override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
        var inside = super.point(inside: point, with: event)
        if !inside {
            let insetFrame =  self.frame.insetBy(dx: -10, dy: -10)
            inside = insetFrame.contains(point)
        }
        return inside
    }
```
试试吧








