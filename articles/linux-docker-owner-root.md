---
title: "LinuxでDocker使うと、Docker側で生成したファイルがrootになる問題"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","Docker"]
published: false
---

英語読んで書いてあることやっただけ。

# 状況

- ホストマシンがLinuxでDockerを使う場合、rootで実行する必要がある
→ いちいち `sudo docker-compose` とrootで実行する必要あり
→ コレについてはzshでalias張れば解決なのでまぁ、手間ではない

- コンテナ内で自動生成する場合 (Laravelとかでファイルを生成させる場合とか)
→ 生成されるファイルが root:root になる
→ Read onlyになってローカルから触れないので、いちいち `sudo chown` する必要が

- そもそもrootでDockerを実行する = dockerに変なの仕込まれたら終わり
→ セキュリティ面でも非常に良くない

これらを解決したい

# Rootlessモード

この問題には `userns-remap` という解決方法があったが、
Docker v19.03からは rootlessモード というものが用意されたようで、こちらを使ってみようと思う。

この違いとしては、 `userns-remap` デーモン自体はrootで動いていたのに対し、
Rootlessモードではすべてがroot権限なしで動くようになっている様子。
(ただしまだ動かないものもあるらしい)

## 環境

- ArchLinux
- yay (AUR)

## インストール

https://docs.docker.com/engine/security/rootless/

一応、Dockerの公式には乗ってるらしいが、ArchLinuxのユーザリポジトリ(AUR)に方法が載っていたので、
AURで楽しようと思う。

20/07/06 ぐらいの話なので、AURのページから下記の情報がまだ使えるか確認しながらどうぞ。

https://aur.archlinux.org/packages/docker-rootless/#pinned-724406

### docker-rootlessをAURからインストール

上記に従ってまず AUR から docker-rootless をいれてやる

```
$ yay -S docker-rootless
```

終わったらこの表示が出る。これは先程のAURのコメントの通りなので、これに従ってやっていく

```
=== Post installation message from docker-rootless ===
This is based on https://docs.docker.com/engine/security/rootless/
To Run the Docker daemon as a non-root user (Rootless mode) for ArchLinux, you need t
o do the following things:

1. configure kernel settings

create '/etc/sysctl.d/99-docker-rootless.conf': 'kernel.unprivileged_userns_clone=1'

and then run: 'sudo sysctl --system'

> see https://docs.docker.com/engine/security/rootless/#distribution-specific-hint for detailed information

2. configure subuid and subgid

and create '/etc/subuid' and '/etc/subgid' with: 'testuser:231072:65536' (for example
, 'testuser' is username)

> see https://docs.docker.com/engine/security/userns-remap/#prerequisites for detailed information

3. start and enable user service: 'systemctl --user status|start|stop docker'

4. finally set docker socket environment variable: 'export DOCKER_HOST=unix://$XDG_RU
NTIME_DIR/docker.sock', you can also add it to '~/.bashrc' or somewhere alike
=========
```

### カーネルの設定

以下を作る

/etc/sysctl.d/99-docker-rootless.conf
```
kernel.unprivileged_userns_clone=1
```

### subuid, subgid の設定

それぞれ下記内容で生成
ユーザ名には自分のユーザ名を入れる

※注意: subgidの方はgroupのidなのかとおもったら、こちらもuserのidで良い模様。
これでドツボにはまって時間浪費した。ちくしょう。

/etc/subuidと/etc/subgid
```
ユーザ名:231072:65536
```
※231072については適当に絶対に被らなさそうな開始idを指定すれば良いので、114514とか適当に
subuid、subgidともに100000とかでも良さそうだった

### ユーザのDocker起動,自動起動有効

```
# まず状態の確認
$ systemctl --user status docker
# State が running でなければ start
$ systemctl --user start docker
# 自動起動を有効に
$ systemctl --user enable docker
```

enableした場合、設定が最後まで終わったあと再起動してdockerが起動されているか確認すると良さそう
最後にもっかいstatusして起動できてるか確認しておくとなおよし

systemctlでの起動の際は、root側と `--user` 側でそれぞれstartしてenableしておく必要あり
rootのdockerが起動していないと、userのdockerだけでは起動がfailedする
あと、起動しない理由としてユーザがdockerグループから抜けていたというのもあったが
そもそもそれをしたくないのでrootless dockerにしたはずなんだが、みたいな話になってきてなんかアレ
https://stackoverflow.com/questions/60490529/docker-rootless-on-ubuntu-overlay2-failed-driver-not-supported
とりあえずコレやってもfailedしてそう

### Docker Host

お使いのshellでPATHを通す
私は `zsh` なので `.zshrc` に

```
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
```

### 再起動

あとはユーザ権限で `docker info` ができるかどうか。
できたらおめでとう :tada:

## 注意

rootless な Docker は、sudo で起動するものとは別にコンテナなどを持つ
つまり同じプロジェクトでもdocker-compose up -dすると新しくbuildが走っちゃうよ、ということである
新幹線でテザリングしてたときにやらかしてコンテナビルド走って慌てて消した
即日通信制限がかかり、その月はひもじい思いをしたので気をつけよう！
