---
layout: post
title: cocos2d-x v3.10でadmob表示(iOS)
---

[cocos2d-xにFirebase導入(iOS)]({{site.baseurl}}/cocos2dxfirebaseiOS/)

AppController.mm
{% highlight Objective-C %}
@interface AppController()
@property (nonatomic, strong) GADBannerView* bannerView;
@end

@implementation AppController

#pragma mark -
#pragma mark Application lifecycle

// cocos2d application instance
static AppDelegate s_sharedApplication;

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {    

    //中略

    app->run();
    [FIRApp configure];
    
    self.bannerView = [[GADBannerView alloc]initWithAdSize:kGADAdSizeSmartBannerPortrait];
    self.bannerView.adUnitID = @"アドユニットID";
    self.bannerView.rootViewController = self.viewController;
    [self.viewController.view addSubview:self.bannerView];
    
    //画面下に配置
    self.bannerView.center = CGPointMake(self.viewController.view.center.x,
                                         self.viewController.view.frame.size.height - self.bannerView.frame.size.height/2);
        
    GADRequest *request = [GADRequest request];
    request.testDevices = @[@"テストデバイスID"];
    [self.bannerView loadRequest:request];
    
    return YES;
}
{% endhighlight %}

GADBannerViewのプロパティ宣言して、admobで取得したアドユニットIDを設定する。
実機テストすると、
`<Google> To get test ads on this device, call: request.testDevices = @[ @"デバイスID" ];`
というログが出るので、それをテストデバイスIDとして設定すると、
そのデバイスではテスト広告が出るようになる。

![4]({{site.baseurl}}/images/2016-07-08_4.png)

参考：[Cocos2d-x 3.5 AdMobのバナー広告を実装する(iOS, Android)](http://studio.cretia.net/blog/344)
