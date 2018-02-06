最近在做一个关于UNUserNotification通知的需求，需要获取当前APP的通知权限，由于iOS10以上通知使用的API发生了很大的变化，在iOS 10以下获取通知设置(notification settings)代码如下：

```
 UIUserNotificationSettings *theSettings = [[UIApplication sharedApplication] currentUserNotificationSettings];
return [theSettings types] & UIUserNotificationTypeAlert;
```
而 iOS10采用UNUserNotificationCenter代替了原有的API。使用UNUserNotificationCenter获取APP通知设置代码如下：
```
   UNUserNotificationCenter *notificationCenter=[UNUserNotificationCenter currentNotificationCenter];
        [notificationCenter getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings * _Nonnull settings) {
            switch (settings.authorizationStatus) {
                case UNAuthorizationStatusAuthorized:
                    enabled = YES;
                    break;
                default:
                    break;
            }
        }];
```
`getNotificationSettingsWithCompletionHandler:`采用的是异步回调的过程。为了写一个兼容两种类型的函数，我的代码如下：

```
   __block BOOL enabled = NO;
    if (@available(iOS 10.0, *)) {
        UNUserNotificationCenter *notificationCenter=[UNUserNotificationCenter currentNotificationCenter];
        [notificationCenter getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings * _Nonnull settings) {
            switch (settings.authorizationStatus) {
                case UNAuthorizationStatusAuthorized:
                    enabled = YES;
                    break;
                default:
                    break;
            }
        }];
    } else {
        UIApplication *application = [UIApplication sharedApplication];
            UIUserNotificationSettings *settings = [application currentUserNotificationSettings];
            if (settings.types != UIUserNotificationTypeNone) {
                enabled = YES;
            }
    }
```
这样貌似很完美，其实这段代码犯了一个很低级的错误。把异步当成同步再用。当时发现这个问题时，APP已经上线了，最后只好向领导老实认错写检讨。伤心太平洋(πーπ)

出现错误肯定要改嘛，最后想到了用信号量来强制将异步变为同步，更改后的代码如下：
```
  __block BOOL enabled = NO;
    if (@available(iOS 10.0, *)) {
        dispatch_semaphore_t sem;
        sem = dispatch_semaphore_create(0);
        UNUserNotificationCenter *notificationCenter=[UNUserNotificationCenter currentNotificationCenter];
        [notificationCenter getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings * _Nonnull settings) {
            switch (settings.authorizationStatus) {
                case UNAuthorizationStatusAuthorized:
                    enabled = YES;
                    break;
                default:
                    break;
            }
            dispatch_semaphore_signal(sem);
        }];
        dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER); //获取通知设置的过程是异步的，这里需要等待
    } else {
          UIApplication *application = [UIApplication sharedApplication];
            UIUserNotificationSettings *settings = [application currentUserNotificationSettings];
            if (settings.types != UIUserNotificationTypeNone) {
                enabled = YES;
            }
    } 
```

另附上Stack Overflow一个获取Notification Settings通用的类，
[地址](https://stackoverflow.com/questions/41564543/ios-10-9-push-notifications)
.h文件
```
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSInteger , PushNotificationType) {
    PushNotificationTypeNone    = 0,      // the application may not present any UI upon a notification being received
    PushNotificationTypeBadge   = 1 << 0, // the application may badge its icon upon a notification being received
    PushNotificationTypeSound   = 1 << 1, // the application may play a sound upon a notification being received
    PushNotificationTypeAlert   = 1 << 2, // the application may display an alert upon a notification being received
};

@interface SpecificPush : NSObject

@property (nonatomic, readonly) PushNotificationType currentNotificationSettings;

+ (SpecificPush *)sharedInstance;
- (PushNotificationType)types;

@end
```
.m文件
```
#import <UserNotifications/UserNotifications.h>

@interface SpecificPush()
@property (nonatomic) PushNotificationType currentNotificationSettings;
@end

@implementation SpecificPush

#pragma mark - Init

static SpecificPush *instance = nil;
+ (SpecificPush *)sharedInstance
{
    @synchronized (self)
    {
        if (instance == nil)
        {
            [SpecificPush new];
        }
    }
    return instance;
}

- (instancetype)init
{
    NSAssert(!instance, @"WARNING - Instance of SpecifishPush already exists");
    self = [super init];
    if (self)
    {
        self.currentNotificationSettings = PushNotificationTypeNone;
    }
    instance = self;
    return self;
}

- (PushNotificationType)types
{
    if (@available(iOS 10.0, *))
    {
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);

        [[UNUserNotificationCenter currentNotificationCenter] getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings *settings) {
            if ((settings.soundSetting == UNNotificationSettingDisabled) && (settings.alertSetting == UNNotificationSettingDisabled) && (settings.soundSetting == UNNotificationSettingDisabled))
            {
                self.currentNotificationSettings = PushNotificationTypeNone;
            }
            if (settings.badgeSetting == UNNotificationSettingEnabled)
            {
                self.currentNotificationSettings = PushNotificationTypeBadge;
            }
            if (settings.soundSetting == UNNotificationSettingEnabled)
            {
                self.currentNotificationSettings = PushNotificationTypeSound;
            }

            if (settings.alertStyle == UNNotificationSettingEnabled)
            {
                self.currentNotificationSettings = PushNotificationTypeAlert;
            }

            dispatch_semaphore_signal(semaphore);

        }];

        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        dispatch_release(semaphore);
    }
    else
    {
        UIUserNotificationSettings *settings = [[UIApplication sharedApplication] currentUserNotificationSettings];

        if (settings.types == UIUserNotificationTypeNone)
        {
            self.currentNotificationSettings = PushNotificationTypeNone;
        }
        if (settings.types & UIUserNotificationTypeBadge)
        {
            self.currentNotificationSettings = PushNotificationTypeBadge;
        }
        if (settings.types & UIUserNotificationTypeSound)
        {
            self.currentNotificationSettings = PushNotificationTypeSound;
        }
        if (settings.types & UIUserNotificationTypeAlert)
        {
            self.currentNotificationSettings = PushNotificationTypeAlert;
        }
    }

    return self.currentNotificationSettings;
}
@end
```
