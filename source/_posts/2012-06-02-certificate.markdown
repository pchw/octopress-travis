---
layout: post
title: "[RubyMotion] 証明書の名前に日本語が入っていた時にrake device出来ない問題の解決策"
date: 2012-06-02 12:10
comments: true
categories: RubyMotion
---

RubyMotionを色々いじって記事書いてたのですが，全然実機で試してませんでした．

BT周りの実装をしてて，ついにシミュレータで確認ができなくなったので，実機に転送しようとして`rake device`したら

    "iPhone Developper: <your escaped name> : no identity found
    rake aborted!
    Command failed with status (1): [CODESIGN_ALLOCATE="/Developer/Platforms/iP...]

とか出やがりました．

マルチバイト文字を何かエスケープして`\123\456`みたいな形式にしてくれて，codesignに渡してしまうので，codesign側がそんなidentifyがねえよと言ってるわけですね．

[RubyMotion Project Management Guide#2.Configuration](http://www.rubymotion.com/developer-center/guides/project-management/#_configuration)
によれば，

    codesign_certificate
    The name of the certificate to use for codesigning, as a String. The default value is the first iPhone Developer certificate found in keychain. Example: "iPhone Developer: Darth Vader (A3LKZY369Q)".

とあるので，Rakefileに指定すればいいかなーとか思って

    $:.unshift("/Library/RubyMotion/lib")
    require 'motion/project'

    Motion::Project::App.setup do |app|
      # Use `rake config' to see complete project settings.
      app.name = 'hoge'
      app.codesign_certificate = "iPhone Developer: <your name>" # added
    end

こんな感じで，`app.codesign_certificate`を追加したらですね，

    rake aborted!
    Rakefile:9: invalid multibyte char (US-ASCII)
    Rakefile:9: invalid multibyte char (US-ASCII)
    Rakefile:9: syntax error, unexpected $end, expecting keyword_end

こういうエラーが出て，もしかしてRakefileにマルチバイト文字は指定出来ないのか，，，ピンチ！

    $ file Rakefile
    Rakefile: UTF-8 Unicode text

ちゃんとUTF8だよな，，，


## 解決策

と，まぁ色々悩んだんですが，結局解決策は

    # -*- coding: utf-8 -*-
    $:.unshift("/Library/RubyMotion/lib")
    require 'motion/project'

    Motion::Project::App.setup do |app|
      # Use `rake config' to see complete project settings.
      app.name = 'hoge'
      app.codesign_certificate = "iPhone Developer: <your name>"
    end

これが正解です．

`# -*- coding: utf-8 -*-` を追加すればいいだけでした．


