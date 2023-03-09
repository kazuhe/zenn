---
title: "Web フロントエンドにおける API モック戦略"
emoji: "🐥"
type: "tech"
topics: ["frontend", "typescript", "vue", "test", "msw"]
published: true
---

# はじめに

新規開発のプロジェクトでテスト戦略を立ててしばらく開発をしています。そのテスト戦略の内の 1 つに Web API モックの運用ポリシーを決めていたのですが、大きな問題がなく運用ができているので「API モック戦略」と大袈裟に題してみました。

特に奇抜なことはしていないですが、振り返りとしてまとめたいと思います。

# 目的

- Web フロントエンドの開発時にモック API を利用して動作確認・自動テストをしたい
  - 開発サーバを起動するだけでブラウザで動作確認をしたい
  - [Testing Library](https://testing-library.com/) などを用いた UI コンポーネントの振る舞いを自動テストしたい
- モックデータの管理を簡単にしたい
  - ブラウザでの動作確認と自動テストで同じモックデータを利用したい

# 前提

API のモックサーバは [Mock Service Worker](https://github.com/mswjs/msw) （以下 MSW と呼ぶ）を利用する。MSW のメリットや利用方法は公式ドキュメントや分かりやすい記事が多いのでこの記事では省略する。

ここで紹介するコードは以下のリポジトリにあるので設定など参考にしていただけると嬉しい。Vue3 を利用しているが、React でも同じ様なことができる。

https://github.com/kazuhe/frontend-mock-strategy

この記事では例として以下の様な書籍の取得と登録のリクエストができる API があるとする。

- 取得
  - リクエストメソッド: GET
  - エンドポイント: `/api/books`
- 登録
  - リクエストメソッド: POST
  - エンドポイント: `/api/books`

# ポリシー

運用方針を決めておかないと後に管理が大変になることが想像できたので、あらかじめ以下のルールを決めて実際に運用した。

1. API のエンドポイントと同じ階層構造でモックを配置する
2. API のレスポンスには型注釈をする

## 1. API のエンドポイントと同じ階層構造でモックを配置する

モックとその設定は `src/mocks` の配下に設定する。全体像は以下に示した様になる。

```bash
src/mocks
├── api # ここより下は API のエンドポイントに合わせる
│   └── books
│       ├── index.ts
│       └── response.ts
├── browser.ts
├── handlers.ts
├── server.ts
└── types.ts
```

それぞれのファイルの中身を説明していく。

### `src/mocks/types.ts`

まず、MSW のハンドラーに登録する関数の型を定義する。

```ts: src/mocks/types.ts
import type {
  DefaultBodyType,
  PathParams,
  ResponseResolver as mswResponseResolver,
  RestContext,
  RestRequest
} from 'msw'

export type ResponseResolver = mswResponseResolver<
  RestRequest<never, PathParams<string>>,
  RestContext,
  DefaultBodyType
>;
```

### `src/mocks/api/books/response.ts`

API のエンドポイントに合わせて `/api/books` ディレクトリを作成してレスポンスのオブジェクトを定義する。このファイルはレスポンスのオブジェクトにのみ関心がある。

```ts: src/mocks/api/books/response.ts
import type { Book } from '@/types/book'

type Response = { bookList: Book[]; totalCount: number }

export const response: Response = {
  bookList: [
    {
      isbn: '111-1-11-1111-1',
      name: 'Book01'
    },
    {
      isbn: '222-2-22-2222-2',
      name: 'Book02'
    }
  ],
  totalCount: 2
}
```

### `src/mocks/api/books/index.ts`

同階層に `handlers.ts` に登録する関数を配置する。この例は簡易だが、クエリパラメータを見てレスポンスを出し分けたりするようになると割と記述量が多くなってくる。

```ts: src/mocks/api/books/index.ts
import type { ResponseResolver } from '@/mocks/api/types'

import { response } from './response'

/**
 * 書籍一覧を取得する
 */
export const getBookList: ResponseResolver = (_, res, ctx) => {
  // 特定の条件で別のレスポンスをする
  // const foo = req.url.searchParams.get('foo')
  // if (foo === 'foo') {
  //   この様な記述が増えていく 
  // }
  return res(ctx.status(200), ctx.json(response))
}

/**
 * 書籍一覧を登録する
 */
export const postBook: ResponseResolver = (_, res, ctx) => {
  return res(ctx.status(201))
}
```

### `src/mocks/handlers.ts`

`handlers.ts` は容易に肥大化してしまうのでルーティングのみに徹する。修正したいモックがあれば、まずこのファイルを見ることになる。

```ts: src/mocks/handlers.ts
import { rest } from 'msw'

import { getBookList, postBook } from '@/mocks/api/books'

export const handlers = [
  rest.get('api/books', getBookList),
  rest.post('api/books', postBook)
]
```

### `src/mocks/browser.ts`

Service Worker の設定と起動をする。このファイルはブラウザで API をモックするための設定をしている。（これは [公式](https://mswjs.io/docs/getting-started/integrate/browser) そのまま）

```ts: src/mocks/browser.ts
import { setupWorker } from 'msw'

import { handlers } from '@/mocks/handlers'

export const worker = setupWorker(...handlers)
```

### `src/mocks/server.ts`

`browser.ts` とほぼ同じ内容だが、こちらは Node.js で実行される自動テストで API をモックするための設定をしている。（これも [公式](https://mswjs.io/docs/getting-started/integrate/node) そのまま）

```ts: src/mocks/server.ts
import { setupServer } from 'msw/node'

import { handlers } from '@/mocks/handlers'

export const server = setupServer(...handlers)
```

## 2. API のレスポンスには型注釈をする

レスポンスのオブジェクトとして作成している `response.ts` に API レスポンスの型を注釈する。

プロジェクトでは [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator) を利用して API クライアントと関連する型を自動生成していたので、生成された型を注釈している。

```ts: src/mocks/api/books/response.ts
// 自動生成されたレスポンスの型
import type { Response } from '@/foo/hoge'

export const response: Response = {
  bookList: [
    {
      isbn: '111-1-11-1111-1',
      name: 'Book01'
    },
    {
      isbn: '222-2-22-2222-2',
      name: 'Book02'
    }
  ],
  totalCount: 2
}
```

API のレスポンスが変更・追加された時には TypeScript がエラーを出してくれる。

# 確認

簡易だが以下の様に `/api/books` に対して書籍一覧を取得する Vue テンプレートを用意した。開発サーバーを立ち上げて（`npm run dev`）ボタンをクリックするとモック API からレンスポンスされていることが確認できる。

```vue: src/views/HomeView.vue
<script setup lang="ts">
import { ref } from 'vue'
import type { Book } from '@/types/book'

const bookList = ref<Book[]>([])

const fetchBookList = async () => {
  const res = await fetch('/api/books')
  const body = await res.json()
  bookList.value = body.bookList
}
</script>

<template>
  <div class="container">
    <button
      type="button"
      data-testid="fetch-book-list-button"
      @click="fetchBookList"
    >
      書籍一覧を取得する
    </button>
    <ul>
      <li v-for="book in bookList" :key="book.isbn">
        isbn: {{ book.isbn }}, name: {{ book.name }}
      </li>
    </ul>
  </div>
</template>
```

モック API に対してリクエストを行い、レスポンスを表示していることを自動テスト（`npm run test:unit`）して合格するこが確認できた。

```ts: src/views/HomeView.test.ts
import { fireEvent, render, screen } from '@testing-library/vue'
import HomeView from '@/views/HomeView.vue'

describe('HomeView.vue', () => {
  // ...略

  it('「書籍一覧を取得する」ボタンを押下すると一覧が表示されること', async () => {
    expect(screen.queryByTestId('book-item')).toBeNull()
    const fetchBookListButton = await screen.findByTestId('fetch-book-list-button')
    await fireEvent.click(fetchBookListButton)
    await screen.findByText('isbn: 111-1-11-1111-1, name: Book01')
    await screen.findByText('isbn: 222-2-22-2222-2, name: Book02')
  })
})
```

# さいごに

- ルーティングに徹する人
- レスポンスを出し分ける人
- レスポンスを定義する人

の様な役割でファイルを分割すると、エンドポイントやテストの為のレスポンスの出し分け条件が増えてきた時も見通しがよいと感じました。

MSW を利用してポリシーに従ってモックデータを作成することで、目的を達成することができました。最後まで読んでいただきありがとうございました。
