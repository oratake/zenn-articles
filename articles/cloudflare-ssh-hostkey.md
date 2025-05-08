---
title: "Cloudflare Zero TrustのTunnelを使ったsshができなくなったときの話"
emoji: "🧹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cloudflare","ssh"]
published: false
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
