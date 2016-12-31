---
layout: post
title: cocos2d-xにFirebaseを入れるとエラー
---

cocos2d-xのiOSプロジェクトにCocoapodsで3.8以降のFirebaseを導入すると、ビルド時に
ld: library not found for -lGoogleToolboxForMac
というエラーが出た。

解決方法は、target->Build Settings->Library Search Pathsに$(inherited)を入れる。

![1]({{site.baseurl}}/images/2016-12-31_1.png)
