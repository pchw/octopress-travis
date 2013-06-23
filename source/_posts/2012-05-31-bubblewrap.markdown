---
layout: post
title: "[RubyMotion]Bubble-Wrapを使ってみた"
date: 2012-05-31 22:44
comments: true
categories: RubyMotion
---

今日はBubble-Wrapというライブラリを使ってみました．

[Bubble-Wrap](http://matt.aimonetti.net/BubbleWrap/)

Bubble-Wrap はめんどくさいことを簡単にしてくれるヘルパーとかRubyっぽく書けない部分をよりRubyっぽくしてくれるライブラリです．

例えば，[昨日書いた](http://pchw.github.com/blog/2012/05/30/resource/)リソースの読み込みの所で，

    path = NSBundle.mainBundle.resourcePath.stringByAppendingPathComponent "test.txt"

とか書きましたけど，`NSBundle.mainBundle.resoucePath`とかめんどくさいですよね．

それを，

    path = resources_path.stringByAppendingPathComponent "text.txt"

にしてくれたりします．
（他にも色々と`documents_path`とか`notification_center`とか）

    alert "Hoge"

とするだけで，

    alert = UIAlertView.alloc.initWithTitle "Hoge",
        message: nil,
        delegate: nil,
        cancelButtonTitle: 'OK',
        otherButtonTitles: nil
    alert.show

とやってくれたりするのもあります．

色々ありますが，Wikiを見ても一部しか載っていないので，何が出来るか全部把握するには，[Githubでコード](https://github.com/mattetti/BubbleWrap/tree/master/lib/bubble-wrap)を見るしか無いっぽいです．

他にも便利なものがあって，HTTPアクセスやJSONのラッパーも提供されています．
例えば，Twitter APIにアクセスして，取得したJSONのデータのアクセスは

    BubbleWrap::HTTP.get("https://api.twitter.com/1/users/show.json?screen_name=TwitterAPI&include_entities=true") do |response|
        json = BubbleWrap::JSON.parse response.body.to_str
        p json['name']
    end

のように書けます．
元が NSURLConnectionを作って，delegateで受けてみたいなことを書かないのと比べるととても簡単です．

`response` も`body`だけでなく，`headers`や`status_code`とか`ok?`（200 OKかどうかを調べる）などもあります．


他にも，device情報を取ったり，gestureを簡単に扱ったりするものもあるので，有効に活用して素早くRubyMotionでアプリ作って行きましょう！

