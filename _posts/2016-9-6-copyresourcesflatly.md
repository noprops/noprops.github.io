---
layout: post
title: cocos2d-x でResoucesフォルダの中身をサブフォルダから出してassetsフォルダにコピーする
---

Resourcesフォルダに↓のようにファイルが入っているとする。

![1]({{site.baseurl}}/images/2016-09-06_resources.png)

cocos compile -p android --android-studio
としてAndroidプロジェクトをビルドするとResourcesフォルダの中身が全て
proj.android-studio/app/assetsフォルダにコピーされる。

上の図でfというファイルを使いたいとすると、iOSでは、fというファイル名を指定すれば使える。

{% highlight C++ %}
Sprite* sprite = Sprite::createWithSpriteFrameName("f.png");
{% endhighlight %}

しかし、これではAndroidで実行した時にエラーになる。
Androidでは、サブフォルダの中にあるファイルをファイル名だけで指定することはできない。

解決方法は２つある。

1.XCodeにResources内のファイルをフォルダごと追加する際にCreate folder references for any added foldersに
チェックを入れ、青いフォルダの状態で追加する。
そして、ファイル名を指定する際にはResourcesフォルダからの相対パスで指定する。
{% highlight C++ %}
Sprite* sprite = Sprite::createWithSpriteFrameName("folderA/folderD/f.png");
{% endhighlight %}

2.FileUtilsにsearchPathを追加する。
{% highlight C++ %}
bool AppDelegate::applicationDidFinishLaunching() {
    /*中略*/
    std::vector<std::string> searchPath;
    searchPath.push_back("folderA");
    searchPath.push_back("folderA/folderB");
    searchPath.push_back("folderA/folderC");
    searchPath.push_back("folderA/folderD");
    FileUtils::getInstance()->setSearchPaths(searchPath);
    /*中略*/
}
{% endhighlight %}
こうすることで、searchPathに指定したサブフォルダの中にあるファイルは全て
ファイル名だけで指定できるようになる。

しかし、例えばfolderAの中にサブフォルダがたくさんあり、
いちいち相対パスを書いたりsearchPathにフォルダ名を全て追加するのが
面倒くさい場合がある。

今回は、Resources/folderA以下のサブフォルダの中身を全て展開して、assetsフォルダにコピーされるようにしたい。
つまり、コピー後のassetsフォルダは下図のようになる。

![1]({{site.baseurl}}/images/2016-09-06_assets.png)

まず、proj.android-studio/build-cfg.jsonを変更する。

{% highlight json %}
"copy_resources": [
      {
          "from": "../Resources",
          "to": "",
          "exclude": [
              "folderA/*"
          ]
      }
  ]
{% endhighlight %}

excludeの部分を追記した。
これによってfolderAはassetsにコピーされなくなる。

次にproj.android-studio/app/build.gradleを変更する。

{% highlight gradle %}
task copyAssets(type: Copy) {
    from fileTree('../../Resources/folderA').files
    into 'assets/folderA'
}
{% endhighlight %}

これでResources/folderAの中身が展開されて、assets/folderAに入れられる。

searchPathにfolderAだけを追加する。
{% highlight C++ %}
bool AppDelegate::applicationDidFinishLaunching() {
    /*中略*/
    std::vector<std::string> searchPath;
    searchPath.push_back("folderA");
    FileUtils::getInstance()->setSearchPaths(searchPath);
    /*中略*/
}
{% endhighlight %}

これで、図のa~fのファイルが、iOSでもAndroidでも、
ファイル名のみの指定で使えるようになった。

{% highlight C++ %}
Sprite* a = Sprite::createWithSpriteFrameName("a.png");
...
Sprite* f = Sprite::createWithSpriteFrameName("f.png");
{% endhighlight %}

注意する点は、folderA内に同名のファイルが存在しないようにすること。
同名のファイルが存在した場合、assetsフォルダにコピーする際に上書きされてどちらかが消える。
