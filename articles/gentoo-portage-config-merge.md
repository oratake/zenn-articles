---
title: "portageでインストールしたと思ったらできてない話"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "gentoo"]
published: true
---

Gentoo Linux を始めたのだが、初歩中の初歩で躓いたので備忘録

# emergeでインストールができてない

## 状況

用があって以下を入れてみた

```
# emerge --ask sys-kernel/genkernel
```

すると、実行しようとするとなぜか command が not found と言われる。  
あれ、間違ったかな？と思ったらタイポもなし。  
shellのprofileをsourceしても変わりなし  

もう一度emerge後のインストールログをみると、なにやら最後に気になる1文が

```
* IMPORTANT: 2 config files in /etc/portage need updating.
```

これは IMPORTANT is IMPORTANT だった...

## 対応

見てみると、emergeというのは新しいconfigファイルが出てきた場合に、下品に上書きせず  
上品に差分をとって待っててくれるらしい...!

甘んじて受け入れるには以下公式の手順を参考に

参考: Gentooハンドブック https://wiki.gentoo.org/wiki/Handbook:AMD64/Portage/Tools/ja

```
# dispatch-conf
```

そして `u` でuse-new、すなわち新しいconfをつかってね、とするとよいわけだ  
このあともう一度emergeのコマンドを打つとちゃんと入る。  

## 結論

ログ読みましょう。  

**IMPORTANT is IMPORTANT.**
