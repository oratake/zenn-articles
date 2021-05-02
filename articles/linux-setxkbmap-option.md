---
title: "Linuxでcapslockとかaltとかの位置をいじる"
emoji: "♻️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux", "Keyboard"]
published: true
---

# TL;DR

Linuxで `setxkbmap` が使える場合は、option指定することでわりと簡単にkeymapができる  
個人的に使用頻度の高い2つ

- CapsLockを潰してCtrlにしちゃう
  - aの左隣はctrlにしちゃう派閥
- AltとWinを入れ替え
  - タイル型WMを使うとSuper(Mod4)を酷使するので、なるべく親指に近いほうがいい

```
$ setxkbmap -option ctrl:nocaps -option altwin:swap_alt_win
```

optionは1つにつき1定義ずつで列挙

# オプション一覧

ここの `! option` 以下が使えるoptionのリスト

```
$ less /usr/share/X11/xkb/rules/base.lst
```

# 参考

https://wiki.archlinux.org/title/Xorg/Keyboard_configuration#Frequently_used_XKB_options
