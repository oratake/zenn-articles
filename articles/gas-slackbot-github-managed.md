---
title: "Slackに潤いを与えるSlackbotをGASで運用しよう"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleAppsScript", "Slackbot"]
published: false
---

# はじめに

この記事は「プロもくチャット Advent Calendar 2023」11/25日目の記事です📚
https://qiita.com/advent-calendar/2023/puromoku

# Slackbot、してますか

世のプログラマが日夜入りびたる場所、Slack。
彼ら/彼女らを惹きつけてやまないSlackには至高のおもちゃ「Slackbot」が幅を利かせているという。
我々スタッフは事実を確かめるべく、魑魅魍魎が跋扈するSlackの奥地へと歩を進めるのであった―――

## Slackbotとは
冒頭で疲れたので普通に書きます。
チームメンバなどと日々のコミュニケーションを取るためにSlackは不可欠です。
しかし皆さんはこう思ったことはないでしょうか。

**混沌が足りない**

と。

:::message
個人の感想です。
:::

そんな殺伐としたSlackに颯爽と現れたのがタイトルにもあります「Slackbot」になるわけです。
Slackでは投稿やチャンネル作成などのイベントがあると、所定の場所にPOSTリクエストを投げて知らせてくれる機能があるため、これを~~悪用~~活用することによってSlackの治安維持に役立てることができます。

ちなみに、通常Slackbotと言った場合はSlack標準で自動メッセージやリマインダなどの時に出てくるコイツ↓のことを指しますが、
![slackbotくん](https://a.slack-edge.com/80588/marketing/img/avatars/slackbot/avatar-slackbot@2x.png =100x)
*Slackbotくん(公式)*

今回の記事では投稿に反応して所定の返答をする自作野良アプリのことをSlacbotということにします。
(広義のChatbotといった方がよさそうな気もする)

## 実例

実際にプロもくのSlackにて稼働しているイメージをいくつか持って来てみました。
解説用に作成したため委細は異なりますことご承知おきください。


