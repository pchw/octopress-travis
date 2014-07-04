---
layout: post
title: "[Synth] ルーティングを増やしてみる"
date: 2014-07-04 10:00
comments: true
categories: synth, Vue.js
---

前説
====
[前回](/blog/2014/06/28/synth-with-vuejs/)でAngularだけではなく自分の好きなクライアントサイドMVCフレームワークが使えるようになったところで，
次はサーバサイドでAPIの追加が必要になってくる頃合いなのでその辺りを触ってみる．

[synth](http://synthjs.com)は初回にも少し触れたが，`back/resources`以下がURLにマッピングされている感じになっている．
そのため，`back/resources`以下にディレクトリを掘って，その中のファイルに`exports.XXXXX`的なのを書いておけば，
そのままURLのルーティングに反映される．

実際の手順を追ってみる．

backend側の改修
====

`back/resources` にディレクトリ追加する．
```
$ synth install -b bluebird
$ mkdir back/resources/messages
$ cat back/resources/messages/filename_you_want.coffee
Promise = require 'bluebird'
sample = ->
  new Promise (resolve)->
    setTimeout ->
      resolve 'something'
    , 3000

exports.getIndex = (req, res)->
  sample()
  .then (d)->
    something: d
```

synthは`back/resources`内の`.js`または`.coffee`のファイルの`exports`を見るため，
ファイル名はなんでもいい，
分かりやすいのを付けるのがいいらしい．

サーバサイドはPromiseを返さないといけないらしいので，[bluebird](https://github.com/petkaantonov/bluebird)を使ってPromiseを返してる．

プリロードテンプレートの改修
====

`front/html`内もbackend側に合わせて変える必要がある．
（`preloadHTML`が必要な場合）

`front/html`内は`back/resources`内と良く似ているが，ディレクトリに入れるファイル名は規則がある．
`exports.getIndex`とかだと`getIndex.jade`または`getIndex.html`というファイル名にする必要がある．
backend側のAPIと合わせなければいけない．
```
$ cat front/html/messages/getIndex.jade
h2 Messages
p {{ something }}
```

frontend側の改修
====

今frontend側にはroutingを入れてなかったので，`/messages`にアクセスしてもVue側でエラーになる．
`v-view`と`Vue.component`を使ってルーティングするロジックを入れる．

まずは`index.jade`の方に`v-view`を
```
$ cat front/index.jade
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
      div(v-view="currentView")

    if preloadHTML
      script.template(type="v-template", id="template")
        != preloadHTML
```

次に`front-app.coffee`の方にURLによって`v-view`の`currentView`をcomponentで置換するように書く．
```
$ cat front/js/front-app.coffee
$ ->

  routes = ['tweets', 'messages']

  getRoute = ->
    path = location.hash.replace(/^#!\?/, '') or 'tweets'
    if routes.indexOf(path) > -1 then path else 'tweets'
    
  Vue.filter "formatDate", (v) ->
    v.replace /T|Z/g, " "

  Vue.component 'tweets',
    template: '#template'
    created: ->
      @$data.newTweet = "write some tweet."
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

    Vue.component 'messages',
      template: '#template'
      created: ->
        if typeof window.preloadedData isnt "undefined"
          @$data = window.preloadedData
          window.preloadedData = null
        else
          $.ajax
            url: "/api/messages"
            success: (data) =>
              @$data.something = data?.something

  app = new Vue
    el: "#content"
    data:
      currentView: do getRoute
      routes: routes
```


アクセスしてみる
====

http://localhost:5000/messages で `something`的な文字が出るページが表示されて，
http://localhost:5000/tweets で今までのTweets一覧が出るページが表示されるようになる


まとめ
====

- synthでルーティングを追加してみた
- back/resource以下にディレクトリを増やしてファイルを突っ込むだけ
- 後から見た時にURLとディレクトリ構造が一致しているので分かりやすい
- front側もプリロードのテンプレートがfront/html以下に同じディレクトリ構造があるので分かりやすい
