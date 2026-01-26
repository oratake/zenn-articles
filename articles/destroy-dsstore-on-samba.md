---
title: "NASでDS_Store汚染を引き起こさないために"
emoji: "☠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["finder", "macosx"]
published: true
---

# 経緯
仕事上NASにMacからファイルを配置する機会があるわけだが、弊社は基本WindowsであるためMacが所構わず撒き散らす.DS_StoreのせいでWinから見るディレクトリ一覧が汚い事になっていた。

```:こんなファイル名
.DS_Store
._.DS_Store
```

こういった ~~ペットの糞の~~ 後始末は飼い主の責任なので、SMB接続先(大体NASはSambaですよね)でDS_Storeを生成しない方法を探したらやっぱりあったので記す。

# コマンド

```
$ defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
$ killall Finder #Finderの強制再起動
```

これで再生成されなくなるので、あとはお掃除お掃除...
