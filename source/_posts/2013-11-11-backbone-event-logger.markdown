---
layout: post
title: "Backbone.jsのイベントをデバッグ出力するやつをモジュール化した"
date: 2013-11-11 09:10
comments: true
categories: backbone.js
---

土日で艦これイベント海域E-4をなんとか突破したぽちです，おはようございます．
今日は島風轟沈の日らしいですね．

ktty1220 さんの[Backbone.jsで全イベントをconsoleに出力するデバッグ用スクリプト](http://qiita.com/ktty1220/items/f1bb5b4eb48839de8394)が便利だったので，使ってたのですが毎回`backbone.debug.js`をプロジェクトにコピーして回るのが大変だったのでbowerでサクッとインストール出来るようにしてみました．

[backbone-event-logger](https://github.com/pchw/backbone-event-logger)

```
$ bower install backbone-event-logger
```
でインストールされます．

あとは
```
<script src="/bower_components/backbone/backbone-min.js"></script>
<script src="/bower_components/backbone-event-logger/backbone-event-logger.min.js"></script>
```
みたいに読みこめば，ガンガンログが出ます．

オリジナルと変更した部分としては
- `Backbone[<ModelとかRouterとか>]::initialize`で自動的にログ出力を有効化
- デフォルトで`@constructor.name`をログに出すようにした
- coffee化
- 名前を`.debug`から`.event-logger`へ（デバッグっていうよりロガーだなと思ったので）
- AMD/RequireJSでも読み込めるようにしてみた（けど試してない）


### こんな感じで出ます
```
23:04:42 Model:ContentModel  remove [> [ContentModel, Contents, Object]
```
`ContentModel` って書いてる部分が`@constructor.name`で出してるところ．

オリジナルにあったスタイルの変更等は`bower install`した後に
`bower_components`内を変更しないといけなくて，悩んでいるところ．
（自動適用にしたので外部から設定する口を用意してもイマイチかなと）

ところで`bower install`した時って`backbone-event-logger.min.js`や`backbone-event-logger.js`
以外のファイルは配置されるべきなんだろうか？ルールがわからない．
（backbone.jsのbowerも配置されてた）

そういうわけでPR/forkどうぞ [backbone-event-logger](https://github.com/pchw/backbone-event-logger)
