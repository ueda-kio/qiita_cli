---
title: Slack APIで「スレッドにリアクションした/していない人を確認するスラッシュコマンド」作った
tags:
  - GAS
  - TypeScript
  - Slack
  - clasp
  - slack-api
private: false
updated_at: '2023-12-12T16:31:26+09:00'
id: 2bd4062579e40537cc4c
organization_url_name: null
slide: false
ignorePublish: false
---
## 作ったもの

Slack で「完了したらリアクションして」という連絡事項系のスレッドがあります。

![連絡事項スレッド（サンプル）](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/0f44b6ee-f075-b1f7-0435-2f35b091c845.png)

現時点で誰がリアクションした/していないのかを確認したいという場合、以下の構文でスラッシュコマンドを実行します。コマンドは任意に設定可能です。

```
/checker {スレッドのurl}
```

チャンネル内のメンバーのうち、スレッドにリアクションした/していないメンバーの一覧を表示します。[エフェメラルメッセージ](https://api.slack.com/messaging/managing#ephemeral) で送信されるため、メンバーにメンションが飛ぶことはありません。

![image.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/a284f7b2-2200-5f95-c971-aeda17cdb637.jpeg)


スラッシュコマンドの第2引数に絵文字を指定することで、その絵文字のリアクションをつけた/つけていないメンバーに絞り込むこともできます。

```
/checker {スレッドのurl} {:絵文字:}
```

![image.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/c02f82bf-a0ed-7b5a-9bb3-3c203eb7a473.jpeg)

## 実装内容

### 環境

スラッシュコマンドで Google Apps Script（以降 GAS）を呼び出して Slack API を実行しています。
GAS は [clasp](https://github.com/google/clasp) および TypeScript を使ってローカルでコーディングしています。

https://github.com/google/clasp

環境構築については以下の記事を参考にしてください。

https://qiita.com/anti-digital/items/9db9ee5ed2e1bd7c73c2

### ソースコード

コンパイル前の TypeScript で掲載します。
各ユーザーIDや Slack へ送信するメッセージは適宜変更してください。

<details>
<summary>ソースコード全文</summary>
<div>

```ts:main.ts
const SLACK_API_TOKEN = PropertiesService.getScriptProperties().getProperty('SLACK_API_TOKEN');
if (SLACK_API_TOKEN === null) {
  Logger.log('SLACK_API_TOKEN is null.');
}

function doPost(e: GoogleAppsScript.Events.DoPost) {
  if (SLACK_API_TOKEN === null) {
    return ContentService.createTextOutput(
      'Error: SlackのAPIトークンが設定されていません。お手数ですが、実装者（<@user_id>）へご連絡お願いします。'
    );
  }

  const paramText = e.parameter.text;
  if (!paramText) return ContentService.createTextOutput('チェックしたいスレッドのURLを引数で渡してください。');

  const { threadUrl, argStamp } = extractUrlAndStampFromParameter(paramText);
  if (argStamp === false) {
    return ContentService.createTextOutput('第1引数にスレッドのurl、第2引数にスタンプを指定してください！スタンプは指定しなくても構いません。');
  }

  const threadData = getThreadData(threadUrl);
  if (!threadData) return ContentService.createTextOutput('Error: スレッドデータの取得に失敗しました。');

  const reactedUserIds = getReactedUserIds(threadData, argStamp);
  if (!reactedUserIds) return ContentService.createTextOutput('Error: スタンプを押したユーザー一覧の取得に失敗しました。');

  const usersList = getUserListOfReactedOrNot(threadData.channelId, reactedUserIds);
  if (!usersList) return ContentService.createTextOutput('Error: ユーザー一覧の取得に失敗しました。');

  const { reactedUsers, nonReactedUsers } = usersList;
  return ContentService.createTextOutput(
    `【${argStamp || 'スタンプ'}つけた人】\n${returnLogText(reactedUsers)}\n\n【${argStamp || 'スタンプ'}つけてない人】\n${returnLogText(nonReactedUsers)}`
  );
}

function extractUrlAndStampFromParameter(parameter: string) {
  const spaceRegExp = /\s+|　+/;
  if (!spaceRegExp.test(parameter)) return { threadUrl: parameter, argStamp: '' };

  const splitParameter = parameter.split(spaceRegExp);
  const threadUrl = splitParameter[0];
  const argStamp = splitParameter[1];
  return /^:(.+):$/.test(argStamp) ? { threadUrl, argStamp } : { threadUrl, argStamp: false as const };
}

function getThreadData(threadUrl: string) {
  try {
    const messages = UrlFetchApp.fetch('https://slack.com/api/search.messages', {
      headers: { Authorization: `Bearer ${SLACK_API_TOKEN}` },
      payload: {
        query: threadUrl,
      },
    });

    const messagesBody = JSON.parse(messages.getContentText());
    const channelId = messagesBody.messages.matches[0].channel.id;
    const ts = messagesBody.messages.matches[0].ts;

    return typeof channelId === 'string' && typeof ts === 'string' ? { channelId, ts } : false;
  } catch (e) {
    return false;
  }
}

function getReactedUserIds({ channelId, ts }: { channelId: string; ts: string }, argStamp: string) {
  try {
    const replies = UrlFetchApp.fetch('https://slack.com/api/conversations.replies', {
      headers: { Authorization: `Bearer ${SLACK_API_TOKEN}` },
      payload: {
        channel: channelId,
        ts,
      },
    });

    const repliesBody = JSON.parse(replies.getContentText());
    const reactions: {
      name: string;
      users: string[];
    }[] = repliesBody.messages[0].reactions;
    if (typeof reactions === 'undefined') return []; // スタンプを押したユーザーがいない場合

    // 引数にスタンプを指定したかどうか
    if (argStamp) {
      const reactionUserIds = reactions.find((reaction) => reaction.name === argStamp.replaceAll(':', ''))?.users ?? [];
      return reactionUserIds;
    } else {
      const _reactionUserIds = reactions.flatMap((reaction) => reaction.users);
      const reactionUserIds = Array.from(new Set(_reactionUserIds)); // 重複削除（いずれか1つ以上のスタンプを押しているか）

      return reactionUserIds;
    }
  } catch (e) {
    return false;
  }
}

function getUserListOfReactedOrNot(channelId: string, reactedUserIds: string[]) {
  // - あらかじめ排除しておきたいユーザーのIDを配列に格納する。（Pollyやbotなど）
  const excludeIds = ['xxx'];

  try {
    const allChannelMembers = UrlFetchApp.fetch('https://slack.com/api/conversations.members', {
      headers: { Authorization: `Bearer ${SLACK_API_TOKEN}` },
      payload: {
        channel: channelId,
      },
    });

    const channelMemberIds = JSON.parse(allChannelMembers.getContentText()).members as string[];
    const reactedUsers: string[] = [];
    const nonReactedUsers: string[] = [];
    for (const id of channelMemberIds) {
      if (excludeIds.includes(id)) continue;
      reactedUserIds.includes(id) ? reactedUsers.push(`<@${id}>`) : nonReactedUsers.push(`<@${id}>`);
    }

    return { reactedUsers, nonReactedUsers };
  } catch (e) {
    return false;
  }
}

function returnLogText(usersList: string[]) {
  return usersList.length === 0 ? 'いません！' : usersList.join('\n');
}
```

</div>
</details>


### Slack App の導入方法

本機能のために必要なパーミッションのスコープは以下の通りです。
- `channels:history`
- `channels:read`
- `groups:history`
- `groups:read`
- `mpim:history`
- `mpim:read`
- `search:read`

詳細な導入方法の手順は他の記事や公式にお任せしますので、ここでは簡単にまとめるのみとします。

https://api.slack.com/start/quickstart

<details>
<summary>導入方法</summary>
<div>

#### GAS でやること

##### 1. コーディング

[GAS の home](https://script.google.com/home) から新規プロジェクトを作成、コーディングします。
clasp を利用している場合、ローカルのエディタでコーディングを行い `clasp push` でプロジェクトに反映します。

##### 2. デプロイ
「ウェブアプリ」としてデプロイし、URLをコピーしておきます。
![image.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/cc7502e3-5e38-e3d2-3f82-a1bf8ce0c8ba.jpeg)
![image.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/b40a971a-45ac-7e6d-bcfc-62710ae2f86d.jpeg)

GAS 側でのやることは以上です。


#### Slack App でやること
##### 1. App の作成

[Your Apps](https://api.slack.com/apps) ページの「Create New App」からアプリを新規作成します。
「From scratch」から任意のアプリ名とインストールしたいワークスペースを選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/c4950b57-d813-6b21-3266-4ae871ddf62b.png)


##### 2. パーミッションの設定

「Features > OAuth & Permissions」画面の「Scopes > User Token Scopes」からパーミッションのスコープを設定します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/fe26eed8-4019-91bb-68e0-68c47dbd3bea.png)

必要なスコープは前述の通りです。
- `channels:history`
- `channels:read`
- `groups:history`
- `groups:read`
- `mpim:history`
- `mpim:read`
- `search:read`

##### 3. スラッシュコマンドの作成

「Features > Slash Commands」画面の「Create New Command」からスラッシュコマンドを作成します。

![スクリーンショット 2023-12-06 12.04.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/395cedbe-3b3d-a689-de21-bc24b24ecb8d.png)

必要な情報を設定します。
- 「Command」に任意のコマンドを設定。ユニークであると望ましいです。
- 「Request URL」に GAS でデプロイしたURLを貼り付けます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/48c7ff14-6ae5-a0aa-c276-2e4acc4652e7.png)

##### 4. Slack のワークスペースにインストール

「Settings > Install App」画面の「Install to Workspace」からワークスペースにアプリをインストールします。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/7e22e085-0a37-b0e5-66df-1d5f4446b50e.png)

これで Slack 内でスラッシュコマンドを呼び出せるようになります。

</div>
</details>

### コード解説

以下はコードの解説となります。ご興味のある方のみご覧ください！

#### 0. APIトークンを GAS の環境変数に設定する

<details>
<summary>GAS の環境変数の設定手順</summary>
<div>

slack api ページの Install App 画面からAPIトークンをコピー。
![image.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/9b684453-a08a-a865-6f72-7509257bfcd1.jpeg)

GAS のプロジェクト設定から スクリプトプロパティ を設定。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/ee95e3ef-e803-f746-15ff-fcf17a9b51c3.png)

![image.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1157551/f105098f-0564-6078-dcda-a95229922d5b.jpeg)

</div>
</details>

`PropertiesService.getScriptProperties().getProperty('SLACK_API_TOKEN');`とすることでコードから環境変数へアクセスできます。

https://developers.google.com/apps-script/reference/properties/properties?hl=ja#getProperties()

#### 1. スラッシュコマンドの引数からスレッドのurlと絵文字を取得する

`e.parameter.text` から引数を取得し、第2引数の有無とそれが絵文字 `:xxx:` であるかどうかを確認する。

```ts
function extractUrlAndStampFromParameter(parameter: string) {
  const spaceRegExp = /\s+|　+/;
  if (!spaceRegExp.test(parameter)) return { threadUrl: parameter, argStamp: '' };

  const splitParameter = parameter.split(spaceRegExp);
  const threadUrl = splitParameter[0];
  const argStamp = splitParameter[1];
  return /^:(.+):$/.test(argStamp) ? { threadUrl, argStamp } : { threadUrl, argStamp: false as const };
}

// 呼び出し側
const paramText = e.parameter.text;
if (!paramText) return ContentService.createTextOutput('チェックしたいスレッドのURLを引数で渡してください。');

const { threadUrl, argStamp } = extractUrlAndStampFromParameter(paramText);
if (argStamp === false) {
  return ContentService.createTextOutput('第1引数にスレッドのurl、第2引数にスタンプを指定してください！スタンプは指定しなくても構いません。');
}
```

- スラッシュコマンドの引数は `doPost` 関数の引数 `e: GoogleAppsScript.Events.DoPost` のプロパティ `e.parameter.text` から取得可能。
- 引数に半角/全角スペースがあるかどうか（`/\s+|　+/.text()`）で第2引数の有無を確認。
- 第2引数が引数が指定されている場合はそれが絵文字かどうかチェック（`/^:(.+):$/.test()`）する。

#### 2. リアクションをしたユーザーID一覧を取得する

[`search.messages`](https://api.slack.com/methods/search.messages) メソッドからスレッドの `channelId` と `ts`（タイムスタンプ）を取得し、[`conversations.replies`](https://api.slack.com/methods/conversations.replies) メソッドからリアクションをしたメンバー一覧を取得する。

```ts
function getThreadData(threadUrl: string) {
  try {
    const messages = UrlFetchApp.fetch('https://slack.com/api/search.messages', {
      headers: { Authorization: `Bearer ${SLACK_API_TOKEN}` },
      payload: {
        query: threadUrl,
      },
    });

    const messagesBody = JSON.parse(messages.getContentText());
    const channelId = messagesBody.messages.matches[0].channel.id;
    const ts = messagesBody.messages.matches[0].ts;

    return typeof channelId === 'string' && typeof ts === 'string' ? { channelId, ts } : false;
  } catch (e) {
    return false;
  }
}
```

- 「特定のスレッドにリアクションしたユーザー」を取得できるメソッド [`conversations.replies`](https://api.slack.com/methods/conversations.replies) を実行するために、スレッドurlの `channelId` と `ts` が必要なため、 [`search.messages`](https://api.slack.com/methods/search.messages) メソッドからそれらを取得する。`payload` の `query` にスレッドurlを指定することでスレッド情報を取得可能。

```ts
function getReactedUserIds({ channelId, ts }: { channelId: string; ts: string }, argStamp: string) {
  try {
    const replies = UrlFetchApp.fetch('https://slack.com/api/conversations.replies', {
      headers: { Authorization: `Bearer ${SLACK_API_TOKEN}` },
      payload: {
        channel: channelId,
        ts,
      },
    });

    const repliesBody = JSON.parse(replies.getContentText());
    const reactions: {
      name: string;
      users: string[];
    }[] = repliesBody.messages[0].reactions;
    if (typeof reactions === 'undefined') return []; // スタンプを押したユーザーがいない場合

    // 引数にスタンプを指定したかどうか
    if (argStamp) {
      const reactionUserIds = reactions.find((reaction) => reaction.name === argStamp.replaceAll(':', ''))?.users ?? [];
      return reactionUserIds;
    } else {
      const _reactionUserIds = reactions.flatMap((reaction) => reaction.users);
      const reactionUserIds = Array.from(new Set(_reactionUserIds)); // 重複削除（いずれか1つ以上のスタンプを押しているか）

      return reactionUserIds;
    }
  } catch (e) {
    return false;
  }
}
```

- [`conversations.replies`](https://api.slack.com/methods/conversations.replies) メソッドの返り値`messages.reactions`から、リアクションの絵文字とユーザーのIDを取得。
- スラッシュコマンドの第2引数に絵文字が指定されていた場合、`messages.reactions.name` から一致するリアクションを絞り込み、`reactions.users` からユーザーIDの一覧を取得。
- スラッシュコマンドの第2引数に絵文字が指定されていない場合、なんらかのリアクションをつけたユーザーを一覧で取得。重複は削除する（`Array.from(new Set(_reactionUserIds))`）。

#### 3. チャンネルのメンバー一覧を取得し、リアクションをした/していないメンバーを分類する

[`conversations.members`](https://api.slack.com/methods/conversations.members) メソッドからチャンネルのメンバー一覧を取得し、リアクションをした/していないメンバーを分類する

```ts
function getUserListOfReactedOrNot(channelId: string, reactedUserIds: string[]) {
  // - あらかじめ排除しておきたいユーザーのIDを配列に格納する。（Pollyやbotなど）
  const excludeIds = ['xxx'];

  try {
    const allChannelMembers = UrlFetchApp.fetch('https://slack.com/api/conversations.members', {
      headers: { Authorization: `Bearer ${SLACK_API_TOKEN}` },
      payload: {
        channel: channelId,
      },
    });

    const channelMemberIds = JSON.parse(allChannelMembers.getContentText()).members as string[];
    const reactedUsers: string[] = [];
    const nonReactedUsers: string[] = [];
    for (const id of channelMemberIds) {
      if (excludeIds.includes(id)) continue;
      reactedUserIds.includes(id) ? reactedUsers.push(`<@${id}>`) : nonReactedUsers.push(`<@${id}>`);
    }

    return { reactedUsers, nonReactedUsers };
  } catch (e) {
    return false;
  }
}
```

- [`conversations.members`](https://api.slack.com/methods/conversations.members) メソッドからチャンネルのメンバー一覧を取得、リアクションした/していないユーザーそれぞれで配列に格納する。
- ユーザーIDを `<@userId>` の形式の文字列で管理することで、Slackに送信した際にメンション扱いになり「ユーザーID → ユーザー名」の変換が自動で行われるようになる。前述の通りエフェメラルメッセージで送信されるため実際にメンバーにメンションが飛ぶことはない。

#### 4. Slack にメンバー一覧を送信する
GAS で Slack にメンバー一覧を [`ContentService.createTextOutput`](https://developers.google.com/apps-script/reference/content/content-service?hl=ja#createTextOutput(String)) メソッドで送信する。

```ts
  const { reactedUsers, nonReactedUsers } = usersList;
  return ContentService.createTextOutput(
    `【${argStamp || 'スタンプ'}つけた人】\n${returnLogText(reactedUsers)}\n\n【${argStamp || 'スタンプ'}つけてない人】\n${returnLogText(nonReactedUsers)}`
  );
```

### 所感
今回初めて GAS を書いたため、諸々ベストプラクティスではない実装となっているかもしれません。改善の余地ありです。
- エラー時に Slack にテキストを送信するようにしているが、これがベストなのか？
- 一度の処理で API と合計で3度もやりとりしているが、もっと通信を削減する方法はないか？
- スラッシュコマンドの引数からスレッドurlと絵文字を切り離す処理を力技で行っているが、よりよい方法はあるのか？
- ハンドリングできていないエラーパターンがあるか？

## おわりに
リアクションを収集する必要があるスレッドで目視確認がつらいと感じている方はぜひ導入検討してみてください！
スラッシュコマンドはワークスペース内のメンバーなら誰でも使えるので、きっと誰かから感謝されるかもしれません。きっと。
