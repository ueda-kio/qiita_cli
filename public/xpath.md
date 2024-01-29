---
title: 【JavaScript】querySelector よりもパワフルに DOM からノードを取得しよう！【XPath】
tags:
  - HTML
  - JavaScript
  - xpath
  - Web
private: false
updated_at: '2024-01-27T15:27:38+09:00'
id: d8d6e72ec7d565500487
organization_url_name: null
slide: false
ignorePublish: false
---
## 1. はじめに

これは [XPath](https://developer.mozilla.org/ja/docs/Web/XPath) を利用して DOM からノードを取得する方法を紹介する記事です。

最近 XPath の存在を知り、「XPath を利用して `document.querySelector()` のように DOM からノード（要素）を取得したい！」と思い立ったため、その方法をまとめていきます。

:::note warn
最近 XPath の存在を知ったばかりの拙いエンジニアによる記事です。誤りや不適切な記述がある場合はご指摘ください。
:::

## 2. 結論

XPath を利用して DOM からノードを取得する関数です。取得したノードは配列に格納して返却します。

```js
const getNodesByXPath = (xpath) => {
  const result = document.evaluate(
    xpath,
    document,
    null,
    XPathResult.ORDERED_NODE_SNAPSHOT_TYPE,
    null
  );
  return [...Array(result.snapshotLength)].map((_, i) => result.snapshotItem(i));
};

const nodes = getNodesByXPath('//div');
nodes; // [div, div, div, ...]
```

## 3. XPath とは

### 3.1. 概要

XPath とは XML 文書（HTML 含む）のさまざまなノードを指し示すことができるパス文字列のことです。

https://developer.mozilla.org/ja/docs/Web/XPath

> XPath は XML Path Language の略称です。非 XML 構文を使って、柔軟な方法で XML 文書の様々な部分をアドレッシングする（指し示す）ことができます。
> （中略）
> `Document.getElementById()` や `Document.querySelectorAll()` メソッドや `Node.childNodes` プロパティ、その他の DOM コア機能に依存することなく、はるかにパワフルな方法でたどることができます。

### 3.2. 構文

HTML 内のノードを URL のパスのような表記で指し示します。これをロケーションパスと呼びます。
以下は XPath の一例です。
```xpath
/html/body/main/div[1]/article/header/h1
```

これは「html の中の body ノードの中の main ノードの中の １つ目の div ノードの中の article ノードの中の header ノードの中の h1 ノード」を指し示します。一致するノードが複数存在する場合、すべてのノードが対象となります。
以下の CSS セレクタと同義です[^1]。
```js
html > body > main > div:first-child > article > header > h1
```

スラッシュ（`/`）が1つの場合は子、つまりコンテキストノードの直下にあるノードのみを指し示しますが、2つ重ねることで子孫のすべてのノードを指し示すことができます。以下はその一例です。

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

### 3.3. XPath は CSS セレクタよりもはるかにパワフル

ここまで CSS セレクタと比較しながら XPath をみてきましたが、XPath は CSS よりもはるかにパワフルにノードを指し示すことができます。CSS セレクタの完全上位互換と言ってもいいと思います[^2]。

▼ チートシート

https://devhints.io/xpath

どのようなパワフルな機能があるのか、いくつか例をあげていきます。

#### 3.3.1. 特定の子ノードを持つ親ノードの指定

特定のノードを XPath で指定し、その親ノードを指定することができます。
以下は「`"dummy-id"` という id 属性を持つ li ノードを子要素にもつ ul ノード」を指し示す XPath です。

```xpath
//ul[li[@id="dummy-id"]]
```

これは li ノードを自身の直下に持つ ul ノードのみ対象となります。

ルートノードまですべてのノードを遡るには [`ancestor`]([text](https://developer.mozilla.org/ja/docs/Web/XPath/Axes#ancestor)) という軸（Axis）を利用します。
以下は「`"dummy-id"` という id 属性を持つ liノードの祖先のすべての ul ノード」を指し示す XPath です。

```xpath
//li[@id="dummy-id"]/ancestor::ul
```

#### 3.3.2. 特定のノードのひとつ前に隣り合うノードを指定

「ひとつ後ろに隣り合う要素」は CSS セレクタでは隣接セレクタ `+` で指定可能ですが、「ひとつ前に隣り合う要素」は CSS セレクタでは指定できません。
XPath では [`preceding-sibling`](https://developer.mozilla.org/ja/docs/Web/XPath/Axes#preceding-sibling) 軸を利用することで指定可能です。

以下は「`"dummy-id"` という id 属性を持つ div ノードのひとつ前に隣り合う p ノード」を指し示す XPath です。

```xpath
//div[@id="dummy-id"]/preceding-sibling::p
```

```html:e.g.
<p>...</p> <!-- ← これ -->
<div id="dummy-id">...</div>
```

#### 3.3.3. XPath 関数
[XPath](https://developer.mozilla.org/ja/docs/Web/XPath/Functions) 関数と呼ばれる関数が用意されており、より高度にノードを探索することができます。

https://developer.mozilla.org/ja/docs/Web/XPath/Functions

##### `text()`

`text()` 関数は、コンテキストノードのテキストノードを指定します。

以下のような HTML があるとします。

```html
<p id="para">詳細は<a href="/">こちら</a>です。</p>
```

このとき、`text()` は「詳細は」と「です。」を指し示します。

```xpath
//p[@id="para"]/text()
```

https://www.educba.com/xpath-text/


##### `contains()`

`contains()` 関数は、第1引数の文字列に第2引数の文字列が含まれているかどうかを判定し、論理値 `true` または `false` を返します。

以下は class 属性に `mt-` という文字列を含んだすべての p ノードを指し示します。

```xpath
//p[contains(@class, "mt-")]
```

`text()` 関数と組み合わせることで、「特定の文字列をテキストノードに持つノード」を指し示すことができます。

```html
<p>詳細は担当者にお問い合わせください。</p>
<p>詳細は<a href="/">こちら</a>です。</p>
```

以下の XPath により、上記2つの p ノードを指し示すことができます。

```xpath
//p[contains(text(), "詳細は")]
```

<details>
<summary>分割された複数のテキストノードを持つ場合</summary>
<div>

以下のようにテキストノード以外のノードを含んだ p ノードを考えます。

```html
<p>XPath を使用するための主となるインターフェイスは <a href="/document">document</a> オブジェクトの <code>evaluate</code> 関数です。</p>
```

> <p>XPath を使用するための主となるインターフェイスは <a href="#">document</a> オブジェクトの <code>evaluate</code> 関数です。</p>

このとき「オブジェクトの」という文字列を含んだ p ノードを `contains()` 関数と `text()` 関数を用いて以下の XPath で取得しようとしても、上記の p ノードは指し示すことができません。

```xpath
//p[contains(text(), "オブジェクトの")]
```

一見するとうまくいってそうですが、ダメでした。
`text()` はコンテキストノードが持つすべてのテキストノードを指し示すことができますが、`contains()` の第一引数には単一の文字列のみ渡るため **`text()` で得られた最初のテキストノードのみが引数として渡されます**。
つまりこの例では「XPath を使用するための主となるインターフェイスは」というテキストノードのみが `contains()` に渡されるため `false` 判定となり上記の p ノードは対象外となります。

これを回避してコンテキストノードが持つ複数あるすべてのテキストノードを `contains()` で判定するために、以下のような XPath を考えました。

```xpath
//p[text()[contains(., "オブジェクトの")]]
```

`//p[text()]` で p ノードのテキストノードすべてを対象とし、[self 軸（`.`）](https://developer.mozilla.org/ja/docs/Web/XPath/Axes#self) を利用してすべてのテキストノードに対して `contains()` で判定しています。

なおこの XPath は自作なので、よりよい方法がありましたらご指摘いただけるとうれしいです。

</div>
</details>

https://developer.mozilla.org/ja/docs/Web/XPath/Functions/contains

## 4. XPath で指し示したノードを JavaScript で取得する

本題です。XPath を利用して DOM からノードを取得します。

JavaScript でノード（要素）を取得するために `document.getElementById` や `document.querySelector` などのメソッドが用意されていますが、これらは XPath を指定できないため、別の方法を用いて取得します。いくつか方法はありますが、その中のひとつをご紹介します。

:::note warn
基本的に標準の DOM API で事足りるという前提のもと、XPath を利用したいというニッチな状況の想定です。
:::

以下の手順で取得します。
1. [`document.evaluate()`](https://developer.mozilla.org/ja/docs/Web/API/Document/evaluate) を実行して DOM から [`XPathResult`](https://developer.mozilla.org/ja/docs/Web/API/XPathResult) オブジェクトを取得
2. [`XPathResult.snapshotItem()`](https://developer.mozilla.org/ja/docs/Web/API/XPathResult/snapshotItem) メソッドからノードの集合を取得

#### 1. [`document.evaluate()`](https://developer.mozilla.org/ja/docs/Web/API/Document/evaluate) を実行して DOM から [`XPathResult`](https://developer.mozilla.org/ja/docs/Web/API/XPathResult) オブジェクトを取得

[`document.evaluate()`](https://developer.mozilla.org/ja/docs/Web/API/Document/evaluate) メソッドを実行して [`XPathResult`](https://developer.mozilla.org/ja/docs/Web/API/XPathResult) オブジェクトを取得します。
第1引数に XPath、第2引数にコンテキストノードとして `document` を指定します。特に、第4引数（`resultType`）に `ORDERED_NODE_SNAPSHOT_TYPE` または `UNORDERED_NODE_SNAPSHOT_TYPE` を指定することで、スナップショットを取得します。

```js
const result = document.evaluate(
  xpath,
  document,
  null,
  XPathResult.ORDERED_NODE_SNAPSHOT_TYPE, // or UNORDERED_NODE_SNAPSHOT_TYPE
  null
);
```

#### 2. [`XPathResult.snapshotItem()`](https://developer.mozilla.org/ja/docs/Web/API/XPathResult/snapshotItem) メソッドからノードの集合を取得

スナップショットを含んだ [`XPathResult`](https://developer.mozilla.org/ja/docs/Web/API/XPathResult) オブジェクトを取得することにより、[`XPathResult.snapshotItem()`](https://developer.mozilla.org/ja/docs/Web/API/XPathResult/snapshotItem) メソッドから取得したノードを取り出すことができます。インデックスを指定することで取り出せるため、すべてのノードを取得するために [`XPathResult.snapshotLength`](https://developer.mozilla.org/ja/docs/Web/API/XPathResult/snapshotLength) の数だけループを回し、ノードを配列に格納します。

```js
[...Array(result.snapshotLength)].map((_, i) => result.snapshotItem(i));
```

これにて XPath からノードの集合を取得することができました。
以下は汎用的にしたコードです。

```js
const getNodesByXPath = (xpath) => {
  const result = document.evaluate(
    xpath,
    document,
    null,
    XPathResult.ORDERED_NODE_SNAPSHOT_TYPE,
    null
  );
  return [...Array(result.snapshotLength)].map((_, i) => result.snapshotItem(i));
};

const nodes = getNodesByXPath('//div');
nodes; // [div, div, div, ...]
```

<details>
<summary>document.querySelectorAll との実行時間の比較</summary>
<div>

簡単な測定ではありますが、`document.querySelectorAll` と実行時間の比較をしてみました。
配列操作をしているためか XPath の方が3倍くらい時間がかかります。
```js:XPath での取得
const startTime = performance.now(); // 開始時間
const nodes = getNodesByXPath('//div');
const endTime = performance.now(); // 終了時間

console.log(endTime - startTime);
// 0.3000001907348633
```

```js: document.querySelectorAll での取得
const startTime = performance.now(); // 開始時間
document.querySelectorAll('div');
const endTime = performance.now(); // 終了時間

console.log(endTime - startTime);
// 0.09999990463256836
```

「特定のテキストノードをもつノードのみ取得する」「親ノードを取得する」など、より高度な条件で取得するために `filter()` や `closest()` などのメソッドを組み合わせた場合、もしかしたら XPath で取得した方が早くなる可能性もあります。

いずれにせよ、この時間差は人間には知覚できないレベルであることは間違いありません。

</div>
</details>

## 5. おわりに
XPath を利用して DOM からノードを取得する方法をまとめました。
`document.getElementById` や `document.querySelector` などの標準メソッドだけではどうしてもアドレッシングが難しい、という限定的な状況で輝いてくれる日が来るかもしれません。私はまだないですが。

[^1]: 「厳密には異なる」という場合はご指摘ください。
[^2]: 「CSS セレクタでできることは XPath でもでき、XPath でできて CSS セレクタにはできないことがある」という意味です。
