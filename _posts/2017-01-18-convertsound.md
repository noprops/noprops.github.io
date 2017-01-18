---
layout: post
title: wavファイルをm4aとoggに変換するスクリプト
---

ツクールMVで音声ファイルを使用するには、m4aとoggの２種類用意する必要がある。
wav形式のファイルをm4aとoggに変換するシェルスクリプトを書いた。

audioconvert.sh

{% highlight bash %}
#!/bin/sh
SCRIPT_DIR=`dirname $0`
cd ${SCRIPT_DIR}

find . -name "*.wav" -type f -maxdepth 1 | while read waveFile
do
#	echo "waveFile = ${waveFile}"
	oggFile=${waveFile%.wav}.ogg
	m4aFile=${waveFile%.wav}.m4a
	ffmpeg -i ${waveFile} -vn -ac 2 -ar 44100 -ab 128k -acodec libvorbis -f ogg ${oggFile}
	ffmpeg -i ${waveFile} -vn -ac 2 -ar 44100 -ab 128k -acodec aac -strict experimental ${m4aFile}
done
{% endhighlight %}

ffmpegがインストールされていない場合は、インストールする。

$ brew install ffmpeg

上記の内容のaudioconvert.shファイルを作って、同じフォルダ内に変換したいwavファイルを置き、audioconvert.shを実行する。
