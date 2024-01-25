---
title: XPath を使って `querySelector()` よりもパワフルに html 要素を取得しよう！
tags:
  - JavaScript
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

## 結論

XPath で DOM から要素を取得する方法。配列に格納して返却します。

```js
const getElementsByXPath = (xpath) => {
  const evaluator = new XPathEvaluator();
  const expression = evaluator.createExpression(xpath);
  const result = expression.evaluate(
    document,
    XPathResult.ORDERED_NODE_SNAPSHOT_TYPE,
  );
  return [...Array(result.snapshotLength)].map((_, i) => result.snapshotItem(i));
};

const elms = getElementsByXPath('//div');
elms; // [div, div, div, ...]
```

## XPath とは

### 概要

XPath とは XML 文書（html 含む）のさまざまなノードを指し示すことができるパス文字列のことです。

https://developer.mozilla.org/ja/docs/Web/XPath

> XPath は XML Path Language の略称です。非 XML 構文を使って、柔軟な方法で XML 文書の様々な部分をアドレッシングする（指し示す）ことができます。
> （中略）
> `Document.getElementById()` や `Document.querySelectorAll()` メソッドや `Node.childNodes` プロパティ、その他の DOM コア機能に依存することなく、はるかにパワフルな方法でたどることができます。

### 構文

html 内のノードを URL のパスのような表記で指し示します。これをロケーションパスと呼びます。
以下は XPath の一例です。
```xpath
/html/body/main/div[1]/article/header/h1
```

これは「html の中の body ノードの中の main ノードの中の １つ目の div ノードの中の article ノードの中の header ノードの中の h1 ノード」を指し示します。一致するノードが複数存在する場合、すべてのノードが対象となります。
以下の CSS セレクタと同義です[^1]。
```js
html > body > main > div:first-child > article > header > h1
```

スラッシュ（`/`）が1つの場合は子孫、つまりコンテキストノードの直下にあるノードのみを指し示しますが、2つ重ねることで子孫のすべてのノードを指し示すことができます。以下はその一例です。

```xpath
/html/body/main//h1
```

これは「html の中の body ノードの中の main ノードの中のすべての h1 ノード」を指し示します。
以下の CSS セレクタと同義です。
```js
html > body > main h1
```

スラッシュ2つで始めることにより、ルートノードの子孫すべてを指し示します。特別な指定をしない限り html におけるルートノードは `html` ノードになります。

```xpath
//h1
```

これは「html の中のすべての h1 ノード」を指し示します。

## XPath は CSS セレクタよりもはるかにパワフル

ここまで CSS セレクタと比較しながら XPath をみてきましたが、XPath は CSS よりもはるかにパワフルにノードを指し示すことができます。CSS セレクタの完全上位互換と言ってもいいと思います[^2]。
どのようなパワフルな機能があるのか、いくつか例をあげていきます。

### 特定の子ノードの持つ親ノードの指定

特定のノードを XPath で指定し、その親ノードを指定することができます。
以下は「`"dummy-id"` という id 属性を持つ li ノードを子要素にもつ ul ノード」を指し示す XPath です。

```xpath
//ul[li[@id="dummy-id"]]
```

これは li ノードを自身の直下にもつ ul ノードのみ対象となります。ルートノードまですべてのノードを遡るには [`ancestor`]([text](https://developer.mozilla.org/ja/docs/Web/XPath/Axes#ancestor)) という軸（Axis）を利用します。

以下は「`"dummy-id"` という id 属性を持つ liノードの祖先のすべての ul ノード」を指し示す XPath です。

```xpath
//li[@id="dummy-id"]/ancestor::ul
```

### 特定のノードのひとつ前に隣り合うノードを指定

「ひとつうしろに隣り合う要素」は CSS セレクタでは隣接セレクタ `+` で指定可能ですが、「ひとつ前に隣り合う要素」は CSS セレクタでは指定できません。
XPath では [`preceding-sibling`](https://developer.mozilla.org/ja/docs/Web/XPath/Axes#preceding-sibling) 軸を利用することで指定可能です。

以下は「`"dummy-id"` という id 属性を持つ div ノードのひとつ前に隣り合う p ノード」を指し示す XPath です。

```xpath
//div[@id="dummy-id"]/preceding-sibling::p
```

<!-- ### 特定のノードを支点に相対パスで指定

まさに url のようですね。 -->

### 関数
XPath 関数と呼ばれる様々な関数が用意されており、より高度にノードを探索することができます。一覧は以下です。

https://developer.mozilla.org/ja/docs/Web/XPath/Functions

#### `text()`

`text()` 関数は、コンテキストノードのテキストノードを指定します。

```html
<p id="para">詳細は<a href="/">こちら</a>です。</p>
```

このような html があったとき `text()` は `詳細は` と `です。` を指し示します。

```xpath
//p[@id="para"]/text()
```

https://www.educba.com/xpath-text/

（[MDN の「XPath > 関数」ページ](https://developer.mozilla.org/ja/docs/Web/XPath/Functions) に `text()` が載っていませんでした。なぜでしょう...？）


#### `contains()`

`contains()` 関数は、第1引数の文字列に第2引数の文字列が含まれているかどうかを判定し、論理値 `true` または `false` を返します。

以下は class 名に `mt-` という文字列を含んだすべての p ノードを指し示します。

```xpath
//p[contains(@class, "mt-")]
```

`text()` 関数と組み合わせることで、「特定の文字列を text ノードに持つノード」を指し示すことができます。

```html
<p id="para">詳細は<a href="/">こちら</a>です。</p>
<p id="para">詳細は<span>お問い合わせください。</span></p>
```

```xpath
$x('//p[contains(text(), "詳細は")]')
```

<details>
<summary></summary>
<div>

</div>
</details>

$x('//p[text()[contains(., "である要素の内、最初の要素だけが格納されます。")]]')


MDN はこちら。

https://developer.mozilla.org/ja/docs/Web/XPath/Axes

チートシートもあります。

https://devhints.io/xpath

## 実行時間
非常に雑な比較ですが、配列操作をしているため3倍くらい時間がかかります。
なお人間には知覚できないレベルであることは間違いないですが。
```js
const startTime = performance.now(); // 開始時間
const elms = getElmsFromXPath('//div');
const endTime = performance.now(); // 終了時間

console.log(endTime - startTime);
// 0.3000001907348633
```

```js
const startTime = performance.now(); // 開始時間
document.querySelectorAll('div');
const endTime = performance.now(); // 終了時間

console.log(endTime - startTime);
// 0.09999990463256836
```

[^1]: 「厳密には異なる」という場合はご指摘ください。
[^2]: 「CSS セレクタでできることは XPath でもでき、XPath でできて CSS セレクタにはできないことがある」という意味です。