---
title: 【JavaScript】アロー関数式における`this`は「スコープを1つスキップする」でOK
tags:
  - 'JavaScript'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

JavaScriptのアロー関数における`this`の挙動について

（function宣言と比較してみていく）


## 対象読者

- JavaScript初学者の方
- function宣言とアロー関数との`this`の挙動の違いは理解しているが、いまいち自信がない/実装時に不安に感じる方

# 結論

アロー関数式における`this`は「スコープを1つスキップする」でOK。
例外としてclassのメソッドはfunction式で宣言しようとアロー関数式で宣言しようと、同じくインスタンスを参照する。

# 

本題が本記事の10割なので以降はその検証。

有名な記事『JavaScriptの「this」は「4種類」？？』の4つで見ていきます。

１. メソッド呼び出しパターン
２. 関数呼び出しパターン
３. コンストラクタ呼び出しパターン
４. apply,call呼び出しパターン

https://qiita.com/takeharu/items/9935ce476a17d6258e27

## １. メソッド呼び出しパターン

```js:メソッド呼び出しパターン
const myObject = {
  value: 10,
  show: function() {
    console.log(this.value);
  }
};
myObject.show(); // 10
```

グローバルオブジェクトを参照する。実行コンテキストにより変動しますが、ブラウザでのスクリプトなら`window`です。
```js:メソッド呼び出しパターン
window.value = 100;

const myObject = {
  value: 10,
  show: () => {
    console.log(this.value);
  }
};
myObject.show(); // 100

this.value; // ここで呼ぶ this と等しい
```

ネストしても同じです。
```js
const myObject = {
  value: 10,
  show: function() {
    const hoge = () => {
      console.log(this.value);
    }

    hoge(); // myObject.valueを参照する
    this.value; // ここで呼ぶ this と等しい
  }
};
myObject.show(); // 10


```

## ２. 関数呼び出しパターン

関数呼び出しパターンでもメソッド呼び出しパターンと全く同じ考え方でOKです。

```js

```


## ３. コンストラクタ呼び出しパターン


## ４. apply,call呼び出しパターン