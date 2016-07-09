---
layout: post
title: cocos2d-x v3.10にFirebase導入(Android)
---

[cocos2d-xにFirebase導入(iOS)]({{site.baseurl}}/1cocos2dxfirebaseiOS/)

[cocos2d-x v3.10でadmob表示(iOS)]({{site.baseurl}}/2cocos2dxadmobiOS/)

Android Studioを開き、Open an existing android studio projectで
プロジェクト名/proj.android-studioフォルダを開く。

SDKManagerでインストールした最新のNDKだとcocos2d-xが対応していなくてビルドエラーになるので、
r10dをダウンロードして使う。(cocos2d-x v3.10を使っています。)
[android ndk r10d](http://dl.google.com/android/ndk/android-ndk-r10d-darwin-x86_64.bin)
参考：[過去のリビジョンのNDKを入手する方法](http://qiita.com/kishi-yama/items/1dab24942c12b9971d3e)

cocos2d-xのフォルダに移動して./setup.pyでNDK_ROOTを設定しておく。

Firebaseを入れる時はAndroid StudioのInstant Runをオフにしないといけないらしいので、
Android Studio->Preferences->Build,~~->Instant Runからオフにする。

[1](2016-07-09_1.png)

プロジェクト下のbuild.gradleのdependenciesに、
`classpath 'com.google.gms:google-services:3.0.0'`
と書く。

[2](2016-07-09_1.png)



{% endhighlight %}

AppController.mm
{% highlight Objective-C %}

