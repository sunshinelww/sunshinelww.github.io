---
layout: post
title: NavigationBar的隐藏与显示
date: 2017-11-14 
---
最近在使用UINavigationView时，rootViewController设置多个子UIView进行切换，发现只有最先显示的UIView能正常显示，通过切换显示的UIView的布局向上偏移了64px，导致部分内容被NavigationBar给遮挡了。

通过查询相关资料才发现iOS6中默认的布局将从navigation bar的底部开始，但到了iOS7中默认布局从navigation bar的顶部开始，这就是为什么所有的UI元素都往上漂移了。

通常有两种解决方案：
1.设置NavigationBar的透明度为NO.

```
navigationBarApperance.translucent=NO;
```

2.设置NavigationBar的backgroundImage.

```
[navigationBarApperance setBackgroundImage:[UIImage imageWithColor:kColorNavBG] forBarMetrics:UIBarMetricsDefault];
```
还有一种方案是在rootViewController的ViewDidLoad方法中设置:

```
self.edgesForExtendedLayout=UIRectEdgeNone;
```
edgesForExtendedLayout是一个类型为UIExtendedEdge的属性，指定边缘要延伸的方向，它的默认值是UIRectEdgeAll，即视图向容器四周延伸。