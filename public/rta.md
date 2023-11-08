---
title: 業務RTA走者による業務爆速化のアプローチ（物理）
tags:
  - '業務効率化'
  - '業務改善'
  - '効率化'
  - 'ショートカット'
  - 'Raycast'
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
この記事は [NIJIBOX Advent Calendar 2023](https://qiita.com/advent-calendar/2023/nijibox) の1日目の記事です🌈

どうも。業務RTA走者の@ment_REです。
普段は「Any%」「No Glitch」「休憩1時間を必ず取る」レギュレーションで走ってます。寒い時期は布団から出てタイマースタートするまでが一番の難所だと思ってます[^1]。

## 仕事はやくなりたいですよね？

みなさん、仕事はやくなりたいですよね！？
私ははやくなりたいです。残業なんてしたくない！定時にｻﾝｯって上がりたい！
とはいえ仕事はこなさなければなりません。なので既定労働時間内でより多くの仕事を片付けるために、業務の効率化が必要です。それも爆速の。

ということで『仕事効率化』で調べてみると以下のようなアプローチがヒットします。

- セルフマネジメント
  - TODO管理
  - 時間管理（ポモドーロとか）
- 作業フローの整備
- 自動化
- etc…

ですが、本記事ではこれらは扱いません。

## 本記事での業務効率化のアプローチ

本記事での業務効率化問題を **物理** で解決します。

![物理](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/9993c699-e80c-c847-851e-ef5ab694feda.png)

つまりどういうことかと言うと、 **手と指の動きを最小限に最速で動かす** ことで仕事を爆速でこなそうぜ、というアプローチです。


# 本記事の結論

**キーボードショートカット** と **ランチャーアプリ** を駆使して業務効率化を図ります。

- ホームポジションから動きたくない。
- よくやる操作は全部キーボードからショートカットで呼び出したい。
- 小さくて押しづらいボタンは押したくない。

がモチベです。

▼ **こんな小さいボタンは絶対に押したくありません**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/2bed6a1d-f178-831c-ad27-d8f64f2c5c17.png)

:::note warn
本記事の内容はすべて **macOSを想定** しています。
macOS以外ですとキーボードショートカットやアプリケーションが適していない内容もありますのでご了承ください。
:::

## 確実にあなたの業務スピードが向上します

断言します。キーボードショートカットとランチャーアプリを取り入れることで **確実にあなたの業務スピードが向上します**（PCワーカーの場合）。
なぜならPCワーカーの我々はキーボードとマウス（またはトラックパッド）を使って会話をし、コードを書き、アプリを操作して仕事をしているため、これらの基本動作がはやくなればそのまま業務のスピードも上がることになるからです。

**向き不向きはありません。大丈夫です、一週間で慣れます。**

# 本題

以下の2点を順に取り上げていきます。

1. キーボードショートカット
2. ランチャーアプリ

## 0. 基本理念

その前にまずは基本的な考え方を書かせてください。

キーボードショートカットもランチャーアプリも、どちらも「そんなの覚えなくてもマウス操作すればいいじゃん」というパンチに殴られがちです。それはその通りなのですが、これらを使うことで **マウスで操作するよりも1秒はやく** なります。そしてこの1秒がアクション毎に発生するため、1日経つ頃には大きな塵積も山になっています。

また、これらに慣れないうちはむしろ時間がかかります。「あれ、あのショートカットってなんだっけな？」「あ、この操作ランチャーアプリ使わなきゃだった」といった具合にです。それでも無理矢理にでも矯正し、何も考えなくともこれらを使えるようになると業務が爆速になります。大丈夫です、一週間で慣れます。

## 1. キーボードショートカット

ここからはキーボードショートカット（以下ショートカット）のうち、 **絶対MUSTに必須で大事なもの** のみをピックアップして掲載します。コピー（`cmd+p`）などは基本的過ぎるため省いています。

:::note info
以降装飾キーをすべて省略して記述しています。
- commandキー: `cmd`
- controlキー: `ctrl`
- shiftキー: `shift`
- optionキー: `opt`
:::

### 1.1. 入力全般

まずは入力全般に関連するショートカットです。これらはすべて **`ctrl`キーを押しながら** というショートカットとなります。

<table>
<tbody>
<tr>
<td>矢印キー</td>
<td>

- 上: `P`
- 下: `N`
- 左: `B`
- 右: `F`

</td>
</tr>
<tr>
<td>削除</td>
<td>

- backspace: `H`
- delete: `D`

</td>
</tr>
<tr>
<td>カタカナ変換</td>
<td>

`K`

</td>
</tr>
<tr>
<td>段落移動</td>
<td>

- 段落の先頭に移動: `A`
- 段落の末尾に移動: `E`

</td>
</tr>
</tbody>
</table>

<!-- - 矢印キー
  - 上: `P`
  - 下: `N`
  - 左: `B`
  - 右: `F`
- 削除
  - backspace: `H`
  - delete: `D`
- 段落移動
  - 段落の先頭に移動: `A`
  - 段落の末尾に移動: `E` -->

今日から **二度と矢印キーとbackspaceは押さないでください！**（過激派）
かなりの頻度で入力する割に、入力時に手がホームポジションから離れるためです。ショートカットを駆使すればホームポジションから動かずに済みます。

:::note info
「`opt`を押しながら」操作すると「単語ごと」、「`cmd`を押しながら」操作すると「行ごと」というショートカットとなります。「ある単語を選択したい」という場合もカーソル移動するのではなく、**矢印キーで移動して`opt+shift`で選択する** という動きのほうがはやい場合があるので、ぜひこの操作に慣れてください。
:::

<details>
<summary>ちょっと寄り道</summary>
<div>

筆者はデフォルトのキーバインドが慣れずに押しづらかったため、[Karabiner-Elements](https://karabiner-elements.pqrs.org/) というアプリを導入してキーバインドを変更しています。とくに矢印キーの配置がデフォルトだとわかりづらかったため、vimと揃えて上下左右を`ctrl+h, j, k, l`にしています。

</div>
</details>



### 1.2. ブラウザ（Google Chrome）

#### 1.2.1 必須of必須

まずは必須of必須、使いこなせないとマズいショートカットたちです。
タブ移動以外は基本的に **`cmd`キーを押しながら** です。また、それぞれに **`cmd + shift`を押しながら** というショートカットがあり`(/xxx)`で記述しています。。こちらも必須ですのでいっぺんに覚えるとよいです。

<table>
<tbody>
<tr>
<td>タブ操作</td>
<td>

- 次のタブへ移動（/前のタブへ移動）: `ctrl + Tab`
- 開く（/直前に閉じたタブを再度開く）: `T`
- 閉じる（/ウィンドウを閉じる）: `W`

</td>
</tr>
<tr>
<td>画面更新<br>（/パードリロード）</td>
<td>

`R`

</td>
</tr>
<tr>
<td>新規ウィンドウ<br>（/新規シークレットウィンドウ）</td>
<td>

`N`

</td>
</tr>
<tr>
<td>設定画面</td>
<td>

`,`

</td>
</tr>
</tbody>
</table>


<!-- - タブ操作
  - 次のタブへ移動: `Tab`
  - 開く: `T`
  - 閉じる: `W`
- 画面更新: `R`
- 新規ウィンドウ: `N`
- 設定画面: `,` -->

:::note info
これらショートカットは **ブラウザに限らず多くのアプリケーションで共通しています**。使いこなせるようになるとすべてのアプリケーションの操作がはやくなるため超時短になります。
必須です。ほんとに。
:::

他にも「リンクを新規タブで開く」を **右クリックから操作するのは絶対にやめてください！** `cmd`押しながらクリックです。「新規ウィンドウで開く」は`shift`押しながらクリックです。

#### 1.2.2 Devtools

Devtools関連のショートカットです。 **これらもwebエンジニアなら必須of必須です**。


<table>
<tbody>
<tr>
<td>開発者モードON</td>
<td>

`F12`

</td>
</tr>
<tr>
<td>インスペクトモード</td>
<td>

`cmd + shift + C`
（Devtoolsパネルが開かれていない場合、パネルを開いて機能する）

</td>
</tr>
<tr>
<td>レスポンシブモード</td>
<td>

`cmd + shift + M`

</td>
</tr>
<tr>
<td>コマンドパレット</td>
<td>

`cmd + P`

</td>
</tr>
<tr>
<td>検索（searchパネル）</td>
<td>

`cmd + opt + F`

<details>
<summary>`cmd + shift + F`へ変更がオススメ</summary>
<div>

VS Codeではリポジトリ検索のショートカットが`cmd + shift + F`なのでこちらと揃えるのがオススメです。Devtoolsの設定画面の「Shortcuts > Sources > Search」から変更可能です。
![searchパネルへのショートカットの設定方法.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/d4153bb5-cd8c-57da-10b9-7939e5cb0ff0.png)

</div>
</details>

</td>
</tr>
</tbody>
</table>

<!-- - 開発者モードON: `F12`
- インスペクトモード: `cmd + shift + C`
- レスポンシブモード: `cmd + shift + M`
- コマンドパレット: `cmd + P`
- 検索: `cmd + shift + F`（オススメ設定） -->

#### 1.2.3 その他

必須of必須というほどではないですが覚えておくと意外と便利なショートカットたちです。

<table>
<tbody>
<tr>
<td>履歴画面を開く</td>
<td>

`Y`

</td>
</tr>
<tr>
<td>タブを検索</td>
<td>

`cmd + shift + A`

（タブ地獄になる人にオススメ）

</td>
</tr>
<tr>
<td>URL入力欄にフォーカス</td>
<td>

`cmd + L`

</td>
</tr>
<tr>
<td>拡大/縮小</td>
<td>

- 拡大: `cmd + shift + ;`
- 縮小: `cmd + -`
- リセット: `cmd + 0`

</td>
</tr>
</tbody>
</table>

<!-- - 履歴画面を開く: `Y`
- タブ検索: `cmd + shift + A`
- URL入力欄にフォーカス: `cmd * L`
- 拡大/縮小
  - 拡大: `cmd + shift + ;`
  - 縮小: `cmd + -`
  - リセット: `cmd + 0` -->


### 1.3. VS Code

続いてVS Codeのショートカットたちです。
VS Codeのショートカットは挙げ出したら無限にありますが（過言）その中でも **必須なんだけどあんまり知られていないかもしれない** ショートカットを載せます。

<table>
<tbody>
<tr>
<td>コメントアウト</td>
<td>

- 行全体: `cmd + /`
- 選択テキストのみ: `opt + shift + a`

</td>
</tr>
<tr>
<td>コマンドパレット</td>
<td>

- ファイル移動: `cmd + P`
- コマンド入力: `cmd + shift + P`

</td>
</tr>
<tr>
<td>メニューバーへのアクセス</td>
<td>

- エクスプローラー: `cmd + shift + E`
- 検索: `cmd + shift + F`
- 拡張機能: `cmd + shift + X`
- Git（ソース管理）: `ctrl + shift + G`
  - （これだけ`ctrl`な点に注意）

</td>
</tr>
</tbody>
</table>



:::note info

タブ切り替えのショートカットを変更することをオススメします。
以下をUserSettingの`keybindings.json`に追記することでエディター上でのタブ切り替えの挙動をブラウザと揃えることができます。必須設定です。
```json:keybindings.json
[
  {
    "key": "ctrl+tab",
    "command": "workbench.action.nextEditor"
  },
  {
    "key": "ctrl+shift+tab",
    "command": "workbench.action.previousEditor"
  }
]
```

:::

## 2. ランチャーアプリ

ランチャーアプリとはアプリケーションやファイルなどにキーボード入力でアクセスできる機能を持つアプリであり、Macには標準で [Spotlight](https://support.apple.com/ja-jp/guide/mac-help/mchlp1008/14.0/mac/14.0) が入っています。
本記事ではこのSpotlightをハチャメチャに拡張した神アプリ **Raycast** を駆使することで業務効率化を図っていきます。

https://www.raycast.com/

とはいえRaycastに関する素晴らしい記事はすでにあらゆるところでまとめられているので、詳しい使い方やインストール方法については本記事よりもそれらを参照したほうがよいと思います。

https://qiita.com/mtaku29/items/31ec3b0ac48ab35d8a80


https://zenn.dev/fumi_sagawa/articles/2ff5fd9c03fbcd



ブクマ検索
起動→入力→Enter
ですむ


# おわりに

ショートカットもランチャーアプリも慣れないうちは逆に時間かかります。
ただ根気強く無理矢理にでも使っていけば1週間もすれば慣れます。アクション毎に1秒の時短です。

業務RTA自己べ更新狙っていきましょう。


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/7b513e34-93f3-d5a2-3b84-0bcb8516bca8.png)

# おまけ
本題はなるべく短くまとめたかったためキーボードショートカットとランチャーアプリの2本建てで構成しましたが、その他にも色々と物理的な業務効率化のために導入しているものがありますのでいくつか付録として紹介します。

## Chrome拡張機能

### 1. Vimium
https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb

### 2. Tampermonkey

https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo

### 3. Pasty

https://chrome.google.com/webstore/detail/pasty/hdjihnnclpjhfdbbinmgoiehhoehhlgf

### 4. simpleGestures

https://chrome.google.com/webstore/detail/simplegestures/flfminafiamnggnldfpilnfnmbgmiegn

### 5. Google検索キーボードショートカット

https://chrome.google.com/webstore/detail/google-search-results-sho/dchaandmcifgjemlhiekookpgjmkcelg


## VS Code extensions

### 1. vscode-input-sequence

連番を簡単に入力できるextensions。

https://marketplace.visualstudio.com/items?itemName=tomoki1207.vscode-input-sequence

## その他
- 隙あらばブックマークレットを作る



[^1]: もちろん「業務RTA」とは茶番です。そんな概念はありません。