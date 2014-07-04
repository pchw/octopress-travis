---
layout: post
title: "[Synth] SynthとVue.jsを組み合わせてみた"
date: 2014-06-28 16:00
comments: true
categories: synth, Vue.js
---

[前回](/blog/2014/06/27/synth/)紹介したAPI-firstなフレームワーク[Synth](http://synthjs.com)と[Vue.js](http://vuejs.org)を組み合わせて使ってみる．

Synthの`$ synth new my_app`で生成されるコードは[Angular.js]()を使うようになっている．
Angular.js派ではないのでVue.js版に書き換えてみる．
ついでに[coffee-script](http://coffeescript.org)派なので，coffee-scriptで動くようにする．

ここまでのコードは[前回](/blog/2014/06/27/synth/)参照．Synth公式の[チュートリアル](http://www.synthjs.com/tutorial/)を済ませた時点のコードです．

モジュールの準備
===

Vue.jsインストール
----
まず，Vue.jsを使うため，インストールする．

```
$ synth install -f vue
$ synth install -f jquery
```

Angularのアンインストール
----

Angularは使わないので抜く．`$ synth uninstall -f angular`とやりたいが，未実装っぽい．
あと`$ synth install -f`時に`.bower.json`に`main`キーがないと`front/js/jsFiles`に記載されないっぽい

```
$ cd front
$ bower uninstall angular
$ bower uninstall angular-route
```

`front/bower.json` からも消しておく．
```
$ cat front/bower.json
{
  "name": "my_app",
  "private": true,
  "dependencies": {
    "vue": "~0.10.5",
    "jquery": "~2.1.1"
  }
}
```

`front/js/jsFiles`から不要になったファイルを削除する
```
$ cat front/js/jsFiles 
../bower_components/jquery/dist/jquery.js
../bower_components/vue/dist/vue.js
front-app.js
```

テンプレートの修正
====
DirectivesをAngular用からVue用に書き換える．
dateの`medium`はVueでは無いので後で`formatDate`を定義することにする．

```
$ cat front/html/tweets/getIndex.jade
textarea(v-text="newTweet")
button(v-on="click: publish()", v-attr="disabled: newTweet.length == 0") Publish
ul.tweet-timeline
  li.tweet(v-repeat="tweet : tweets")
    .content {{ tweet.content }}
    .date {{ tweet.created_at | formatDate}}
```

`front/index.jade` もVue用に色々修正
```
doctype html
html
  head
    meta(charset='utf-8')
    meta(http-equiv='X-UA-Compatible', content='IE=edge,chrome=1')
    meta(name='viewport', content='width=device-width, initial-scale=1.0')
    title= appName

    for cssFile in cssFiles
      link(rel="stylesheet", href=cssFile)

    // Preloaded Data
    script.
      window.preloadedData = !{data};
    for jsFile in jsFiles
      script(src=jsFile)
  body
    h1= appName

    div#content

    if preloadHTML
      script.template(type="v-template", id="template")
        != preloadHTML
```

分からなかった点
----
```
script(type="text/ng-template", id="#{preloadHTMLPath}")
```
とかなっててngRouteの`templateUrl`らへんでバインドしてた方法のVue版が分からなかったから普通にid指定にしてしまった．

Vueのバインディングとcoffee-script化
====
`front/front-app.js`を消して`front/front-app.coffee`にしてVue版にする
```
$ ->
  Vue.filter "formatDate", (v) ->
    v.replace /T|Z/g, " "

  content = new Vue
    el: "#content"
    template: "#template"
    data:
      newTweet: "write some tweet."

    created: ->
      if typeof window.preloadedData isnt "undefined"
        @$data = window.preloadedData
        window.preloadedData = null
      else
        $.ajax
          url: "/api/tweets"
          success: (data) =>
            @$data.tweets = data?.tweets


    methods:
      publish: ->
        $.ajax
          type: "POST"
          url: "/api/tweets"
          data:
            content: @$data.newTweet

          dataType: "json"
          success: (tweet) =>
            @$data.tweets.unshift tweet
            @$data.newTweet = ""
```

合わせて`front/js/jsFiles`も書き換え
```
$ cat front/js/jsFiles 
../bower_components/jquery/dist/jquery.js
../bower_components/vue/dist/vue.js
front-app.coffee
```

![image](https://cloud.githubusercontent.com/assets/1183484/3419737/e84b3ec4-fe76-11e3-8d09-132d59ed6c48.png)
これでめでたくTweetが表示されて，投稿できるようになった．

まとめ
====

- synthのAngularで動いてたのをVue版に書き直してみた
- そんなに変更は難しくなかった．これから新しいクライアントサイドのライブラリが出てきてもサッと変えてテスト出来そう．
- よく考えたら当たり前だけどbackend側全然いじってない
- API増やしたり，色々するとまたbackend側のディレクトリ構成とかfrontのhtml以下のディレクトリ構成とかハマるポイントが有るけどそれは次の話にする．