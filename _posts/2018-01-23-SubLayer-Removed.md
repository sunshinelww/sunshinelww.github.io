---
layout: post
title: 删除CALayer的子layer的正确姿势
date: 2018-01-23 
---
在做动画时候，往往需要在当前CALayer中添加许多子图层(subLayers)，当动画完成后，需要将这些subLayers从当前的CALayer中移除，加入我需要移除当前CALayer中类型为CAShapeLayer的subLayers,通常我们会采用这种做法：

```Objective-C
 NSArray<CALayer *> *subLayers = self.layer.sublayers;
 [subLayers enumerateObjectsUsingBlock:^(CALayer * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if([obj isKindOfClass:[CAShapeLayer class]]){
            [obj removeFromSuperlayer];
        }
  }];
```

然而，这时候会出现crash,具体错误堆栈信息如下：
![](/images/posts/2018-01-23/crash_info.png)


可以看到修改不变的Array导致的。也就是说CALayer的subLayers是不变的数组。推测是因为removeFromSuperlayer会修改self.layer.sublayers,而此时self.layer.sublayers调用了enumerateObjectsUsingBlock函数，也就是在遍历的时候出现了修改，所以会出现crash。

网上有一种解决方案是：

```Objective-C
[self.layer.sublayers makeObjectsPerformSelector:@selector(removeFromSuperlayer)];
```
但是我需要移除部分的subLayers时候就需要另想其他办法了。其实这里很简单就是将self.layer.sublayers数组中的需要删除的子Layer选择出来，存储到另一个数组中，然后在进行删除，这时候不会出现crash。

代码如下：

```Objective-C
    NSArray<CALayer *> *subLayers = self.layer.sublayers;
    NSArray<CALayer *> *removedLayers = [subLayers filteredArrayUsingPredicate:[NSPredicate predicateWithBlock:^BOOL(id  _Nullable evaluatedObject, NSDictionary<NSString *,id> * _Nullable bindings) {
        return [evaluatedObject isKindOfClass:[CAShapeLayer class]];
    }]];
    [removedLayers enumerateObjectsUsingBlock:^(CALayer * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        [obj removeFromSuperlayer];
    }];

```

是不是很easy，😆

**附注：**

刚才同事给我说了另一个方案，就是对self.layer.sublayers数组逆序遍历进行修改，测试下也是可以的。贴上代码：

```Objective-C
NSArray<CALayer *> *subLayers = [self.view.layer sublayers];
    [subLayers enumerateObjectsWithOptions:NSEnumerationReverse usingBlock:^(CALayer * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj isKindOfClass:[CAShapeLayer class]]){
            [obj removeFromSuperlayer];
        }
    }];
```
查了下原因，对数组逆序遍历时,遇到匹配的元素删除后，位置改变的是遍历过的元素，而没有遍历到的元素位置却没有改变，所以遍历能够正常进行。
