---
title: "Cloudflare Zero TrustのTunnelを使ったsshができなくなったときの話"
emoji: "🧹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cloudflare","ssh"]
published: true
---

# 概要
Cloudflare Zero TrustのTunnel機能をつかって、外部からsshを行うということをしていた。
だたこれがなにかの設定中に接続できなくなって、sshしようとすると以下のエラーを出すようになってしまったという話。

```
$ ssh -vvv sshしたいホスト
---中略---
kex_exchange_identification: Connection closed by remote host
Connection closed by UNKNOWN port 65535
```

# 詳細

## Cloudflare Tunnelの設定

パブリックホスト名: `hogehoge.example.com`
サービス: `ssh://192.168.0.XXX:22`

この辺は設定できていた

## ssh時の設定

```:~/.ssh/config
Host hogehoge
  User piyouser
  HostName hogehoge.example.com
  ProxyCommand /opt/homebrew/bin/cloudflared access ssh --hostname %h
```
この辺もできていた

## 接続状態

- OK✅️
ローカルからの接続
`ssh piyouser@192.168.0.XXXX`

- NG❌️ 
cloudflare越しの接続
`ssh hogehoge`

この時点で多分cloudflare側の設定か?と思っていた

# 対処

一度Cloudflare Tunnelの設定を削除して、再度登録する
こんな簡単なことで...
