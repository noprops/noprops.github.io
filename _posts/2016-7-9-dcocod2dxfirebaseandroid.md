---
layout: post
title: cocos2d-x v3.10にFirebase導入してadmobのバナーとインタースティシャル表示(Android)
---

[cocos2d-xにFirebase導入(iOS)]({{site.baseurl}}/1cocos2dxfirebaseiOS/)

[cocos2d-x v3.10でadmob表示(iOS)]({{site.baseurl}}/2cocos2dxadmobiOS/)

[cocos2d-x v3.10でadmobインタースティシャル表示(iOS)]({{site.baseurl}}/cocos2dxadmobinterstitial/)

Android Studioを開き、Open an existing android studio projectで
プロジェクト名/proj.android-studioフォルダを開く。

SDKManagerでインストールした最新のNDKだとcocos2d-xが対応していなくてビルドエラーになるので、
r10dをダウンロードして使う。(cocos2d-x v3.10使用)

参考：[過去のリビジョンのNDKを入手する方法](http://qiita.com/kishi-yama/items/1dab24942c12b9971d3e)

cocos2d-xのフォルダに移動して./setup.pyでNDK_ROOTを設定しておく。

Firebaseを入れる時はAndroid StudioのInstant Runをオフにしないといけないらしいので、
Android Studio->Preferences->Build,~~->Instant Runからオフにする。

![1]({{site.baseurl}}/images/2016-07-09_1.png)

Firebaseコンソールでアプリを登録して、
`google-services.json`を取得する。
それをappフォルダに入れる。

![2]({{site.baseurl}}/images/2016-07-09_2.png)

プロジェクト下のbuild.gradleのdependenciesに、

`classpath 'com.google.gms:google-services:3.0.0'`

と書く。

{% highlight java %}
dependencies {
    classpath 'com.android.tools.build:gradle:x.x.x'
    classpath 'com.google.gms:google-services:3.0.0'
}
{% endhighlight %}

![3]({{site.baseurl}}/images/2016-07-09_3.png)

appフォルダ以下のbuild.gradleを以下のように変更する。

{% highlight java %}
dependencies {
  // ...
  compile 'com.google.firebase:firebase-core:9.2.0'
  compile 'com.google.firebase:firebase-ads:9.2.0'
}

// ADD THIS AT THE BOTTOM
apply plugin: 'com.google.gms.google-services'
{% endhighlight %}

![4]({{site.baseurl}}/images/2016-07-09_4.png)

gradleを変更したら、syncする。

app/src/org.cocos2dx.cpp/AppActivity.javaを変更する。

バナーとインタースティシャルを表示する。

参考：[Cocos2d-x 3.5 AdMobのバナー広告を実装する(iOS, Android)](http://studio.cretia.net/blog/344)

{% highlight java %}
package org.cocos2dx.cpp;

import org.cocos2dx.lib.Cocos2dxActivity;
import android.graphics.Color;
import android.os.Bundle;
import android.view.Gravity;
import android.view.WindowManager;
import android.widget.FrameLayout;
import com.google.android.gms.ads.*;

public class AppActivity extends Cocos2dxActivity {
    static AdView _adView;
    static final String BANNER_AD_UNIT_ID = "バナーのアドユニットID";

    static InterstitialAd _interstitialAd;
    static final String INTERSTITIAL_AD_UNIT_ID = "インタースティシャルのアドユニットID";

    static final String TestDeviceID = "テストデバイスID";

    static AppActivity _instance;

    public static AppActivity getInstance(){
        return _instance;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        _instance = this;

        getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);

        FrameLayout.LayoutParams adParams = new FrameLayout.LayoutParams(
                FrameLayout.LayoutParams.WRAP_CONTENT,
                FrameLayout.LayoutParams.WRAP_CONTENT);
        adParams.gravity = (Gravity.BOTTOM | Gravity.CENTER); // 最下部中央へ表示

        _adView = new AdView(this);
        _adView.setAdSize(AdSize.BANNER);
        _adView.setAdUnitId(BANNER_AD_UNIT_ID);

        AdRequest adRequest = new AdRequest.Builder()
                .addTestDevice(TestDeviceID)
                .build();

        _adView.loadAd(adRequest);
        _adView.setBackgroundColor(Color.TRANSPARENT);
        addContentView(_adView, adParams);

        //interstitial
        _interstitialAd = new InterstitialAd(this);
        _interstitialAd.setAdUnitId(INTERSTITIAL_AD_UNIT_ID);

        _interstitialAd.setAdListener(new AdListener() {
            @Override
            public void onAdClosed() {
                requestNewInterstitial();
            }
        });

        requestNewInterstitial();
    }
    private void requestNewInterstitial() {
        AdRequest adRequest = new AdRequest.Builder()
                .addTestDevice(TestDeviceID)
                .build();
        _interstitialAd.loadAd(adRequest);
    }
    public void showIntersAd() {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                if (_interstitialAd.isLoaded()) {
                    _interstitialAd.show();
                }
            }
        });
    }
    @Override
    protected void onResume() {
        super.onResume();
        if (_adView != null) {
            _adView.resume();
        }
    }
    @Override
    protected void onPause() {
        if (_adView != null) {
            _adView.pause();
        }
        super.onPause();
    }
    @Override
    protected void onDestroy() {
        _adView.destroy();
        super.onDestroy();
    }
}
{% endhighlight %}

interstitial.isLoaded()はmainUIスレッドで呼ばれないとクラッシュするので、
このように書いてある。
テストデバイスIDは、アプリをrunした時にlog catに

`Use AdRequest.Builder.addTestDevice("***************") to get test ads on this device.`

と出るので、それをコピペすればよい。

AppActivityのshowIntersAd()を呼ぶ、jniのメソッドを作る。

srcフォルダを右クリックして、new->java classで
com.domain.appname.PlatformUtilというファイルを作る。
はじめのcom.domain.appnameの部分は、アプリのpackageと同じにする。

![5]({{site.baseurl}}/images/2016-07-09_5.png)

{% highlight java %}
package info.mygames888.escapeFromEscapeGame;

import org.cocos2dx.cpp.AppActivity;
import android.content.Context;
import android.util.Log;

public class PlatformUtil {
    public static void showIntersAd(){
        Log.d("PlatformUtil", "showIntersAd");
        AppActivity.getInstance().showIntersAd();
    }

    public static boolean isTablet(){
        Context context = AppActivity.getContext();
        return (context.getResources().getConfiguration().smallestScreenWidthDp >= 600);
    }
}
{% endhighlight %}

XCodeに戻って

PlatformUtil.h
{% highlight C++ %}
#ifndef PlatformUtil_h
#define PlatformUtil_h

class PlatformUtil {
public:
    static void showIntersAd();
    static bool isTablet();
};
#endif /* PlatformUtil_h */
{% endhighlight %}

PlatformUtil.cpp
{% highlight C++ %}
#include "PlatformUtil.h"
#if (CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)

#include "cocos2d.h"
#include "platform/android/jni/JniHelper.h"

using namespace std;
USING_NS_CC;

const static char className[] = "info/mygames888/escapeFromEscapeGame/PlatformUtil";

void PlatformUtil::showIntersAd()
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, className, "showIntersAd", "()V"))
    {
        t.env->CallStaticVoidMethod(t.classID, t.methodID);
    }
}
bool PlatformUtil::isTablet()
{
    JniMethodInfo t;
    
    if (JniHelper::getStaticMethodInfo(t, className, "isTablet", "()Z"))
    {
        return t.env->CallStaticBooleanMethod(t.classID, t.methodID);
    } else {
        return false;
    }
}

#endif
{% endhighlight %}

PlatformUtil.cppは、XCodeの右ペインのTarget Membershipのチェックボックスを全部外して、
iOSではビルドされないようにする。

これで、任意のタイミングでPlatformUtil::showIntersAd()を呼べば、
Androidでもインタースティシャル広告が出る。
