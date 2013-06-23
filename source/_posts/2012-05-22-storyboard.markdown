---
layout: post
title: "Storyboardを使ってRubyMotionで開発する方法"
date: 2012-05-22 21:53
comments: true
categories: RubyMotion
---

![](https://lh6.googleusercontent.com/-ptxn4wJhqOc/T69nu2xHtLI/AAAAAAAAAYY/JKxXu3OAsYE/B6AD8776-C7D5-4F4A-A9AA-A8143868D413.jpg)

RubyMotionでもStoryBoardが使えるらしいので，試してみた．

# ひな形作成

    $ motion create SB
    $ cd SB


# XcodeでStoryboardを作成
おもむろにXCodeを開きます．

`File>New>File`で`iOS>User Interface>Storyboard`を選びます．
![](https://lh4.googleusercontent.com/-mLcH1PENo2w/T69nyXr7yTI/AAAAAAAAAY4/RT_Q6EIAjIY/6335FCDB-E334-486F-9A97-C62A4212F9CE.jpg)

## 初めに表示されるViewControllerを作成

UIViewController設置．
![](https://lh4.googleusercontent.com/-A3bJEOrvzrE/T69n2eaXo3I/AAAAAAAAAZM/gMeA45JyTv4/C59A9802-25A6-4A66-B8B7-F247989BD60B.jpg)

Identifierを`First`とか付けます．
![](https://lh6.googleusercontent.com/-m0viQeqlxi8/T69nv3v32qI/AAAAAAAAAYg/RR6q5pe4rRg/655CD9E6-3C77-4930-95CD-D71E1CCEAFD1.jpg)

UILabelとUIButtonを追加します．

## Segueで遷移する先のViewControllerを作成
もう一つUIViewController設置．
![](https://lh5.googleusercontent.com/-pzw1VqoabCw/T69n3leryQI/AAAAAAAAAZY/wpds7icQfMk/3D867F91-1A11-442A-9E71-EE5F0475A81C.jpg)

Identifierを`Second`とか付けときます．


## Segueの設定
`First`に配置したButtonを，Ctrlを押しながら引っ張って`Second`のViewControllerへ繋ぎます．
Storyboard Seguesの設定が出るので，`Modal`とかを選ぶ．
![](https://lh3.googleusercontent.com/-XX9-8SPsWAg/T69n1E6_nlI/AAAAAAAAAZI/R9xnsie7vnE/26CE3AAA-C5FE-4209-9C35-AF20533C15B3.jpg)

![](https://lh5.googleusercontent.com/-7zTj7o1d3dQ/T69nwxc2G2I/AAAAAAAAAYk/eEjmStwoOlA/78C994F8-EFE6-4643-93FD-1D8F1EBB5050.jpg)

## Storyboardの保存
`Cmd+s`でStoryboardを`resources/Storyboard.storyboard`とか名前を付けて保存します．

# storyboardのコンパイル

    $ ibtool --compile resources/Storyboard.storyboardc resources/Storyboard.storyboard

# コーディング
`app/app_delegate.rb`を書く．
Storyboardからロードします．

    class AppDelegate
      def application(application, didFinishLaunchingWithOptions:launchOptions)
        @window = UIWindow.alloc.initWithFrame(UIScreen.mainScreen.bounds)
        @sb = UIStoryboard.storyboardWithName('Storyboard', bundle: nil)
        rvc = @sb.instantiateViewControllerWithIdentifier("First")
        @window.rootViewController = rvc
        @window.rootViewController.wantsFullScreenLayout = true
        @window.makeKeyAndVisible
        true
      end
    end


ちなみに，Buttonの動作を変えたい場合などは，`First`と付けたUIViewControllerをCustom Classに変更してUIViewControllerを継承したクラスを作成して，`prepareForSegue:sender:`で遷移前に処理を行うとか，UIButtonの`addTarget:sender`とかにselectorを登録して処理を行うとかします．

# コンパイルして実行
      $rake
とすれば，Simulatorが立ち上がります．
設置したButtonを押せばSecondのViewへ切り替わります．

![](https://lh4.googleusercontent.com/-XyNt1C6cfKo/T69nxjkqNrI/AAAAAAAAAYs/V4i7HQ9rDV8/8619164C-B90D-4D8F-913F-ED11086D700A.jpg)

![](https://lh5.googleusercontent.com/-4ugWozivZrg/T69nziIvs3I/AAAAAAAAAZA/HYZfkyB-WEQ/22854A71-7125-43CF-802C-7630629537F5.jpg)
