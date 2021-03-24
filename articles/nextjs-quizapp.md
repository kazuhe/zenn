---
title: "Next.js+TypeScript+Web Storageの小説クイズアプリを作ったので振り返ってみる"
emoji: "🎩"
type: "tech"
topics: ["react", "nextjs", "redux", "typescript", "webstorage"]
published: true
---
# はじめに
仕事でNext.js+TypeScript+Web Storageの小説クイズアプリを作ったので意識した点を振り返ってみました。

また、機能は簡略化していますがほぼ同じ構成のデモリポジトリを用意しています。
改善点があればコメントいただけると大変嬉しいです。

# 概要
1つの小説に対して1つのクイズが紐付けられており、クイズに正解するとWeb Storageに小説のidが保存されます。実際のプロジェクトでは全てのクイズに正解すると特別コンテンツを表示したりと他にも機能があるのですが、以下リポジトリでは最低限の機能になっています。

https://github.com/kazuhe/nextjs-quizapp

https://nextjs-quizapp.vercel.app


# アプリ構成
以下の様なディレクトリ構成にしています。
```bash
src
 ├─ components  # DOMとロジックを持つコンポーネント郡
 ├─ data  # 小説とクイズのデータを扱う
 ├─ pages  # 各ページを表す
 ├─ stores  # Redux Toolkitに関するファイル郡
 ├─ styles  # アプリ全体で使うcss
 ├─ test  # testファイル郡（仮）
 └─ utils  # 利用頻度が高いfunction/class/typeを定義
```
当プロジェクトにおいての`/components`, `/data`, `/stores`ディレクトリの役割について詳しくふれていきます。

※デモではテストを書いていないので`/test`の中身は仮です。

# ▼ components
`/components`ディレクトリはDOMとロジック分離して持つコンポーネント郡です。

基本のコンポーネントは下記の4層で構成しています。
- Import 層
- Types 層
- DOM 層
- Container 層

DOM層は一切状態を持たずContainer層でDOM層に依存を注入しています。この構成によって「見た目」と「状態」責務を完全に分離することができています。

```tsx:Example.tsx
/*
 * Import
 */
import React, { useState } from 'react'
import styles from './style.module.scss'

/*
 * Types
 */
export type Props = {
  name: string
  nameHandler: (e: React.ChangeEvent<HTMLInputElement>) => void
}

/*
 * DOM
 */
export const Example: React.FC<Props> = (props) => (
  <div className={styles.example}>
    <label htmlFor="username">Name</label>
    <input
      id="username"
      typeof="username"
      type="text"
      value={props.name}
      onChange={props.nameHandler}
      autoComplete="username"
    />
  </div>
)

/*
 * Container
 */
export const ExampleContainer: React.FC = () => {
  const [name, setName] = useState('')

  const nameHandler = (e: React.ChangeEvent<HTMLInputElement>): void => {
    setName(e.target.value)
  }

  return <Example name={name} nameHandler={nameHandler} />
}
```

下記の記事を参考にさせていただいています。
https://qiita.com/Takepepe/items/41e3e7a2f612d7eb094a

# ▼ data
`/data`ディレクトリは小説記事とクイズのコンテンツデータを扱います。

今回のプロジェクトではコンテンツの更新頻度が少なく、非エンジニアがデータを編集することは想定していません。

また、手早く構築する必要がありコンテンツ管理に費用を使えない背景もあって記事データは`data/articles.ts`に、クイズデータは`data/quizzez.ts`にそれぞれオブジェクトとして記述しています。

TypeScriptの型チェックとESLintのおかげで楽にコンテンツ管理できました。プロトタイプとか作るときにも良いかも。
```ts:data/articles.ts
export const articles: Articles = [
  {
    section_title: 'SECTION 1',
    section_subtitle: '宮沢賢治',
    section_contet: [
      {
        id: '1-1',
        title: 'セロ弾きのゴーシュ',
        introduction:
          'ゴーシュは町の活動写真館で...',
        content: [
          'ゴーシュは町の活動写真館で...',
          'ひるすぎみんなは楽屋に円く...',
          // ~~~
        ],
      },
    ],
  },
  // ~~~
]
```
```ts:data/quizzez.ts
export const quizzes: Quiz[] = [
  {
    id: '1-1',
    question: 'Question 1-1',
    choices: ['Option A', 'Option B', 'Option C'],
    answer: 'Option A',
  },
  // ~~~
]
```

また、コンテンツを取得するための関数を`utils/helper.ts`に用意しており、各コンポーネントからは下記関数を一貫して利用してコンテンツにアクセスしています。

```ts:utils/helper.ts
export function getArticleById(id: string): Article {
  const sectionContets = articles.flatMap((data) => data.section_contet)
  return sectionContets.find((data) => data.id === id)
}
```
今後、「頻繁に記事を更新することになりそうなのでコンテンツ管理をヘッドレスCMSに置き換えたい」となった場合も比較的簡単にリプレースできるように意識しました。（最善かどうかは分かりませんが..）

# ▼ stores
`stores/`ディレクトリはRedux(Toolkit) を扱う為のファイル郡です。

どの様にWeb Storage <-> アプリ間でデータの受け渡しを行うかはかなり悩んだのですが、各コンポーネントからWeb Storageには直接アクセスさせるとデータの流れが複雑になる気がしたので
1. Web Storageからデータ取得 (`stores/articles.ts`)
2. Reduxへデータ保存 (`stores/articles.ts`)
3. 各コンポーネントからデータにアクセス

の様なデータフローにしてみました。~~ここまでやらなくても良かった気がする~~

![altテキスト](https://user-images.githubusercontent.com/57878514/111955047-54a73c00-8b2c-11eb-9432-19f866c79105.png =500x)

なお、Web StorageからReduxへの保存のタイミングは`components/initialize`の初回マウント時にReduxの初期化（Web Storage のデータセット）メソッドをコールしています。

```tsx:components/initialize
// ~~~
export const Initialize: React.FC<Props> = (props) => <>{props.children}</>

export const InitializeContainer: React.FC = (props) => {
  const dispatch = useDispatch()
  const router = useRouter()

  // 初回マウント時に'Web Strage'から正解したクイズidを'Redux'へセット
  useEffect(() => {
    dispatch(initialArticleQuiz())
  }, [router.pathname])

  return <Initialize>{props.children}</Initialize>
}
```

`_app.tsx`で下記の様に`components/initialize`コンポーネントを呼び出しています。
```tsx:_app.tsx
<Provider store={store}>
  <InitializeContainer>
    <ProgressContainer />
    <Main>
      <Component {...pageProps} />
    </Main>
    <Footer />
  </InitializeContainer>
</Provider>
```
Web Storageへのデータセット・取得関数を用意しておくだけでも良かった気はしますが、開発中はReduxを意識するだけなので特に煩わしさは感じませんでした。

# さいごに
シンプルなアプリケーションほど「どれだけ拡張性を考慮しておくか」が重要になるのかなと感じたプロジェクトでした。

全く関係ないですが同時期にNuxt.jsのプロジェクトも担当していたのでキャッチアップにちょっと苦労しました... フレームワークはどれか1つを使い倒したいですね。
