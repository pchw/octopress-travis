---
layout: post
title: "RubyMotionでUIViewを継承したViewを作る！"
date: 2012-06-25 11:34
comments: true
categories: RubyMotion
---

アプリを作っていると独自UIViewを作りたいことが多々あります．

ObjCなら

    self = [super initWithFrame:frame];
    if (self) {
        // 初期化処理
    }
    return self;

という感じのをinitWithFrameに書いて`//初期化処理`の部分にその独自クラスで持ってるものを初期化したりします．

RubyMotionに簡単に翻訳すると，

    class AwesomeView < UIView
        def initWithFrame(frame)
            self = super.initWithFrame(frame)
            if self
                # 初期化処理
            end

            self
        end
    end

と書きたいですが，これはダメダメでエラーになります．（selfを書き換えれないとか，initWithFrameがループして落ちるとか，，，）

RubyMotionでは

    class AwesomeView < UIView
        def initWithFrame(frame)
            if super(frame)
                # 初期化処理
            end

            self
        end
    end

とするのが正しそうです．
（おいおいそれイケてねーよっていう指摘があったら是非教えてください！！！）

あとは，AwesomeなViewを実装してくだけです．