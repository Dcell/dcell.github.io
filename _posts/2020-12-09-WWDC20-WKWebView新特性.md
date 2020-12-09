---
title: 'WWDC20 WKWebView新特性'
layout: post
comments: true
---

WWDC20中，有一个讲座是对于iOS14中WKWebView的新特性，其中一个特性还是比较有意思的，那就是**WKScriptMessageHandlerWithReply** 。

## **WKScriptMessageHandlerWithReply** 和 **WKScriptMessageHandler** 的区别

**WKScriptMessageHandlerWithReply**其实是对**WKScriptMessageHandler**的扩展，在**WKScriptMessageHandler**中JS调用**PostMessage**，返回的是undefine;而**WKScriptMessageHandlerWithReply**中调用**PostMessage** 返回的Promise；在Native中，**WKScriptMessageHandlerWithReply**多了一个**replyHandler**回调调

```objective-c
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message replyHandler:(void (^)(id _Nullable reply, NSString *_Nullable errorMessage))replyHandler API_AVAILABLE(macos(11.0), ios(14.0));
```

当Native调用**replyHandler**回调后，在JS端就执行了Promise的resolve函数，从而完成了这个JS调用Native的闭环。

从这个现象来看，在iOS14的WKWebView中，系统支持了JS调用Native，并且返回Native执行结果。可以通过新的API来替换原先那些开源的JSBridge方案。现在越发手痒想自己搞一个JSBridge了。

