---
title: dialog 要素の罠 - Esc キーを連打すると preventDefault が効かない
tags:
  - HTML
  - JavaScript
  - Web
  - フロントエンド
  - React
private: false
updated_at: '2025-04-18T09:08:43+09:00'
id: 1d90d8bfbd059340de09
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

最近の Web 開発では、モーダル UI のマークアップに `<dialog>` 要素を利用することが主流となってきました。

https://developer.mozilla.org/ja/docs/Web/HTML/Reference/Elements/dialog

- 最小限の JavaScript でモーダルを実現可能
- フォーカス制御や Top layer などのアクセシビリティへの配慮が標準実装されている
- **Esc キーで閉じる機能が標準搭載**（本記事のテーマ）
- Chrome, Firefox, Safari, Edge など、主要なモダンブラウザでサポートされている

この記事では、`<dialog>` 要素に標準搭載されている Esc キーで閉じる機能が、とある条件下で制御できない問題についてまとめます。

## 結論

最初に結論まとめです:

- `<dialog>` 要素の Esc キーによる閉じる操作は `cancel` イベントを発火する
- Chromium では、Esc キーを連打すると2回目以降の `cancel` イベントは `event.cancelable` が `false` であるため `preventDefault()` できない
- 完全にこの挙動をブロックする完璧な方法は現時点では存在しない

## `<dialog>` 要素を使った基本的なモーダルコンポーネントの実装

まず、React を使った典型的な `<dialog>` 要素の実装例を見てみましょう（コアな部分のみ抜粋）：

```tsx:Modal.tsx
type Props = {
  open: boolean;
  onClose: () => void;
  children: React.ReactNode;
}

export const Modal: React.FC<Props> = ({ open, onClose, children }) => {
  const ref = useRef<HTMLDialogElement>(null);

  useEffect(() => {
    const element = ref.current;
    open ? element.showModal() : element.close();
  }, [open]);

  return (
    <dialog ref={ref} onClose={onClose}>
      <button onClick={onClose}>
        x
      </button>
      {children}
    </dialog>
  )
}
```

利用側では以下のように `open` 状態を管理します：

```tsx:Page.tsx
import { Modal } from '@components/Modal';

export const PageComponent = () => {
  const [open, setOpen] = useState(true);

  return (
    <Modal open={open} onClose={() => setOpen(false)}>
      {/* モーダルの中身 */}
    </Modal>
  )
}
```

:::note info
`<dialog>` 要素の `open` 属性に直接 open state を渡すことは推奨されないため、open state を参照して `showModal()` と `close()` メソッドを制御する形式にします。

参考: [HTML Living Standard - The dialog element](https://momdo.github.io/html/interactive-elements.html#the-dialog-element:~:text=open%E5%B1%9E%E6%80%A7%E3%82%92%E5%89%8A%E9%99%A4%E3%81%99%E3%82%8B%E3%81%A8%E3%80%81%E9%80%9A%E5%B8%B8%E3%83%80%E3%82%A4%E3%82%A2%E3%83%AD%E3%82%B0%E3%81%8C%E9%9D%9E%E8%A1%A8%E7%A4%BA%E3%81%AB%E3%81%AA%E3%82%8B%E3%80%82%E3%81%97%E3%81%8B%E3%81%97%E3%80%81%E3%81%9D%E3%81%AE%E3%82%88%E3%81%86%E3%81%AB%E3%81%99%E3%82%8B%E3%81%A8%E3%80%81%E3%81%84%E3%81%8F%E3%81%A4%E3%81%8B%E3%81%AE%E5%A5%87%E5%A6%99%E3%81%AA%E8%BF%BD%E5%8A%A0%E3%81%AE%E7%B5%90%E6%9E%9C%E3%81%A8%E3%81%AA%E3%82%8B%EF%BC%9A)
:::

## 発見した問題点

プロジェクトで「モーダルを閉じる際に『本当に閉じますか？』という確認ダイアログを表示してほしい」という要件があり、対応するため以下のような実装をしました：

```tsx:Page.tsx
export const PageComponent = () => {
  const [open, setOpen] = useState(true);

  return (
    <Modal open={open} onClose={() => {
      const confirm = window.confirm('本当に閉じますか？')
      if (!confirm) return; // 確認ダイアログでキャンセルした場合は閉じない
      setOpen(false)
    }}>
      {/* モーダルの中身 */}
    </Modal>
  )
}
```

`onClose` ハンドラー内で `window.confirm` を呼び出し、その応答に応じて `setOpen(false)` を実行することで要件を満たしています。

しかし動作確認中に次の問題が判明しました：

**「Esc キーを押すと確認ダイアログは表示されるものの、モーダルがそのまま閉じてしまう」**

閉じるボタンをクリックした場合は想定通り確認ダイアログが表示され、ユーザーの選択に応じた挙動となります。しかし、Esc キーを押した場合は確認プロセスをバイパスしてしまうため、要件を満たしていません。

## 原因の究明

調査の結果、**Esc キー押下時には `cancel` イベントが発火し、モーダルが閉じられた後に `close` イベントが発火する** ことがわかりました。

https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/cancel_event

HTML 仕様書では、この動作を以下のように定義しています:

> dialog 要素に対する close request（閉じる要求）が送信されると、まず cancel イベントが発火し、そのイベントが preventDefault() によってキャンセルされない場合、dialog 要素を閉じる処理が行われる

https://momdo.github.io/html/interactive-elements.html#the-dialog-element

技術的な動作フローを HTML 仕様に基づいてより詳細に説明すると以下のようになります：

1. `showModal()` メソッドによって表示された `<dialog>` 要素は、Esc キーによる閉じる操作が可能な状態になる
2. ユーザーが Esc キーを押すと、ブラウザは内部的に `requestClose()` メソッドを呼び出す
3. `requestClose()` メソッドは以下の順序で処理を行う:
   - まず `cancel` イベントを発火する（この時点で `event.cancelable` は `true`）
   - `cancel` イベントが `preventDefault()` によって阻止されなかった場合:
     - dialog 要素の `open` 属性を削除する
     - `close` イベントを発火させる
4. `close` イベントが発火する時点で、dialog は既に閉じられた状態になっている

現在の Modal コンポーネントの実装では:
- 外部で管理している `open` state によって `showModal()` と `close()` メソッドを制御している
- `close` イベントハンドラー内で条件付きで `setOpen(false)` を実行している
- しかし、`cancel` イベント発火後に dialog 要素は自動的に閉じられ、その後 `close` イベントが発火するため、`close` イベントハンドラー内での制御では既に遅い状態になっている

## `preventDefault()` を使った対応

対応は比較的単純です。`cancel` イベントの発火時に `preventDefault()` を適用して閉じる動作を阻止します：

```tsx:Modal.tsx
type Props = {
  open: boolean;
  onClose: () => void;
  onCancel: (e: Event) => void; // cancel イベント用のハンドラーを追加
  children: React.ReactNode;
}

export const Modal: React.FC<Props> = ({ open, onClose, onCancel, children }) => {
  useEffect(() => {
    open ? element.showModal() : element.close();
  }, [open]);

  return (
    <dialog onClose={onClose} onCancel={onCancel}>
      <button onClick={onClose}>x</button>
      {children}
    </dialog>
  )
}
```

利用側では、両方のイベントハンドラーに確認ロジックを実装します：

```tsx:Page.tsx
import { Modal } from '@components/Modal';

export const PageComponent = () => {
  const [open, setOpen] = useState(true);

  return (
    <Modal
      open={open}
      onClose={() => {
        // cancel イベント後に close イベントが発火するためこのままでは連続して window.confirm が呼ばれる
        // 実際には別途条件を管理しておく必要があるが、ここでは省略
        const confirm = window.confirm('本当に閉じますか？')
        if (!confirm) return;
        setOpen(false)
      }}
      onCancel={(e) => {
        e.preventDefault() // Esc キーによる自動閉じを防止
        const confirm = window.confirm('本当に閉じますか？')
        if (!confirm) return;
        setOpen(false)
      }}
    >
      {/* モーダルの中身 */}
    </Modal>
  )
}
```

一見解決したように見えますが、ここで新たな問題が発生しました。

## Chromium の仕様による予期せぬバグ

意気揚々と動作確認していると、意図せぬ挙動が確認されました。
**Esc キーを連打すると 2 回目以降のキー押下でモーダルが閉じてしまう** のです。

この問題は Chromium の issue トラッカーにも報告されており、詳細な議論が行われています：

https://issues.chromium.org/issues/41491338

issue の内容を簡単に要約すると以下のようになります：

1. **当初の問題**：
   - Chromium において、Esc キーを1回押した後すぐに2回目を押すと、`cancel` イベントが発火せず直接 `close` イベントが発火する
   - Firefox などの他のブラウザでは、Esc キーを押すたびに常に `cancel` イベントが発火する（期待される動作）
2. **Google 開発チームの見解**：
   - 当初 Google の開発者はこの挙動を「意図的な動作」として説明
   - 2024年5月のアップデートで一部改善され、「2回目の `cancel` イベントは発火するようになったが、`preventDefault()` を呼んでも効果がない」状態に修正
3. **最新の状況（2025年4月時点）**：
   - 複数のユーザーから「依然として問題が解決していない」「この問題のために `<dialog>` 要素が使用できない」という報告がされている
   - 技術的には、2回目以降の `cancel` イベントは `event.cancelable: false` という状態で発火するため、`preventDefault()` が効かない状況が続いている

ということで 2025年4月現在でも、2回目以降の `cancel` イベントは `event.cancelable: false` で発火するため、`preventDefault()` を実行しても効果がありません。

## 暫定的な対応策

現時点では完全な解決策は存在しませんが、実用的な対応策として以下の方法が考えられます：

```tsx
onCancel={(e) => {
  // cancelable プロパティをチェックし false ならそのまま閉じる
  if (!e.cancelable) {
    setOpen(false);
    return;
  }

  e.preventDefault();
  const confirm = window.confirm('本当に閉じますか？')
  if (!confirm) return;
  setOpen(false);
}}
```
- `event.cancelable` プロパティを確認し、`true` の場合のみ `preventDefault()` を実行する
- `event.cancelable: false` の場合（Esc キー連打時の2回目以降）は、やむを得ず閉じる処理を行う

Esc キー連打時の挙動を完全に制御することはできないため、`cancelable: false` の場合はそのまま閉じる処理を行うしかありません。

## 結論
- `<dialog>` 要素の Esc キーによる閉じる操作は `cancel` イベントを発火する
- Chromium では、Esc キーを連打すると2回目以降の `cancel` イベントは `event.cancelable` が `false` であるため `preventDefault()` できない
- 完全にこの挙動をブロックする完璧な方法は現時点では存在しない

この制限を理解した上で、以下のような設計上の検討が必要です：

- **重要なデータを扱うコンテンツには `<dialog>` 要素を使用しない**：
  Esc キーによる意図しない閉じる操作を完全に防げないため、データ損失が深刻な問題となるような重要な情報やフォームは、別の方法（例：ページ内のフォーム）で実装することを検討する
- **より堅牢な UI パターンを採用する**：
  重要な操作が必要な場合は、モーダルダイアログではなく専用の画面遷移を設計し、ブラウザバックや画面更新などの操作でも安全性を確保する

