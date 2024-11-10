---
title: 古くてデカくて詳細度の高い common.css を倒す
tags:
  - 'HTML'
  - 'CSS'
  - 'Web'
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---
# はじめに

[カスケードレイヤー](https://developer.mozilla.org/ja/docs/Web/CSS/@layer) を活用して [詳細度](https://developer.mozilla.org/ja/docs/Web/CSS/Specificity) が高い CSS ファイルを気にせずに新しいスタイルを適用できる方法を提案します。

## 対象読者

- 歴史のあるウェブサイトを運用している開発者
- 古い CSS ファイルの高い詳細度に苦しんでいる開発者

# 「古くてデカくて詳細度の高い common.css」とは？
歴史のあるウェブサイトを長年支えている古い CSS ファイルのことです。[CSS 設計](https://developer.mozilla.org/ja/docs/Learn/CSS/Building_blocks/Organizing) という考え方が普及する前の時代に作成された CSS ファイルは、往々にして詳細度が高いセレクターが見受けられ保守性が低くなっており、記述されている量も多くファイルサイズも大きくなりがちです。また、このようなファイルは common.css という命名がなされていることが多いため（筆者の主観です）、この記事ではこのような CSS ファイルを「古くてデカくて詳細度の高い common.css」と呼称します。

:::note alert
このようなファイルのことを卑下するつもりはありません。長年の運用によりこれらの CSS ファイルは多くのユーザーに価値を提供してきましたが、時代の変化に伴い保守性や拡張性の観点から見直しが必要になることがあるのも事実です。
:::

## なにが困るの？
common.css に以下のような詳細度の高いスタイルが記述されているとします。

```css:common.css
html body div#mainContents div.container p {
  font-size: 14px;
  line-height: 21px;
  color: black;
}
/* 詳細度: 1-1-5 */
```

上記に該当する全ての `p` 要素にこのスタイルが当たることになります。新たなコンポーネントを実装する際にこれを上書きしようとすると、さらに詳細度の高いセレクターを作成しなければなりません。

```css:modern.css
html body div#mainContents div.container p.c-text {
  font-size: 16px;
  color: blue;
}
/* 詳細度: 1-2-5 */
```

本来は `.c-text` という詳細度 `0-1-0` のセレクターで定義したいスタイルも、common.css を上書きするために高い詳細度でスタイルを記述する必要があります。これにより CSS はどんどん複雑化し可読性が低下します。加えて、古い CSS ファイルが多くの場所で使用されている場合、新たに追加したスタイルが他のスタイルと競合することが頻繁に発生します。これにより意図しないスタイルが適用されたり修正が必要になったりするため、保守性が著しく低下します。

# カスケードレイヤーで倒す

上記の問題を解消する方法を検討します。

## カスケードレイヤーとは
[カスケードレイヤー](https://developer.mozilla.org/ja/docs/Web/CSS/@layer) は2022年から Widely available[^1] となった CSS の機能で、スタイルシート内のルールの適用順序を管理するための仕組みです。**レイヤー** という格納器を作成し、各レイヤーの中で詳細度の勝ち負けを計算します。各レイヤーには優先順位を設定できるため、たとえ詳細度が高いセレクターであっても、優先順位の低いレイヤーに格納すれば、より優先度の高いレイヤーのスタイルが適用されます。

詳細については以下をご確認ください。

https://developer.mozilla.org/ja/docs/Web/CSS/@layer

### カスケードレイヤーのイメージ
これまでは詳細度のみで適用されるスタイルの勝ち負けを計算していました。

![詳細度計算の例](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/4a580a8d-43f0-bb72-9c80-b57a6bdf1a8a.png)

レイヤーで区切ることで、最も優先度が高いレイヤーのセレクターが優先されます。

![カスケードレイヤーのスタイル適用の例](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/b63d88c2-739d-c3b6-262f-eb10a29b3073.png)

## どう common.css を倒すのか

common.css を優先度が低いレイヤーに格納し、新たに作成する UI のスタイルはそれよりも優先度の高いレイヤーに格納することで、common.css の詳細度の高いスタイルに干渉されることなく実装を進めることができます。

common.css を含む古いスタイルをすべて `legacy` レイヤーに格納し、新規実装のスタイルをすべて `modern` レイヤーに格納するとします。

```css:例
@layer legacy, modern;

/* ↓ common.css ↓ */
@layer legacy {
  html body div#mainContents div.container p {
    font-size: 14px; /* 詳細度では勝っているがレイヤーの優先度で負け */
  }
  ...
}

/* ↓ modern.css ↓ */
@layer modern {
  .c-text {
    font-size: 16px; /* 勝ち */
  }
  ...
}
```

新たに実装するすべての UI のスタイルを `modern` レイヤーに記述することで、common.css を意識することなく低い詳細度のまま新規スタイルを実装できます。

## common.css を編集することなくレイヤーに割り当てたい
common.css のような歴史のあるファイルは往々にして修正を加えることが難しいことが多いです。特に歴史のあるウェブサイトのエンハンスにおいて既に価値を生み出している既存のファイルを修正するためには、ディレクターやプランナーをはじめ様々なステークホルダーにリファクタリングの意義と費用対効果を説明する必要があり、その合意を得ることは難しい印象です。

そのため『common.css を修正する方法』と『common.css を修正しない方法』の2点を検討します。

### 1. common.css を修正する方法
common.css を修正してよい場合は以下のようにします。

1. レイヤーの優先順を指定する
2. 読み込まれている全ての CSS ファイルをそれぞれレイヤーに割り当てる

#### 1. レイヤーの優先順を指定する

CSS ファイルが読み込まれるより前段でレイヤーの優先順を指定する必要があります。これは layer.css のような CSS ファイルを用意して読み込むか、HTML の `head` 要素などの内部スタイルシートに記述することも可能です。

```css:layer.css
@layer legacy, modern; /* 優先順位: 1.modern, 2.legacy */
```

```html:layer.cssを読み込む場合
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <link rel="stylesheet" href="layer.css"> <!-- 他のCSSファイルよりも前段で読み込む -->
  <link rel="stylesheet" href="common.css">
  <link rel="stylesheet" href="modern.css">
  <!-- ... -->
</head>
```

```html:内部スタイルシートでレイヤーを指定する場合
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    /* 他のCSSファイルよりも前段で記述する */
    @layer legacy, modern;
  </style>
  <link rel="stylesheet" href="common.css">
  <link rel="stylesheet" href="modern.css">
  <!-- ... -->
</head>
```

#### 2. 読み込まれている全ての CSS ファイルをそれぞれレイヤーに割り当てる

common.css を以下のように修正します。

```css:common.css - before
html body div#mainContents div.container p {
  font-size: 14px;
}
...
```

```css:common.css - after
@layer legacy {
  html body div#mainContents div.container p {
    font-size: 14px;
  }
  ...
}
```

common.css よりも優先度を高めたい CSS ファイルはすべて `legacy` レイヤーよりも優先度の高いレイヤーに割り当てたうえでスタイルを実装します。

```css:modern.css
@layer modern {
  .c-text {
    font-size: 16px;
  }
  ...
}
```

:::note info
どのレイヤーにも割り当てられていないレイヤー外のスタイルはすべてのレイヤーよりも優先度が高いため、単に common.css の優先度を下げたいだけであればその他の CSS ファイルはレイヤーに割り当てなくとも目的は達成できます（[参考 - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/@layer#:~:text=%E3%83%AC%E3%82%A4%E3%83%A4%E3%83%BC%E3%81%AE%E5%A4%96%E3%81%A7%E5%AE%A3%E8%A8%80%E3%81%95%E3%82%8C%E3%81%9F%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB%E3%81%AF%E3%80%81%E3%83%AC%E3%82%A4%E3%83%A4%E3%83%BC%E3%81%AE%E4%B8%AD%E3%81%A7%E5%AE%A3%E8%A8%80%E3%81%95%E3%82%8C%E3%81%9F%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB%E3%82%88%E3%82%8A%E3%82%82%E5%84%AA%E5%85%88%E3%81%95%E3%82%8C%E3%82%8B%E3%81%A8%E3%81%84%E3%81%86%E3%81%93%E3%81%A8%E3%81%A7%E3%81%99%E3%80%82)）。
ですが、[後述する通り](#すべてのスタイルシートをレイヤーに割り当てることを推奨します。) 保守性の観点からすべてのスタイルシートをレイヤーに割り当てることを推奨します。
:::

### 2. common.css を修正しない方法

前述の通り大ベテランの common.css に修正を加えることがプロダクト的に難しいという場合は、その中身に修正を加えることなく `legacy` レイヤーに割り当てる方法を考えます。
手順は以下の通りです。

1. common.css を `@import` でインポートしつつ `legacy` レイヤーに割り当てる
2. `link` 要素で common.css を読み込まないようにする

スタイルをレイヤーに割り当てる方法はレイヤーブロック内にスタイルを記述する以外に、**[`@import`](https://developer.mozilla.org/ja/docs/Web/CSS/@import) でスタイルシートをインポートしつつその CSS ファイル全体を指定のレイヤーに割り当てる方法** があります。

https://developer.mozilla.org/ja/docs/Web/CSS/@import

`@import` はスタイルシートから他のスタイルシートを読み込むアットルールですが、インポートの際にファイル全体を特定のレイヤーに割り当てることができます。

```css:構文
@import "common.css" layer(legacy);
```

レイヤーの優先順位の定義はインポートするより前段で行う必要があります。また、HTML の `link` 要素で common.css を読み込んでいると、`legacy` レイヤーに割り当てられた common.css とそうでない common.css を重複して読み込むことになってしまうため、`link` 要素を削除し `@import` のみで読み込むようにします。

```html:common.css は @import のみで読み込む
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    @layer legacy, modern;
    @import "common.css" layer(legacy);
  </style>
  <!-- <link rel="stylesheet" href="common.css"> --> <!-- link 要素での読み込みは削除する -->
  <link rel="stylesheet" href="modern.css">
  <!-- ... -->
</head>
```

common.css 以外のすべての CSS ファイルも `@import` で読み込むようにすると管理が楽になりつつ、CSS ファイル側で逐一 `@layer` を指定する必要がなく読み込む画面側にレイヤーを指定する責務を任せることができます。

```css:layer.css
@layer legacy, modern;

@import "common.css" layer(legacy);
@import "modern.css" layer(modern);
```

```html:link 要素で読み込むファイルは layer.css のみ
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <link rel="stylesheet" href="layer.css">
  <!-- ... -->
</head>
```


このようにして、common.cssのスタイルをそのままにしておきつつ、新たなスタイルを適用できます。

:::note info
#### すべてのスタイルシートをレイヤーに割り当てることを推奨します。

いわゆる reset.css も簡単に上書きできるように優先度を下げたい CSS であり、また全画面に共通するコンポーネントを components.css などのファイルで管理している場合これも画面固有のスタイルで簡単に上書きしたいため、優先度を下げておくことが望ましいでしょう。
このように common.css ファイル以外にも優先度を管理したいというモチベーションは生まれやすいため、すべてのファイルをレイヤーに割り当てることを推奨します。

```css:例: layer.css
/* 優先順位: 1.画面固有, 2.コンポーネント, 3.reset.css など, 4.common.css など */
@layer legacy, base, component, page;

@import "common1.css" layer(legacy);
@import "common2.css" layer(legacy);

@import "reset.css" layer(base);
@import "fonts.css" layer(base);

@import "components.css" layer(component);

@import "topPage.css" layer(page);
```
:::

# まとめ
詳細度とは別の概念でスタイルの優先度を決定するカスケードレイヤーという機能の登場により新たな CSS 設計が生まれるだけでなく、特に古くからある保守性の低い CSS を管理しやすくなったのではないかと考えています。さらにこの記事では当該ファイルを直接修正することなくその恩恵を受ける方法を考えていきました。
もし common.css に日々悩まされている方がいらっしゃいましたら、こちらの方法をぜひご検討ください。

[^1]: [Baseline \(互換性\) - MDN Web Docs 用語集](https://developer.mozilla.org/ja/docs/Glossary/Baseline/Compatibility) 参照