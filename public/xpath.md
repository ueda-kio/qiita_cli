---
title: XPath を使って `querySelector()` より強力に html 要素を取得しよう！
tags:
  - JavaScript
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

# 結論

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

# XPath とは


# 実行時間
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