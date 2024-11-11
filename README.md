# Qiita CLI
## 公式 README
https://github.com/increments/qiita-cli

## プレビュー起動
URL: http://localhost:8888
```shell
npx qiita preview
```

## 画像アップロード
https://qiita.com/settings/uploading_images

## 記事の作成
Qiita Preview 上の「新規記事作成」ボタン、または以下のコマンドで新規記事を作成できます。
```shell
npx qiita new 記事のファイルのベース名
```

## 記事の投稿・更新
### 投稿
Qiita Preview 上の「記事を投稿する」ボタン、または以下のコマンドで投稿・更新ができます。
```shell
npx qiita publish 記事のファイルのベース名
```

以下のコマンドで全ての記事を反映させることができます。
```shell
npx qiita publish --all
```

--forceオプションを用いることで、強制的に記事ファイルの内容を Qiita に反映させます。
```shell
npx qiita publish 記事ファイルのベース名 -f
```

### 更新
記事ファイルを Qiita と同期します。 Qiita 上で更新を行い、手元で変更を行っていない記事ファイルのみ同期されます。

```shell
npx qiita pull
```

--forceオプションを用いることで、強制的に Qiita 上の内容を記事ファイルに反映させます。
```shell
npx qiita pull -f
```