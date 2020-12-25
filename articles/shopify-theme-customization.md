---
title: "Shopifyテーマカスタマイズを4ヶ月して感じた事と設計について"
emoji: "🛍"
type: "tech"
topics: ["shopify"]
published: true
---
# はじめに
約4ヶ月間、実際のプロジェクトでShopifyのテーマカスタマイズをしたので振り返ってみました。
設計についても記述していますが、探りながら自分なりに良いなと思った設計ですので1つの参考程度にしていただければ幸いです。
# 4ヶ月やって感じた事
### メリット
まず、Shopifyのコレクション（商品のグルーピング機能）が優秀でUIも良くてユーザーは使いやすいだろうなと思ったのが第一印象。1年前EC-CUBEを使った事がありますが圧倒的にShopifyの方が使いやすく感じました。
ドキュメントは基本英語ですが、日本語対応も目下進められている様です。サポートも手厚い印象があります。

### デメリット
しかし、開発者としての感想は別で何点か辛みは感じました。
* SCSSなどのCSSプリプロセッサが使いにくい
* 独自テンプレート言語のliquid学習コスト
* assets配下に独自のディレクトリを追加できず肥大化してしまう
  
などなど... まあ、どんな技術にもデメリットはあると思うので許容しています。
上記を解決する為に、ヘッドレスコマースとしてGatsbyJSなどのフレームワークと[Shopify Storefront API](https://shopify.dev/docs/storefront-api)を活用する手もあるみたいですが、それはそれで互換性があるアプリが少ないなどデメリットはあるみたいです。しかし、凝ったデザインや本格的な運用を見越すなら[テーマカスタマイズ](https://shopify.dev/docs/themes)では辛みが大きいので、個人的にはヘッドレスコマースを試す価値はあるかなと考えます。

また、
* お気に入り機能
* ギフト配送
* ポイント管理
* 予約販売

などの機能実装にはかなり手間がかかります。これらをShopifyの「アプリ（WordPressでいうところのプラグイン）」によって解決しようとすると想定以上に運用コストが掛かってしまいます。詳しくは調べていませんが多くはサブスクリプション方式の印象です。なので、導入したい機能をあらかじめ洗い出しアプリの選定から運用コストの算出は必須かと思います。

# 設計について
さて、所感はこれくらいにしておいてここからは設計について。
今回は工数削減の為に要件に最も近い有料テーマをベースにカスタマイズ。また、ベーステーマは影響範囲が大きいので原則として編集せず、今回の追加分とは分離させることを意識してコーディングしました。

## ディレクトリ
特に変わった点はないかと思いますが、後述するJavaScriptをコンパイルする為の`webpack.config.js`と`package.json`と`src`ディレクトリは独自に追加しています。
```bash
src
 ├─ assets # CSSやJavascriptや画像など
 ├─ config # テーマの設定
 ├─ layout # サイト共通レイアウトの大枠
 ├─ locales # 翻訳の定義と内容
 ├─ sections # テーマエディタから編集可能な共通パーツの定義
 ├─ snippets # ページに組み込むパーツの定義
 ├─ src # JavaScriptファイル群
 └─ templates # 各ページのレイアウト雛形
  - config.yml # Theme Kitの設定
  - package.json # JavaScript用
  - webpack.config.js # JavaScript用
```
各ディレクトリの詳しい説明は下記の記事がとても分かりやすいのでオススメです。
https://www.non-standardworld.co.jp/23013/


## 開発環境
Shopifyはテーマの変更を監視して自動アップロードする為のコマンドラインツール「Shopify Theme Kit」を提供しています。
https://shopify.github.io/themekit/
変更監視のWatch以外にもブラウザのカスタマイズ画面で変更した内容をテーマにダウンロードするコマンド等も用意されているので迷わず導入しておくべきです。この他にもコマンドラインツールが存在する様ですがTheme Kitで特に不便は感じませんでした。

:::message alert
Shopify Theme Kitを使う場合ローカルで`config/settings_data.json`を保存してしまうと、ブラウザ側のカスタマイズ画面で設定した保存内容が上書きされてしまいます。
上書き対策で`config.yml`に`ignore_files:- config/settings_data.json`の記述を追加して監視対象から外す事をオススメします。
:::

## コンポーネント
コンポーネントの概念があるか不明ですが、なるべく各機能やパーツを分離して記述したいのでVue.jsでいうSFCっぽいファイル構成にしています。簡単な例を紹介します。
各ページで使用するメインタイトルをコンポーネント化したファイルが下記になります。
```liquid:snippets/page-header.liquid
<div class="c-page_header">
  <h2>{{ title }}</h2>
</div>

{% style %}
.c-page_header {
  background: #333;
  /* ~~~ */
}
{% endstyle %}
```
`{{ title }}`は下記の呼び出し元のファイルから渡されます。
```liquid:呼び出し元のファイル
{% render 'page-header', title: section.settings.title %}

{% schema %}
  {
    "name": "Blog pages",
    "settings": [
      {
        "type": "text",
        "id": "title",
        "label": "ページタイトル"
      }
    ]
  }
{% endschema %}
```
上記ファイルによって構成されているページのカスタマイズ画面で設定されたページタイトルを`snippets/page-header.liquid`に渡しています。
注意点としては、このSFCっぽい方法だと**SCSSなどは使うことができない**のでコードの分割とCSSの書きやすさはトレードオフかなと思います。

## CSSの取り扱いについて
Shopifyは公式でSassの使用を非推奨としています。
https://www.shopify.jp/blog/partner-theme-sass-depricated
といっても、Gulpなどを用いてローカルでコンパイルするのは問題ないので、SCSSなど使いたい場合は各々用意すればOKです。
今回は前述した通りコンポーネントとして扱う1つのliquidファイルにCSSを記述しているのでSCSSは使っていません。辛かったです。

ファイルを越えて使用する共通のCSSは、ベーステーマとは別に`assets/style.css.liquid`を作成して`layout/theme.liquid`に純粋なCSSを読み込んでいます。（記述量が少なくSCSSコンパイル環境を用意するのが手間だったので）
```liquid:layout/theme.liquid
{{ 'style.css' | asset_url | stylesheet_tag }}
```

## JavaScriptの取り扱いについて
各ページ共通で使うスクリプトはテーマのルートに作成した`src`ディレクトリの配下のJavaScriptファイル群にそれぞれ記述。それらをWebpackで`assets`配下に`script.js`としてコンパイルさせるように設定していました。
なので、JavaScriptを書く際にはWebpackのWatchとShopifyのテーマWatchを同時に行って開発していました。

# さいごに
ブラウザで入力・選択された値によって表示を変更する必要がある場面が多く、肥大していくコードを眺めモヤモヤしながらなんとか4ヶ月間を耐えました。とはいえ、既存コードを崩さず開発を進める経験ができたり勉強になったことは多かったなと思います。
次にShopify案件をやるなら、何らかのフレームワークとストアフロントAPIを使ってヘッドレスコマースを実装してみたいです。