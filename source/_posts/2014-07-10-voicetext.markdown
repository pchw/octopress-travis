---
layout: post
title: "VoiceText Web APIのNode.jsライブラリ作った"
date: 2014-07-10 09:00
comments: true
categories: voicetext
---

GitHub: [https://github.com/pchw/node-voicetext](https://github.com/pchw/node-voicetext)

npm: [https://www.npmjs.org/package/voicetext](https://www.npmjs.org/package/voicetext)

前説
====

HOYAの[VoiceText Web API](https://cloud.voicetext.jp/webapi)というのが公開されていたのでNode.jsから叩くライブラリを作った．

VoiceText Web API(β版)は[ショウ君](http://dic.nicovideo.jp/a/%E3%82%B7%E3%83%A7%E3%82%A6%E5%90%9B)で有名な[VoiceText](http://voicetext.jp/)がスタンドアロンのソフトじゃなくてWebで公開されたやつ．

しかも今のところ無償で利用出来る．

Voice Text Web API自体はHTTPSのPOSTでデータを投げつけたらwavファイルを送り返してくるゴキゲンなやつ．

はじめにやること
====

VoiceText Web APIを使うには，まず利用登録が必要．
[利用登録画面](https://cloud.voicetext.jp/webapi/api_keys/new) から`名前``メールアドレス`などその他入れて登録すればすぐAPIキーが書かれたメールが来る．

Node.jsからVoiceText Web APIを使う
====

おもむろにnpm installする
```
$ npm install voicetext
```

`require('voicetext')`したらVoiceTextクラスが返ってくるからnewしつつAPIキーをブチ込む
```
VoiceText = require('voicetext');
voice = new VoiceText('APIキーをここにコピペする');
```

あとは色々パラメータをいじりつつメソッドチェーンして`speak`する．
基本的にメソッドは[公式ドキュメント](https://cloud.voicetext.jp/webapi/docs/api)のパラメータに合わせるようにした．
```
voice
.speaker(voice.SPEAKER.HIKARI)
.emotion(voice.EMOTION.HAPPINESS)
.emotion_level(voice.EMOTION_LEVEL.HIGH)
.pitch(200)
.speed(400)
.volume(200)
.speak('きょうもいちにちがんばるぞい', function(e, buf){
})
```
コールバックの第二引数にwavが詰まったBufferインスタンスが返ってくるので，
`fs.writeFile`とかしてwavに書き出せばもうすぐに再生できる形だ！

たぶん[node-wav](https://github.com/TooTallNate/node-wav)とか使ったらそのまま再生出来るかも．

というわけでNode.js版のVoiceTextライブラリのリポジトリは[https://github.com/pchw/node-voicetext](https://github.com/pchw/node-voicetext)

chatopsなところはBOTとかに喋らせるといいんじゃないかな．

