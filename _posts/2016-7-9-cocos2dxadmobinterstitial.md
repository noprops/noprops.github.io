---
layout: post
title: cocos2d-x v3.10でadmobインタースティシャル広告表示(iOS)
---

[cocos2d-xにFirebase導入(iOS)]({{site.baseurl}}/1cocos2dxfirebaseiOS/)
[cocos2d-x v3.10でadmob表示(iOS)]({{site.baseurl}}/2cocos2dxadmobiOS/)

AppController.hでshowInterstitialAdというメソッドを宣言する。
AppController.h
{% highlight Objective-C %}
@class RootViewController;

@interface AppController : NSObject <UIApplicationDelegate> {
    UIWindow *window;
}

@property(nonatomic, readonly) RootViewController* viewController;

- (void)showInterstitialAd;

@end
{% endhighlight %}

AppController.mm
{% highlight Objective-C %}
@interface AppController() <GADInterstitialDelegate>
@property (nonatomic, strong) GADBannerView* bannerView;
@property (nonatomic, strong) GADInterstitial* interstitial;
@property (nonatomic, strong) NSArray* testDevices;
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
    
    self.testDevices = @[@"テストデバイスID"];
    
    self.bannerView = [[GADBannerView alloc]initWithAdSize:kGADAdSizeSmartBannerPortrait];
    self.bannerView.adUnitID = @"アドユニットID";
    self.bannerView.rootViewController = self.viewController;
    [self.viewController.view addSubview:self.bannerView];
    
    //画面下に配置
    self.bannerView.center = CGPointMake(self.viewController.view.center.x,
                                         self.viewController.view.frame.size.height - self.bannerView.frame.size.height/2);
        
    GADRequest *request = [GADRequest request];
    request.testDevices = self.testDevices;
    [self.bannerView loadRequest:request];
    
    [self createAndLoadInterstitial];
    
    return YES;
}

- (void)createAndLoadInterstitial {
    self.interstitial =
    [[GADInterstitial alloc] initWithAdUnitID:@"インタースティシャルのアドユニットID"];
    self.interstitial.delegate = self;
    
    GADRequest *request = [GADRequest request];
    request.testDevices = self.testDevices;
    [self.interstitial loadRequest:request];
}

- (void)showInterstitialAd {
    if (self.interstitial.isReady) {
        [self.interstitial presentFromRootViewController:self.viewController];
    } else {
        NSLog(@"The interstitial didn't finish loading or failed to load");
    }
}

#pragma mark GADInterstitialDelegate implementation

- (void)interstitial:(GADInterstitial *)interstitial
didFailToReceiveAdWithError:(GADRequestError *)error {
    NSLog(@"%s: %@", __PRETTY_FUNCTION__, [error localizedDescription]);
}

- (void)interstitialDidDismissScreen:(GADInterstitial *)interstitial {
    NSLog(@"%s", __PRETTY_FUNCTION__);
    [self createAndLoadInterstitial];
}
{% endhighlight %}

GADInterstitialは使い捨てのオブジェクトで、一度表示すると二度と使えなくなるので、
interstitialDidDismissScreenで次のインタースティシャルを作り直す。

C++からこれを呼ぶには、
iOSとAndroidで処理をわけるやつを作って、そこから呼ぶ。

PlatformUtil.h
{% highlight C++ %}
#ifndef PlatformUtil_h
#define PlatformUtil_h

class PlatformUtil {
public:
    static void showIntersAd();
};
#endif /* PlatformUtil_h */
{% endhighlight %}

PlatformUtil.mm
{% highlight Objective-C %}
#include "PlatformUtil.h"
#include "AppController.h"

using namespace std;
USING_NS_CC;

void PlatformUtil::showIntersAd()
{
    auto appController = (AppController *)[[UIApplication sharedApplication] delegate];
    [appController showInterstitialAd];
}
{% endhighlight %}

あとは任意の場所で`PlatformUtil::showIntersAd()`と書けばインタースティシャル広告が出る。
