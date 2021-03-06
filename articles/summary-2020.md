---
title: "自身の2020年を総括する"
emoji: "🔍"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["振り返り", "ポエム"]
published: true
---
# はじめに
自身の2020年の活動を振り返り、反省と翌年の目標を書き殴っているだけの記事です。
文字に起こして晒しておくことで動機付けと責任感の刺激になれば良いなと思っています。

# 振り返り
## @2020年以前
学生時代から特に目標もなく何となく某旅客鉄道会社へ就職。その後は電気工事や会計事務所など異業種を約1年半ペースで転々としてきた私だが、2017年後半に前職の地方Web制作会社に入社してからはWeb業界一筋で何とか頑張っている。

前職は中小企業をターゲットに
* コーポレートサイトやランディングページの制作
* チラシやパンフレット等の広告物の制作
* 既存顧客への営業
* ディレクション業務

などを浅く広く担当していてコーディングはWordPressを弄るだけ。コーディングよりもデザインや営業をしていた時間の方が多い気がする。デザインも営業もコーディングもどれも中途半端になっていて危機感をおぼえたので2020年に都内の受託制作会社へ転職。

-----

## @2020年
最も大きい変化は言わずとも転職なわけだが、2019年の10月に【将来プロフィールに書きたいスキル】としてtwitterに投稿していた項目が達成できているか振り返る。

![kazuhe](https://storage.googleapis.com/zenn-user-upload/38n4iv11s3ayfho3fyr4j4t2k2w3 =500x)

ちなみに、これらの項目は某企業のフロントエンドエンジニア募集要項に「求めるスキル」として記載されていた。

**結論を述べると、「Node.jsによるBFFの知見」以外は実際の業務で達成している。**

ユニットテストの経験については、12月中旬から始まったプロジェクトで実践しており滑り込みセーフな感じはあるが...少しだけ掘り下げて振り返る。

### ■ HTML5/CSS/JavaScript(ES6以降)を利用したWebサイト開発
前職ではJavaScriptをあまり書くことはなく前職退職後から現職の入社までに学習した。
改訂新版JavaScript本格入門がとても分かりやすくて今でも忘れた部分を読み返している。
https://gihyo.jp/book/2016/978-4-7741-8411-1


### ■ SPAの実装
SPAについては、業務で2案件関わった。どちらもVueでメイン担当ではないが先輩のコードを見れたので学びが大きかった。

### ■ Vue/Reactを使用した開発
業務では主にVueを使っているが、個人開発でReactも使って開発した。
個人的にはReactを使っていきたいので、12月中旬から始まった私がメイン担当のプロジェクトはNext.js(React)を使わせてもらっている。

### ■ TypeScriptでの開発
TypeScriptは業務でも個人でも書いた。
O'ReillyのTypeScript書籍を読んだ。基本部分はしっかりと読んで理解できたと思うが、非同期処理やクラス構文は読み飛ばしているので時間を作ってしっかりと学びたい。
https://www.oreilly.co.jp/books/9784873119045/

### ■ ユニットテストの経験
先にも書いたが、インテグレーション・E2Eテストも含め現プロジェクトで実践中。

### ■ Webpackを用いた開発環境の構築運用経験
ここ最近は自分でWebpackの環境を用意することはあまり無いが、何度かゼロベースで構築はした。基本は理解して分からない設定は公式ドキュメントで調べている。

### ■ Git/Githubフローでの開発
業務でGitを使って開発している。しかし、本格的なルールの元git運用できている訳ではないので、ある程度大きな規模で複数人開発するタイミングで導入したい。
https://guides.github.com/introduction/flow/

### ■ Node.jsによるBFFの知見
全く分からない。以下の記事は読ませてもらった。
https://zenn.dev/yuri/articles/f33216517c38a6d4306b

### 【その他】
#### golang
趣味でgolangを勉強し始めた。以下の書籍を読んだ。
https://www.shoeisha.co.jp/book/detail/9784798142418
自分でAPIサーバをたててマイクロサービスを構築するのが目標。それによってフロントエンドだけではなくサービス全体の仕組みをあやふやでも把握したい。

#### Write Code Every Day
以下の様なルールがある様だが、今の所は11月末から毎日何らかのコードを書くことだけを目標に実践した。もちろんこれからも続けたい。
https://johnresig.com/blog/write-code-every-day/

![kazuhe](https://storage.googleapis.com/zenn-user-upload/pkdiuvjtyveo5gb1tq4r700rgg2o)

# 2021年の目標
大きな目標は「**フレームワークや言語の具体的な使い方を学ぶことに加えて、その技術が生まれた経緯を知りソフトウェア開発の本質を垣間見る** 」こと。（理解したいとは言えない..）

細か目標は下記の様な項目
* パフォーマンスを意識してReactを扱う
* TypeScriptの深い理解
* フロントエンドでTDDを実践
* 様々なアーキテクチャに触れる
* golangでAPIサーバを立ててマイクロサービスを構築する
* OSS活動
* Write Code Every Day

抽象度・粒度がバラバラだが目標を達成することでフロントエンドエンジニアとして市場価値を高めたい。

# さいごに
全くまとまっていない文章ですが読んでくださってありがとうございます。
2021年はもっと自分を追いこんで、かっこ悪くても貪欲に過ごしていこうと思います。