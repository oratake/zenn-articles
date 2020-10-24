---
title: "Linuxでhttpsでgitするときにtokenをわからせる方法"
emoji: "🗝️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","Git"]
published: true
---

Linux (当方Arch) で `git push` でgithubなどに上げる際、
大本営からは [sshよりhttpsが推奨されているので](https://docs.github.com/en/github/using-git/which-remote-url-should-i-use#cloning-with-https-urls-recommended) そのとおりやっているが
いかんせんLinuxくんは覚えが悪いのでTokenを覚えてくれない。
ので、必要のない起き攻め昇竜で"わからせる"

# 環境

- ArchLinux
- GNOME Keyring

# 方針

gitくんでは credential helper なるものを使用してやると、わからせられるようである

今回件のhelperには GNOME Keyringをつかってみることにする

# 手順

https://wiki.archlinux.jp/index.php/GNOME_Keyring#GNOME_Keyring_.E3.81.A8_Git

上記手順に沿っていく

## GNOME Keyringをインストール

公式リポジトリから GNOME Keyringをインストール

```
$ sudo pacman -S libgnome-keyring
```

## helperをmake

```
$ cd /usr/share/git/credential/gnome-keyring
$ sudo make
```

## git configで設定

$ git config --global credential.helper /usr/lib/git-core/git-credential-gnome-keyring

## 使えるか確認

任意のremoteをhttpsで指定しているリポジトリでpush
