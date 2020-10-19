---
layout: post
title:  "自定义一个简单的SegmentedControl"
date:   2017-02-20 14:41:25 +0800
categories: jekyll update
---
# 自定义一个简单的SegmentedControl

先大概看下我们想实现简单的效果
### ![segment]({{ site.url }}/assets/media/14872996299734/segment.gif)

## 源码

```
//
//  DSegmentedControl.swift
//  IOS学习之自定义UISegmentedControl
//
//  Created by ding_qili on 16/9/12.
//  Copyright © 2016年 ding_qili. All rights reserved.
//

import UIKit

@IBDesignable class DSegmentedControl: UIControl {

    /*
    // Only override drawRect: if you perform custom drawing.
    // An empty implementation adversely affects performance during animation.
    override func drawRect(rect: CGRect) {
        // Drawing code
    }
    */
    var titleColor:UIColor = UIColor.gray //选择状态
    var titleColorDisable:UIColor = UIColor.darkGray //未选中状态
    let slide =  CALayer() //滑块
    
    @IBInspectable var titles:[String]{ //标题
        didSet{
            guard titles.count > 0 else {
                return
            }
            for title in 0..<titles.count {
                let label =  UIButton();
                label.setTitle(titles[title], for: UIControlState());
                label.setTitleColor(titleColor, for: UIControlState.disabled)
                label.setTitleColor(titleColorDisable, for: UIControlState())
                label.addTarget(self, action: #selector(changeValue), for: UIControlEvents.touchUpInside);
                self.addSubview(label);
            }
            setNeedsLayout()
        }
        
    }
    
    override func tintColorDidChange() {
        super.tintColorDidChange()
        
    }
    
    var selecteIndex:Int = 0 {
        didSet{
            refrushSubView()
        }
    }

    override init(frame: CGRect) {
        titles = []
        super.init(frame: frame)
        finishInit()
    }
    
    required init?(coder aDecoder: NSCoder) {
        titles = []
        super.init(coder: aDecoder)
        finishInit()
    }
    
    func changeValue(_ sender:UIView){
        if let index =  self.subviews.index(of: sender){
            selecteIndex = index;
            self.sendActions(for: UIControlEvents.valueChanged)
        }
    }
    
    func finishInit(){
        titleColor = self.tintColor
        slide.frame = CGRect(x: 0, y: 0, width: self.frame.width/2, height: 1)
        slide.backgroundColor = titleColor.cgColor
        self.layer.addSublayer(slide)
        self.backgroundColor = UIColor.clear
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        refrushSubView()
    }
    
    func refrushSubView(){
        for item in self.subviews.enumerated() {
            item.element.frame = CGRect(x: self.frame.width/CGFloat(titles.count) * CGFloat(item.offset), y: 0, width: self.frame.width/CGFloat(titles.count), height: self.frame.height)
            if selecteIndex ==  item.offset{
                var center =  item.element.center;
                center.y = self.frame.height;
                slide.position = center;
            }
            if let uIControl = item.element as? UIControl {
                uIControl.isEnabled = (selecteIndex !=  item.offset);
            }
            
        }
    }


}


```

```
1. 选择继承UIControl,是想要通过UIControlEvents.valueChanged的方式来通知调用者切换状态
2. 默认是用了TintColor作为主色调来使用
3. `segment.titles = ["简介","评论"]` 就可以使用了
4. `segment.addTarget(self, action: #selector(segmentValueChange), for: UIControlEvents.valueChanged)` 回调选择切换
5. 代码比较简单，用于简单的学习
```



