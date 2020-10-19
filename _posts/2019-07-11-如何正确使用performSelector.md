---
layout: post
comments: true
---
## 最近总是神经嘻嘻的，有时候代码写着写着，就去看头文件了，看看有没有其他好玩的函数，就忘记了写代码了；直到有一天看到了`performSelector`头文件

```
- (id)performSelector:(SEL)aSelector;
```
这个函数最为iOS开发人员，平时接触的可不少，target-action调用方式。粗看一下好像没啥问题，实例函数，调用函数，返回数据；返回的也是`id`超类，符合常理设计。
但好像忽略了点东西... 如果返回的基础类型怎么办？或者返回的`void`。
## 猜想1
如果是基础类型，系统默认帮忙封装成`NSNumber/NSValue`类型
## 猜想2
如果是`void`返回值，默认返回nil

-------

> 我们来做个实验吧

首先我们先写一个最简单的类

![截屏2020-09-28 下午3.32.34.png](https://i.loli.net/2020/09/28/JP1i9jpxlyHhNKB.png)

最简单的调用

![截屏2020-09-28 下午3.35.57.png](https://i.loli.net/2020/09/28/WNlERU74gT3XFqt.png)

Crash！

![截屏2020-09-28 下午3.38.38.png](https://i.loli.net/2020/09/28/Qztj5F4IGAHaOfX.png)

* 调用`testBoolean`crah，因为返回的`YES`，在ARC下，默认一次实例引用都会retain
 `0x01`明显不是一个对象，报野指针异常。
*  接下来好玩的来了，如果我返回`NO`呢？--> 没有Crash,因为返回的地址`0x00`，默认认为是nil，不会crash。那么基本我们可以确定，如果你返回的int类型，除了返回0，其他都会crash。

###  猜想1，你猜对了么
-------
如果返回的`void`是怎么样的呢？
![截屏2020-09-28 下午3.47.09.png](https://i.loli.net/2020/09/28/HEIyZkYmNxsJOu3.png)
`testPre`和`r1`其实是同一个对象，也就是说，如果返回的`void`，默认就是返回当前对象，而不是nil。
### 猜想2，你猜对了么
## 那么正确调用的姿势是什么呢
### NSMethodSignature
每个函数都有对应的一个函数签名，可以通过对象/实例方法来获取。

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector OBJC_SWIFT_UNAVAILABLE("");

+ (NSMethodSignature *)instanceMethodSignatureForSelector:(SEL)aSelector OBJC_SWIFT_UNAVAILABLE("");
```
一般一个函数基本都是`@:{arg1}{arg2}...`,其中第一，第二个参数基本都表示对象和函数的意思。第3个参数开始，表示参数的类型。
参考表（亲自测试，官方不是很准，后续会单独写个文章阐述下）：https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html

既然有参数表，肯定也返回类型
`methodSignature.methodReturnType`可以知道函数返回的类型。
比如前面Demo中，返回的`BOOL`，methodReturnType 返回类型应该是`B`。
这里不再展开讨论，怎么用`NSMethodSignature`和`NSInvocation`，有时间我会单独写个文章来说明。
