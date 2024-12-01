---
title: "低予算で独自ドメインメールを使ってみる(Cloudflare+Resend)"
emoji: "📩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cloudflare", "resend"]
published: true
---

この記事は「プロもくチャットAdvent Calendar 2024」1/25日目の記事です🎄
https://qiita.com/advent-calendar/2024/puromoku

# 概要

この記事ではCloudflareで自分のドメインを取得(有償)し、各サービスの無料範囲内でメールの受信、受信ができるようになることを目指します。
以下では個人のGmailアカウントでドメインのメールを送受信するように設定していきます。

# はじめに

最近連絡先交換のため名刺を検討しているのだが、その内容を考えているときにふと思ったのが

**あれ、弊社独自ドメインのメールなくね...? 🤔**

ということである。

まぁ昔から続く町工場なので使う機会も少ないのだが、今時メールぐらいはプロバイダのドメインが入ったやつじゃなく、見栄えのいい独自ドメインを使いたいのが人間のサガというやつである。
ただ無料サービスでなんとかスモールスタートできんかなという思いがあり、ざざっと調べたところいい感じの手段があったのでまとめてみる。

# 手順

## ドメイン取得

何はともあれドメインを取得する。
TLDについて、わりと周りのメーカさんは `.co.jp` ドメインを使っていることが多いのだが、正直 `.net` とか `.com` を使うメーカさんもいるので、ひとまずcloudflareで安めの `.com` でとることにした。
というのと弊社の名前が割と平凡なので、すでに `.co.jp` で取りたいドメインが取られていたといこともある。

この記事ではCloudflareでドメインを取得した体で解説を進める。

## メール受信設定

メール受信設定はCloudflareにて行う。
Cloudflareのホーム画面にドメインが並んでいるので、自分のドメインを選択

![](/images/freecompanymail-on-cloudflare/domain_selection.webp)

### メールアドレスの作成

今回は作成したメールアドレスにきたメールを、自分の持っているアドレス(今回はGmail)に転送する設定をしていく。
「メールアドレス」→「Email Routing」→「ルーティングルール」から、
「カスタムアドレス」→「アドレスを作成」

![](/images/freecompanymail-on-cloudflare/custom_address.webp)

上記「カスタムアドレス」には使いたいアドレス、  
「宛先」に転送先のアドレスを入力して「変更」を押下。

![](/images/freecompanymail-on-cloudflare/custom_address_approve.webp)

指定したメールアドレスに確認のVerify emailが来ているので、「Verify email address」を押下。

![](/images/freecompanymail-on-cloudflare/verify_email.webp)

設定したemailにメールを送って転送がされていればOK

## メール送信設定(Resend)

次に送信。
今回はわりとZennで見かけたのでResendをつかう。

### ドメインを登録

Resendは無料の場合1ドメインのみの登録となるため、追加で画像収録がしづらかったため画像すくなめ。

「Domains」から「Add domains」を選択し、自分のドメインを登録する。

![](/images/freecompanymail-on-cloudflare/resend_dns_info.webp)

### DNSレコードをCloudflareに登録

Domains設定画面にDNSレコードが出てくるので、Requiredのほうの3レコードをCloudflareに登録していく。
Cloudflareのドメインごとの設定画面から「DNS」→「レコード」を選択。

![](/images/freecompanymail-on-cloudflare/dns_record.webp)

ここのレコードの追加から各レコードを追加すればOK。
なお上の画像はすでに追加されている状態。
ちなみにこのあとDNSの検証が必要だと思うが、画像をとっていなかったため手順は不明...。
他の手順を参照いただければと思う。

### API Keyを作成

次にAPI Keyを作成する。
このkeyがSMTP時のパスワードにもなる。
「API Keys」から「+Create API Key」を選択。
Permissionは今回送信にしか使わないのでSending access、Domainは登録したドメインとする。

![](/images/freecompanymail-on-cloudflare/resend_apikey.webp)

### SMTPの設定

メーラーでやり取りするならSMTPの設定が必要になる。
Resendの「Settings」から「SMTP」をみると各設定があるので参照のこと。

![](/images/freecompanymail-on-cloudflare/resend_SMTP.webp)

これで送信も工事完了です...。

# 注意

無料で使えているということは当然限りはあるので確認。

## 受信

https://developers.cloudflare.com/email-routing/limits/
CloudflareのLimitsを確認したが、おそらくは1通25MiBまでが限界と思われる。

https://developers.cloudflare.com/workers/platform/limits/#worker-limits
またあまり複雑な内容だとCloudflareのWorkerの制限をこえるからメールが落ちるぞという話だと思うんだが、
ひとまず受信メールの転送のみなのであまり問題にはならないであろう(楽観視)。
1000リクエスト/分、10万リクエスト/日まで良いらしいので、比較的こちらはいけそう。

## 送信

制限としてはこちらが多そう。
とはいえResendは月に3000通、日に100通までなら大丈夫そうである。
そこまでHeavyに使わないので最初のうちは余裕がありそう。

# まとめ

無料で使う分もあって色々ガバガバなのであまり重要なメールには使えないかもしれないが、ひとまず体裁は整えられたと思う。
使用する場合はあくまでタダで使わせていただいているという認識で自己責任にて。
