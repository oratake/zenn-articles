---
title: "LinuxでDocker使うと、Docker側で生成したファイルがrootになる問題"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux","Docker"]
published: true
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
(一部機能はまだ動かないものもあるらしい)

## 環境

- ArchLinux
- yay (AUR)

## インストール

https://docs.docker.com/engine/security/rootless/

一応、Dockerの公式には乗ってるらしいが、ArchLinuxのユーザリポジトリ(AUR)に方法が載っていたので、
AURで楽しようと思う。

~~20/12/09 ぐらいの話なので、AURのページから下記の情報がまだ使えるか確認しながらどうぞ。
https://aur.archlinux.org/packages/docker-rootless/~~

~~21/01/03 上記見れなくなってた。以下で入りそう。なんか最近このあたりゴタゴタしてるので公式とか見たほうがいいかも
https://aur.archlinux.org/packages/docker-rootless-extras-bin/~~

22/06/01 確認時はこれ
https://aur.archlinux.org/packages/docker-rootless-extras/

### docker-rootlessをAURからインストール

公式を確認すると以下のインストールが推奨されていたので追記
```
$ sudo pacman -S fuse-overlayfs
```

上記AURに従ってまず AUR から docker-rootless-extras をいれてやる
```
$ yay -S docker-rootless-extras
```

このメッセージがでる

```
=== Post installation message from docker-rootless ===
This is based on https://docs.docker.com/engine/security/rootless/
To Run the Docker daemon as a non-root user (Rootless mode) for ArchLinux, you need to do the following things:

1. Configure subuid and subgid

Create '/etc/subuid' and '/etc/subgid' with the following:

    testuser:231072:65536
    # replace 'testuser' with your username.

2. Enable socket-activation for the user service:

    systemctl --user enable --now docker.socket

3. Finally set docker socket environment variable:

    export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock

You can also add it to '~/.bashrc' or somewhere alike.

=========
Optional dependencies for docker-rootless-extras
    fuse-overlayfs: overlayfs support [installed]
    slirp4netns: faster network stack
```

最後にOptional dependenciesと書かれているものは、実はどちらもほぼ必須っぽい。
dockerがうまく起動しなくて色々みていたら見つけたやつ。

```
$ sudo pacman -S slirp4netns 
```

:::details rootlesskitを使ってdocker起動時に出たエラー

`$ systemctl --user status docker` して出たログ

```
Jun 01 00:31:20 PC名 systemd[428]: docker.service: Main process exited, code=exited, status=1/FAILURE
Jun 01 00:31:20 PC名 systemd[428]: docker.service: Failed with result 'exit-code'.
Jun 01 00:31:22 PC名 systemd[428]: docker.service: Scheduled restart job, restart counter is at 3.
Jun 01 00:31:22 PC名 systemd[428]: Stopped Docker Application Container Engine (Rootless).
Jun 01 00:31:22 PC名 systemd[428]: docker.service: Start request repeated too quickly.
Jun 01 00:31:22 PC名 systemd[428]: docker.service: Failed with result 'exit-code'.
Jun 01 00:31:22 PC名 systemd[428]: Failed to start Docker Application Container Engine (Rootless).
```

よくわからんので `journalctl -xe` した

```
Either slirp4netns (>= v0.4.0) or vpnkit needs to be installed
```

slirp4netns 要るやんけ。
なにがOptional dependenciesじゃ。

```
$ sudo pacman -S slirp4netns 
```

ﾖｼ。

:::

:::details 旧内容
上記AURに従ってまず AUR から docker-rootless をいれてやる

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
:::

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
subuid、subgidともに100000から65536個確保、とかでも良さそうだった

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

~~systemctlでの起動の際は、root側と `--user` 側でそれぞれstartしてenableしておく必要あり
rootのdockerが起動していないと、userのdockerだけでは起動がfailedする
あと、起動しない理由としてユーザがdockerグループから抜けていたというのもあったが
そもそもそれをしたくないのでrootless dockerにしたはずなんだが、みたいな話になってきてなんかアレ
https://stackoverflow.com/questions/60490529/docker-rootless-on-ubuntu-overlay2-failed-driver-not-supported
とりあえずコレやってもfailedしてそう~~

追記: 20/12/09
rootlessモードなdockerをstartするだけでいけた。なんでや。

### Docker Host

お使いのshellでPATHを通す
小生は `zsh` なので `.zshrc` に

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


## 追記

dockerのファイル保存先をかえれば、root側のdockerと設定関係のファイルがかち合わないのでは？とおもった

コンテナをbuildしてる最中にこんな感じのが出てくる

```
Status: Downloaded newer image for php:8-fpm
 ---> edc6bf79d9ba
Step 2/2 : RUN apt update && docker-php-ext-install pdo pdo_mysql
 ---> Running in 5616e0ab9a5a
ERROR: Service 'php' failed to build : /run/containerd/s/60ceecf888f9382eafe6d7308496a508a83e2bd819ef77fb285590aaf9a7985b: mkdir /run/containerd/s: permission denied: unknown
```

これひょっとしてrootとrootlessのdockerの環境が分けられてないかな？という気がしたので
rootlessなdockerは別なとこを指定してみる

試しにそれぞれのstatusを確認

root
```
$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: >
     Active: inactive (dead)
TriggeredBy: ● docker.socket
```

rootless
```
$ systemctl --user status docker
● docker.service - Docker Application Container Engine (Rootless)
     Loaded: loaded (/usr/lib/systemd/user/docker.service; disabled; vendor preset:
     Active: inactive (dead)
       Docs: https://docs.docker.com
```

このそれぞれのdocker.serviceに、起動時の設定などをぶちこんでおける

rootless dockerのdocker.service
こいつのExecStartを変更していく
```
[Unit]
Description=Docker Application Container Engine (Rootless)
Documentation=https://docs.docker.com

[Service]
Environment=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
Environment=DOCKERD_FLAGS="--experimental"
ExecStart=/usr/bin/dockerd-rootless.sh $DOCKERD_FLAGS
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
Type=simple

[Install]
WantedBy=default.target
```

今回はユーザ向けのdockerのディレクトリを ~/.docker にしてみた

```
ExecStart=/usr/bin/dockerd-rootless.sh $DOCKERD_FLAGS --data-root /home/<USERの名前>/.docker
```

参考: https://xvideos.hatenablog.com/entry/move_docker_location

## bindが1024以下の特権ポートを使えない

更に追記。  
普通にやるとnginxとかをビルドしたときに1024以下のポートは使えんぞって怒られる  

```
ERROR: for hoge_nginx  Cannot start service hoge_nginx: driver failed programming external connectivity on endpoint hoge_nginx (<HASH_IS_HERE_114514>): Error starting userland proxy: error while calling PortManager.AddPort(): cannot expose privileged port 443, you might need to add "net.ipv4.ip_unprivileged_port_start=0" (currently 1024) to /etc/sysctl.conf, or set CAP_NET_BIND_SERVICE on rootlesskit binary, or choose a larger port number (>= 1024): listen tcp 0.0.0.0:443: bind: permission denied
ERROR: Encountered errors while bringing up the project.
```

怒られるけど使いたいときはあるので許可していく

Docker デーモンをルート以外のユーザで実行（Rootless モード） - docker-docs-ja  
https://docs.docker.jp/engine/security/rootless.html#rootless-exposing-privileged-ports

エラー文で勧められた通りにrootlesskitバイナリに一部権限を渡していきます(setcap)  
上記URLの通りに作業するが、Archのrootlesskitの場所はdocker大本営の話とはちょっと違っている？っぽいので
```
$ which rootlesskit
```
に置き換えておいてみた。

`/etc/sysctl.d/` に書き足すところも、どこぞの設定に習い `/etc/sysctl.d/99-docker-rootless.conf` に置いているので、そこに追記する
