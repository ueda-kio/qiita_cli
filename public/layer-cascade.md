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

## この記事はなに？
CSSのカスケードレイヤーを活用して、詳細度が高いCSSファイルを気にせずに新しいスタイルを適用できる方法を提案します。

## 対象読者
この問題に直面している方々、つまり古くてデカくてムダに詳細度の高いcommon.cssに苦しんでいるウェブ開発者やデザイナーの皆さんを想定しています。特に、既存のスタイルに対して新たなスタイルを適用しようとしている方々には、非常に有用な情報を提供します。

- 歴史のあるウェブサイトを運用している開発者
- 古いCSSファイルの詳細度に苦しんでいる開発者

# 「古くてデカくて詳細度の高い common.css」とは？

古くて大きいCSSファイルとは、歴史のあるウェブサイトで使用されるスタイルシートのひとつで、リリース当初から多くのスタイルが追加されてきました。こういったファイルは「common.css」のようなファイル名がつけられていることが多く、一昔前まではCSS設計という考え方がなかったため、保守性が低いことが一般的です。その結果、詳細度が非常に高くなっていることが多いです。

## なにが困るの？
新たなUIを実装する際、これらの古いCSSが邪魔をしてくることがよくあります。たとえば common.css に以下のようなスタイルが記述されているとします。

```css:common.css
html body div#mainContents div.container p {
  font-size: 14px;
  line-height: 21px;
  color: black;
}
```

上記に該当する全ての `p` 要素にこのスタイルが当たることになります。新たなコンポーネントを実装する際にこれを上書きしようとすると、さらに詳細度の高いセレクターを作成しなければなりません。

具体的には、次のように書く必要が出てきます：

```css:modern.css
html body div#mainContents div.container p.custom-text {
  font-size: 16px;
  color: blue;
}
```

ここでは `p.custom-text` というクラスを追加することで、特定のスタイルを適用していますが、これによりCSSはどんどん複雑化し、可読性が低下します。さらに、他の開発者がこのスタイルを見たとき、どの部分がどのスタイルに影響しているのかを理解するのが難しくなります。

加えて、古いCSSファイルが多くの場所で使用されている場合、新たに追加したスタイルが他のスタイルと競合することが頻繁に発生します。これにより、意図しないスタイルが適用されたり、修正が必要になったりするため、保守性が著しく低下します。結果的に、開発者はCSSの詳細度を上げ続けることに疲れ果て、プロジェクト全体の管理が一層難しくなってしまいます。

# カスケードレイヤーで倒す

## カスケードレイヤーとは
カスケードレイヤーは2022年から Widely available となった CSS の機能で、スタイルシート内のルールの適用順序を管理するための仕組みです。「レイヤー」という格納器を作成し、各レイヤーの中で詳細度の勝ち負けを計算します。各レイヤーには優先順位を設定できるため、たとえ詳細度が高いセレクターであっても、優先順位の低いレイヤーに格納すれば、より優先度の高いレイヤーのスタイルが適用されます。

これまでは詳細度のみで勝ち負けを計算していました。

<画像>

レイヤーで区切ることで、最も優先度が高いレイヤーのセレクターが優先されます。

<画像>

詳細は以下をご確認ください。

https://developer.mozilla.org/ja/docs/Web/CSS/@layer

## どう common.css を倒すのか

common.css を優先度が低いレイヤーに格納し新たに作成する UI のスタイルをそれよりも優先度の高いレイヤーで実装することで、common.css の詳細度の高いスタイルに干渉されることなく実装を進めることができます。
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

これで common.css のムダに高い詳細度を意識することなく低い詳細度のまま新規スタイルを実装できます。

## common.css を編集することなくレイヤーに割り当てる
common.cssは往々にして修正が難しいことが多いです。特に歴史のあるウェブサイトのエンハンスにおいては、既に価値を生み出している既存のファイルを修正することは困難であり、ディレクターやプランナーなどの様々なステークホルダーにリファクタリングの意義と費用対効果を説明する必要があります。そのため、合意を得ることは難しい印象です。

### common.css を修正する方法
common.css を修正してよい場合は以下のようにします。

1. レイヤーの優先順を指定する
2. 読み込まれている全ての CSS ファイルをそれぞれレイヤーに割り当てる

#### 1. レイヤーの優先順を指定する

CSSファイルが読み込まれるより前段でレイヤーの優先順を指定する必要があります。これは layer.css のような CSS ファイルを用意して読み込ませるか、HTML の head 要素などに直接インラインスタイルで記述してもよいと思います。

```css:layer.css
@layer legacy, modern;
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

```html:インラインスタイルでレイヤーを指定する場合
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

common.css よりも優先度を高めたい CSS ファイルはすべて legacy レイヤーよりも優先度の高いレイヤーに割り当てたうえでスタイルを実装します。

```css:modern.css
@layer modern {
  .c-text {
    font-size: 16px;
  }
  ...
}
```

:::note info
どのレイヤーにも割り当てられていないレイヤー外のスタイルはすべてのレイヤーよりも優先度が高いため、単に common.css の優先度を下げたいだけならばその他の CSS ファイルはレイヤーに割り当てなくとも目的は達成できます（[参考 - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/@layer#:~:text=%E3%83%AC%E3%82%A4%E3%83%A4%E3%83%BC%E3%81%AE%E5%A4%96%E3%81%A7%E5%AE%A3%E8%A8%80%E3%81%95%E3%82%8C%E3%81%9F%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB%E3%81%AF%E3%80%81%E3%83%AC%E3%82%A4%E3%83%A4%E3%83%BC%E3%81%AE%E4%B8%AD%E3%81%A7%E5%AE%A3%E8%A8%80%E3%81%95%E3%82%8C%E3%81%9F%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB%E3%82%88%E3%82%8A%E3%82%82%E5%84%AA%E5%85%88%E3%81%95%E3%82%8C%E3%82%8B%E3%81%A8%E3%81%84%E3%81%86%E3%81%93%E3%81%A8%E3%81%A7%E3%81%99%E3%80%82)）。
:::

### common.css を修正しない方法

前述の通り大ベテランの common.css に修正を加えることがプロダクト的に難しいという場合は、その中身に修正を加えることなく `legacy` レイヤーに割り当てる方法を考えます。

スタイルをレイヤーに割り当てる方法はレイヤーブロック内にスタイルを記述する以外に、**[`@import`](https://developer.mozilla.org/ja/docs/Web/CSS/@import) でスタイルシートをインポートしつつその CSS ファイル全体を指定のレイヤーに割り当てる方法** があります。

https://developer.mozilla.org/ja/docs/Web/CSS/@import

`@import` はスタイルシートから他のスタイルシートを読み込むアットルールですが、インポートの際にファイル全体を特定のレイヤーに割り当てることができます。

```css:構文
@import "common.css" layer(legacy);
```

レイヤーの優先順位の定義はインポートするより前段で行う必要があります。また、HTML の `link` 要素で common.css を読み込んでいると、`legacy` レイヤーに割り当てられた common.css とそうでない common.css を重複して読み込むことになってしまうため、`@import` のみで読み込むようにします。

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

common.css 以外のすべての CSS ファイルも `@import` で読み込むようにすると管理が楽になりつつ、CSS ファイル側で逐一 `layer` を指定する必要がなく読み込む画面側にレイヤーを指定する責務を任せることができます。

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


このようにして、common.cssのスタイルをそのままにしておきながら、新たなスタイルを適用できます。これにより、保守性が向上し、開発者は新しいUIに集中しやすくなります。

:::note info
いわゆる reset.css も簡単に上書きできるように優先度を下げたい CSS であり、また全画面に共通するコンポーネントを components.css などのファイルで管理している場合これも画面固有のスタイルで簡単に上書きしたいため優先度を下げておきたいでしょう。
このように common.css ファイル以外にも優先度を管理したいというモチベーションは生まれやすいため、すべてのファイルをレイヤーに割り当てることを推奨します。

```css:e.g. layer.css
/* 優先順: 1.画面固有, 2.コンポーネント, 3.reset.css など, 4.common.css など */
@layer legacy, base, component, page;

@import "common1.css" layer(legacy);
@import "common2.css" layer(legacy);

@import "reset.css" layer(base);
@import "fonts.css" layer(base);

@import "components.css" layer(component);

@import "top_page.css" layer(page);
```
:::

# まとめ
古くてデカくて詳細度の高いcommon.cssは、ウェブ開発において避けがたい問題のひとつですが、カスケードレイヤーを利用することで効果的に管理できます。レイヤーを使うことで、古いスタイルに影響を受けることなく、自由に新しいスタイルを追加できるようになります。これにより、開発の効率が向上し、保守性も高まります。

古いプロジェクトを更新する際には、ぜひカスケードレイヤーを取り入れて、スムーズなCSS管理を実現してみてください。