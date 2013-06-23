---
layout: post
title: "[RubyMotion] Simulatorでは動くのに実機で動かなくてハマった話"
date: 2012-06-05 23:29
comments: true
categories: RubyMotion
---

## はじめに
アプリを作ってた時に，Simulatorでは動くのに`rade device`や`rake testflight`して実機で動かしたときに起動した瞬間落ちるという現象が起こって困った話です．

## RubyMotionは実機でのデバッグに弱い！
実機を繋ぐのはめんどくさいので，基本はSimulatorで開発してるのですが，さぁ実機でうごかすかーとなったときに何故か即死して困りました．

RubyMotionは`rake`したときにSimulatorが自動的に立ち上がってインタラクティブシェルで色々出来るという利点があるのですが，これは実機では出来ません．（よね？）

なので，どうしてもSimulatorがメインになってしまう気がします．

今回実機で動かして落ちるとかいうことでハマったわけですが，結果的には実機で動かしてXCodeのOrganizerのConsoleからログを見て原因調査をしました．
それ以外で実機オンリーのバグ調査はどうやるんでしょうか，，，XCodeだと実機で繋いでブレイクポイントとか貼れると思うのですが，RubyMotionだと方法がない，，，


## 今回のケース
さて，今回はどういう原因で落ちていたかです．

こういうSourceでした．

    class AppDelegate
      def application(application, didFinishLaunchingWithOptions:launchOptions)
        @window = UIWindow.alloc.initWithFrame UIScreen.mainScreen.bounds
        @storyboard = UIStoryboard.storyboardWithName(
            "StoryBoard", 
            bundle: nil)
        @window.rootViewController = @storyboard.instantiateViewControllerWithIdentifier "NavigationController"

        @window.makeKeyAndVisible
        true
      end
    end

リソースは `Storyboard.storyboard`です．

        @storyboard = UIStoryboard.storyboardWithName(
            "StoryBoard", 
            bundle: nil)

この部分で指定しているStoryboard名の大文字小文字が間違っているからですね．（正しくは`"Storyboard"`）
Simulatorでは何故か動くのですが，実機ではStoryboardが見つからないというログを吐いて落ちます．

## まとめ
RubyMotionを使っていると，どうしてもSimulator頼りの開発になりがちですが，あまりに実機で確認しないと最終フェーズで痛いことになるっていう話でした！

TwitterをRubyMotion検索で張ってるのですが，Twitter上でもTypoで動かなくて困ったぜみたいな話とか見ましたし，実行時評価も色々と大変ですね．
MethodとかPropertyなら静的解析とかでなんとかなりそうですが，今回みたいに文字列部分に関するところだとどうしても発見出来ない気がする，，，