---
layout: post
title: "Backbone.jsでView.renderを書く場合のセオリー"
date: 2013-12-04 05:26
comments: true
categories: Backbone.js
---

こんにちは、ぽちです．
みなさんBackbone.js使ってますか？Marionetteですか？Chaplinですか？それともngが付いちゃうアレですか？

さて今日は、Backbone.jsのお話です．

タイトルは「Backbone.jsでrenderした時に空divを無くす方法」とかにしようと思ったのですが，なんだかそれに限らない気もしたのでブチあげました．

## 時間がない人へのまとめ
忙しい人のために，一言でまとめるとこうです．
```
Class HogeView extends Backbone.View
  initialize: ->
    @listenTo @model, ‘change’, render
    do @render
  render:->
    $oldel = @$el
    $newel = $(<render HTML by template engine>)
    @setElement $newel
    $oldel.replaceWith $newel
```

次の段落からは簡単な形から流れに沿って最終系になるように説明していきます．

## ベーシックなよくある書き方

```
class HogeView extends Backbone.View
  el: '.hoge'
  initialize: ->
    do @render
  render:->
    @$el.html ich.hoge_tmpl()

<hoge.jade>
.hoge
script#hoge_tmpl(type="text/html")
  p this is hoge
```
よくあるのだと、こういう書き方しますよね．
この場合、elを指定しているので、既に存在する要素にViewを紐付けています．

_ここで使っているテンプレートエンジンは[ICanHaz.js](http://icanhazjs.com/)です．_

## 要素を作成する場合
次は，既存のの要素ではなく，あるボタンをクリックして
要素を作成するケースを考えてみてください．
クリック時に始めてclassがnewされ，それによってDOMに追加されるケースです．
```
Class HogeView extends Backbone.View
  initialize: ->
    do render
  render:->
    @$el.html(<HTML rendering by template engine>)

Class ButtonView extends Backbone.View
  el: '.add-btn'
  events:
    ‘click': 'append'
  append:->
    v = new HogeView()
    @$el.append(v.el)

<hoge.jade>
a.add-btn add
script#hoge_tmpl(type="text/html")
  .hoge
    p this is hoge 
```

こんな感じですかね．

ここで問題が．
Hogeviewにはelを指定していないので，new時にデフォルトで新しい空divがelになります．
そのあと，renderで.htmlするので空divの中にテンプレートの内容が入ったものがelになります．
```
<div>
  <div class=“hoge”>
    <p>this is hoge</p>
  </div>
</div>
```

## className/tagNameの導入？
これを回避するには，`className`や`tagName`といったpropertyを使うのが一つの方法です．
```
class HogeView extends Backbone.View
  className: '.hoge'
  initialize: ->
    do @render
  render:->
    @$el.html ich.hoge_tmpl()

class ButtonView extends Backbone.View
  el: '.add-btn'
  events:
    'click': 'append'
  append:->
    v = new HogeView()
    @$el.append v.el

<hoge.jade>
a.add-btn add
script#hoge_tmpl(type="text/html")
    p this is hoge 
```

ただし，`className`を用いるためにテンプレートの一番親の要素(`.hoge`)をHogeViewに持ってきたため見通しが悪い．

## setElementを使う
そこで`setElement`を使います．
```
class HogeView extends Backbone.View
  initialize: ->
    do @render
  render:->
    @setElement ich.hoge_tmpl()

class ButtonView extends Backbone.View
  el: '.add-btn'
  events:
    'click': 'append'
  append:->
    v = new HogeView()
    @$el.append v.el

<hoge.jade>
a.add-btn add
script#hoge_tmpl(type="text/html")
  .hoge
    p this is hoge 
```

とすると空divに挟まずに設定できます．

## Modelの更新をトリガーにViewを更新する場合
この方法でも，以下のケースのように一度`setElement`をしたものを更に`setElement`で置き換えるようなコードでは正しく動作しません．
```
class HogeModel extends Backbone.Model
  defaults:
    index: 0
  initialize: ->
    @listenTo Backbone, 'inc', @inc
  inc: ->
    i = @get 'index'
    @set 'index', i+1
class HogeView extends Backbone.View
  initialize: ->
    @index = 0
    @listenTo @model, 'change', @render
    do @render
  render:->
    @setElement ich.hoge_tmpl
      index: @index++

class ButtonView extends Backbone.View
  el: '.add-btn'
  events:
    'click': 'render'
  initialize: ->
    v = new HogeView
      model: new HogeModel()
    @$el.append v.el
  render:->
    Backbone.trigger 'inc'

<hoge.jade>
script#hoge-tmpl(type=“text/html“)
  .hoge
    p this is hoge
```

## 最終系
これを解決するにはこのようにしてあげれば良いです．
```
class HogeModel extends Backbone.Model
  defaults:
    index: 0
  initialize: ->
    @listenTo Backbone, 'inc', @inc
  inc: ->
    i = @get 'index'
    @set 'index', i+1
class HogeView extends Backbone.View
  initialize: ->
    @index = 0
    @listenTo @model, 'change', @render
    do @render
  render:->
    $oldel = @$el
    $newel = ich.hoge_tmpl
      index: @index++
    @setElement $newel
    $oldel.replaceWith $newel

class ButtonView extends Backbone.View
  el: '.add-btn'
  events:
    'click': 'render'
  initialize: ->
    v = new HogeView
      model: new HogeModel()
    @$el.append v.el
  render:->
    Backbone.trigger 'inc'

<hoge.jade>
script#hoge-tmpl(type=“text/html“)
  .hoge
    p this is hoge
```

## Conclusion
```
Class HogeView extends Backbone.View
  initialize: ->
    @listenTo @model, ‘change’, render
    do @render
  render:->
    $oldel = @$el
    $newel = $(<render HTML by template engine>)
    @setElement $newel
    $oldel.replaceWith $newel
```
のようにすることで，テンプレートの構造そのままをViewに適用することが出来て，正しく表示も更新されます．

## refs
[http://stackoverflow.com/questions/11594961/backbone-not-this-el-wrapping](http://stackoverflow.com/questions/11594961/backbone-not-this-el-wrapping)
