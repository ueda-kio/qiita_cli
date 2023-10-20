---
title: タブUIの非機能要件を満たしたい時に見る記事
tags:
  - HTML
  - JavaScript
  - UI
  - アクセシビリティ
private: false
updated_at: '2022-12-04T07:04:09+09:00'
id: 7ef9c0c3ecfe5468cb9f
organization_url_name: nijibox
slide: false
ignorePublish: false
---
## はじめに
複数のコンテンツを並列にグループ化し各コンテンツに紐づいたタブ要素をアクティブにすることでコンテンツを切り替えるUI、タブUI。
モーダルやカルーセルなどのUIと比べると実装難易度は低めのUIかと思いますが、特にキーボード操作周りで是非に満たしたい非機能要件がいくつかあります。
今回はその非機能要件にフォーカスをあて、具体例をあげながら実装方法を見ていきます。
（ほとんどまるっと[WAI-ARIA Authoring Practices 1.2](https://w3c.github.io/aria-practices/#tabpanel)の写経です）

## 完成形
まずは非機能要件を意識せずに作ったタブUIです。

<p class="codepen" data-height="500" data-default-tab="html,result" data-slug-hash="eYKKNXO" data-user="ueda-kio" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/ueda-kio/pen/eYKKNXO">
  Untitled</a> by Men_Tori (<a href="https://codepen.io/ueda-kio">@ueda-kio</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

そしてこちらが非機能要件を意識して作ったタブUIです


<p class="codepen" data-height="500" data-default-tab="html,result" data-slug-hash="XWYqpOe" data-user="ueda-kio" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/ueda-kio/pen/XWYqpOe">
  Tab UI - Accessible</a> by Men_Tori (<a href="https://codepen.io/ueda-kio">@ueda-kio</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

ぱっと触った限りでは、両者の違いはわからないかと思います。
そんな「ぱっと触った限りではわからない部分のお話」ということになります。


## 機能要件
まずはタブUIの一般的な機能要件を確認しましょう。Authoring Practices 1.2から引用します。


> Tabs are a set of layered sections of content, known as tab panels, that display one panel of content at a time. Each tab panel has an associated tab element, that when activated, displays the panel. The list of tab elements is arranged along one edge of the currently displayed panel, most commonly the top edge.
> *タブは、タブパネルと呼ばれるコンテンツのレイヤーセクションのセットで、一度に1つのパネルを表示します。各タブパネルには、関連するタブ要素があり、これをアクティブにするとパネルが表示されます。タブエレメントのリストは、現在表示されているパネルの一辺（通常は上辺）に沿って配置されます。*
（DeepLにより翻訳）

https://w3c.github.io/aria-practices/#tabpanel

- タブパネルに紐づいたタブが存在し、タブをアクティブにするとタブパネルが表示される
- 一度に表示されるタブパネルは１つ
- タブのリストは表示されているタブパネルの上辺に沿って配置されている

くらいでしょうか。
シンプルですし我々の思っている通りの機能かと思います。
（以下、本記事ではクリック可能なコンテンツを切り替えるトリガーの部分を「タブ」、切り替わるコンテンツ部分を「タブパネル」と呼ぶことにします。）

```js
class Tab {
    constructor() {
        this.tabs = tabs; // タブ
        this.tabPanels = tabPanels; // タブパネル
        this.activeIndex = this.tabPanels.findIndex((elm) =>
            elm.classList.contains('is-active')
        ); // 初期表示時にアクティブなタブのインデックス
        this.activeTab = this.tabs[this.activeIndex]; // 現在アクティブなタブ
        this.activeTabPanel = this.tabPanels[this.activeIndex]; // 現在アクティブなタブパネル

        this.#handleClick();
    }

    #handleClick() {
        this.tabs.forEach((tab, i) => {
            tab.addEventListener('click', (e) => {
                const target = e.currentTarget;
                if (target === this.activeTab) return;

                const targetPanel = this.tabPanels[i];
                target.classList.add('is-active');
                targetPanel.classList.add('is-active');
                this.activeTab.classList.remove('is-active');
                this.activeTabPanel.classList.remove('is-active');

                this.activeIndex = i;
                this.activeTab = target;
                this.activeTabPanel = targetPanel;
            });
        });
    }
}
```

[完成形](#完成形)でお見せした「非機能要件を意識せずに作ったタブUI」のjsのソースとしてはこんな感じです。（一部省略しています）
アクティブなタブ・タブパネルには`is-active`クラスを付与しスタイルを定義、非アクティブなタブをクリックすることでタブ・タブパネルの`is-active`クラスをトグルして表示を切り替えます。

前述した機能要件はこれで満たせているので、非機能要件のことを考えなければこれで問題ないとも思えます。
では、ここからどのような非機能要件をどのように実装するのか見ていきます。



## 非機能要件
以下の要件を実装していきます。
1. `role`属性の追加
2. `ARIA`属性の追加
3. **キーボード操作のサポート**


### 【非機能要件①】`role` 属性の設定
タブ・タブパネルそれぞれを示すhtmlの要素（`<tab></tab>`のようなもの）はないので、マークアップ自体は`div`, `span`などで実装することになります。その上で、タブUIに適した`role`属性がWAI-ARIA[^1]に用意されているのでそれらをありがたく使っていきます。
（`role`属性についての詳細は避けますが「html標準の要素だけでは説明しきれない、その要素の役割を示すための属性」という認識でよいかと）

https://developer.mozilla.org/ja/docs/Web/Accessibility/ARIA/Roles

まずはタブのラッパー要素に`role="tablist"`を、タブ自身には`role="tab"`を付与します。
そしてタブパネルには`role="tabpanel"`を付与します。


```html:role属性の付与
<div class="tabList" role="tablist">
	<div id="tab-1" class="tabList__tab is-active" role="tab">tab-1</div>
	<div id="tab-2" class="tabList__tab" role="tab">tab-2</div>
	<div id="tab-3" class="tabList__tab" role="tab">tab-3</div>
</div>
<div class="tabPanle is-active" role="tabpanel">...</div>
<div class="tabPanle" role="tabpanel">...</div>
<div class="tabPanle" role="tabpanel">...</div>
```
これでマシン・ブラウザに「この要素たちはタブUIなんですよ」ということを伝えることができました。


`role="tablist"`

https://w3c.github.io/aria/#tablist

`role="tab"`

https://w3c.github.io/aria/#tab

`role="tabpanel"`

https://w3c.github.io/aria/#tabpanel


<details>
<summary>[余談...]</summary>
<div>

- **`ul`, `li`要素でリスト化しなくてもよい**
「リスト構造なのだから`ul`, `li`要素で実装する方が適しているのでは？」と感じる方もいるかもしれません。が、その必要はありません。
`ul`要素は`role="list"`、`li`要素は`role="listitem"`というロールを持っていますが、これらよりも`role="tablist"`, `role="tab"`の方が適したロールであり、かつ**ロールは`role`属性により上書きされるため**、`div`, `span`要素などのなんのロールも持たない要素でマークアップしても問題ありません。もちろん、`ul`, `li`要素でマークアップしてもかまいません。

- **タブは`button`要素でマークアップしなくてもよい**
「タブはインタラクティブなUIなので`button`要素で実装する方が適しているのでは？」と感じる方もいるかもしれません。が、上記と同様の理由でその必要はありません。
`button`要素が持っている`button`ロールは結局`role="tab"`によって上書きされるため、必ずしも`button`要素でマークアップしなくても問題ありません。現に今回は`button`要素のデフォルトスタイルをリセットするのが面倒だったので`div`要素でマークアップしました。
が、ロールだけの問題ではなく、Tabキーでアクセス可能だったりSpace, Enterキーでアクティブにできたりと、`button`要素はブラウザ標準で様々な機能を持っているため **`div`でマークアップするからにはその辺りも漏らさずに自分で実装する必要があります**。（その辺りは後述で実装します）


</div>
</details>


### 【非機能要件②】`ARIA`属性の設定
`role`属性以外にも、付与するべき属性がいくつかあります。

#### `tabindex`属性

https://developer.mozilla.org/ja/docs/Web/HTML/Global_attributes/tabindex

（厳密にはWAI-ARIAの機能ではありませんが...）
`tabindex`属性はTabキーでのフォーカス順序を決定する属性で、主に`"0" | "-1"`の値で「Tabキーでフォーカスさせるかどうか」の制御に用いる属性ですが、こちらをタブ・タブパネル両方に指定します。
また、非アクティブなタブにはTabキーでフォーカスできないようにするため、アクティブなタブのみに`tabindex="0"`を指定します。
タブを`button`要素などデフォルトでTabアクセスが可能な要素でマークアップした場合、非アクティブなタブに`tabindex="-1"`を付与して上書きする必要があります。

```html:tabindex属性の付与
<div class="tabList" role="tablist">
	<div id="tab-1" class="tabList__tab" role="tab" tabindex="0">tab-1</div>
	<div id="tab-2" class="tabList__tab" role="tab">tab-2</div>
	<div id="tab-3" class="tabList__tab" role="tab">tab-3</div>
</div>
<div class="tabPanle is-active" role="tabpanel" tabindex="0">...</div>
<div class="tabPanle" role="tabpanel" tabindex="0">...</div>
<div class="tabPanle" role="tabpanel" tabindex="0">...</div>
```

なぜ非アクティブなタブはTabキーでアクセスできないようにしているかという理由は、**Tabキーでの移動順序が「タブ→タブパネル」とするため**です。
非アクティブなタブにも`tabindex="0"`を付与するとTab移動の順序が「アクティブなタブ→非アクティブなタブ」となってしまい、それを避けるためです。

「それでは非アクティブなタブにキーボードでアクセスできず、問題ではないのか」と思われるかもしれませんが、大丈夫です。
詳細は後述しますが、方向キーを使用して非アクティブなタブも操作可能にします。


#### `aria-controls`, `aria-labelledby`, `aria-selected`属性

タブ・タブパネルの紐付けやラベル付け、どのタブがアクティブになっているかの指定など、各種`ARIA`属性を用いて示していきます。
以下が完成形です。

```html:各種ARIA属性の付与
<div class="tabList" role="tablist">
	<div id="tab-1" class="tabList__tab is-active" role="tab" aria-selected="true" aria-controls="tabpanel-1">tab-1</div>
	<div id="tab-2" class="tabList__tab" role="tab" aria-selected="false" aria-controls="tabpanel-2">tab-2</div>
	<div id="tab-3" class="tabList__tab" role="tab" aria-selected="false" aria-controls="tabpanel-3">tab-3</div>
</div>
<div id="tabpanel-1" class="tabPanel is-active" role="tabpanel" aria-labelledby="tab-1" tabindex="0">...</div>
<div id="tabpanel-2" class="tabPanel" role="tabpanel" aria-labelledby="tab-2" tabindex="0">...</div>
<div id="tabpanel-3" class="tabPanel" role="tabpanel" aria-labelledby="tab-3" tabindex="0">...</div>
```

- **紐付け**
まずタブ・タブパネルを紐づける、つまり「このタブをアクティブにするとこのタブパネルが表示される」ということを示すために、タブに`aria-controls`属性を付与しその値に該当するタブパネルの`id`属性を挿入します。
これで紐付けができました。
- **ラベル付け**
次にタブパネル内のコンテンツの内容を示すために、タブパネルに`aria-labelledby`属性を付与しその値にタブの`id`を挿入します。これによりタブパネルのコンテンツの内容が、タブの文字列によって説明されることになります。
これでラベル付けができました。<br>（なお例ではタブの文字列が「tab-1」となっており、コンテンツの内容がさっぱりわからないため適していません。）
- **アクティブタブの指定**
最後に「現在どのタブがアクティブになっているか」を示すために、アクティブなタブに`aria-selected="true"`を付与し、非アクティブなタブに`aria-selected="false"`を付与します。<br>これでアクティブなタブを示すことができました。



`aria-controls`

https://w3c.github.io/aria/#aria-controls

`aria-labelledby`

https://w3c.github.io/aria/#aria-labelledby

`aria-selected`

https://w3c.github.io/aria/#aria-selected


<details>
<summary>[余談...]</summary>
<div>

- **`is-active`クラスでは「アクティブであること」を示せていない**
「そのタブがアクティブであることは`is-active`クラスで示せているのでは？」と感じる方もいるかもしれません。が、残念ながら示せていません。
クラスはあくまでスタイルを指定するための識別子にすぎずhtmlの構造上の意味はなにもありません。そのため、しっかりとマシン・ブラウザに「このタブが今アクティブですよ」と示すため`aria-selected="true"`属性を指定します。
- **`aria-hidden`は不要**
「非アクティブなタブパネルに`aria-hidden="true"`の指定が必要では？」と感じる方もいるかもしれません。が、不要です。
`aria-hidden="true"`は付与することによってアクセシビリティAPIから隠蔽することができスクリーンリーダーなどの読み上げをスキップすることができる属性で、確かに非表示の要素には付与するべき属性です。が、 **cssのスタイルによって`display: none;`が当たっている要素はそれだけでアクセシビリティAPIから隠蔽されている**ため、`aria-hidden="true"`の指定は重複した指定となってしまいます。
- **タブの機能要件は自分で実装する必要がある**
注意していただきたいのは「紐付け」や「アクティブタブの指定」を`ARIA`属性で指定したからといって、ブラウザがよしなにタブ機能を実現してくれるわけではありません。
これらはあくまで要素の状態や性質を説明するためのものにすぎないため、タブの機能はjsなりで実装する必要があります。

</div>
</details>




これらの属性はインタラクティブに変化させる必要があるため、js側にも追加実装が必要です。
`is-active`クラスの付け替えだけでなく、各種`ARIA`属性の付け替えも行いましょう。

```diff_javascript:属性の切り替え
tab.addEventListener('click', (e) => {
    const target = this.tabs[i];
    if (target === this.activeTab) return;

    const targetPanel = this.tabPanels[i];
+   target.setAttribute('aria-selected', 'true');
+   target.setAttribute('tabindex', '0');
    targetPanel.classList.add('is-active');

+   this.activeTab.setAttribute('aria-selected', 'false');
+   this.activeTab.removeAttribute('tabindex');
    this.activeTabPanel.classList.remove('is-active');

    this.activeIndex = i;
    this.activeTab = target;
    this.activeTabPanel = targetPanel;
});
```


### 【非機能要件③】キーボード操作のサポート

さて、こちらが**もっとも大事な非機能要件**と言っても過言ではないかもしれません、キーボード操作のサポートです。
すべてのインタラクティブなUIはマウスがなくても操作可能、つまりキーボードだけで操作可能である必要があります[^2]。タブUIも例外ではありません。
ということで、キーボードだけでタブの切り替えが可能になるように以下の要件を満たしていきます。

1. **方向キー左右でタブの切り替えが可能（垂直タブの場合方向キー上下）**
2. **`Home`, `End`キーで最初・最後のタブパネルを表示する**

アクティブタブにフォーカスがあるとき、方向キー左右でタブを切り替えます。
また、最初（最後）のタブがアクティブのとき左（右）キーを押下すると最後（最初）のタブがアクティブになります。つまりぐるぐるとループする形になります。（垂直タブの場合、方向キー上下で同じ機能）

また、こちらは非機能要件の中でもさらにoptionalな要件ではあるのですが、アクティブタブにフォーカスがあるときに **`Home`, `End`キーを押下すると最初（最後）のタブをアクティブにする**という要件もあります。

これらの機能により、キーボードのみでもタブUIの操作が可能となります。
では実装方法を見ていきましょう。

```js
constructor() {
	// 一部抜粋
	this.maxIndex = tabs.length - 1;
	...
}

/**
 * @param {number} index
 */
#changeActiveTab(index) {
	const targetTab = this.tabs[index];
	// 以下これまでに実装したクリックイベントハンドラと同様
}

#handleClick() {
	this.tabs.forEach((tab, index) => {
		tab.addEventListener('click', () => {
			this.#changeActiveTab(index);
		});
	});
}

#handleKeyPress() {
	for (const tab of this.tabs) {
		tab.addEventListener('keydown', (e) => {
			switch (e.key) {
				case 'ArrowRight': {
					const targetIndex = (() => {
						if (this.activeIndex === this.maxIndex) {
							return 0; // アクティブタブが最後の場合最初のタブを表示
						}
						return this.activeIndex + 1;
					})();
					this.#changeActiveTab(targetIndex);
					break;
				}
				case 'ArrowLeft': {
					const targetIndex = (() => {
						if (this.activeIndex === 0) {
							return this.maxIndex; // アクティブタブが最初の場合最後のタブを表示
						}
						return this.activeIndex - 1;
					})();
					this.#changeActiveTab(targetIndex);
					break;
				}
				case 'Home': {
					const targetIndex = 0;
					this.#changeActiveTab(targetIndex);
					break;
				}
				case 'End': {
					const targetIndex = this.maxIndex;
					this.#changeActiveTab(targetIndex);
					break;
				}
				default:
					return;
			}
		});
	}
}
```

`click`イベントと`keydown`イベントに同じ処理を登録するため、タブ切り替え処理の部分を`changeActiveTab`という関数に切り出しています。
`keydown`イベントのハンドラに渡される`Event`オブジェクトの`key`プロパティを参照し、アクティブタブの切り替えを操作しています。


最後にソースコードの全体を載せておきます。
<details>
<summary>[クリックすると展開]</summary>
<div>

```html:html
<section class="section">
	<div class="tabList" role="tablist">
		<div id="tab-1" class="tabList__tab is-active" role="tab" aria-selected="true" aria-controls="tabpanel-1" tabindex="0">tab-1</div>
		<div id="tab-2" class="tabList__tab" role="tab" aria-selected="false" aria-controls="tabpanel-2">tab-2</div>
		<div id="tab-3" class="tabList__tab" role="tab" aria-selected="false" aria-controls="tabpanel-3">tab-3</div>
	</div>
	<div>
		<div id="tabpanel-1" class="tabPanel is-active" role="tabpanel" aria-labelledby="tab-1" tabindex="0">
			<!-- contents -->
		</div>
		<div id="tabpanel-2" class="tabPanel" role="tabpanel" aria-labelledby="tab-2" tabindex="0">
			<!-- contents -->
		</div>
		<div id="tabpanel-3" class="tabPanel" role="tabpanel" aria-labelledby="tab-3" tabindex="0">
			<!-- contents -->
		</div>
	</div>
</section>
```

```js:javascript
(() => {
	'use strict';
	class Tab {
		/**
		 * @param {NodeListOf<Element>} tabs
		 */
		constructor(tabs) {
			const tabPanels = [...tabs]
				.map((tab) => {
					const id = tab.getAttribute('aria-controls') ?? '';
					return document.getElementById(id);
				})
				.filter((el) => !!el);
			if (tabPanels.length !== tabs.length) return;

			this.tabs = tabs;
			this.tabPanels = tabPanels;
			this.activeIndex = this.tabPanels.findIndex((elm) =>
				elm.classList.contains('is-active')
			);
			this.activeTab = this.tabs[this.activeIndex];
			this.activeTabPanel = this.tabPanels[this.activeIndex];
			this.maxIndex = tabs.length - 1;

			this.#init();
		}

		/**
		 * @param {number} index
		 */
		#changeActiveTab(index) {
			const targetTab = this.tabs[index];
			const targetPanel = this.tabPanels[index];
			if (targetTab === this.activeTab) return;

			targetTab.setAttribute('aria-selected', 'true');
			targetTab.setAttribute('tabindex', '0');
			targetTab.focus();
			targetPanel.classList.add('is-active');

			this.activeTab.setAttribute('aria-selected', 'false');
			this.activeTab.removeAttribute('tabindex');
			this.activeTabPanel.classList.remove('is-active');

			this.activeIndex = index;
			this.activeTab = targetTab;
			this.activeTabPanel = targetPanel;
		}

		#handleClick() {
			this.tabs.forEach((tab, index) => {
				tab.addEventListener('click', () => {
					this.#changeActiveTab(index);
				});
			});
		}

		#handleKeyPress() {
			for (const tab of this.tabs) {
				tab.addEventListener('keydown', (e) => {
					switch (e.key) {
						case 'ArrowRight': {
							const targetIndex = (() => {
								if (this.activeIndex === this.maxIndex) {
									return 0;
								}
								return this.activeIndex + 1;
							})();
							this.#changeActiveTab(targetIndex);
							break;
						}
						case 'ArrowLeft': {
							const targetIndex = (() => {
								if (this.activeIndex === 0) {
									return this.maxIndex;
								}
								return this.activeIndex - 1;
							})();
							this.#changeActiveTab(targetIndex);
							break;
						}
						case 'Home': {
							const targetIndex = 0;
							this.#changeActiveTab(targetIndex);
							break;
						}
						case 'End': {
							const targetIndex = this.maxIndex;
							this.#changeActiveTab(targetIndex);
							break;
						}
						default:
							return;
					}
				});
			}
		}

		#init() {
			this.#handleClick();
			this.#handleKeyPress();
		}
	}

	const tabs = document.querySelectorAll('[role="tab"]');
	new Tab(tabs);
})();

```

スタイルは省略しています。
実装例のスタイルが気になる方はCodePenのソースをご参照ください。

</div>
</details>



以上で非機能要件を意識したタブUIを作成することができました！




## おわりに
冒頭でお伝えしている通り、今回紹介した非機能要件はすべて[WAI-ARIA Authoring Practices 1.2](https://w3c.github.io/aria-practices/#tabpanel)内で紹介されているものです。私が勝手に「こういう要件あるといいヨネ」と言っているわけではありません()
余談ですが、チラリと確認したところ[MUI](https://mui.com/material-ui/react-tabs/)や[Chakra UI](https://chakra-ui.com/docs/components/tabs)などモダンなCSSフレームワークではこれら要件が満たされているようでした。（[React-Bootstrap](https://react-bootstrap.github.io/components/tabs/)は`Home`キーによるタブ切り替えは実装されていなかったです）

WAI-ARIAを除くとキーボード操作周りの要件がほとんどで、スマホファーストな今の時代ではそこまでMUSTな機能とは言えなくなってきているのかもしれない、という所感でした。が、簡単なUIとはいえ色々と意識しなければいけない非機能要件はあるんだよ、という紹介がしたかったがために、今回記事にしてまとめました。

[WAI-ARIA Authoring Practices 1.2](https://w3c.github.io/aria-practices/)ではタブUI以外にも様々なUIの非機能要件をまとめていますので、ぜひUI実装時には参考にしてみてください。
また、本記事で誤った内容がありましたらご指摘いただけますと幸いです。



## 参考

本記事作成にあたり参考にさせていただきました。ありがとうございました。

https://accessible-usable.net/2016/01/entry_160118.html

https://w3c.github.io/aria-practices/#tabpanel

https://mui.com/material-ui/react-tabs/

https://momdo.github.io/wai-aria-1.2/




[^1]: [WAI-ARIAの基本 - ウェブ開発を学ぶ | MDN](https://developer.mozilla.org/ja/docs/Learn/Accessibility/WAI-ARIA_basics)
[^2]: [Web Content Accessibility Guidelines (WCAG) 2.1](https://waic.jp/docs/WCAG21/#keyboard-accessible)
