---
layout: post
title: cocos2d-x v3.10にFirebase導入(iOS)
---

[このページ](https://firebase.google.com/docs/ios/setup?hl=ja)を参考にして、CocoaPodsで入れる。
AppController.mmは下のように書く。

```Objective-C:AppController.mm
#import "Firebase.h"

@implementation AppController

#pragma mark -
#pragma mark Application lifecycle

// cocos2d application instance
static AppDelegate s_sharedApplication;

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {    
    //中略
    app->run();
    [FIRApp configure];
    return YES;
}
```


![1]({{site.baseurl}}/images/2016-07-08_1.png)
![2]({{site.baseurl}}/images/2016-07-08_2.png)
![3]({{site.baseurl}}/images/2016-07-08_3.png)
