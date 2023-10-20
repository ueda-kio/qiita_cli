---
title: ハンバーガーメニュー最適解（デザインのみ）
tags:
  - HTML
  - CSS
  - Web
  - Webデザイン
private: false
updated_at: '2022-09-04T18:16:20+09:00'
id: af10e2a5ce8fc8fb56ee
organization_url_name: null
slide: false
ignorePublish: false
---
## 結論
**個人的**最適解です。
<p class="codepen" data-height="400" data-default-tab="html,result" data-slug-hash="OJZLJpo" data-user="ueda-kio" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/ueda-kio/pen/OJZLJpo">
  hamburgerMenu</a> by Men_Tori (<a href="https://codepen.io/ueda-kio">@ueda-kio</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>


### ポイント
- デザインである横棒は擬似要素で実装
- ボタン内にテキストを持たせ、ボタンの目的を表す
- `type="button"`をしっかり書く


## よくあるやつ
見た目・機能は全く同じです。
<p class="codepen" data-height="400" data-default-tab="html,result" data-slug-hash="abGobGJ" data-user="ueda-kio" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/ueda-kio/pen/abGobGJ">
  hamburgerMenu__notGood</a> by Men_Tori (<a href="https://codepen.io/ueda-kio">@ueda-kio</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>


### よくないポイント
- デザインを空要素（`span`）で実装している
- `button`要素にアクセシブルなテキストがセットされていない
- デフォルトの`type="submit"`になっている



## 気をつけたいポイント
### 1. デザインの実装はCSS or 擬似要素で行う
よくある実装の方では、デザインである横棒を`span`要素で実装しています。が、基本的にデザインの実装はcssのみで行い、それでは実装が難しい場合でも、空要素をおくのではなく擬似要素で実装するようにしましょう。
HTMLの仕様上では、ほぼすべての要素が該当する*パルパブルコンテンツ*は、**空要素ではいけない**という仕様となっています。
[HTML Standard - パルパブルコンテンツ](https://momdo.github.io/html/dom.html#palpable-content)

> …コンテンツ内に少なくとも1つのノードを持つべきである。これがパルパブルコンテンツであり、hidden属性が指定されていない。


ということで、空の`span`要素でデザインを実装することは避けましょう。

<details>
<summary>【余談】</summary>

`div.spacer`みたいな空`div`で余白を取る実装があると考えてみてください。とっても違和感＆奇妙に感じますよね👀
空の`span`でハンバーガーを実装することもこれくらい奇妙に私は感じます。

</details>

### 2. `button`要素にアクセシブルなテキストを挿入する
`button`要素や`a`要素などには、その目的を表すアクセシブルな（識別可能な）テキストが必要です。テキストが含まれていない`button`要素はいけませんよ〜、ということですね。
ということで、**`button`要素内は空っぽにせず「メニューを開く」などのテキストを入れておきましょう**。
（詳細は避けますが、アクセビリティAPIに伝わればよいので、`aria-label`でもよきです）

▼ devtoolによってはおこってくれます
![0D6F31CF-8CE2-4A89-9E13-0E27A4971B46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/637090ab-f85c-8a7f-5eff-19e08ed7fe9f.png)


### 3. `button`の`type`属性を省くと`type="submit"`扱いになる
`button`要素の`type`属性は3種類あり（[参考](https://momdo.github.io/html/form-elements.html#attr-button-type)）、初期値は`submit`となっています。
**ハンバーガーメニューはもちろん送信ボタンではないので、`type="button"`を明示したほうがよい**でしょう。

とはいえ、親要素に`form`要素が存在しない`button`要素の場合は処理が自動で`return`されますが、それでも明示した方がよいと考えています。


> A button element element's activation behavior is:
>1. If element is disabled, then return.
>2. If element does not have a form owner, then return.
>3. If element's node document is not fully active, then return.
>4. Switch on element's type attribute's state:




## おわりに

個人的ベストプラクティスでした。
「**仕様の解釈ちがうぞ！**」「**もっとこっちのほうがいいぞ！**」「**そんなんじゃぜんぜんだめだ！**」
などありましたらどしどしくださいませ。
