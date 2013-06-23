---
layout: post
title: "AppReviewViewerが良い！"
date: 2012-05-26 22:52
comments: true
categories: AppStore
---

今日は「第２回 iphone_dev_jp 東京iPhone/Mac勉強会」に参加してきました．

そこで面白いものが紹介されていて，超便利だったので紹介します．

![appreviews](/images/appreviews.png)

mixiのiOSアプリ開発という発表をなさっていた@k_kinukawaさんがAppReviewViewerというツールを紹介されていました．

[AppReviewViewer](https://github.com/punchdrunker/AppReviewViewer)

これは，アプリのIDを登録すると，レビューページをスクレイピングしてどのバージョンでどんな評価が付いたかを一覧したり見やすく表示してくれるものです．

また，mecabが入っているとレビュー文を形態素解析してくれて頻出名詞をタグクラウドみたいに見せてくれたりします．

**Gemfileが追加されたので，Gemfileを自分で追加する必要はなくなりました！**

上記の[github](https://github.com/punchdrunker/AppReviewViewer)ページに行ってもらうと分かりますが，依存ライブラリが書いてあるんですが，Gemfileが無くてめんどくさいので書きました．

{% gist 2793067 Gemfile %}

AppReviewViewerをGithubからcloneした後に入れちゃってください．

    $ git clone git://github.com/punchdrunker/AppReviewViewer.git
    $ cd AppReviewViewer
    $ cat > Gemfile
    (上記gistを入れる)
    $ bundle install
    $ ruby app.rb

こんな感じでやれば，AppReviewViewerが`http://localhost:4567`に起動します．

左端に`+Add app`があるので，選択してアプリのIDと名前を入れます．

![appreviewsadd](/images/appreviewsadd.png)

ちなみにアプリのIDはAppStoreのアプリのページを開くと，

![id](/images/appreviewsid.png)

ココらへんに書いていたりします．

アプリのIDと名前が入れられたら，

    $ ruby scripts/fetch.rb

とすれば，スクレイピングが始まるので，終わったら

    $ ruby app.rb

としてまた`http://localhost:4567`を開いてみましょう．Reviewが見えるはずです！