---
layout: post
title: iOS开发之NSPredicate的使用
date: 2016-11-19 
---
最近在看一个开源项目，里面用到了NSPredicate类，感觉Foundation提供的NSPredicate类及其过滤表达式特别强大，所以特地去学习了下。话不多说，下面我们来一起学习下吧。

NSPredicate有三个子类，如图是其类结构图

![NSPredicate类图.png](http://upload-images.jianshu.io/upload_images/4297538-e0b249e8a09a8ee3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

NSPredicate主要用生成基本的条件表达式，如：

```
   NSMutableArray *array=[NSMutableArray array];
    [array addObject:[[Student alloc] initWith:@"lww" age:20]];
    [array addObject:[[Student alloc] initWith:@"wy" age:20]];
    [array addObject:[[Student alloc] initWith:@"LWW" age:21]];
    [array addObject:[[Student alloc] initWith:@"sunshinelww" age:22]];
    NSPredicate *basicPredicate=[NSPredicate predicateWithFormat:@"name = 'lww'"];
   [array filterUsingPredicate: basicPredicate]; //通过条件表达式筛选数组元素
```
输出结果：
![](http://upload-images.jianshu.io/upload_images/4297538-3725b2d1fb4b160c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当然NSPredicate还可以生成很多复杂的谓词表达式，在此我们不详细展开。

下面我们来看下NSCompoundPredicate类，这个类主要用于组合谓词表达式，有三种组合方式：AND,OR和NOT

```
NSMutableArray *predicateArray=[NSMutableArray array];
NSPredicate *basicPredicate1=[NSPredicate predicateWithFormat:@"name = 'lww'"];
NSPredicate *basicPredicate2=[NSPredicate predicateWithFormat:@"age = 20"];[predicateArray addObject: basicPredicate1];
[predicateArray addObject: basicPredicate2];

NSCompoundPredicate *orMatchPredicate=[NSCompoundPredicate orPredicateWithSubpredicates:predicateArray]; ///对数组中的谓词表达式取或
NSCompoundPredicate *andMatchPredicate=[NSCompoundPredicate andPredicateWithSubpredicates:predicateArray];///对数组中的谓词表达式取与
[NSCompoundPredicate notPredicateWithSubpredicate: basicPredicate1]; ///对basicPredicate1取反
```
通过NSCompoundPredicate我们可以灵活实现多谓词条件相结合的方式进行过滤。

最后我们看下NSComparisonPredicate类的使用，NSComparisonPredicate主要用来比较两个表达式的结果，其中创建NSComparisonPredicate对象需要指定左表达式、右表达式和运算符。构造方法如下：

```
- (instancetype)initWithLeftExpression:(NSExpression *)lhs 
                       rightExpression:(NSExpression *)rhs 
                              modifier:(NSComparisonPredicateModifier)modifier 
                                  type:(NSPredicateOperatorType)type 
                               options:(NSComparisonPredicateOptions)options;
```
表达式类为NSExpression,NSExpression分为constant Value和key path等。
其次是操作符枚举NSPredicateOperatorType，主要有：

```
typedef NS_ENUM(NSUInteger, NSPredicateOperatorType) {
    NSLessThanPredicateOperatorType = 0, // compare: returns NSOrderedAscending
    NSLessThanOrEqualToPredicateOperatorType, // compare: returns NSOrderedAscending || NSOrderedSame
    NSGreaterThanPredicateOperatorType, // compare: returns NSOrderedDescending
    NSGreaterThanOrEqualToPredicateOperatorType, // compare: returns NSOrderedDescending || NSOrderedSame
    NSEqualToPredicateOperatorType, // isEqual: returns true
    NSNotEqualToPredicateOperatorType, // isEqual: returns false
    NSMatchesPredicateOperatorType,
    NSLikePredicateOperatorType,
    NSBeginsWithPredicateOperatorType,
    NSEndsWithPredicateOperatorType,
    NSInPredicateOperatorType, // rhs contains lhs returns true
    NSCustomSelectorPredicateOperatorType,
    NSContainsPredicateOperatorType NS_ENUM_AVAILABLE(10_5, 3_0) = 99, // lhs contains rhs returns true
    NSBetweenPredicateOperatorType NS_ENUM_AVAILABLE(10_5, 3_0)
};
```
通过指定操作符我们可以对两个表达式进行比较。
该函数最后一个参数是枚举NSComparisonPredicateOptions用来指定比较选项，定义如下：

```
typedef NS_OPTIONS(NSUInteger, NSComparisonPredicateOptions) {
    NSCaseInsensitivePredicateOption = 0x01, //大小写不敏感
    NSDiacriticInsensitivePredicateOption = 0x02, //忽视发音符号
    NSNormalizedPredicateOption NS_ENUM_AVAILABLE(10_6, 4_0) = 0x04, /* 表示待比较的字符串已经被预处理了。这一选项取代了NSCaseInsensitivePredicateOption和NSDiacriticInsensitivePredicateOption，旨在用作性能优化的选项 */
};
```
最后用一个例子综合应用上述三个类。代码如下：

```
    NSMutableArray *array=[NSMutableArray array];
    [array addObject:[[Student alloc] initWith:@"lww" age:20]];
    [array addObject:[[Student alloc] initWith:@"wy" age:20]];
    [array addObject:[[Student alloc] initWith:@"LWW" age:21]];
    [array addObject:[[Student alloc] initWith:@"sunshinelww" age:22]];

    
    NSExpression *lhs=[NSExpression expressionForKeyPath:@"name"];
    NSExpression *rhs=[NSExpression expressionForConstantValue:@"lww"];
    
    NSPredicate *predicate1=[NSComparisonPredicate predicateWithLeftExpression:lhs rightExpression:rhs modifier:NSDirectPredicateModifier type:NSContainsPredicateOperatorType options:0]; //name字符串包含“lww”
    
    lhs=[NSExpression expressionForKeyPath:@"age"];
    rhs=[NSExpression expressionForConstantValue:@20];

    NSPredicate *predicate2=[NSComparisonPredicate predicateWithLeftExpression:lhs rightExpression:rhs modifier:NSDirectPredicateModifier type:NSEqualToPredicateOperatorType options:NSCaseInsensitivePredicateOption]; //age=20

    NSMutableArray *predicateArray=[NSMutableArray array];
    [predicateArray addObject:predicate1];
    [predicateArray addObject:predicate2];
    
    NSCompoundPredicate *andMatchPredicate=[NSCompoundPredicate andPredicateWithSubpredicates:predicateArray]; //对两个谓词求与

    [array filterUsingPredicate:andMatchPredicate];
   
    for (Student *stu in array) {
        [stu printAddress];
    }
```
输出结果

![](http://upload-images.jianshu.io/upload_images/4297538-40e226582565f361.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)