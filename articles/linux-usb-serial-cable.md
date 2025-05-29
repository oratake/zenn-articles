---
title: "Linuxから業務用ルータにUSBシリアルケーブルで接続する"
emoji: "🔌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux"]
published: false
---

# 概要

- 初めて業務用ルータ(NEC UNICERGE IX2105)というものをメルカリで仕入れた
- 巷はWindows+Tera Termでの設定情報ばかり
- 弊家はMacかLinuxしかない
- しゃーなしArchLinuxで接続したときの記録

# 登場人物

- ルータ
  - NEC UNIVERGE IX2105
- 設定用PC
  - ArchLinux 
- コンソールケーブル
  - USB-A to RJ45

# 手順

コンソールケーブルとLinuxを接続。今回購入したIX2105はRJ45でコンソールに接続するタイプなのでUSB-A to RJ45のものを購入。

IX2105とLinuxをコンソールケーブルで接続。接続する口はCONSOLEと記載がありそこだけ上下てれこになっているので間違えないはず。

接続に使う[cu](https://archlinux.org/packages/extra/x86_64/uucp/)をインストールしておく
```shell
$ sudo pacman -S uucp
```

dmesgでUSBの接続を確認

```
$ sudo dmesg
~中略~
[ 6461.166489] usb 1-4: USB disconnect, device number 8
[ 6461.167134] ftdi_sio ttyUSB0: FTDI USB Serial Device converter now disconnected from ttyUSB0
[ 6461.167181] ftdi_sio 1-4:1.0: device disconnected
[ 6464.209451] usb 1-4: new full-speed USB device number 9 using xhci_hcd
[ 6464.334046] usb 1-4: New USB device found, idVendor=0403, idProduct=6001, bcdDevice= 6.00
[ 6464.334061] usb 1-4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 6464.334066] usb 1-4: Product: USB Serial Converter
[ 6464.334071] usb 1-4: Manufacturer: FTDI
[ 6464.334075] usb 1-4: SerialNumber: FTB6SPL3
[ 6464.336395] ftdi_sio 1-4:1.0: FTDI USB Serial Device converter detected
[ 6464.336481] usb 1-4: Detected FT232R
[ 6464.336644] ftdi_sio ttyUSB0: Unable to read latency timer: -32
[ 6464.337547] usb 1-4: FTDI USB Serial Device converter now attached to ttyUSB0
```

`ttyUSB0` これを覚えておく。
接続。

```shell
$ sudo cu -l ttyUSB0
```

一発Enterでも押して反応があればOK

```
Router# 
```

接続をどう終了したらいいかわからなくてちょっと困った。  
`~.` と入力してEnterで終了

