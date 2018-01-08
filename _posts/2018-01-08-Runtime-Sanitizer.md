---
layout: post
title: iOS中并发问题导致的Data Race
date: 2018-1-8
---

首先看一段代码

```Objective-C
@interface ViewController ()

@property (nonatomic,strong)NSObject *obj;

@end

@implementation KVOViewController

- (void)viewDidLoad {
 for (NSUInteger i =0 ; i <10000; i++) {
        dispatch_async(dispatch_get_global_queue(0, 0), ^{
            [self set]
            if (i == 9999) {
                NSLog(@"执行完成");
            }
        });
    }
}

@end
```
这段代码在执行过程中会出现`EXC_BAD_ACCESS`类型的crash，很多人都知道这个由于在多个线程中操作同一个数据区引起不安全的问题。

根本原因在于在属性obj的赋值不是原子性，我们知道在ARC机制下，对属性的赋值等价于MRC的如下代码：

```
- (void)setObj:(NSObject *)obj{
 if (_obj) {
        [_obj release];
        _obj = nil;
    }
    _obj = [obj retain];
    }
    
```
由于不是原子性，导致两个线程同时进入到setObj:方法，有可能对_obj变量同时释放两次。这种类型的问题用Xcode的 Thread Sanitizer线程检查工具检查出来，Xcode称这类问题为Data Race.

使用Xcode 的Thread Sanitizer工具详见：[Thread Sanitizer and Static Analysis](https://developer.apple.com/videos/play/wwdc2016/412)

上述只是一个极端例子，其实平时开发中，我们往往会忽略代码中隐藏的Data Race问题，特别是某个公共库的方法可能被多次调用时，下面是比较常见的例子：


```Objective-C

typedef void (^GlobalBlock) (NSMutableDictionary *dic);

+ (void)setConfigType:(GlobalBlock) newParamsBlock  {
    
    static GlobalBlock block = nil;
    static BOOL isInitilized = NO;
    if (isInitilized) {
        isInitilized = YES;
        block = newParamsBlock;
    }
}
```
例如以上方法是一个类方法，该方法隐含定义了两个静态变量，由于静态变量的生命周期是整个APP运行期间。如果该方法在多个线程中被同时调用，则可能引发Data Race问题。

上述问题可以通过加锁的方式解决，如下：

```Objective-C
+ (void)setConfigType:(GlobalBlock) newParamsBlock  {
    
    static GlobalBlock block = nil;
    static BOOL isInitilized = NO;
     @synchronized(self){
       if (isInitilized) {
          isInitilized = YES;
          block = newParamsBlock;
      }
     }
}
```
