---
layout: post
title: "MongoDBのShellを使いやすくする方法"
date: 2013-11-12 07:49
comments: true
categories: mongodb
---

最近DBは[MongoDB](http://www.mongodb.org/)しか使ってないぽちです，おはようございます！

Node.js + MongoDBの組み合わせで使うことがほとんどなので，スクリプトからは[mongoose](http://mongoosejs.com/)を使うのですが，ちょっとDBの中身を確認したりするのでmongodbのshellを使うことも多いです．

普段mongooseを使っていると，mongodbのshellが貧弱で困ることがあります．

```
> db.users.findOne({_id:"<存在するObjectId>"})
null
```

みたいな．

正しくはこう書かねばなりません．
```
> db.users.findOne({_id:ObjectId("<存在するObjectId>")})
{
  "_id":ObjectId("<存在するObjectId>")
  :
}
```

めんどくさい！`ObjectId`と毎回タイプするのがめんどくさい！

```
> db.users.findOneById("<存在するObjectId>")
```
と書ければ楽だなーと思って探したら，

```
> DBCollection.prototype.findOneById = function(id){return this.findOne({_id:ObjectId(id)});};
```
とかやってあげると
```
db.users.findOneById("<存在するObjectId>")
{
  "_id":ObjectId("<存在するObjectId>")
  :
}
```
楽ちんだ！

`findOneById`とか長い！


```
> DBCollection.prototype.findOneById = DBCollection.prototype.fbi = function(id){return this.findOne({_id:ObjectId(id)});};
db.users.fbi("<存在するObjectId>")
{
  "_id":ObjectId("<存在するObjectId>")
  :
}
```

楽ちんだ！！！！さすがFBI！！！連邦捜査局！！！

はい，でもこれを毎回打つのが超めんどくさい．

そういう時は，`~/.mongorc.js`を配置しましょう．
```
$ echo "DBCollection.prototype.findOneById = DBCollection.prototype.fbi = function(id){return this.findOne({_id:ObjectId(id)});};" > ~/.mongorc.js
$ mongo hoge
> db.users.fbi("<存在するObjectId>")
{
  "_id":ObjectId("<存在するObjectId>")
  :
}
```

楽ちん！

上では`findOneById`だけですが，どんどん追加できるので，決まったaggregateがあるならパラメータを渡す簡単構文でaggregateしたりmongooseでいう`populate`的な動作も出来ると思います．

[mesh](https://github.com/skratchdot/mesh)というmongo shellに色々ライブラリを読み込んでくれるのもあるのですが，欲しいのがなかったので自分で作って公開するといいよ！

## まとめ
- `~/.mongorc.js`にmongo shellを拡張するスクリプトを置くと便利！





