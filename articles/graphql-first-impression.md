---
title: "🔰GraphQLをLaravel+Reactで使ってみる"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GraphQL"]
published: true
---

初アドベントカレンダーなので初投稿です
プロもくチャット Advent Calendar 2023 の2/25日目です。 ~~あれ、投稿日...?~~
https://qiita.com/advent-calendar/2023/puromoku

# まえがき
エンジニアを退職して久しいが、「たとえ仕事は変わっても刃は研ぎ続けなさい」という新卒の頃の上司に言われた言葉を受けて、少しずつだがプログラムを書き続けている工場のおじさんです。
そんなこんなで、エンジニア時代に触っていなかったNextやらGraphQLに手を出すことにしているのだが、GraphQLが面白かったので、アドベントカレンダにかこつけてまとめてみようと思う。

# GraphQLについて
## 比較: REST APIのつらみ
通常、サーバから情報をもらってくるときにはREST APIをよく使っていたが、これには不満がある。
例えばUserの情報を取ってくるとして、nameとageしか必要ないのに全部の情報が取れてしまうということがある。
```json
{
  id: "114514",
  email: "tadokoro@example.com",
  name: "田所 浩二",
  age: "24",
  gender: "2",
  // その他特に興味のない情報etc...etc...
}
```
もしあまりにも取得するコストが高いときは、別にエンドポイントを作ればよいかと思うが、
これがgenderだけ確認したいとかageだけ見たいなどあれば、その都度エンドポイントが切られて管理がどんどん煩雑になってしまう。

- `/API/v1/user/email_name_age`
- `/API/v1/user/name_age`
- `/API/v1/user/name_gender`
- ...他、細々した違いで作られる有象無象のURL

これは大変。どうしよう。
お客様。そんなときはこのGraphQLはいかがでしょうか。となるわけである。
以下いくつか利点を示す。

## いいところ
### URLがごちゃごちゃしない
RESTfulなAPIでは上記の例のように都度エンドポイントを切って、ということをしていたわけだが、
GraphQLでは `/graphql` など1つのエンドポイントに対して、POSTで特定のクエリを投げることで様々な情報を取得できる。
この点でURLがごちゃごちゃしないので気持ちがいい。
:::message
エンドポイントについては自分で設定ができるので適当に設定したらよい
:::

### クライアントが要る情報だけもらえる
またこれもGraphQLの大きな違いとしては、クライアント側から欲しい要素を選択できる点にある。
実際に情報を取得するクエリがこれだが、

```graphql
query ($id: ID!) {
  User(id: $id) {
    name
    email
    age
  }
}
```

これで取得できるのはこんな感じ

```json
{
  "data": {
    "User": {
      "name": "田所 浩二",
      "email": "tadokoro@example.com",
      "age": 24
    }
  }
}
```

こうやって欲しい情報を記述したクエリを投げることで、欲しい分だけの情報が手に入れられることになる。SDGsだねぇ。(よくわかってない)

また、上記ではqueryで取得を行っていたが、DB操作におけるCRUDのうち、ReadのみがQuery、あとはMutationと2つに分けられている。
これらは読み込み系がQuery、それ以外の変更をかける系の処理がMutation、とかなり整理されているように思う。

|DB操作|GraphQL|
|:--:|:--:|
|Create|Mutation|
|Read|Query|
|Update|Mutation|
|Delete|Mutation|

:::message
ほかにもリアルタイムに通信ができるSubscriptionというものがあるが、使ったことがないため割愛
:::

# 使ってみる
と、さっくり把握したところで、実際に使ってみる。

## 構成
今回試した構成はこちら
- Laravel
  - Lighthouse
- Next v13.4
  - Urql

LighthouseはLaravelでGraphQL APIを提供してくれるフレームワーク、
GraphQLサーバからの取得はUrqlでやってみる。

:::message
細かな手順などについては手が回らなかったため端折っています。
大枠の流れをつかむことをメインにしていますので、実際にやる場合は公式の手順を追ってください。
:::

### 1. Lighthouseの設定

[公式のInstallation](https://lighthouse-php.com/5/getting-started/installation.html)を参考にスキーマ定義というものを行う。
手順に従うとlaravelのディレクトリに `graphql/schema.graphql` というファイルが生成される。
これの中にフロントからアクセスできるようにスキーマを定義していく。
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  created_at: DateTime!
  updated_at: DateTime!
}
```

ぱっと見でお分かりかと思うが、GraphQLでは型指定をする必要があるため、バックフロント別で型定義をすることなく、一気通貫で型安全な通信ができてしまうのである。
:::message
しこたまどうでもいいのだが、「一気通貫」という言葉が一般語彙か確認するためにggった所、麻雀用語だったらしい。
一度も麻雀やったことないのだが。
:::

ちなみにこのlighthouseくんはわりと横着もできて、以下のようなスキーマでモデルの内容をそっくりそのまま使えたりする。
```graphql
type Query {
  users: [User!]! @all
}
```
Queryはルートの指定になるが、ここのUserという部分がモデル名になり、@allは全権取得のためのディレクティブになる。
これは@whereや@paginationなどがあり、ORM(EloquentORM)になじみがあればほぼ迷わずかけると思われる。
モデルを書いていくというあたりはRESTfulなAPIからの移行の時には大きく構造を崩さずに導入が進められそう。

### 2. GraphiQLからGraphQLをいじる
GraphiQL(グラフィカルって呼ぶのかな)というツールを使うとDBに簡単にクエリを投げられるようになるので、開発時には重宝する。
php屋さんとしてはGraphQLでできることに特化した **使いやすい** PHPMyAdminという印象である。
```bash
$ composer require mll-lab/laravel-graphiql
```
`/graphiql` とかからアクセスできるようになるので、本番環境では消し忘れないように注意。

GraphQLに慣れないうちはここでQueryやMutationを試し打ちして慣れるとよさそう。

### 3. Reactで取得してみる

さて、フロントから取得してみる。
フロント側の細かい構成やらインストールやらについてはバッサリカットして、とりあえずReactでUrqlを使う前提で書いてみる。
以下はReact+UrqlのGetting started

https://formidable.com/open-source/urql/docs/basics/react-preact/

使い方としては、接続情報を詰めたclientインスタンスを使ってクエリを投げる形か、
```typescript
import { Client, cacheExchange, fetchExchange } from 'urql';

const client = new Client({
  url: 'http://localhost:3000/graphql',
  exchanges: [cacheExchange, fetchExchange],
});
```
プロバイダで囲ってやる形になる。
```typescript
import { Client, Provider, cacheExchange, fetchExchange } from 'urql';

const client = new Client({
  url: 'http://localhost:3000/graphql',
  exchanges: [cacheExchange, fetchExchange],
});

const App = () => (
  <Provider value={client}>
    <YourRoutes />
  </Provider>
);
```
おそらく後者をよく使うことになるかと思う。

実際にユーザを取得するコードを書くとこう。

```typescript
import { gql, useQuery } from 'urql';

const UsersQuery = gql`
  query {
    users {
      id
      name
    }
  }
`;

const Users = () => {
  const [result, reexecuteQuery] = useQuery({
    query: UsersQuery,
  });

  const { data, fetching, error } = result;

  if (fetching) return <p>Loading...</p>;
  if (error) return <p>Oh no... {error.message}</p>;

  return (
    <ul>
      {data.users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};
```

まずUsersQueryにgraphqlのクエリを書いておき、UrqlのuseQueryにクエリを渡して取得、という流れになる。

# まとめ
今回気力の都合で取得のみをまとめたが、実際GraphQLを使い初めて最初のうちはどのプラグインを使えばいいか迷ったり、取得の流れがわからなかったりという事があったが、
ひとたび勝手がわかれば普段作るようなプロジェクトであれば大概GraphQLにしてもよいのでは...？と思う程度には開発中の体験はよくなったように思う。
ぜひぜひ使ったことがない方は使ってみてほしい。
