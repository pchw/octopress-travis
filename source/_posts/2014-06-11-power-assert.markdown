---
layout: post
title: "coffee-scriptでpower-assertを使ったテストを書く"
date: 2014-06-11 00:20
comments: true
categories: power-assert
---

tl;dr
----
1. @t_wadaさん神
2. coffee-scriptで[power-assert](https://github.com/twada/power-assert)使うときは[espower-coffee](https://github.com/twada/espower-coffee)を使おう

あらすじ
----
ちょっと前に話題になっていた[power-assert](https://github.com/twada/power-assert)を使ってみようとしました．

こんな感じのを用意しました．

```
assert = require 'power-assert'

describe 'array', ->
  beforeEach ->
    @arr = [1,2,3]
  describe '#indexOf()', ->
    it 'should return index when the value is present', ->
      zero = 0
      two = 2
      assert(@arr.indexOf(zero) is two)
```
んで
```
$ mocha --require intelli-espower-loader test/power-assert.coffee
```
みたいに実行したら
```
  array
    #indexOf()
      1) should return index when the value is present


  0 passing (4ms)
  1 failing

  1) array #indexOf() should return index when the value is present:
     AssertionError: false == true
```
あれ，，，ログがショボい，，，
```
  array
    #indexOf()
      1) should return index when the value is present


  0 passing (10ms)
  1 failing

  1) array #indexOf() should return index when the value is present:
     AssertionError: # /Users/pochi/Documents/development/(snip)/test/power-assert.js:16

assert(this.arr.indexOf(zero) === two)
            |   |       |     |   |   
            |   |       |     |   2   
            |   -1      0     false   
            [1,2,3]                   

[number] two
=> 2
[number] this.arr.indexOf(zero)
=> -1

```
こんな感じのを期待したのに！！

<blockquote class="twitter-tweet" lang="en"><p>power-assert使ってみたのに思ったように出力でない．「AssertionError: false == true」って出てしまった</p>&mdash; ぽち＠ (@pchw) <a href="https://twitter.com/pchw/statuses/475911097069744130">June 9, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

とか言ってたら[@yosuke_furukawa](https://twitter.com/yosuke_furukawa)が色々助言をくれて，こうなって

<blockquote class="twitter-tweet" data-conversation="none" lang="en"><p><a href="https://twitter.com/pchw">@pchw</a> intelli-espower-loaderへのpull request案件な気がしてきましたね。 <a href="https://t.co/F5tHM0NDqh">https://t.co/F5tHM0NDqh</a></p>&mdash; Yosuke FURUKAWA (@yosuke_furukawa) <a href="https://twitter.com/yosuke_furukawa/statuses/475914266860482560">June 9, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

さらに中をみていくと

<blockquote class="twitter-tweet" lang="en"><p>いや待てよこれ*.coffeeにしてもならないぞ，，，espower-loaderの中の方まで見ていかないと</p>&mdash; ぽち＠ (@pchw) <a href="https://twitter.com/pchw/statuses/475914928562266112">June 9, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

さらに色々調べてたら数カ月前の@mizchiの発言が

<blockquote class="twitter-tweet" lang="en"><p>昨日power-asset 使おうとしてみたけど coffeeと espower が上手く動かんかった</p>&mdash; 俺は平気だよ (@mizchi) <a href="https://twitter.com/mizchi/statuses/468274394368708608">May 19, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

という感じで動作しない感じでした．

TypeScriptでの前例があるように

<blockquote class="twitter-tweet" data-conversation="none" lang="en"><p><a href="https://twitter.com/pchw">@pchw</a> なるほど。。vvakameさんがtypescriptでどうやってやってんだろーと思って中身見たらgrunt-espower使ってたのでそういうことですね。 <a href="https://t.co/mFYljl9QRw">https://t.co/mFYljl9QRw</a></p>&mdash; Yosuke FURUKAWA (@yosuke_furukawa) <a href="https://twitter.com/yosuke_furukawa/statuses/475915916807376896">June 9, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

のようにgruntを使ってjsにコンパイルしてそれを食わせるという方法しか無いように思えた時に @t_wada 降臨．

<blockquote class="twitter-tweet" data-conversation="none" lang="en"><p><a href="https://twitter.com/pchw">@pchw</a> <a href="https://twitter.com/yosuke_furukawa">@yosuke_furukawa</a> 反応遅れてすみません。 CoffeeScript で power-assert 使いたい人意外といらっしゃるみたいですね……Node のローダーではちょっと向かい風ですが、手を考えます <a href="https://t.co/P1WY2nV1fz">https://t.co/P1WY2nV1fz</a></p>&mdash; Takuto Wada (@t_wada) <a href="https://twitter.com/t_wada/statuses/475935334874636288">June 9, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

そしてその翌日

<blockquote class="twitter-tweet" data-conversation="none" lang="en"><p><a href="https://twitter.com/pchw">@pchw</a> <a href="https://twitter.com/yosuke_furukawa">@yosuke_furukawa</a> espower-coffee を作ってみたので試してみてください <a href="https://t.co/U8NM7Eji9k">https://t.co/U8NM7Eji9k</a></p>&mdash; Takuto Wada (@t_wada) <a href="https://twitter.com/t_wada/statuses/476235243221835776">June 10, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

[espower-coffee](https://github.com/twada/espower-coffee) が公開されてました！

使い方
----
```
$ npm install espower-coffee --save-dev
$ mocha --require 'espower-coffee/guess' test/**/*.coffee
```

`grunt-mocha-test`を使っているなら
```
mochaTest:
    options:
      reporter: 'spec'
      require: 'espower-coffee/guess'
      colors: true
      timeout: 10000
    src: [
      'test/power-assert.coffee'
      ] 
```
こんな感じ．`compilers: 'coffee:coffee-script'`は要らないみたいです．

すると
```
  array
    #indexOf()
      1) should return index when the value is present
 
 
  0 passing (10ms)
  1 failing
 
  1) array #indexOf() should return index when the value is present:
     AssertionError: # /Users/pochi/Documents/development/(snip)/test/power-assert.coffee:15
 
assert(this.arr.indexOf(zero) === two)
            |   |       |     |   |   
            |   |       |     |   2   
            |   -1      0     false   
            [1,2,3]                   
 
[number] two
=> 2
[number] this.arr.indexOf(zero)
=> -1
```
こんなのが得られて，coffee-scriptでpower-assertが使えました！

ちなみに他のファイルとかはこんな感じでやりました．
{% gist 3fbed13bd69fb31faca6 %}

注意点
----
- coffee-scriptは1.7.1以上を要求します
- coffee-errorsなどを併用するとショボいログに戻ります
- AssertionErrorの行数が実際の.coffeeの行数ではありません．（多段SourceMap対応するまでは仕様とのこと）
