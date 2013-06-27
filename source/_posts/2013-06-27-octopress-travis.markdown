---
layout: post
title: "OctopressとTravis CIを連携させてBlog生成を自動にする"
date: 2013-06-27 10:00
comments: true
categories: octopress
---

Octopressってめんどくさいよね
====
Octopressはmarkdownでブログ書けて，見た目も良い感じで導入した当初はすげーテンションがあがって記事を書きまくってたのですが，`rake generete`とかするのが超めんどくさい．さらにちゃんとセットアップしてるマシンでやらないとrubyのバージョン違いでなんかエラー出たりして超めんどくさいので，最近記事かくのがおっくうでした．

そこで，Travis CIとOctopressをforkしてるリポジトリを連携させることでそういう煩わしさを減らしてブログを書けることが判明しました！色々ハマりどころがあって1日ぐらいかかったので解説記事を書くよ！

ちなみに，TravisCIと連携させてgenerateするようにしておけば，[Prose.io](http://prose.io/)（github連携のWebエディタ）を使ったりするとOctopressでgenerate出来ない環境とかでもブログが書ける！
[GitMongo](https://itunes.apple.com/jp/app/gitmongo/id593450102)使ったりすれば，iPad/iPhoneからOctopressでブログが書ける！

というわけで手順解説です．

OctopressとTravisを連携させる
====

今回の構成はこんな感じ
![octopress-travis](/images/octopress-travis.png)

作業は全部↑で言うOctopress(forked)に対して行います．

下に書く`.travis.yml`のREPOで指定している所が配置するリポジトリです．（上図のGitHub Pagesと書いてるところ）

まず，Travis CIと連携するための`.travis.yml`をプロジェクトのrootに置きます．
.travis.yml
----
```
---
language: ruby
rvm:
- 1.9.3
before_script:
- git config --global user.name "ユーザ名(via TravisCI)"
- git config --global user.email "メールアドレス"
- git remote set-url origin $REPO.git
- if [ -z "$id_rsa_{1..32}" ]; then echo 'No $id_rsa_{1..32} found !' ; exit 1; fi
- echo -n $id_rsa_{1..32} >> ~/.ssh/travis_rsa_64
- base64 --decode --ignore-garbage ~/.ssh/travis_rsa_64 > ~/.ssh/id_rsa
- chmod 600 ~/.ssh/id_rsa
- echo -e "Host github.com\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
- rake setup_github_pages[$REPO]
script:
- rake generate
after_script:
- rake deploy
env:
  global:
  - REPO="git@github.com:ユーザ名/GitHubページを配置するリポジトリ.git"
```

Travis CI用の鍵作成
----
```
# 鍵生成
# passphraseは空で．
ssh-keygen -t rsa -C "<メールアドレス>" -f ~/.ssh/travis_rsa

# クリップボードに公開鍵をコピー
cat ~/.ssh/travis_rsa.pub | pbcopy


# SSH鍵設定ページを開いてAdd SSH keyする
open  https://github.com/<GitHubアカウント名>/<Octopress用リポジトリ名>/settings/keys
```

ブラウザが開くので，Add SSH Keyボタンを押して追加します．

```
# 元からOS Xに入っているbase64コマンドだと --wrap=0オプションが使えないのでbase64コマンドをインストール
brew install base64

# 鍵をbase64化
# brewのバージョンなどによってinstallされるPathは異なることがある
/usr/local/Cellar/base64/1.5/bin/base64 --wrap=0 ~/.ssh/travis_rsa > ~/.ssh/travis_rsa_64


# travisコマンドをインストール
brew install travis

# travis encryptが100文字までしか受け付けないので，100文字ごとにtravis encryptに渡す
--addで.travis.ymlに自動追記してくれる
bash <(cat ~/.ssh/travis_rsa_64 | perl -pe 's/(.{100})/$1\n/g' | nl | perl -pe 's/\s*(\d+)\s*(.*)/travis encrypt -r <GitHubアカウント名>\/<Octopress用リポジトリ名> id_rsa_$1="$2" --add env.global/')
 
# id_rsa_Xが何個あるかをカウント
cat ~/.ssh/travis_rsa_64 | perl -pe 's/(.{100})/$1\n/g' | nl | tail
```

.travis.ymlの書き換え
----

`$id_rsa_{1..32}` みたいに書いている場所は↑の`id_rsa_Xが何個あるかをカウント`の部分でカウントした数にします．

僕の場合は`32`でした．

リポジトリにpushする
----
`$ git push remote source`

Travis CI側の設定
----

[https://travis-ci.org/profile](https://travis-ci.org/profile) からリポジトリの所をonにする．

https://github.com/ユーザ名/リポジトリ名/settings/hooks#travis_minibucket
からtravisのhookが設定されているので，TestHookを押す．

[https://travis-ci.org/](https://travis-ci.org/)にアクセスするとなんかビルドが始まって勝手に更新される！

ハマりどころ
===
Enter passphrase for key '/home/travis/.ssh/id_rsa': とか出やがって止まる
----
1. travis encryptに指定するユーザ名，リポジトリ名をミスってる
2. id_rsa_Xの番号をミスってる（ちゃんとid_rsaXの数分Secureの行数分指定しているか）
3. 鍵生成でミスってる（passpraseをemptyに）

base64コマンドがbase64: unrecognized option `--wrap=0'とか吐く
----
`brew install base64`して入ったbase64で実行する．
`/usr/bin/base64` を使うと`--wrap`オプションが無い．


参考資料
====
- [Auto-deploying to My Octopress Blog With Travis-CI](http://www.harimenon.com/blog/2013/01/27/auto-deploying-to-my-octopress-blog/)
- [Prose.io + Octopress + Travis-CI + GitHub Pages = ♥](http://darvin.github.io/2013/01/13/Prose_Octopress_TravisIO/)
- [https://gist.github.com/lukewpatterson/4242707](https://gist.github.com/lukewpatterson/4242707)
- [https://gist.github.com/douglasduteil/5525750](https://gist.github.com/douglasduteil/5525750)
- [https://gist.github.com/wakefield/5519563](https://gist.github.com/wakefield/5519563)
