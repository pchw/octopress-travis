---
layout: post
title: "Githubの二段階認証を設定しよう"
date: 2013-11-22 11:13
comments: true
categories: github
---

Github 2段階認証の設定方法

Githubがログインアタック受けたみたいですね．
そんなときにハラハラしないために2段階認証（two-factor authentication）を入れましょう．

Githubにログインして右上の工具マーク（Account Setting）から設定画面に行きます．
![2013-11-22 10 56 33](https://f.cloud.github.com/assets/1183484/1597813/a4207532-531a-11e3-9d50-bf483cdbdf8e.png)

`Account Settings` を選んで，`Two-factor authentication`の`Set up two-factor authentication`を選択します．
![2013-11-22 10 56 47](https://f.cloud.github.com/assets/1183484/1597826/d2c7f22a-531a-11e3-9ede-143c4dc9cc88.png)

SMSかアプリか選べと言われるので`Set up using app`を選びます．
![2013-11-22 10 57 09](https://f.cloud.github.com/assets/1183484/1597829/e5300aa6-531a-11e3-8e56-fcf313c4f7ef.png)

すると，右側にQRコードが表示された画面になるので，[Google Authenticator](https://itunes.apple.com/jp/app/google-authenticator/id388497605) でQRコードを読み込ませます．
すぐに6桁のコードが表示されるので，`2. Enter the 6-digit code that the application generates`に入力して`Enable`します．
![2013-11-22 10 57 21](https://f.cloud.github.com/assets/1183484/1597842/32d6a058-531b-11e3-99ac-de54e9e0c108.png)

これで2段階認証の設定が完了です！

エジプトからのアクセスでファラオに呪いをかけられる心配しなくてよくなりましたね！