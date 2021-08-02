---
title: "TypeScriptで型駆動開発"
emoji: "🦅"
type: "tech"
topics: ["typescript", "vue"]
published: false
---

# はじめに
型駆動開発とはどんなもので実践すると何が嬉しいのかを自分なりに理解するためにこの記事を書きます。
2021年8月時点では「型駆動開発」でググっても意図した内容がヒットせず、「Type-Driven Development」と検索して英語の記事が何件かヒットする程度です。
自分の「型駆動開発」に対しての理解・認識が世間一般のそれと相違がある場合もありますので、何か思うところがあればご指摘いただければ大変嬉しく思います。

また、この記事ではTypeScriptとVue.jsでフロントエンドのコードを書いていきます。TypeScriptは必須の前提ですがReactでも同じような考え方を活かせるのではないかなと思っています。

# 型駆動開発とは
型駆動開発とはなにか。書籍「プログラミングTypeScript――スケールするJavaScriptアプリケーション開発」から引用させてもらいます。

> まず型シグネチャで概略を記述し、その後で値を埋め込むプログラミングのスタイル

https://www.oreilly.co.jp/books/9784873119045/

日本語で表現するならプログラミングの手法でしょうか。ではこれによって何が嬉しいのか？個人的に考えるのは以下の点です。

- 型を先行して記述することによって実装中にタイプチェッカーがエラーをキャッチしてくれる
- 型シグネチャがプログラムの仕様になるので、型を見るだけでその関数について理解できる
- 型を定義することによって脳内で実装を整理できるので脱printデバッグへの近道（大声）
- チームで開発する際には（状況にもよると思うが）型シグネチャすなわちインターフェースを共有することで同時作業がやりやすい
- テスト駆動開発とも親和性が高い

他にもたくさんあると思うので、「追加しろや」って意見があればコメントください。

# 型駆動開発を試してみる

こちらに完成例を置いておきます。commit履歴を追っていただければ見やすいかもしれません。

https://github.com/kazuhe/type-driven-development-with-typescript

下のgif画像の様な絵文字をクリックすると絵文字の一覧モーダルが表示され、任意の絵文字に変更することができるシンプルな`BaseIcon`コンポーネントを型駆動開発で実装してみようと思います。

![](https://storage.googleapis.com/zenn-user-upload/88c75befd6f737c05ab0f219.gif)

ディレクトリの構成は以下の様になっています。
```bash
.
└── BaseIcon
    ├── compositions
    │   ├── index.test.ts
    │   └── index.ts # インターフェースとそれに対応する実装
    ├── icons.ts # 絵文字一覧の変数を格納したファイル
    └── index.vue
```

## 1_ 機能を箇条書き
機能を実装する為に何が必要なのかを箇条書きするところから始めます。後ほど削除するのでこの段階では雑に以下の様なコメントを書いておけば良いかと思います。
```ts:BaseIcon/compositions/index.ts
// [必要な状態]
// - 現在選択されていてる絵文字
// - アイコン選択モーダルの表示状態

// [必要な関数]
// - 絵文字がクリックされたらアイコン選択モーダルを表示する関数
// - 既存の絵文字が変更されたらアイコン選択モーダルを非表示にする関数
// - アイコン選択モーダルの中の絵文字がクリックされたら既存の絵文字を変更する関数
// - Vue.jsから利用する為のuseXXX関数
```

アイコン選択モーダルの表示・非表示の切り替えは1つの関数でも良いかも。と思えば少し調整します。

```diff ts:BaseIcon/compositions/index.ts
- // - 絵文字がクリックされたらアイコン選択モーダルを表示する関数
- // - 既存の絵文字が変更されたらアイコン選択モーダルを非表示にする関数
+ // - アイコン選択モーダルが表示されていれば非表示に、非表示であれば表示する関数
```

これで一旦メモとしては完成です。メモに沿って5つの`type`を定義していきます。

```ts:BaseIcon/compositions/index.ts
// [必要な状態]
// - 現在選択されていてる絵文字
// - アイコン選択モーダルの表示状態

// [必要な関数]
// - アイコン選択モーダルが表示されていれば非表示に、非表示であれば表示する関数
// - アイコン選択モーダルの中の絵文字がクリックされたら既存の絵文字を変更する関数
// - Vue.jsから利用する為のuseXXX関数
```

## 2_ TypeScriptでインターフェースを定義
上記のコメントを元に必要なインターフェースを記述していきます。個人的にはこの工程が最も時間が掛かると思っています（コードの仕様書を作成している感覚）。
インターフェースさえ固まってしまえば残りはこれに沿ってテストを書いて関数を実装するだけなのでしっかりと考えます。

```ts: BaseIcon/compositions/index.ts
import { Ref } from 'vue'

// [必要な状態]
// - 現在選択されていてる絵文字
type Icon = Ref<string>

// - アイコン選択モーダルの表示状態
type IsShownIconSelectionModal = Ref<boolean>

// [必要な関数]
// - アイコン選択モーダルが表示されていれば非表示に、非表示であれば表示する関数
type ToggleDisplayOfIconSelectionModal = (
  isShownIconSelectionModal: IsShownIconSelectionModal
) => () => void

// - アイコン選択モーダルの中の絵文字がクリックされたら既存の絵文字を変更する関数
type ChangeIcon = (
  icon: Icon,
  toggleDisplayOfIconSelectionModal: ReturnType<ToggleDisplayOfIconSelectionModal>
) => (newIcon: string) => void

// - Vue.jsから利用する為のuseXXX関数
type UseIcon = (
  isShownIconSelectionModal: IsShownIconSelectionModal,
  icon: Icon
) => {
  toggleDisplayOfIconSelectionModal: ReturnType<ToggleDisplayOfIconSelectionModal>
  changeIcon: ReturnType<ChangeIcon>
}
```

テスト容易性を考慮して状態や依存関数はカリー化関数に注入するようなインターフェースにしています。例えば、`ChangeIcon`はリアクティブな値である`icon: Icon`と`toggleDisplayOfIconSelectionModal`関数に依存するので、実行前に引数を事前に注入するといった要領。

`UseIcon`はReactで言うところのHookだと思っていただければ良いかと思います。ここで依存性の注入をしてvueファイル側からは特に意識することなく関数を利用することができます。


## 3_ 空の関数を記述

テストをする為にインターフェースに沿って空の関数を記述します。

```diff ts:BaseIcon/compositions/index.ts
type ToggleDisplayOfIconSelectionModal = (
  isShownIconSelectionModal: IsShownIconSelectionModal
) => () => void

+ export const toggleDisplayOfIconSelectionModal: ToggleDisplayOfIconSelectionModal =
+   (isShownIconSelectionModal) => () => {}

type ChangeIcon = (
  icon: Icon,
  toggleDisplayOfIconSelectionModal: ReturnType<ToggleDisplayOfIconSelectionModal>
) => (newIcon: string) => void

+ export const changeIcon: ChangeIcon =
+   (icon, toggleDisplayOfIconSelectionModal) => (newIcon) => {}

type UseIcon = (
  isShownIconSelectionModal: IsShownIconSelectionModal,
  icon: Icon
) => {
  toggleDisplayOfIconSelectionModal: ReturnType<ToggleDisplayOfIconSelectionModal>
  changeIcon: ReturnType<ChangeIcon>
}

+ export const useIcon: UseIcon = (isShownIconSelectionModal, icon) => {}

```

## 4_ テストのtodoを記述

テストのtodoを記述します。ここでインターフェースからさらに詳細な仕様を作成していると思えば後の作業が楽になる気がします。
また、`useIcon`は依存を注入して公開しているだけなのでテストを実施しません。

```ts: BaseIcon/compositions/index.test.ts
import { toggleDisplayOfIconSelectionModal, changeIcon } from '.'

describe('toggleDisplayOfIconSelectionModal', () => {
  test.todo('アイコン選択画面が「表示されている」場合は関数の実行によって非表示になること')
  test.todo('アイコン選択画面が「表示されていない」場合は関数の実行によって表示されること')
})

describe('changeIcon', () => {
  test.todo('iconが既存のアイコンから新しいアイコンに変更されること')
  test.todo('toggleDisplayOfIconSelectionModalがコールされていること')
})
```

![](https://storage.googleapis.com/zenn-user-upload/a94f57b3fad2ccd932de9f5a.png)

## 5_ toggleDisplayOfIconSelectionModalのテストを記述

テストを記述します。

```diff ts: BaseIcon/compositions/index.test.ts
+ import { ref } from 'vue'
import { toggleDisplayOfIconSelectionModal, changeIcon } from '.'

describe('toggleDisplayOfIconSelectionModal', () => {
+  test('アイコン選択画面が「表示されている」場合は関数の実行によって非表示になること', () => {
+    const isShownIconSelectionModal = ref(true)
+    const testTarget = toggleDisplayOfIconSelectionModal(isShownIconSelectionModal)
+    testTarget()
+    expect(isShownIconSelectionModal.value).toBe(false)
+  })
+  test('アイコン選択画面が「表示されていない」場合は関数の実行によって表示されること', () => {
+    const isShownIconSelectionModal = ref(false)
+    const testTarget = toggleDisplayOfIconSelectionModal(isShownIconSelectionModal)
+    testTarget()
+    expect(isShownIconSelectionModal.value).toBe(true)
+  })
})
```

テストを実行して意図した通りに失敗していることを確認します。

![](https://storage.googleapis.com/zenn-user-upload/11f3f933d74d9b327491690a.png)

## 6_ toggleDisplayOfIconSelectionModalを実装

テストに合格する関数を記述します。

```diff ts:BaseIcon/compositions/index.ts
export const toggleDisplayOfIconSelectionModal: ToggleDisplayOfIconSelectionModal =
- (isShownIconSelectionModal) => () => {}
+ (isShownIconSelectionModal) => () =>
+   (isShownIconSelectionModal.value = !isShownIconSelectionModal.value)
```

テストに合格したので実装完了。

![](https://storage.googleapis.com/zenn-user-upload/f9c901500aa0be8f020ac3e0.png)

## 07. changeIconのテストを記述

```diff ts: BaseIcon/compositions/index.test.ts
describe('changeIcon', () => {
-  test.todo('iconが既存のアイコンから新しいアイコンに変更されること')
-  test.todo('toggleDisplayOfIconSelectionModalがコールされていること')
+  const icon = ref('🐶')
+  const newIcon = '🐱'
+  const toggleDisplayOfIconSelectionModalMock = jest.fn()
+  beforeEach(() => {
+    toggleDisplayOfIconSelectionModalMock.mockClear()
+    const testTarget = changeIcon(icon, toggleDisplayOfIconSelectionModalMock)
+    testTarget(newIcon)
+  })
+  test('iconが既存のアイコンから新しいアイコンに変更されること', () => {
+    expect(icon.value).toMatch(newIcon)
+  })
+  test('toggleDisplayOfIconSelectionModalがコールされていること', () => {
+    expect(toggleDisplayOfIconSelectionModalMock).toHaveBeenCalledTimes(1)
+  })
})
```

`toggleDisplayOfIconSelectionModal`と同じ様にテスト記述後に実行して意図した通りに失敗していることを確認します。

![](https://storage.googleapis.com/zenn-user-upload/78d77a9412f1d383ab8a5eab.png)

## 08. changeIconを実装

`toggleDisplayOfIconSelectionModal`と同じ様にテストに合格する関数を記述します。

```diff ts:BaseIcon/compositions/index.ts
- export const changeIcon: ChangeIcon = (icon) => (newIcon) => {}
+ export const changeIcon: ChangeIcon = (
+   icon,
+   toggleDisplayOfIconSelectionModal
+ ) => (newIcon) => {
+   icon.value = newIcon
+   toggleDisplayOfIconSelectionModal()
+ }
```

![](https://storage.googleapis.com/zenn-user-upload/7158ccbba9f96a017d7a1af6.png)

これでテストに合格する実装が完了しました。

## 09. useIconを実装

最後に願いを込めてuse関数を書きます。型エラーも出ていないし、テストも通っているので問題なく動くはず... !!

```diff ts:BaseIcon/compositions/index.ts
- export const useIcon: UseIcon = (isShownIconSelectionModal, icon) => {}
+ export const useIcon: UseIcon = (isShownIconSelectionModal, icon) => ({
+   toggleDisplayOfIconSelectionModal: toggleDisplayOfIconSelectionModal(
+     isShownIconSelectionModal
+   ),
+   changeIcon: changeIcon(
+     icon,
+     toggleDisplayOfIconSelectionModal(isShownIconSelectionModal)
+   ),
+ })
```

## 10. Vueファイルを記述

あとはチョチョイとマークアップして`useIcon`に依存を注入してやれば`toggleDisplayOfIconSelectionModal`と`changeIcon`が良い感じに使える様になります。

```vue: BaseIcon/index.vue
<template>
  <div class="base-icon">
    <div @click="toggleDisplayOfIconSelectionModal" class="icon current">
      {{ icon }}
    </div>
    <div v-if="isShownIconSelectionModal" class="icon-selection-modal">
      <ul class="list">
        <li
          v-for="(icon, index) in icons"
          :key="index"
          @click="changeIcon(icon)"
          class="icon item"
        >
          {{ icon }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
import { useIcon } from './compositions'
import { icons } from './icons'

export default defineComponent({
  name: 'BaseIcon',

  setup() {
    const icon = ref('🦅')
    const isShownIconSelectionModal = ref(false)
    return {
      ...useIcon(isShownIconSelectionModal, icon),
      icons,
      icon,
      isShownIconSelectionModal,
    }
  },
})
</script>
```

絵文字は何となく別ファイルにおきました。

```ts: BaseIcon/icons.ts
export const icons = [
  '🐶',
  '🐺',
  '🐱',
  '🐭',
  '🐹',
  '🐰',
  '🐸',
  // ~~~
]
```

# さいごに

