---
layout: post
title: RegexKitLite的使用
date: 2016-12-15 
---
RegexKitLite是一个很强大的第三方正则表达式开源库，开源地址为：[点我](http://regexkit.sourceforge.net/) ，下面我们通过几个Demo来讲解这个库的具体使用。

首先从地址 http://regexkit.sourceforge.net/ 下载RegexKitLite开源包，然后解压，得到RegexKitLite.h和RegexKitLite.m两个文件，将其导入到工程项目中。然后`#import "RegexKitLite.h"`就可以使用。

第二中方式是通过CocoaPod的方式引入，在Podfile文件中添加RegexKitLite依赖库，通过pod install命令即可下载RegexKitLite库，同样通过`#import "RegexKitLite.h"`就可以使用。

下面我们具体看下RegexKitLite相关Api的使用，核心函数为：

```
- (NSString *)stringByMatching:(NSString *)regex capture:(NSInteger)capture
```

参数regex表示正则表达式，capture表示匹配选项。
capture为0时，表示返回第一次匹配的字符串，带完全匹配的结果
capture为1时，表示返回第一次匹配的结果中第一处模糊匹配的子字符串
capture为2时，表示返回第一次匹配的结果中第二处模糊匹配，依次类推。

如下Demo代码：

```
    NSString *htmlStr = @"oauth_token=1a1de4ed4fca40599c5e5cfe0f4fba97&oauth_token_secret=3118a84ad910967990ba50f5649632fa&name=foolshit";
    NSString *regexString = @"oauth_token=(\\w+)&oauth_token_secret=(\\w+)&name=(\\w+)";
    NSString *matchedString1 = [htmlStr stringByMatching:regexString capture:0L]; //返回第一次匹配的结果,带完全匹配的字符
    NSString *matchedString2 = [htmlStr stringByMatching:regexString capture:1L]; //返回第一次匹配的结果中第1处模糊匹配，不带完全匹配的字符中
    NSString *matchedString3 = [htmlStr stringByMatching:regexString capture:2L]; //返回第一次匹配的结果中第2处模糊匹配，
    NSString *matchedString4 = [htmlStr stringByMatching:regexString capture:3L];
    
    NSLog(@"%@",matchedString1);
    NSLog(@"%@",matchedString2);
    NSLog(@"%@",matchedString3);
    NSLog(@"%@",matchedString4);

    htmlStr = @"0_T123F_0_T2F_0_T3F_0";
    NSString *regex = @"T123F";
    NSString *result = [htmlStr stringByMatching:regex capture:0L];
    NSLog(@"匹配结果:%@",result);
```

输出结果为：
![](http://upload-images.jianshu.io/upload_images/4297538-4492f4e5e44b5373.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从输出结果我们可以看到，当capture为0时，返回的是匹配的整个字符串，当capture为1时，返回的是模糊匹配的第一个子字符串。

在上面字符串`0_T123F_0_T2F_0_T3F_0`，由于`T123F`是完全匹配，不存在模糊匹配，所以当capture=1时，会抛出异常。