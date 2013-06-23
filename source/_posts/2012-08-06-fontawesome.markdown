---
layout: post
title: "[RubyMotion] motion-fontawesomeっていうgems作りました"
date: 2012-08-06 01:09
comments: true
categories: RubyMotion
---

1ヶ月ぶりの記事になってしまった！お久しぶりです．

motion-fontawesomeというgemsを作ったので宣伝がてらエントリー書きます！

## Retina環境の増加

Retina iPhone/iPadはかなり普及したんじゃないでしょうか？
ちょっと何か作るのにTabBarやToolbarのアイコンをいちいち探すのが面倒で，GlyphiconsのFREEを使っていたりしたんですが，非Retinaで荒いし他のアイコンを探そうとすると種類が少なかったり，Retinaじゃなかったりで面倒くさいことこの上ないです．
（そもそも無料で探してるのが悪かったりするんですけど）

そこで[FontAwesome](http://fortawesome.github.com/Font-Awesome/)というWebfontを使うと便利です．

## FontAwesomeとは

元々はTwitter Bootstrapで使えるようにデザインされたアイコンフォント集です．

フォントなので拡大縮小しても荒くならないため，Retinaでも例え超Retina的なのが出てきても対応できますし，色んなサイズが必要になる場面でも力を発揮してくれます．

## FontAwesomeをRubyMotionで使おうとすると

しかし，普通に使おうとすると，色々とやることがあって，

- `resources`にフォントを追加
- `app.fonts`に追加
- 欲しい記号を出すためのUTF8の文字コードを調べないといけない
- chr(Encoding::UTF_8)とか使って文字コードから文字へ変換してやらないといけない

みたいなことをやらないといけないわけです．

## motion-fontawesomeを作りました

そんな面倒なことやってられないので，上記のことを簡単にしてくれるgemsを作りました．

使い方は，

### Rakefileでrequire

    require 'rubygems'
    require 'motion-fontawesome'

### 使う

    label = UILabel.alloc.initWithFrame([[0, 0],[200, 200]])
    label.text = FontAwesome.icon(key)
    label.font = FontAwesome.fontWithSize(200.0)

### アイコンが出る！

![simulator](/images/fontawesome.png)

これだけです！上のスクリーンショットを見てもらってもわかると思いますけど，こんなに大きくしても綺麗です．

`FontAwesome.icon()`に指定する文字列は[ここ](http://fortawesome.github.com/Font-Awesome/#all-icons)から選んで，`icon-`を取ったものを指定してください．

（`icon-camera`だったら`camera`だけで良いです．）


