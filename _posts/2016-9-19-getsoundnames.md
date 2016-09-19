---
layout: post
title: cocos2d-xで音声ファイルの名前を自動で取得してpreloadする
---

cocos2d-xのSimpleAudioEngineやexperimental::AudioEngineで音声ファイルを鳴らすときは、アプリ起動時等にpreloadしてから使う。

{% highlight C++ %}
experimental::AudioEngine::preload("a.mp3");
{% endhighlight %}

音声ファイルがたくさんある場合は、それらのファイル名を全て指定するのが面倒だし、
ファイルを追加、削除、リネームした場合に
ファイル名を間違える可能性がある。

音声ファイル名を自動で取得して、preloadするようにしたい。

Resourcesフォルダ内に、soundNames.txtという空のテキストファイルを追加する。
![1]({{site.baseurl}}/images/2016-09-19-1.png)

Resources/seフォルダ内のmp3ファイル名一覧を取得して、soundNames.txtに書き込む。

Target->Build Phasesを開き、
左上の+ボタンを押してNew Run Script Phaseを追加する。

![2]({{site.baseurl}}/images/2016-09-19-2.png)

{% highlight shell %}
output="${PROJECT_DIR}/../Resources/soundNames.txt"
echo '' > $output

se="${PROJECT_DIR}/../Resources/se"
for file in `find $se -name "*.mp3"`; do
echo ${file##*/} >> $output
done
{% endhighlight %}

このRun Script PhaseはCopy Bundle Resourcesの上に移動する。

![3]({{site.baseurl}}/images/2016-09-19-3.png)

これで、ビルドした時にsoundNames.txtに音声ファイル名が書き込まれる。

![4]({{site.baseurl}}/images/2016-09-19-4.png)

あとは、アプリ起動時にsoundNames.txtから読み込んだファイル名を指定してpreloadする。

このプロジェクトではRRGSoundManagerというクラスで
音声を扱っているので、全ての音声ファイルをpreloadするメソッドを追加する。

RRGSoundManager.h
{% highlight C++ %}
#include "cocos2d.h"

#define sharedSoundManager RRGSoundManager::getInstance()

class RRGSoundManager : public cocos2d::Ref
{
private:
    bool _isPlaying;
    std::string _bgm;
    int _audioID;
public:
    RRGSoundManager();
    virtual ~RRGSoundManager();
    static RRGSoundManager* getInstance();
    static void destroyInstance();
    void playBGM(const std::string& bgm);
    void stopBGM();
    void playSE(const std::string& sound);
    void pauseAll();
    void resumeAll();
    //追加
    void preloadAll();
};
{% endhighlight %}

RRGSoundManager.cpp
{% highlight C++ %}
#include "RRGSoundManager.h"
#include "audio/include/AudioEngine.h"

using namespace std;
USING_NS_CC;

/*中略*/

namespace {
    string trim(const string& str, const char* trimCharacterList = "; \t\v\r\n\"\'") {
        string result;
        string::size_type left = str.find_first_not_of(trimCharacterList);
        
        if (left != string::npos) {
            string::size_type right = str.find_last_not_of(trimCharacterList);
            result = str.substr(left, right - left + 1);
        }
        
        return result;
    }
    
    list<string> split(string str, string delim) {
        list<string> result;
        long cutAt;
        
        
        while( (cutAt = str.find_first_of(delim)) != str.npos ) {
            if(cutAt > 0) {
                result.push_back(trim(str.substr(0, cutAt)));
            }
            str = str.substr(cutAt + 1);
        }
        if(str.length() > 0) {
            result.push_back(trim(str));
        }
        
        return result;
    }
}

void RRGSoundManager::preloadAll()
{
    FileUtils* fileUtil = FileUtils::getInstance();
    string fullpath = fileUtil->fullPathForFilename("soundNames.txt");
    string strings = fileUtil->getStringFromFile(fullpath);
    
    if (strings.empty()) return;
    
    list<string> stringLines = split(strings, "\n");
    for(list<string>::iterator begin = stringLines.begin(),
        end = stringLines.end();
        begin != end;
        ++begin)
    {
        CCLOG("soundName = %s",(*begin).c_str());
        experimental::AudioEngine::preload(*begin);
    }
}
{% endhighlight %}

preloadAllメソッドを起動時に呼ぶ。

AppDelegate.cpp
{% highlight C++ %}
bool AppDelegate::applicationDidFinishLaunching() {
	/*中略*/
	sharedSoundManager->preloadAll();
	/*中略*/
	return true;
}
{% endhighlight %}
