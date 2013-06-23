---
layout: post
title: "チュートリアルを実現するjQueryプラグイン達"
date: 2013-04-07 17:14
comments: true
categories: jquery
---

最近チュートリアルを実現したり，特定の要素をハイライトしたり，用途的に似たようなライブラリがぽこぽこ出てきてどれがいいのか試したのでまとめてみる．

# Chardin

![chardin](/images/chardin.png)

<!--more-->

http://heelhook.github.com/chardin.js/

Apache License v2.0

## 概要
一度に複数の要素に対して，説明を書ける．

ステップで実行などはないが，completion handlerで再度実行すればできる．

Gmailの新機能で使われたチュートリアル風味．

実現方法はbody直下に黒半透明オーバーレイ要素を重ね，その後ろに説明用のレイヤーを追加する．ハイライトする要素をオーバーレイより大きいz-indexを追加し，上に持ってくる．

ハイライトする要素にz-indexがinline styleで付いていると隠れてしまう．

## 導入方法
css追加
```
<head>
  :
  <link href="/stylesheets/chardinjs.css" rel="stylesheet" />
  :
</head>
```

js追加
```
<script type="text/javascript" src="chardinjs.min.js"></script>
```

`data-intro`をハイライトする要素の属性に追加．
`data-position`に`left`・`top`・`right`・`bottom`のどれかを指定（説明文が出る位置）
```
<h1 data-step="1" data-intro="Hello all! :) This project's called Intro.js." class="">Intro<span style="font-weight: normal;">.js</span></h1>
```

ロード時に実行
```
<script type="text/javascript">
  $(function(){
    $('body').chardinJs('start');
  })
</script>
```

ハイライト開始時と終了時のイベントを受け取れる
```
<script type="text/javascript">
  $(function(){
    $('body').on('chardinJs:stop', function() {
      // 完了時の処理
    });
    $('body').chardinJs('start');
  })
</script>
```



# jQuery Highlighter

![jhighlighter](/images/jhighlighter.png)

http://www.dailymarkup.com/

MIT

## 概要
白いBoxshadowをつけてて、要素が光ってるようなハイライトができてかっこいい。

ハイライトしたい要素をcloneして、オーバーレイの上にコピペして実現．

指定した要素だけをcloneして，上に重ねるため，cssの書き方によっては（親要素を指定しているなど）デザインが崩れる．

アルファ版らしく，本番運用非推奨．

## 導入方法
css追加
```
<head>
  :
  <link href="/stylesheets/jquery.highlighter.css" rel="stylesheet" type="text/css"/> 
  :
</head>
```

js追加
```
<script type="text/javascript" src="/js/jquery.highlighter.js"></script> 
```

コントローラにハイライトしたい要素のid（ここでは`menu`）を指定し，relにハイライト時に表示されるメッセージを指定する
```
<div id="menu">ハイライトしたい要素<div/>
:
<a href="javascript:void(0);" title="menu" class="jhighlight" rel="<h1>HTML Markup for the Description</h1>">1</a> 
```

ロード時に実行
```
<script type="text/javascript">
  $(function(){
    $('.jhighlight').jhighlight(); 
  })
</script>
```


# jquery-tutor

![jquery-tutor](/images/jquery-tutor.png)

https://github.com/crocos/jquery-tutor

3-clause BSD license

## 概要
ステップ1、ステップ2のように流れに沿って要素をハイライトしたり，jsの処理を順次実行出来る．

実現方法は要素の周りに太い半透明borderを重ねて実現．

borderで実現しているのが影響してか見た目が良くない．

## 導入方法

css追加
```
<head>
  :
  <link rel="stylesheet" type="text/css" href="/stylesheets/jquery.tutor.min.css">
  :
</head>
```

js追加
```
<script src="/js/jquery.tutor.min.js"></script>
```

stepsの配列に順次ハイライトしたい要素とメッセージのオブジェクト(target, message)やjsのfunctionをそのまま入れたりしていく．
deferredを使うと非同期実行などもできる．
```
<script type="text/javascript">
  $(function(){
    $.tutor({
      steps:[
      {
        target: '.selector',
        message: 'Step 1: Push the button.'
      },
      function() {
        alert('Step 2: Click "OK"');
      },
      function(dfd) {
        // async callback
        setTimeout(function() {
          alert('Step 3: Hi.')
        }, 1500);

        return dfd.promise();
      },
      function() {
        // `$.ajax` returns *Deferred Object*.
        return $.ajax({
          // ...
        });
      },
      {
        target: '.selector',
        message: 'That\'s all. Enjoy!'
      }
    ]
    });
  })
</script>
```


# Introjs

![introjs](/images/introjs.png)

http://usablica.github.io/intro.js/

MIT

## 概要
角丸四角でハイライトされて，ステップ数のバッヂや次のステップへ進む，前のステップへ戻るボタンなどが表示されて親切な感じ

黒半透明のオーバーレイ要素を対象要素の後ろにz-index高めで追加して対象の要素に`.introjs-showElement`を付けてz-indexをオーバーレイ要素より高めてハイライトする形式．
inline styleでz-indexを追加していると後ろに隠れてしまってハイライト要素が見えないという状態になる．
画面端の要素をハイライトすると，バッヂが隠れてしまったりもする．

## 導入方法
css追加
```
<head>
  :
  <link href="/stylesheets/introjs.min.css" rel="stylesheet" />
  :
</head>
```

js追加
```
<script type="text/javascript" src="/js/intro.min.js"></script>
```

`data-step`と`data-intro`をハイライトする要素の属性に追加．
`data-position`は指定したければ指定する．
```
<h1 data-step="1" data-intro="Hello all! :) This project's called Intro.js." class="">Intro<span style="font-weight: normal;">.js</span></h1>
```

ロード時に実行
```
<script type="text/javascript">
  $(function(){
    introJs().start();
  })
</script>
```

ステップ移行時や完了，離脱時のcallbackを設定できる．

# まとめ

見た目的にはchardinやintro.jsが良い感じ．

チュートリアル的に使うならintro.jsかな．

1枚画面でさっと機能を提示したいだけならchardinが良いのではないか．

この種のライブラリは複雑な構成（z-indexをjsでinline styleとして書き換えたり，absoluteやfixedで配置をダイナミックにしているなど）を取ってると微妙に要素が隠れたり，ハイライトがずれたりするので注意が必要．