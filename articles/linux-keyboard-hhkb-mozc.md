---
title: "ArchLinuxでHHKB Lite2 for Macを使う"
emoji: "⌨️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["archlinux","Linux"]
published: true
---

ArchLinuxでHappy Hack Keyboard Lite2 for Mac (JIS)を使おうとした話

# 環境

- ThinkPad X1 Carbon (ふるいやつ)
- keymap: jp106
- HHKB Lite2 for Mac

ThinkPad本体のCapsLockはCtrlと入れ替え済
このワンライナーを.zshrcで登録している

```
$ setxkbmap -model jp106 -layout jp -option ctrl:nocaps
```

# 状況

左右のCommandが使用できなかったのでxevで確認したところ、
どうやら左⌘は全角/半角に、右⌘はひらがな/カタカナになっていた模様
以下参考にkeycodeに割当てる

参照: https://www.komee.org/entry/2018/10/24/150000

```
# もともとの設定
keycode 133 Super_L
keycode 101 Hiragana_Katakana
keycode 49 Zenkaku_Hankaku

# Superに割当されたキーとSuperをどう指定するかの確認
keycode 133 = Super_L NoSymbol Super_L
keycode 134 = Super_R NoSymbol Super_R
keycode 206 = NoSymbol Super_L NoSymbol Super_L
```

変更後の設定はこのようにした

```
keycode 101 = Super_R
keycode 49 = Super_L
```

やったぜ。
