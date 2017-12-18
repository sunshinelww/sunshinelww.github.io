---
layout: post
title: 设置UIButton背景颜色的正确姿势
date: 2017-12-12 
---
通常设置按钮背景颜色，我们采用 setBackgroundColor:(UIColor*)color方法，但这只能设置UIControlStateNormal状态下的按钮背景颜色。采用根据color创建按钮的背景图片，然后设置按钮的背景图片可以有效设置按钮在不同状态下的背景颜色。

```
//根据背景颜色创建图片
- (UIImage *)imageFromColor:(UIColor *)color
{
    CGRect rect = CGRectMake(0, 0, 1, 1);
    UIGraphicsBeginImageContextWithOptions(rect.size, NO, [UIScreen mainScreen].scale);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, color.CGColor);
    CGContextFillRect(context, rect);
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}

//设置按钮背景颜色
 UIColor *color = [self colorWithRGBHexString:@"#049DFF"];
 UIImage *normalBack = [self imageFromColor:color];
 [button setBackgroundImage:normalBack forState:UIControlStateNormal];
```

通过这种方式，我们可以通过这种方式同时设置UIControlStateNormal和UIControlStateHighlighted两种状态的背景颜色