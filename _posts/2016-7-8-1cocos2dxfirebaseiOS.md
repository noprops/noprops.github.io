---
layout: post
title: cocos2d-x v3.10にFirebase導入(iOS)
---

[このページ](https://firebase.google.com/docs/ios/setup?hl=ja)を参考にして、CocoaPodsで入れる。
AppController.mmは下のように書く。

{% highlight Objective-C %}
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
{% endhighlight %}

ビルドするとなんかエラーが出るので、
Targets->Build Settings->Other Linker Flagsに`$(inherited)`を追加する。

![1]({{site.baseurl}}/images/2016-07-08_1.png)

ビルドするとまた以下の画像のようなGCControllerがなんとかいうエラーが出る。

![2]({{site.baseurl}}/images/2016-07-08_2.png)

Targets->Build Phases->Link Binary With LibrariesからGameController.frameworkを入れると解決する。

![3]({{site.baseurl}}/images/2016-07-08_3.png)

ビルドしてまた
ld: library not found for -lGoogleToolboxForMac
というエラーが出たら、
target->Build Settings->Library Search Pathsに$(inherited)を入れる。

![1]({{site.baseurl}}/images/2016-12-31_1.png)

[cocos2d-xでadmob表示(iOS)]({{site.baseurl}}/2cocos2dxadmobiOS/)
