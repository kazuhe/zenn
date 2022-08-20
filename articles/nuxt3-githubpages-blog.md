---
title: "Nuxt3 + GitHub Pages で ﾎｰﾑﾍﾟｰｼﾞ兼 ﾌﾞﾛｸﾞを作成してみた"
emoji: "🏠"
type: "tech"
topics: ["nuxtjs", "typescript", "github"]
published: true
---

# はじめに
簡易的なホームページ兼ブログを作成しました。作りながら以下のスクラップに雑にまとめていたので、少しだけ整理して記事にしてみました。

https://zenn.dev/kazuhe/scraps/3dd0d2c83c2e4e

実際の手順はスクラップに記載しているので要点だけ書きたいなと思います。

# 構成
とにかく手をかけずに作成したかったので以下の様な構成にしました。

- フレームワーク
  - [Nuxt3](https://v3.nuxtjs.org/) (ホームページを作成開始した当時 `3.0.0-rc.6`)
  - https://v3.nuxtjs.org/
- ブログコンテンツ管理
  - [@nuxt/content](https://content.nuxtjs.org/) (`2.0.1`)
- ホスティングサービス
  - [GitHub Pages](https://pages.github.com/)

そろそろ Nuxt3 が正式リリースされる？などの噂を耳にしていましたが、気になっていたので試してみることにしました。また、コンテンツ管理は CMS を導入するほど大袈裟にしたくなかったので `@nuxt/content` に決定しました。

実際のコードは以下のリポジトリです。

https://github.com/kazuhe/kazuhe.github.io

# ポイント

以下に簡単に要点をまとめます。

## GitHub Actions で GitHub Pages へ継続的デプロイ

ブログ作成開始の少し前に以下の機能がリリースされたので早速利用しました。

https://github.blog/changelog/2022-07-27-github-pages-custom-github-actions-workflows-beta/

リポジトリの `Settings > Pages` の `Build and deployment` の `Sorce` を `GitHub Actions` に設定して、

![](https://storage.googleapis.com/zenn-user-upload/fc1a41c3bce7-20220820.png)

以下のファイルを作成しました。

```yml: .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: "16"
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v1
      - name: Install dependencies
        run: npm i
      - name: Static HTML export with Nuxt
        run: NUXT_APP_CDN_URL=https://kazuhe.github.io/ npm run generate
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./dist

  deploy:
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1
```

GitHub Pages のサイト URL を環境変数 `NUXT_APP_CDN_URL` にセットしていますが必須ではないかもしれません。

main ブランチにマージすると意図した通り自動でリソースを公開してくれました。

https://github.com/kazuhe/kazuhe.github.io/actions/runs/2769003556

## @nuxt/content でブログコンテンツを管理

ブログ作成開始の少し前に Nuxt3 に対応した @nuxt/content を利用しました。

[公式ドキュメント](https://content.nuxtjs.org/get-started)に従ってセットアップしました。

```bash
npm i -D @nuxt/content
```

nuxt.config の `modules` に追記します。私は `srcDir: "src/"` を設定してたので、`content` に `@nuxt/content` で管理したいコンテンツの場所を教えてあげました。

```ts:  nuxt.config.ts
export default defineNuxtConfig({
  modules: ["@nuxt/content"],
  srcDir: "src/",
  content: {
    sources: [path.join(__dirname, "content")],
  },
});
```

`/content` ディレクトリに新しくマークダウンを作成して、

```md: content/blog/2022/08/new-blog.md
---
title: ホームページを開設しました
description: ふと思い立ってホームページを開設しました
created_at: Sat Aug 06 2022 18:02:39 GMT+0900
tags: ["Nuxt", "nuxt/content"]
icon: 🎉
---

ふと思い立ってホームページを開設しました。
~~
```

pages ディレクトリに .vue ファイルを作成して、

```vue: src/pages/blog/[...slug].vue
<template>
  <div>
    <ContentDoc v-slot="{ doc }">
      <!-- ~省略~ -->
      <ContentRenderer class="markdown-body" :value="doc" />
    </ContentDoc>
  </div>
</template>
```

`npm run dev` して `http://localhost:3000/blog/2022/08/new-blog` にアクセスすると表示されました。

:::message
作成当時にはバグがありました。[スクラップ](https://zenn.dev/link/comments/38a984466f6eb7) に記載しているのでご参考までに
:::

## zenn の自分の記事をブログの一覧に表示

Zenn の記事も `@nuxt/content` のコンテンツらと合わせてブログに表示させたかったので、`rss-parser` というライブラリを利用して、[Zenn の RSS](https://zenn.dev/zenn/articles/zenn-feed-rss) から記事を取得しました。

```bash
npm i -D rss-parser
```

RSS から取得した記事らと、先に設定した nuxt/content の記事をそれぞれ取得してコンポーネントが期待する型にコンバートしています。その後に最新順に並び替えて表示しています。

```vue: src/pages/index.vue
<script setup lang="ts">
import Parser from "rss-parser";
import { Blog, sortBlogs } from "@/domain/blog"; // コンテンツの型
import BlogCard from "@/components/BlogCard.vue"; // コンテンツを表示する UI コンポーネント

/**
 * zenn の RSS からコンテンツを取得する
 */
const fetchZennContent = (): Promise<Blog[]> =>
  Promise.resolve(
    new Parser().parseURL("https://zenn.dev/kazuhe/feed?all=1").then((res) =>
      res.items.map((d) => ({
        title: d.title,
        description: d.content,
        path: d.link,
        createdAt: d.pubDate,
        icon: "z",
        type: "zenn",
      }))
    )
  );

/**
 * `/content` からコンテンツを取得する
 */
const fetchOwnContent = (): Promise<Blog[]> =>
  queryContent("blog")
    .find()
    .then((res) =>
      res.map((d) => ({
        title: d.title,
        description: d.description,
        path: d._path,
        createdAt: d.created_at,
        icon: d.icon,
        type: "own",
      }))
    );

const { data: blogs } = await useAsyncData("blogs", () =>
  Promise.all([fetchZennContent(), fetchOwnContent()]).then(sortBlogs)
);
</script>

<template>
  <div>
    <ul>
      <li v-for="blog in blogs" :key="blog.path">
        <blog-card
          :title="blog.title"
          :description="blog.description"
          :path="blog.path"
          :created-at="blog.createdAt"
          :icon="blog.icon"
          :type="blog.type"
        />
      </li>
    </ul>
  </div>
</template>

```

日付とかいろいろ雑ですが...~~（面倒くさくなってきた）~~ 画像の様に一覧表示されました。

![](https://storage.googleapis.com/zenn-user-upload/3e8611da56e0-20220820.png)

## GitHub の PR 情報を取得して表示

GitHub API の実行回数の上限が予想以上に厳しかったので (Client Secrets を設定すれば緩和されるっぽいですが GitHub Pages でホスティングしているので...) ローカルで実行した結果を .vue コンポーネントにコピペすることにしました。良い方法があれば教えてほしい。

:::message alert
これは **かなりイケてない** と思っていますが勇気をもって公開
:::

```js: scripts/github-pulls.js
import fs from "fs";
import path from "path";
import fetch from "node-fetch";

/**
 * GitHub の Pull Request を取得する
 */
const fetchPulls = async (owner, repo, number) =>
  fetch(`https://api.github.com/repos/${owner}/${repo}/pulls/${number}`)
    .then((reponse) => reponse.json())
    .then((d) => ({
      title: d.title,
      description: d.body.substring(0, 150) + "...",
      url: d.html_url,
      createdAt: d.created_at,
      number: d.number,
      repo: d.base.repo.full_name,
    }))
    .catch((err) => {
      console.error(err);
      return undefined;
    });

(async () => {
  // 取得したい PR の情報を渡して実行する
  const pulls = await Promise.all([
    fetchPulls("zenn-dev", "zenn-editor", 151),
    fetchPulls("microcmsio", "microcms-js-sdk", 10),
  ]);

  await fs.promises.writeFile(
    path.resolve("json", "github-pulls.json"),
    JSON.stringify(pulls)
  );
})();
```

本当は自分以外のリポジトリに対して作成した PR を一覧で取得したい。

```json: package.json
{
  "scripts": {
    "generate:github-pulls": "node scripts/github-pulls",
  },
}
```

`npm run generate:github-pulls` を実行すると json ファイルが作成されるので、コピーして直接 .vue ファイルに貼り付けます。
更新頻度はかなり少ないので面倒過ぎますが妥協しました。

```vue: src/pages/activity.vue
<script setup lang="ts">
/**
 * from json/github-pulls.json
 */
const activities = [
  {
    title: "chore: Added sidebar minimize button to Zenn CLI preview",
    description:
      "https://github.com/zenn-dev/zenn-editor/issues/141\r\n\r\n1画面で執筆している場合には Zenn CLI のプレビューでサイドバーを非表示にできると執筆体験が向上すると思いましたのでPRさせていただきました。\r\n\r\nなるべく既存コードを変更しない様に...",
    url: "https://github.com/zenn-dev/zenn-editor/pull/151",
    createdAt: "2021-05-31T08:22:33Z",
    number: 151,
    repo: "zenn-dev/zenn-editor",
  },
  {
    title: "chore: Error handling of makeRequest function",
    description:
      'https://github.com/wantainc/microcms-js-sdk/issues/9\r\nエラーハンドリング関連でPRさせていただきます。\r\n\r\n```javascript\r\nconst client = createClient({\r\n  serviceDomain: "YOUR...',
    url: "https://github.com/microcmsio/microcms-js-sdk/pull/10",
    createdAt: "2021-05-26T08:50:05Z",
    number: 10,
    repo: "microcmsio/microcms-js-sdk",
  },
];
</script>

<template>
  <div>
    <ul>
      <li
        v-for="activity in activities"
        :key="activity.createdAt"
      >
        <!-- ~~ -->
      </li>
    </ul>
  </div>
</template>
```

実装はどうであれやりたかったことはできました。

![](https://storage.googleapis.com/zenn-user-upload/11b9fd62f539-20220820.png)

## GoogleAnalytics を設定

これはサクッと

```sh
npm i -D vue-gtag-next
```

```ts: src/plugins/vue-gtag.client.ts
import VueGtag from "vue-gtag-next";

export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.use(VueGtag, {
    property: {
      id: "<ID>",
    },
  });
});
```

# 最後に

いろいろと雑な部分がありますが、供養できたのでスクラップをクローズしちゃいます。
もっと良い方法がたくさんあると思いますので、気付いた点があればコメントいただけるとすごく嬉しいです。
