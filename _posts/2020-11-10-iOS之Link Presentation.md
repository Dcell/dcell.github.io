---
title: 'iOS Link Presentation'
layout: post
comments: true
---
iOS13后，Framework家族迎来了新的成员Link Presentation；这是一个帮你解析URL的库，方便读取连接里面包含的[标题，图片，媒体]等。常见的应用场景，比如微信分享链接，新闻类分享链接等。

## LPMetadataProvider
通过指定的URL，解析对应的文本数据。
### startFetchingMetadata(for:completionHandler:)
开始解析指定的URL，并且返回结果对象`LPLinkMetadata`，如果有异常则返回`LPError`
。
### cancel() 
取消该次请求。
### timeout & shouldFetchSubresources
设置超时和是否默写下载资源

## LPLinkMetadata
一个链接包含的数据对象，该对象包含大概4类数据[链接，标题，图片，媒体]，具体参考[LPLinkMetadata](https://developer.apple.com/documentation/linkpresentation/lplinkmetadata)
## LPLinkView
官方提供的一个内容展示view，使用非常简单。构造函数，可选项输入URL和LPLinkMetadata（本人测试使用URL无效果）；点击后默认跳转到浏览器。

```
let linkView = LPLinkView(metadata: metadata)
self.view.addSubview(linkView)
linkView.sizeToFit()
```

![simulator_screenshot_5D6DE094-3DEE-4FC8-AB06-17CA49060CF1.png](https://i.loli.net/2020/11/10/I4pZnBxqU59QFNK.png)

[Demo](https://github.com/Dcell/my-test/tree/master/Link-Presentation-Demo)