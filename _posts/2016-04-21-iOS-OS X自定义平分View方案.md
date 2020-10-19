---
layout: post
title:  "iOS/OS X自定义平分View方案"
date:   2016-02-29 18:00:53 +0800
categories: jekyll update
---
#### 先前在android平分一个View其实还是蛮简单的可以自定义，也可以用GridView。

现在在IOS/Mac中想要平分一个View，想尝试用NSCollectionView(IOS 中为 UICollectionView，后面以Mac为例)

#CollectionView
用我自己的理解就是一个ViewGroup，高自由性，可塑化，相信以后很多绚丽的布局都有可能基于它而实现。

言归正传，如果做出一个简单，实用的多分屏View。
首先你必须了解NSCollectionViewLayout，你只需要如何布局你想要的View，而不需要关心如何管理这些View。如果让开发者自己来实现一个类似TableView的布局，我相信最麻烦的应该就是管理View。

网上对应NSCollectionViewLayout的描述还是很多，我还是希望自己能去官方文档中看一遍。

````
//
//  BisectCollectionViewLayout.swift
//  DSS Desktop
//
//  Created by ding_qili on 16/2/26.
//  Copyright © 2016年 ding_qili. All rights reserved.
//

import Cocoa

class BisectCollectionViewLayout: NSCollectionViewLayout {
    
    var collectionSize:NSRect!;
    var collectionItem:Int = 0;
    var itemWidth:CGFloat = 0;
    var itemHeight:CGFloat = 0;
    var disectNum:Double = 0;
    
    override var collectionViewContentSize: NSSize {
        return (self.collectionView?.superview?.bounds.size)!;
    }
    
    override func prepareLayout() {
        super.prepareLayout();
        collectionSize = self.collectionView?.bounds;
        collectionItem = (self.collectionView?.numberOfItemsInSection(0))!;
        disectNum =  sqrt(Double(collectionItem));
        itemWidth = collectionSize.width/CGFloat(disectNum);
        itemHeight = collectionSize.height/CGFloat(disectNum);
    }
    
    override func layoutAttributesForElementsInRect(rect: NSRect) -> [NSCollectionViewLayoutAttributes] {
        let  itemCount =  self.collectionView?.numberOfItemsInSection(0)
        var  layoutAttributesArray:[NSCollectionViewLayoutAttributes] = [];
        for var i = 0;i < itemCount ;i++ {
            let indexPath = NSIndexPath(forItem: i, inSection: 0);
            if let layoutAttributes = self.layoutAttributesForItemAtIndexPath(indexPath){
                layoutAttributesArray.append(layoutAttributes);
            }
        }
        return layoutAttributesArray;
    }
    
    override func layoutAttributesForItemAtIndexPath(indexPath: NSIndexPath) -> NSCollectionViewLayoutAttributes? {
        let attributes =  NSCollectionViewLayoutAttributes(forItemWithIndexPath: indexPath);
        let item = indexPath.item;
        let xu  = item%Int(disectNum);
        let yu  = item/Int(disectNum);
        let xc = CGFloat(xu);
        let yc = CGFloat(yu);
        attributes.frame = NSRect(x: xc*itemWidth, y: yc*itemHeight, width: itemWidth, height: itemHeight);
        attributes.zIndex = indexPath.section;
        return attributes;
    }
    
    override func shouldInvalidateLayoutForBoundsChange(newBounds: NSRect) -> Bool {
        return true;
    }
    
    
}
````

![效果](https://raw.githubusercontent.com/dingqili/HexoRes/master/IOS-OS-X%E8%87%AA%E5%AE%9A%E4%B9%89%E5%B9%B3%E5%88%86View%E6%96%B9%E6%A1%88/1.png)

