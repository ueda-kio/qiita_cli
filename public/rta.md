---
title: 業務RTA走者による業務爆速化のアプローチ（物理）
tags:
  - ショートカット
  - 効率化
  - Raycast
  - 業務改善
  - 業務効率化
private: true
updated_at: '2023-11-13T17:11:07+09:00'
id: bfc41536bd211fbc6eb8
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
この記事は [NIJIBOX Advent Calendar 2023](https://qiita.com/advent-calendar/2023/nijibox) の1日目の記事です🌈

:::note warn
**本記事の内容は、あくまでも私の個人的な経験から得たオススメのやり方をまとめたものになります。**
:::


どうも。業務RTA走者の@ment_REです。
普段は「Any%」「No Glitch」「休憩1時間を必ず取る」レギュレーションで走ってます。寒い時期は朝布団から出てタイマースタートするまでが一番の難所だと思ってます[^1]。

## 仕事早くなりたいですよね？

みなさん、仕事早くなりたいですよね！？
私も早くなりたいです。残業なんてしたくない！定時にｻｯと上がりたい！
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

という思想です。

▼ **「タブを閉じる」のような小さなボタンは絶対に押したくありません！**
![ブラウザのタブを閉じるボタンが小さい参考画像](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/2bed6a1d-f178-831c-ad27-d8f64f2c5c17.png)

## 確実にあなたの業務スピードが向上します

断言します。キーボードショートカットとランチャーアプリを取り入れることで **確実にあなたの業務スピードが向上します**（PCワーカーの場合）。
なぜならPCワーカーの我々はキーボードとマウス（またはトラックパッド）を使ってコミュニケーションを取り、コードを書き、アプリを操作して仕事をしているため、これらの基本動作が早くなればそのまま業務のスピードも上がることになるからです。

**向き不向きはありません。**

## 対象読者

- キーボードショートカットをあまり使いこなせていないと感じている方。
- ランチャーアプリを知らない方。または使いこなせていない方。

# 本題

以下の2点を順に取り上げていきます。

1. キーボードショートカット
2. ランチャーアプリ

:::note warn
本記事の内容はすべて **macOSを想定** しています。
:::

## 0. 基本理念

キーボードショートカットもランチャーアプリも、どちらも「そんなの覚えなくてもマウス操作すればいいじゃん」というパンチに殴られがちです。それはその通りなのですが、これらを使うことで **マウスで操作するよりも動作が1秒早く** なります。そしてこの1秒がアクション毎に発生するため、1日経つ頃には大きな塵積も山になっています。大きな業務効率化です。

しかしこれらに慣れないうちはむしろ時間がかかります。「あれ、あのショートカットってなんだっけな？」「あ、この操作ランチャーアプリからできるんだっけ。え〜っとどうやるんだっけ。」といった具合にです。それでも無理矢理にでも矯正し、何も考えなくともこれらを使えるようになると、業務が爆速になります。大丈夫です、一週間で慣れます。

## 1. キーボードショートカット

ここからはキーボードショートカット（以下ショートカット）のうち、 **超絶頻出で大事なもの** のみをピックアップして掲載します。コピー（`cmd + P`）などは基本的過ぎるため省いています。

:::note info
装飾キーをすべて省略して記述しています。
- commandキー: `cmd`
- controlキー: `ctrl`
- shiftキー: `shift`
- optionキー: `opt`
:::

### 1.1. テキスト入力関連

まずはテキスト入力に関連するショートカットです。これらはすべて **`ctrl`キーを押しながら** のショートカットとなります。

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

:::note info
「`opt`を押しながら」操作すると「単語ごと」、「`cmd`を押しながら」操作すると「行ごと」というショートカットとなります。特定の単語を選択したい場合、カーソル移動するのではなく **矢印キーで移動して`opt+shift`で単語を選択する** ほうが早いことがあるので、ぜひこの操作に慣れてください。

`ctrl + shift + opt(/cmd) + P/N/B/F`と複数の装飾キーを同時に押す必要があるので最初は難しいですが、指の形が慣れるまで練習するとよいです。
:::

今日から **二度と矢印キーとbackspaceは押さないでください！**（懇願🙇‍♂️）
かなりの頻度で入力する割に入力時に手がホームポジションから離れるため時間のロスとなります。ショートカットを駆使すればホームポジションから動かずに済みます。

<details>
<summary>ちょっと寄り道</summary>
<div>

筆者はデフォルトのキーバインドが慣れずに押しづらかったため、[Karabiner-Elements](https://karabiner-elements.pqrs.org/) というアプリを導入してキーバインドを変更しています。特に矢印キーの配置がデフォルトだとわかりづらかったため、vimと揃えて上下左右を`ctrl + h, j, k, l`にしています。

</div>
</details>

その他macOSの各ショートカットの詳細はこちらに記載されています。

https://support.apple.com/ja-jp/HT201236

### 1.2. ブラウザ（Google Chrome）

続いてブラウザのショートカットです。

#### 1.2.1. 必須of必須

まずは必須of必須、使いこなせないとマズいショートカットです。
タブ移動以外は基本的に **`cmd`キーを押しながら** です。また、それぞれに **`cmd + shift`を押しながら** というショートカットがあり`(/xxx)`で記述しています。こちらも必須ですので同時に覚えるとよいです。

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

:::note info
これらショートカットは **ブラウザに限らず多くのアプリケーションで共通しています**。使いこなせるようになるとすべてのアプリケーションの操作が早くなるため超時短になります。
必須です。ほんとに。
:::

**このへんの小さなボタンは二度と押さないでください！**
![ブラウザの「タブを閉じるボタン」と「タブを開くボタン」画像](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/77a315fc-f12d-6cfc-0553-53c5527b08b3.png)

他にも「リンクを新規タブで開く」を **右クリックから操作するのはやめてください！** `cmd`キーを押しながらクリックして行いましょう。「新規ウィンドウで開く」場合は`shift`キーを押しながらクリックです。

#### 1.2.2. Devtools

以下はDevtools関連のショートカットです。 **これらもweb開発者にとって必須of必須です**。


<table>
<tbody>
<tr>
<td>開発者モードを有効化</td>
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

<details>
<summary>ファイルへのアクセスに便利です</summary>
<div>

画面に読み込まれているファイルを探すときにnetworkパネルから頑張って探す必要はありません。コマンドパレットを立ち上げてファイル名を入力すればサジェストしてくれます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/36f049de-630b-b98d-33a7-13dbaaabad6c.png)

</div>
</details>

</td>
</tr>
<tr>
<td>検索（searchパネル）</td>
<td>

`cmd + opt + F`

<details>
<summary>`cmd + shift + F`へ変更がオススメ</summary>
<div>

VS Codeではリポジトリ検索のショートカットが`cmd + shift + F`なのでこちらと揃えるのがオススメです。Devtoolsの設定画面から「Shortcuts > Sources > Search」にて変更が可能です。
![searchパネルへのショートカットの設定方法.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/d4153bb5-cd8c-57da-10b9-7939e5cb0ff0.png)

</div>
</details>

</td>
</tr>
</tbody>
</table>

**このへんの小さなボタンは二度と押さないでください！**
![インスペクトモードとレスポンシブモードのボタン](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/19e9215b-a29a-2d3c-9be4-fbfee4db292c.png)

#### 1.2.3. その他

以下はブラウザでのその他覚えておくと便利なショートカットです。
必須of必須というほどではないですが、必須レベルです。

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

その他Google Chromeの各ショートカットの詳細はこちらに記載されています。

https://support.google.com/chrome/answer/157179?hl=ja&co=GENIE.Platform%3DDesktop

### 1.3. Visual Studio Code

続いてVisual Studio Code（以下VS Code）のショートカットです。
VS Codeのショートカットは挙げ出したら無限にありますが（過言）その中でも **必須なんだけどあまり知られていないかもしれない** ショートカットを載せます。

<table>
<tbody>
<tr>
<td>コマンドパレット</td>
<td>

- ファイル移動: `cmd + P`
- コマンド入力: `cmd + shift + P`

</td>
</tr>
<tr>
<td>一括選択</td>
<td>

テキストを選択した状態で入力することで複数の単語を一括選択できます。

- 次のマッチ（同じテキスト）を選択: `cmd + D`
- ファイル内のすべてのマッチを選択: `cmd + shift + L`

</td>
</tr>
<tr>
<td>行操作</td>
<td>

- カーソル行を削除: `cmd + shift + K`
- カーソル行を上下に移動: `opt + ↑(/↓)`
  - 矢印キーショートカット込み: `ctrl + opt + P(/N)`
- カーソル行を上下に複製: `opt + shift + ↑(/↓)`
  - 矢印キーショートカット込み: `ctrl + shift + opt + P(/N)`

</td>
</tr>
<tr>
<td>コメントアウト</td>
<td>

- 行全体: `cmd + /`
- 選択テキストのみ: `opt + shift + a`

</td>
</tr>
<tr>
<td>メニューバーへのアクセス</td>
<td>

- エクスプローラーパネル: `cmd + shift + E`
- 検索パネル: `cmd + shift + F`
- 拡張機能パネル: `cmd + shift + X`
- Git（ソース管理）パネル: `ctrl + shift + G`
  - （このパネルだけ`ctrl`な点に注意）

</td>
</tr>
</tbody>
</table>

特にコマンドパレットの操作にはぜひ慣れてください。コマンド入力で大体の操作は呼び出せますし、ファイルにアクセスするときもエクスプローラーから探すのではなく`cmd + P`からファイル名を入力してアクセスするようにしてください。**巨大なリポジトリの中から目的をファイルを目視で探すよりもこちらのほうが圧倒的に早いです**。

:::note info

**タブ切り替えのショートカット（`ctrl + Tab`）の動作を変更することをオススメします。**
以下をUserSettingの`keybindings.json`に追記することでエディター上でのタブ切り替えの動作をブラウザと揃えることができます。必須設定です。
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

その他、ほとんどの操作にはショートカットが割り当てられている、または割り当て可能ですので、よくやる操作はすべてショートカットで行うとよいです。こちらの記事にVS Codeのショートカットが詳細かつわかりやすくまとめられているので参照してください。

https://qiita.com/naru0504/items/99495c4482cd158ddca8

## 2. ランチャーアプリ

ランチャーアプリはPC内のアプリケーションやファイルへキーボード入力で素早くアクセスできるアプリケーションで、Macには [Spotlight](https://support.apple.com/ja-jp/guide/mac-help/mchlp1008/14.0/mac/14.0) というランチャーアプリが標準で入っています。
本記事ではこのSpotlightをハチャメチャに拡張したアプリ **[Raycast](https://www.raycast.com/)** を駆使することで業務効率化を図っていきます。

https://www.raycast.com/

とはいえRaycastに関する参考になる記事はすでにあらゆるところでまとめられているので、詳しい使い方やインストール方法については本記事よりもそれらを参照したほうがよいと思います。

https://qiita.com/mtaku29/items/31ec3b0ac48ab35d8a80


https://zenn.dev/fumi_sagawa/articles/2ff5fd9c03fbcd


### 2.1. 基本的な使い方

ホットキーを入力すると画面上に入力フィールドが表示され、そこにアプリケーション名やファイル名、その他さまざまなコマンドを入力することで操作できます。

![Raycast参考画像](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/29d47547-1d97-e00a-7b4e-6b313eefef77.png)

以下は、筆者が特に便利だと思うRaycastの主要な機能です。

<table>
<tbody>
<tr>
<td><strong>ブックマーク検索</strong></td>
<td>

もうブラウザのブクマ一覧から探す必要はありません。**まごうことなき神機能**。

</td>
</tr>
<tr>
<td><strong>クリップボードヒストリー</strong></td>
<td>

検索（filter）機能とサムネイル機能がすばらしい。**まごうことなき神機能**。

</td>
</tr>
<tr>
<td><strong>スニペット</strong></td>
<td>

登録した文字列を任意の識別子で瞬時に出力する機能。たまに反応してくれない時がありますが **神機能**。

</td>
</tr>
<tr>
<td><strong>Quicklinks</strong></td>
<td>

クエリを指定することでさまざまなwebサイトの検索結果画面へ飛ぶことが可能です。**神機能**。

</td>
</tr>
<tr>
<td><strong>ウィンドウマネージャー</strong></td>
<td>

アプリのウィンドウサイズの調節機能。全画面・右半分・左側に3/4などの画面サイズへコマンド一発でリサイズ可能。**神機能**。

</td>
</tr>
<tr>
<td><strong>システム機能の実行</strong></td>
<td>

Sleep, Lock, Shut Down, Log OutなどOSの機能を呼び出せます。いい機能。

</td>
</tr>
<tr>
<td><strong>その他数多のextensions</strong></td>
<td>

先人たちのすばらしい拡張機能をアプリから直接または[Store](https://www.raycast.com/store)からインストール可能。

</td>
</tr>
</tbody>
</table>

すべての機能を紹介すると記事が長くなってしまうので、特にオススメな機能のみピックアップして以下にまとめます。

### 2.2. 必須の設定

設定画面の「Advanced > Auto-switch input Source」から「Romaji（またはEnglish）」を選択してください。これにより **Raycastを開くと自動で入力ソースがアルファベット状態になります**。基本的にRaycastは英字入力する機能がほとんどですので、これは必須の設定です。
<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/ef97a7c1-7837-d521-740f-fafad479ef5a.png" width="700">

### 2.3. ブックマーク検索

以下から拡張機能をインストールしてください。

https://www.raycast.com/raycast/browser-bookmarks

これによりブラウザのブクマ一覧から一生懸命目的のサイトを探す手間はなくなります。部分一致でサジェストしてくれるので登録名を忘れてても見つけられます。

![ブクマ検索の例](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/7cd6536a-6f90-9721-1dc4-d3a9f7e5db00.png)

かなり頻繁に使う機能ですのでホットキーの登録を推奨します！（私は`cmd + shift + ctrl + space`を登録してます）
「起動 → 入力 → Enter」で済みます。マウス操作と比べて1秒どころではない時短になります。ほんとにオススメです。


### 2.4. クリップボードヒストリー

Raycastには標準でクリップボードヒストリー機能が搭載されています。この機能のために [Clipy](https://github.com/Clipy/Clipy) というアプリケーションを導入している方も多いかと思いますが、**乗り換え推奨** です。

Clipyではできない、検索（filter）機能やサムネイル機能が本当にすばらしいです。

▼ 検索機能がすばらしい。
![クリップボードヒストリーの検索機能](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/4771b3ae-05af-11fe-6770-288979702931.png)

▼ サムネイル機能。特にコピーした画像を表示してくれるのがすばらしすぎる。`cmd + shift + ctrl + 4`との相性最高。
![クリップボードヒストリーのサムネイル機能](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/4a8e7d31-75d1-e4d3-3801-fa00e3daf6d2.png)

### 2.5. Quicklinks

任意のwebサイトを開くことができ、かつクエリを引数として渡すことができるため検索結果画面へ即座に飛ぶことができます。つまり **Raycast上から任意のサイト内検索が可能** ということです。

▼ QuicklinkでGoogle検索する例（`g`をエイリアスに設定）
![Google検索の例](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/f5adfb68-b45a-dbc5-4702-01e397cfb704.gif)

▼ 設定画面の「Create Quicklink」から作成可能です。
![Quicklinks設定画面](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/281fd51b-73e6-c470-aec3-805af7239060.png)

お好きなwebサイトを登録してRaycastから爆速検索しましょう！
Google検索、Google画像検索、YouTube、[CanIUse](https://caniuse.com/)、[GitHub](https://github.com/search)、etc…

# おわりに

改めてとはなりますが、ショートカットもランチャーアプリも慣れないうちは逆に時間がかかります。ただ根気強く無理矢理にでも使っていけば、1週間もすれば慣れます。各アクション毎に1秒の時短です。

業務RTA自己べスト更新狙っていきましょう🏃💨

<!-- ![ありがとうございました](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/7b513e34-93f3-d5a2-3b84-0bcb8516bca8.png) -->

![ありがとうございました](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/3b87b25e-9ab0-e2ba-0810-03119b57053c.png)

[^1]: もちろん「業務RTA」とは茶番です。そんな概念はありません。
