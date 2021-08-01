---
title: "TypeScriptで型駆動開発"
emoji: "💭"
type: "tech"
topics: ["typescript", "vue"]
published: false
---

- 型駆動開発とは？
  - 型駆動開発で何が嬉しいのか
  - 実際にどうやるのか
    - まず欲する機能のインターフェースを定義
    - それにあてるように実装をする

# はじめに
型駆動開発とはどんなもので実践すると何が嬉しいのかを自分なりに理解するためにこの記事を書きます。
2021年7月時点では「型駆動開発」でググっても意図した内容がヒットせず、「Type-Driven Development」と検索して英語の記事が何件かヒットする程度です。
自分の「型駆動開発」に対しての理解・認識が世間一般のそれと相違がある場合もありますので、何か思うところがあればご指摘いただければ大変嬉しく思います。

また、この記事ではTypeScriptとVue.jsでフロントエンドのコードを書いていきます。TypeScriptは必須の前提ですがReactでも同じような考え方を活かせるのではないかなと思っています。

# 型駆動開発とは


# 型駆動開発を試してみる
実際に下記の様な絵文字をクリックすると絵文字の一覧モーダルが表示され、任意の絵文字に変更することができる機能を実装してみようと思います。

[gif]

## 01. TypeScriptでインターフェースを定義
機能を実装する為に、何が必要なのかを箇条書きにしてみるところから始めます。

まずザックリと脳内で必要な手順を
```ts
// - 絵文字がクリックされたらアイコン選択モーダルを表示する
// - 絵文字一覧の中の絵文字がクリックされたら既存の絵文字を変更する
// - 既存の絵文字が変更されたらアイコン選択モーダルを非表示にする
```

```ts
import { Ref } from 'vue'

/**
 * アイコン（絵文字）
 */
type Icon = Ref<string>

/**
 * アイコン選択画面が表示されている状態
 */
type IsShownIconSelectionModal = Ref<boolean>

/**
 * アイコン選択画面を表示/非表示する関数
 *
 * @param isShownIconSelectionModal アイコン選択画面が表示されている状態
 * @returns アイコン選択画面を表示/非表示するカリー化関数
 */
type ToggleDisplayOfIconSelectionModal = (
  isShownIconSelectionModal: IsShownIconSelectionModal
) => () => void

/**
 * アイコンを設定する関数
 *
 * @param icon アイコン
 * @returns アイコンを設定するカリー化関数
 */
type ChangeIcon = (icon: Icon) => (newIcon: string) => void

type UseIcon = (
  isShownIconSelectionModal: IsShownIconSelectionModal,
  icon: Icon
) => {
  toggleDisplayOfIconSelectionModal: ReturnType<ToggleDisplayOfIconSelectionModal>
  changeIcon: ReturnType<ChangeIcon>
}
```
