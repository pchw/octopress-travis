---
layout: post
title: "RubyMotionを使ってる人はTestFlight使うの簡単だからやった方がいいよ"
date: 2012-06-17 20:45
comments: true
categories: RubyMotion
---

iOSアプリを開発していて，友達とかチームの人に配布する時のAdHoc配布はTestFlightが便利です．

XCodeでやってたときは，AdHoc用の構成を作ってEntitlementがどうとかしてオーガナイザでIPAファイルにしてTestFlightのサイトにアクセスしてUpload BuildでIPAをアップロードして配布するメンバーを選んで，，，というような事をしないといけませんでした．

※ 最近はTestFlight Desktop Appとか出てて何かもうちょっと楽になってそうな感じだけど．

RubyMotionでは`motion-testflight`というgemがありまして，そこらへんを楽チンに出来ます．

![developer center](/images/testflight_devcenter.png)

公式の[Developer Center](http://www.rubymotion.com/developer-center/articles/testflight/)にもガイドがあるので，みんなやってるかな？

まぁとりあえず導入とどう設定してるか解説してみるよ．

`motion-testflight`を使うには4つ準備が必要です．

1. motion-testflightのインストール
2. API TokenをRakefileに設定する
3. Team TokenをRakefileに設定する
4. TestFlight SDKをvendorに入れる

## motion-testflightのインストール

    $ sudo gem install motion-testflight


## API TokenをRakefileに設定する
次に，TestFlightのWebページに行ってAPI Tokenを取得する必要があります．

![api token](/images/testflight_apitoken.png)

TestFlightのWebページに行ってログインします．
その後，右上のをクリックしてアカウント設定を開きます．
その中の下の方にAPI Tokenが書いてあるのでコピーします．

`Rakefile`の`app.testflight.api_token`に設定します．

    Motion::Project::App.setup do |app|
        # Use `rake config' to see complete project settings.
        app.name = 'stickey'
        app.testflight.api_token = '<API Token>'
    end

こんな感じになります．

## Team TokenをRakefileに設定する
TestFlightにはチームとか言う配布する範囲的なものが決めれます．

※ チーム内でもdistribution_listという更に細かく配布先を設定することも出来ます．

そして，そのチームごとにTeam TokenというAPI Tokenとは別のものがあります．これも設定する必要があります．

![TeamToken](/images/testflight_teamtoken.png)

TeamInfoからTeamTokenをコピペして`Rakefile`の`app.testflight.team_token`に設定します．

    Motion::Project::App.setup do |app|
        # Use `rake config' to see complete project settings.
        app.name = 'stickey'
        app.testflight.api_token = '<API Token>'
        app.testflight.team_token = '<Team Token>'
    end

こんな感じになります．

## TestFlight SDKをダウンロードする
`motion-testflight`はTestFlight SDKが無いと動きません．

そのため，さっきのTeamTokenをコピーした近くにTestFlight SDKへのリンクがあるのでそこから落としておきます．

[TestFlight SDK ダウンロード](http://testflightapp.com/sdk/download)

TestFlight SDKはzipなので，解凍して`vendor/TestFlight`とかに置いておきます．

Rakefile にも置き場所を設定する必要があります．

    Motion::Project::App.setup do |app|
        # Use `rake config' to see complete project settings.
        app.name = 'stickey'
        app.testflight.api_token = '<API Token>'
        app.testflight.team_token = '<Team Token>'
        app.testflight.sdk = 'vendor/TestFlight'
    end

## TestFlightで配布する

    $ rake testflight notes="リリースノートを入れる"

とすれば，buildしてTestFlightへアップロードされていきます．

## distribution_listを作った方がいい
ここまでだと，AdHoc BuildされたパッケージがTestFlightに上がるのは楽になりましたが，
まだTestFlightのWebサイトにアクセスしてどのTeammateを配布先にするかを選択しないといけません．

これは非常にめんどくさいので，distribution_listを作ります．

![people](/images/testflight_people.png)

メニューのPeopleの所を選びます．

![distribution](/images/testflight_distribution.png)

distribution_listに入れたい人を選択して，右のActionsから`+New Distribution List`を選択します．
あとは，そのdistribution_listに名前を付けてSaveすれば完了です．

※ distribution_listは何個も作れるっぽいので，「アプリ毎」とか「このアプリのテスターはこの人達」とか「どうせ全員に配るから全員」など色んな単位で作っておくと色々指定出来て楽ちんです．

distribution_listを作ったら，`Rakefile`にdistribution_listを設定します．

    Motion::Project::App.setup do |app|
        # Use `rake config' to see complete project settings.
        app.name = 'stickey'
        app.testflight.api_token = '<API Token>'
        app.testflight.team_token = '<Team Token>'
        app.testflight.sdk = 'vendor/TestFlight'
        app.testflight.distribution_lists = ['<distribution_list>']
    end

こんな感じで設定します．Arrayで設定できるので，この集団とあの集団に配布したいという時も2つ設定すればいいだけです．

## 証明書とProvisioningProfile
証明書(codesign_certificate)とProvisioningProfileは何も設定しないと，
keychainの初めに見つかった証明書と`~/Library/MobileDevice/Provisioning`内にある初めの`.mobileprovision`を使ったりします．

割りとTestFlight配布だと上手くいかなくなることが多いので，`Rakefile` で指定するといいです．


    Motion::Project::App.setup do |app|
        # Use `rake config' to see complete project settings.
        app.name = 'stickey'
        app.testflight.api_token = '<API Token>'
        app.testflight.team_token = '<Team Token>'
        app.testflight.sdk = 'vendor/TestFlight'
        app.testflight.distribution_lists = ['<distribution_list>']

        app.codesign_certificate = "iPhone Distribution: なまえがはいったり"
        app.provisioning_profile = "/Users/<Username>/Library/MobileDevice/Provisioning Profiles/XXXXXXXXXXXXXXXX.mobileprovision"
    end
        
こんな感じに指定すると良いです．

`app.provisioning_profile`は`~/...`とか入れるとエラーになるので，普通にフルパス入れるとよいです．

## まとめ
motion-testflight を入れるとAdHoc配布がすごく楽になります．
今まではXCodeのオーガナイザとブラウザとエディタを行ったり来たりでしたが，
simulatorの起動等と同じようにターミナルで完結するのですごく楽．

他人へのAdHoc配布だけでなく，自分で複数のdeviceに入れるときもUSBを繋ぎ変えて`rake device`を連発するより楽だと思うので，積極的に使っていくと良いと思います．



