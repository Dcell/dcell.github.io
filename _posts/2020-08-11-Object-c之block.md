---
title: "Object-c之block"
layout: post
comments: true
---

> 以下测试都是在ARC模式下，并不适用MRC下；SDK版本13.0。
> MRC下的情况我不再去演示，平常开发已经没有涉及到。

# NSGlobalBlock

```
id block = ^{};

##############################

(lldb) po [block class]
__NSGlobalBlock__
```
结论：不访问外部变量的`block`类型是`__NSGlobalBlock__`


```
const static NSString *s = @"";
id block = ^{
    NSLog(s);
};

########

(lldb) po [block class]
__NSGlobalBlock__
```
结论：访问全局静态变量，`block`类型是`__NSGlobalBlock__`


# NSMallocBlock

```
int i = 0;
id block = ^{
    NSLog(@"%d",i);
};

###########

(lldb) po [block class]
__NSMallocBlock__
```
结论：block访问了块外面的局部变量，类型是`__NSMallocBlock__`


```
@property(nonatomic,assign) int i;
id block = ^{
    NSLog(@"%d",self.i);
};

###########

(lldb) po [block class]
__NSMallocBlock__
```
结论：block访问了当前对象属性，类型是`__NSMallocBlock__`


# block copy

```
id block = ^{
};
    
block_t block2 =  [block copy];

###########

(lldb) po [block2 class]
__NSGlobalBlock__
```
结论：`__NSGlobalBlock__` copy 并不会更改类型


```
int i = 0;
id block = ^{
    NSLog(@"%d",i);
};
    
block_t block2 =  [block copy];

############

(lldb) po [block2 class]
__NSMallocBlock__

```

结论：`__NSMallocBlock__` copy 并不会更改类型

# 消失的 NSStackBlock
测试一波常用的使用方法，网上说的`NSStackBlock`好像都没碰到过

```
int i = 0;
NSLog(@"%@",^{
    NSLog(@"%d",i);
});

#########

 <__NSStackBlock__: 0x7ffee758cb98>

```
1. block未做任何强引用
2. block访问了局部变量
在这种情况下，block 类型是 `__NSStackBlock__`，但是一旦加了强引用，ARC模式下系统默认会Copy到`__NSMallocBlock__`。

如果这样不够直观的话，我们再写一个用例。

```
- (void)sayHi:(block_t)block{
    id m_block = block;
}
int a = 1;
[self sayHi:^{
    NSLog(@"%d",i);
}];

###########
(lldb) po [block class]
__NSStackBlock__

(lldb) po [m_block class]
__NSMallocBlock__

```

非常直观的看了Block类型的变化。


# 其他
我们继续测试

```
- (void)sayHi:(block_t)block{
    dispatch_async(dispatch_get_main_queue(), ^{
        block();
    });
}

int a = 1;
[self sayHi:^{
    NSLog(@"%d",i);
}];

我们分别在`dispatch_async`之前和进入block添加断点。

(lldb) po [block class]
__NSStackBlock__

(lldb) po [block class]
__NSMallocBlock__
```
当我们把block作为参数，从一个调用栈到另外一个调用栈，系统默认进行一次Copy操作。
