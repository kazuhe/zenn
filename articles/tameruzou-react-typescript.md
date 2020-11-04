---
title: "React+TypeScriptで簡単なアプリを制作したので振り返ってみる"
emoji: "👻"
type: "tech"
topics: ["react", "redux", "typescript"]
published: true
---
# はじめに
最近Reactの勉強を始めたので、実際に手を動かすべく簡単なアプリを作りました。
まだまだ勉強中で至らない点もあるかと思いますが、個人的に意識した点を振り返ってみました。
改善点があればコメントいただけると大変嬉しいです。

# アプリ概要
ひとことで言うなら貯金シミュレーションアプリです。
- アプリ - https://tameruzou.netlify.app/
- GitHub - https://github.com/kazuhe/tameruzou

目標貯金額・貯金期間・ボーナス月（任意）・ボーナス月の貯金額（任意）をユーザに入力させて、目標貯金額を達成する為には毎月何円貯金する必要があるのかを計算するシンプルな作りです。

# 構成
State管理とViewの責務を分割したコンポーネント設計を意識して下図の様な構成にしました。
![tameruzou構成](https://storage.googleapis.com/zenn-user-upload/tr5rx2y769b6f73vah07da5r6lx5)
*tameruzou構成*

## Presentational Component
Presentational Component（以下、「Component」とする）はViewを担当するコンポーネントで、どの様に見えるかだけに関心を持ちます。今回のアプリでは`components/`以下に配置してPropsとして受け取ったデータを変更せずに表示させています。
```tsx:components/bonus-month.tsx
import React, { FC } from 'react'
import styles from '../styles/components/bonus-month.module.scss'

type Props = {
  months: { id: number; value: string }[]
  activeMonth: number[]
  toggleMonth: (month: number) => void
}

export const BonusMonthComponent: FC<Props> = (props: Props) => (
  <div className={styles.wrap}>
    <p>クリックしてアクティブにしてください（複数選択可）</p>
    <ul className={styles.list}>
      {props.months.map((month) => (
        <li
          key={month.id}
          onClick={() => props.toggleMonth(month.id)}
          className={props.activeMonth.some((n) => n === month.id) ? styles.active : ''}
        >
          {month.value}
        </li>
      ))}
    </ul>
  </div>
)
```
ほぼマークアップの記述のみしか書かれてないので、パッと見でも見た目を想像しやすいかなと思います。

## Container Component
Container Component（以下、「Container」とする）は状態管理を担当するコンポーネントで、どの様に機能するかに関心を持ちます。今回のアプリでは`containers/`以下に配置しており、ReduxのStoreとデータのやり取りをして必要であれば変更を加えてComponentに渡しています。
また、ルールとしてContainerはデータを渡すComponentと同ファイル名で作成する様にしています。
```tsx:containers/bonus-month.tsx
import React, { FC } from 'react'
import { BonusMonthComponent } from '../components/bonus-month'
import { useSelector } from 'react-redux'
import { RootState } from '../stores'
import { useDispatch } from 'react-redux'
import { toggleBonusMonth } from '../stores/simulation'

export const BonusMonthContainer: FC = () => {
  const months = useSelector((state: RootState) => state.simulation.bonus.months)
  const dispatch = useDispatch()

  const monthsCollection = [
    { id: 1, value: '1月' },
    { id: 2, value: '2月' },
    // ~~~~
  ]

  // activeな月だけの配列を作成
  const activeMonth: number[] = []
  Object.entries(months).forEach(([key, value]) => {
    if (value) activeMonth.push(Number(key))
  })

  // ボーナス月をアクティブ・非アクティブにする関数
  const toggleMonth = (month: number) => {
    dispatch(toggleBonusMonth(month))
  }

  return <BonusMonthComponent months={monthsCollection} activeMonth={activeMonth} toggleMonth={toggleMonth} />
}
```
上記の様に責務を分割することで状態管理がしやすく1つのコンポーネントのコード量も減るので非常に気持ちが良いです。（Vue.jsのSFCだといつも大量のコードになってしまうので..）

## Redux Toolkit
公式でRedux Toolkitを推奨しているので採用しました。今回のアプリでは`stores/`以下に関連ファイル配置しています。
Reduxに関しては正直なところ理解が及ばずですが、Redux Toolkitはボイラープレートコードも少なく公式がストアの分割方法をヘルパー関数として提供しているのでかなり取っ付きやすく感じました。

実際に使ってみた感想としてはContainerとComponentを意識していれば、小規模アプリでもReduxはデメリットなく状態管理の効率化を助けてくれる気がしました。

# さいごに
簡単なアプリですが作ってみて責務の分割であったりと学びが多かったです。同時に、出来なかった事・次にやりたい事も発見できました。
- パフォーマンス
- テスト
- スタイルガイド

ほんの一部ですがNEXT STEPとしては上記3点。今回のアプリではパフォーマンスは ~~全く~~ あまり意識できていないので、`React.memo`などを用いてパフォーマンス改善も行っていきたいです。また、今回の規模では必要ないかもしれませんがスタイルガイドも導入できたらなと思っています。