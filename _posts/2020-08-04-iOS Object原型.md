开发iOS也快3年了，其中对objc底层的原理并不是很清楚，第一是以前写java，很少碰到c++结构图相关知识；第二在开发过程中，其实很难碰到这方面的知识点。今天稍微对NSObject原型学习记录下。
# 知识点
* id类型
* 实体指针
* 类指针

```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```
查看`objc.h`头文件，