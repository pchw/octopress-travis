---
layout: post
title: "[RubyMotion] NanoStoreを使ってデータを格納しよう！"
date: 2012-06-21 09:28
comments: true
categories: RubyMotion
---

今日はRubyMotionでちょっとしたデータの保存がしたくなったときに使えるNanoStoreのWrapperを紹介します．

[NanoStoreInMotion](https://github.com/siuying/NanoStoreInMotion)

NanoStoreというのは，sqliteを保存先としてスキーマレスなKeyValueStoreを実現するものらしいです．NanoStoreInMotionはそれをRubyMotionから使えるようにするWrapperです．

## インストール

まずcocoapodsを入れます

    $ gem install motion-cocoapods
    $ pod setup

NanoStoreInMotionを入れます．

    $ gem isntall nano-store

プロジェクトを作って，Rakefileを編集します．

    $ motion create nanostoretest
    $ cat Rakefile
    # -*- coding: utf-8 -*-
    $:.unshift("/Library/RubyMotion/lib")
    require 'motion/project'
    require 'rubygems'
    require 'motion-cocoapods'
    require 'nano-store'

    Motion::Project::App.setup do |app|
      # Use `rake config' to see complete project settings.
      app.name = 'nanotest'
      app.pods do
        dependency 'NanoStore', '~> 2.1.4'
      end
    end

## 使ってみる

### 前処理
まず，NanoStoreの保存先を決めてインスタンス化します．

    NanoStore.shared_store = NanoStore.store(:file, App.documents_path + '/store.db')

`App.documents_path`というのは，BubbleWrapの便利Methodで，
`/Users/xxxxxx/Library/Application Support/iPhone Simulator/5.1/Applications/EDC08B50-2004-42D9-8B67-2E290EF55CFF/Documents`みたいなアプリ毎の保存場所を返してくれます．
中身の実装的には`NSSearchPathForDirectoriesInDomains`でNSDocumentDirectoryを返してるだけです．

    NanoStore.shared_store = NanoStore.store(:memory)

とすることで，ファイルに保存するのではなく，オンメモリでDbを使うこともできます．

### Model定義
次に，保存するModelを定義します．

Modelを定義するには`NanoStore::Model`を継承したクラスを定義すれば良いです．

    class Label < NanoStore::Model
      attribute :text
      attribute :x
      attribute :y
      attribute :height
      attribute :width
    end

保存するフィールドは`attribute <シンボル名>`で定義できます．
上記の例では，text, x, y, height, width というフィールド（というかKey）が定義されます．

★ `attribute :key`は自動的に付与されるので，`:key`は書いてはいけません．（書いてもエラーにはならないが，実際に使うときに予期せぬ動作になります．なりました．）

NanoStore はNSDataも入れれるようですが，インデックスが効かないようです．

### データ投入
先ほど定義したModelクラスのnewメソッドを呼んでsaveすれば良いです．

    label = Label.new(
        :text=>"test",
        :x => 100,
        :y => 200,
        :height => 50,
        :width => 250)
    label.save

この時点で保存されて，検索などが出来るようになります．

### 検索

検索も先ほど定義したModelクラスを通じて行います．

    # もうひとつLabelを追加しておく
    hogeLabel = Label.new(
        :text=>"test1",
        :x => 50,
        :y => 400,
        :height => 100,
        :width => 50)
    hogeLabel.save

    allLabels = label.all
    # => [#<Label:0x8e622e0>, #<Label:0x8e62720>]

条件を指定して検索は

    Label.find(:text=>"test")
    # => [#<Label:0x8e622e0>]

とすればできるので，あとは`.each`とかユニークなKeyで検索するなら`.first`
とかすれば目当てのものが手に入ります．


    Label.find(:x => { NSFGreaterThan => 10 }).each do |lbl|
        p lbl.text
    end
    # => "test1"
    # => "test"

### アップデート
更新処理は，上記の検索で得られたもののプロパティを編集して再度saveを呼べば良いです．

    update = Label.find(:text=>"test").first
    update.text = "test2"
    update.save

    Label.all.each do |lbl|
        p lbl.text
    end
    # => "test2"
    # => "test1"

### 削除
削除処理は，同じく上記の検索で得られたものの`delete`を呼べば，削除できます．

    del = Label.find(:text=>"test1").first
    del.delete

    Label.all.each do |lbl|
        p lbl.text
    end
    # => "test2"

全部削除したい場合は，

    Label.delete

とすれば，全部消えます．

    Label.delete(:text=>"test1")

のように条件をつければ，条件にヒットしたものを一気に削除できます．


## 保存されるデータ
さて，実際に保存されているデータはどうなっているのでしょうか？

`NanoStore.store`時に`:file`を指定した場合は指定したPathにsqliteのデータベースファイルが生成されています．
その中をsqlite3コマンドかなんかで見ていけば保存されている形を見ることができます．

    $ sqlite3 store.db
    sqlite> .tables
    NSFKeys    NSFValues

どうやら`NSFKeys`と`NSFValues`というテーブルが存在するようです．

### NSFKeys

    $sqlite> select * from NSFkeys;
    ROWID|NSFKey|NSFPlist|NSFCalendarDate|NSFObjectClass

- ROWIDはautonumber
- NSFKeyはオブジェクトごとに異なるID
- NSFPlistはオブジェクトのattributeと値をplist形式で表したもの
- NSFCalendarDateは更新日かな？
- NSFObjectClass はNanoStore::Modelを継承したクラス名

### NSFValues

    sqlite > select * from NSFValues;
    ROWID|NSFKey|NSFAttribute|NSFValue|NSFDatatype


- ROWIDはautonumber
- NSFKeyはNSFKeysテーブルのNSFKey（外部キー）
- NSAttributeはattributeで定義したシンボル名
- NSFValueは実際に入っている値
- NSFDatatypeは型．TEXTとかREALとか


NSFValuesの方にModelクラスの違い関係なくattribute名と値が入っています．
クラス名とかの情報はNSFKeysの方に入っていますね．

## まとめ
NanoStoreInMotionはデータを保存，検索，削除などが簡単なステップで出来て，ちょっとしたデータの取り扱いに役立つと思います．
`:memory`を使うことで，Web APIからデータをごっそり取ってきてさっとフィルタリングするような用途にも簡単に使えそうです．