---
title: "OC实现Swift协议 如何正确tryCatch"
layout: post
comments: true
---
自从Swift3.0发布以后，我们项目中也开始尝试使用Swift来实现一些模块。混编已经是一种比较常用的方式了。
在oc中，需要函数抛出一个异常，一般使用`Error指针`注入的方式，很多系统函数也这样处理；在Swift中，我们可以 `throws/throw` 来抛出异常。当我们oc类实现了Swift协议后，会出现怎么样的情况呢？

## Swift协议没有返回值

```
func sayHi() throws
```
当oc实现了这个协议后，系统默认生成的函数

```
-(BOOL)sayHiAndReturnError:(NSError * _Nullable __autoreleasing *)error
```
系统默认会在函数末尾添加`NSError **`，并且返回值是`BOOL`。

那么什么情况下，我们才能catch到异常呢？
我们分几种情况来测试下：
1. 返回NO，不对Error赋值 --------> catch `nilError`
2. 返回NO，对Error赋值 --------> catch 自定义的Error
3. 返回YES，对Error赋值 --------> 无catch

总结：一级逻辑是返回`YES/NO` ， 二级逻辑只有返回`NO`，才能catch Error，如果Error未定义则默认 `nilError`

## Swift协议返回对象

```
func sayHi3() throws -> NSObject
```

```
- (NSObject *)sayHi3AndReturnError:(NSError * _Nullable __autoreleasing *)error
```
同理，我也测试几种情况。
1. 返回nil，不对Error赋值 --------> catch `nilError`
2. 返回nil，对Error赋值 --------> catch 自定义的Error
3. 返回Object，对Error赋值 --------> 无catch

总结：一级逻辑是返回对象是否为空 ， 二级逻辑只有返回`nil`，才能catch Error，如果Error未定义则默认 `nilError`


## 总结
OC实现Swift throws协议，一级判断都是返回的内容，如果返回内容是`YES或者对象不为空`，就算有`Error`赋值，也无法catch，你猜对了么。