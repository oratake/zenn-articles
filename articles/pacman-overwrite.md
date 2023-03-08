---
title: "pacmanのインストールを中断して大量の /path/is/hoge exists in filesystemを出した"
emoji: "🟡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "archlinux", "pacman" ]
published: true
---

# TL;DR

上書きをする

```shell
$ sudo pacman -S package --overwrite="*"
```

yay(AUR)でも同じ `--overwrite` 可だった。
zshでは*をダブルクォートで囲う必要があった

# 細かい話
## 構成
- OS: Manjaro (Archlinuxベース)

## 経緯
yayでvscode(visual-studio-code-bin)をインストールをしている途中で `<C-c>` してしまったので途中で終わってしまった。再び同様のコマンドを実行すると大量の `/path/is/hoge exists in filesystem` が出てしまい途方に暮れた。
`$ yay -Rs package` するも変わらず。
なので探したところ上書きのオプションがあったので上記で解決と相成った