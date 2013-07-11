---
layout: post
title: "Sublime Text2で1行が80文字以内かわかりやすくする方法"
date: 2013-07-12 07:20
comments: true
categories: SublimeText2
---

別に80文字以上でもいいじゃんとか思っていたんですが，
GitHubでレビューする時に文字数が多いと横スクロールしにくくてやりにくいので．

Preferences > Settings - User を開いて（または`Cmd+,`）
```
"rulers":
  [
    80,
    120
  ],
```
を追加して保存すれば80行目と120行目に線が表示されるようになります．
