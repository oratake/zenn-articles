---
title: "npm -g したらpathが通ってるはずなのに実行できなかった話(nodenv絡み)"
emoji: "🧶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nodejs", "nodenv"]
published: false
---
# TL;DR

# 詳細
## 状況
vimの起動時にcoc.vimのエラーが出ていた。状況としては以下
https://qiita.com/Taichi-yzrh/items/5868e618c82e328c89f6
vimの初回起動時にyarnがインストールされていなかったため、installができずcoc.vimが起動していなかったという話の様子。

で、node関係についてはanyenv経由でnodenvを入れているので、 `npm -g yarn` でグローバルにyarnを入れることで最新のyarnを入れようとした。

## 与太話
ちなみに、wslにすでに相当古いyarnが入っていたようで、whichすると `/usr/bin/yarn` とのこと。
aptでremoveしてみたが消えてくれないので仕方なく `sudo rm -f /usr/bin/yarn` したという。よい子はマネしないでね。
