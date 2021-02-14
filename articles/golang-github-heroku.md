---
title: "GolangのWebアプリケーションをとにかく簡単に公開したいのでHerokuにデプロイしてみた"
emoji: "🏃‍♀️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go", "github", "heroku"]
published: true
---

# はじめに
フロントエンドエンジニアとしてお仕事をさせていただいている私ですが、HTTPやデータベースなどを学びたくて勉強がてらGolangで簡単なAPIを作ってみました。
DockerやAWSを使ってデプロイしたいと考えましたが現在学びたい領域はインフラではないので、とにかく簡単に公開するために普段使い慣れているHerokuを使って公開してみました。

本当に分からない事だらけなので、間違っている点があれば優しくご指摘いただけると幸いです。

# 概要
GitHubにソースコードをpushするとあらかじめ連携しておいたHerokuに自動でデプロイされる様に設定しています。また、データベースはPostgreSQLを使用しています。

リポジトリは下記になりますが、これから手を加えていく予定ですので2021/02/15時点のコードを参考にしてもらえればと思います。
https://github.com/kazuhe/gocomm

:::message
GitHub及びHerokuの登録と正常に動作するGolangのコードが用意されている前提で進めます。
:::
:::message
今回はHerokuのサービス名を`go-comm`としています。これは一意の名前になるので適宜ご自身のサービス名に変更してコマンドの実行をしてください。
:::

# コードの変更
Herokuに公開する場合は一部コードを変更する必要があります。
Herokuにデプロイする場合はポート番号を自由に設定できない様なので環境変数からポート番号を読み取るように変更します。
```go
server := http.Server{
  // 環境変数からポート番号を取得
  Addr: ":" + os.Getenv("PORT"),
}
```

次に、データベースドライバ名とデータソース名を指定する記述を変更します。こちらも環境変数から取得される様に設定しておきます。
```go
// データソース名を環境変数から取得
DB, err = sql.Open("postgres", os.Getenv("DATABASE_URL"))
```

# 依存ファイル郡を取得
Helokuではgodepを使って依存ファイルを管理します。`go get`でgodepを取得します。
```bash
$ go get github.com/tools/godep
```

次にWebアプリケーションのルートディレクトリで下記コマンドを実行します。これによって依存ファイルがディレクトリ内に作成されます。
```bash
$ godep save
```

# Procfileの作成
ルートディレクトリに`Procfile`の名前でファイルを作成します。これはHerokuのビルドが完了した後に実行するファイルを指定しています。
```Procfile:Procfile
web: gocomm
```
:::message
Herokuのサービス名は`go-comm`ですが、コード自体のディレクトリ名は`gocomm`なのでここでは`web: gocomm`となります。
:::

# HerokuにPostgreSQLを用意
まずはHerokuにログインします。
```bash
$ heroku login
```

アドオンを使ってデータベースを追加します。`--app=go-comm`の`go-comm`はご自身のサービスに変更してください。
```bash
$ heroku addons:create heroku-postgresql:hobby-dev --app=go-comm
```

これによってデータベースが作成され環境変数が設定されます。以下のコマンドでアプリの構成変数を一覧表示し、`DATABASE_URL`の値を確認することができます。
```bash
$ heroku config --app go-comm

=== go-comm Config Vars
DATABASE_URL: postgres://dlrhbionqjnobw:988d2b~~~~~
```

データベースが作成できたら、Herokuのpsqlにログインします。
```bash
$ heroku pg:psql --app go-comm
```

ログインした状態でテーブルを作成します。今回はセットアップ用のSQLファイルを用意していたのでそれを実行しました。適宜書き換えてください。
```bash
$ \i ./data/setup.sql
```

テーブルが作成されているのを確認できればOKです。
```bash
$ \d
```

![](https://storage.googleapis.com/zenn-user-upload/9e0f4yfp5wg7c684koypdmm0cojv)

# GitHubとHerokuの連携
Herokuのプラットフォームにログインして`Deploy`タブから対象のGitHubリポジトリの設定と自動デプロイを有効化するだけで連携は完了です。
![](https://storage.googleapis.com/zenn-user-upload/vtmkzin1dokjinvnjsfa0m5morr7)

連携が完了したらあとはGitHubにコードの変更をpushすると自動デプロイが実行されるはずです。

# curlでAPIの動作確認
**API自体の説明は記事の趣旨から逸れるので省きます**が、curlを使ってAPIが正常に動作するか確認してみます。新しい投稿を追加するためにJSONデータをPOSTします。
```bash
$ curl -i -X POST -H "Content-Type: application/json" -d '{"content": "My first post", "author": "kazuhe"}' https://go-comm.herokuapp.com/post/
```

先ほどPOSTした投稿をGETリクエストで取得してみます。
```bash
$ curl -i -X GET https://go-comm.herokuapp.com/post/1
```

ｷﾀｰ----!!
![](https://storage.googleapis.com/zenn-user-upload/2bzfe8re9q6bdg69nlhpdbjjvleq)

意図したレスポンスを取得できたのでこれにて公開完了です。

# 注意
ひとまず公開は完了しましたが今のままでは誰でもAPIを叩けてしまいます。
セキュリティ的によろしくないと思うのでAPIキーの設定等（まだ勉強不足ですが他にもやるべきことはありそう..）を今後していこうと思います。

※このWebアプリケーション（Herokuサービス）は一旦非公開にしています。

# 参考
公式が分かりやすいので詳細はこちらからご確認ください。日本語対応もしているみたいです。
https://devcenter.heroku.com/articles/getting-started-with-go#use-a-database
https://devcenter.heroku.com/articles/go-dependencies-via-godep