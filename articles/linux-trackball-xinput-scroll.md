---
title: "Linuxでトラックボールをゴロゴロしてスクロールしたいじゃない"
emoji: "🖲️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","Xorg","libinput"]
published: true
---

PCをinitするたびに毎回探してる気がするのでメモ

# TL;DR

`/usr/share/X11/xorg.conf.d/40-libinput.conf` に以下のようなものをぶっこむ。  
以下は ELECOM のトラボEX-G (M-XT3DR) の例。  
右クリック脇のボタンを薬指で抑えながらゴロゴロやると、スクロールができて気持ちがいい。

```
Section "InputClass"
        Identifier "ELECOM ELECOM TrackBall Mouse"
        MatchProduct "ELECOM ELECOM TrackBall Mouse"
        Driver "libinput"
        Option "ScrollButton" "10"
        Option "ScrollMethod" "button"
EndSection
```

## 環境

Thinkpad X1 Carbon 4th  
Ubuntu 20.10  
(ArchLinuxでも使ってたのでいける)  
(というかXorgの設定なのでLinux大体いけそう)

## 設定

### デバイス名確認

まずトラボを接続して名前が何で認識されているか確認

```
$ xinput list
⎡ Virtual core pointer                          id=2    [master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
⎜   ↳ ELECOM ELECOM TrackBall Mouse             id=12   [slave  pointer  (2)]
⎜   ↳ SynPS/2 Synaptics TouchPad                id=15   [slave  pointer  (2)]
⎜   ↳ TPPS/2 IBM TrackPoint                     id=16   [slave  pointer  (2)]
⎣ Virtual core keyboard                         id=3    [master keyboard (2)]
    ↳ Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
    ↳ Power Button                              id=6    [slave  keyboard (3)]
    ↳ Video Bus                                 id=7    [slave  keyboard (3)]
    ↳ Sleep Button                              id=8    [slave  keyboard (3)]
    ↳ ARCHISS PTR70 ARCHISS PTR70               id=9    [slave  keyboard (3)]
    ↳ ARCHISS PTR70 ARCHISS PTR70 System Control        id=10   [slave  keyboard (3)]
    ↳ ARCHISS PTR70 ARCHISS PTR70 Consumer Control      id=11   [slave  keyboard (3)]
    ↳ Integrated Camera: Integrated C           id=13   [slave  keyboard (3)]
    ↳ AT Translated Set 2 keyboard              id=14   [slave  keyboard (3)]
    ↳ ThinkPad Extra Buttons                    id=17   [slave  keyboard (3)]
    ↳ ELECOM ELECOM TrackBall Mouse             id=18   [slave  keyboard (3)]
```

今回だと `ELECOM ELECOM TrackBall Mouse`

### キーの確認

ちょうどよく薬指のボタンが遊んでいるので、こいつを押している間大玉ゴロゴロスクロールできるようにしてみたい  
薬指ボタンの番号を `xev` あたりで確認する  
手元のものは10だった

```
ButtonPress event, serial 31, synthetic NO, window 0x3000001,
    root 0x7b7, subw 0x0, time 108376160, (7,950), root:(651,1010),
    state 0x0, button 10, same_screen YES

ButtonRelease event, serial 31, synthetic NO, window 0x3000001,
    root 0x7b7, subw 0x0, time 108376160, (7,950), root:(651,1010),
    state 0x0, button 10, same_screen YES
```

### 設定記入

tldrに書いた通り。  
追加で注釈をくわえると、Optionに並んでいる ScrollMethod や ScrollButton については libinput のmanに記載がある様子

https://man.archlinux.org/man/libinput.4

## 参考

- https://wiki.archlinux.jp/index.php/Libinput
