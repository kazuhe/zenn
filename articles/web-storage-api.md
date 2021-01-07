---
title: "Web Storageの基本とclass構文を用いた汎用的な例"
emoji: "💾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript", "webstorage"]
published: true
---
# はじめに
仕事でWeb Storageを扱う必要があったので少し調べてまとめてみました。間違っていたりより良い方法があればコメントいただけると幸いです。

# Web Storageとは
> Web Storage API は、クッキーを使用するよりも直感的な方法で、ブラウザーがキーと値のペアを保存できる仕組みを提供します。
> *参照元：https://developer.mozilla.org/ja/docs/Web/API/Web_Storage_API*

とMDNに記載されています。クッキーとの大きな違いは以下の点です。
```bash
# データサイズの上限
Web Storage > クッキー

# データの有効期限
Web Storage: なし
クッキー: あり

# データ通信
Web Storage: 発生しない
クッキー: リクエストの都度でサーバーにも送信
```
また、Web StorageにはsessionStorageとlocalStorageの2種類が存在しており、

**sessionStorage**はページのセッション中 (ブラウザーを開いている間) のデータのみ保存され、ブラウザ又はタブが閉じられたタイミングでデータは破棄される

**localStorage**も同じくセッション中のデータを保存しており、ブラウザーを閉じたり再び開いてもデータは持続される

といった特徴を持っています。ちなみに、どちらもページの再読み込みや復元をしてもデータは持続されています。
:::message
sessionStorageで事足りるならsessionStorageを優先して使用するべきですが、今回の例ではlocalStorageを扱う前提で記載しています。また、両者はアクセスのプロパティが異なるだけで操作方法は共通です。
:::

# 基本的な使い方
`Application`>`Local Storage`から保存されているデータを確認できます。
![](https://storage.googleapis.com/zenn-user-upload/j63nth3669xg5yfz3ki28tmy41tk)
*Chrome 開発者ツール*

ブラケット構文（`storage['key']`）等も用意されていますが、メソッド構文の方が明示的で好みなので以降メソッド構文以外のコードは記載しません。

## Storageオブジェクトを代入
```ts
const storage = localStorage
```

## データの保存
保存できる値は基本stringが前提です。
```ts
storage.setItem('key', 'value')
```

オブジェクトとして保存したい場面が多いかと思いますが、そのまま保存しようとすると内部的に`toString`で文字列化されて読み取れなくなってしまいます。その為、JSONに変換した後に保存する必要があります。
```ts
const obj = {
  key1: 'value1',
  key2: 'value2',
  key3: 'value3',
}
storage.setItem('obj', JSON.stringify(obj))
```

## データの取得
文字列を取得する場合
```ts
storage.getItem('key')
```

オブジェクトを取得する場合はJSONから復元して取得
```ts
JSON.parse(storage.getItem('obj'))
```

## データの削除
keyを指定して特定の値を削除する場合
```ts
storage.removeItem('key')
```

指定されたStorageオブジェクトのすべてのデータを削除する場合
```ts
storage.clear()
```

## その他
その他のプロパティとメソッド。
```ts
// データ数を整数で返す
storage.length

// パラメータに渡したn番目のキーを返す
storage.key(n)
```

# class構文で汎用的にWeb Storageを扱う
各所で雑多にWeb Storageとデータをやり取りするとスマートではないのでclassを定義してみます。ちなみに下記の例はTypeScriptで記述しています。
```ts:helper.ts
export class MyStorage {
  private _storage = localStorage
  // ストレージが存在しない場合は空のオブジェクトを生成
  private _data = JSON.parse(this._storage.getItem(this._name) || '{}')
  constructor(private _name: string) {}

  getItem(key: string): string {
    return this._data[key]
  }

  setItem(key: string, value: string): void {
    this._data[key] = value
  }

  // saveメソッドが実行されたタイミングでlocalStorageに保存される
  save(): void {
    this._storage.setItem(this._name, JSON.stringify(this._data))
  }
}
```
今回は例として`storageA`インスタンスと`storageB`インスタンスを作成してそれぞれにデータを追加しています。
```ts:index.ts
import { MyStorage } from './helper'

// storageAインスタンス
const storageA = new MyStorage('MyStorageA')
storageA.setItem('keyA', 'valueA')
storageA.setItem('keyA2', 'valueA2')
console.log('storageA: ' + storageA.getItem('keyA')) // storageA: valueA
storageA.save()

// storageBインスタンス
const storageB = new MyStorage('MyStorageB')
storageB.setItem('keyB', 'valueB')
console.log('storageB: ' + storageB.getItem('keyB')) // storageB: valueB
storageB.save()
```
Chromeの開発者ツールで確認すると、意図した様にそれぞれのオブジェクトが保存されています。
![](https://storage.googleapis.com/zenn-user-upload/0ki2hduenvd3wggn3nl3geq8dqa1)
*Chrome 開発者ツール*

必要であれば削除のメソッドや特定の値を持ったデータがオブジェクトの中に存在するか確認できるメソッドもあれば便利ですね。

# さいごに
別のタブやウィンドウで発生したWeb Storageの変更を監視することもできる様なので、試した後に内容を追加できたらと思っています。
また、Web Storageは単純で扱いやすかったのですが、データが永続的に残るのでゴミが溜まりやすい印象も受けました。どこかのタイミングで意図的に削除することも検討した方が良いのか。あまり気にする必要がないのか、この辺りも調べていけらと思います。
