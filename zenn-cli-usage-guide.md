# Zenn CLI 使い方ガイド

## 概要

このプロジェクトでは Zenn CLI v0.1.148 を使用して Zenn の記事とブックを管理しています。

## 現在の状況

- ✅ **zenn-cli**: v0.1.148 がインストール済み
- ✅ **articles**: 2 つの記事が存在
  - e71cb001f179bf: "振る舞いに応じて型を分けることで複雑さに対処する"
  - b1c513c32fe15e: "Domain Modeling Made Functional の読書メモ"
- ✅ **books**: 設定済み（まだブックはなし）

## 基本的なコマンド

### 1. 初期化（初回のみ）

```bash
npx zenn init
```

※ このプロジェクトではすでに実行済みです

### 2. プレビュー

```bash
npx zenn preview
```

- ローカルサーバーが起動してブラウザでコンテンツをプレビューできます
- 通常は http://localhost:8000 でアクセス可能
- ファイルの変更をリアルタイムで反映

### 3. 新しい記事の作成

```bash
npx zenn new:article
```

- 自動的にユニークなスラッグが生成されます
- `articles/` ディレクトリに新しい Markdown ファイルが作成されます

### 4. 新しいブックの作成

```bash
npx zenn new:book
```

- 自動的にユニークなスラッグが生成されます
- `books/` ディレクトリに新しい設定ファイルとチャプターが作成されます

### 5. 記事一覧の表示

```bash
npx zenn list:articles
```

現在の記事一覧が表示されます

### 6. ブック一覧の表示

```bash
npx zenn list:books
```

現在のブック一覧が表示されます

### 7. バージョン確認

```bash
npx zenn --version
```

### 8. ヘルプ表示

```bash
npx zenn help
```

## 記事のフロントマター

記事ファイルの先頭に以下のようなフロントマターを記述します：

```markdown
---
title: "記事のタイトル"
emoji: "🦄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [javascript, nodejs]
published: true # 公開状態
publication_name: "organization_name" # 組織での投稿の場合
---
```

### 主要な設定項目

- **title**: 記事のタイトル
- **emoji**: 記事の絵文字
- **type**: `tech`（技術記事）または `idea`（アイデア）
- **topics**: タグの配列（最大 5 つ）
- **published**: `true`で公開、`false`で下書き
- **publication_name**: 組織での投稿の場合に設定

## ワークフロー

### 1. 新しい記事を書く場合

```bash
# 1. 新しい記事を作成
npx zenn new:article

# 2. 作成されたファイルを編集
# articles/[生成されたスラッグ].md を編集

# 3. プレビューで確認
npx zenn preview

# 4. Git で管理
git add .
git commit -m "新しい記事を追加"
git push
```

### 2. 既存の記事を編集する場合

```bash
# 1. プレビューを起動
npx zenn preview

# 2. 記事ファイルを編集
# articles/既存ファイル名.md を編集

# 3. リアルタイムでプレビューを確認

# 4. Git で管理
git add .
git commit -m "記事を更新"
git push
```

### 3. ブックを作成する場合

```bash
# 1. 新しいブックを作成
npx zenn new:book

# 2. 設定ファイルとチャプターを編集
# books/[生成されたスラッグ]/config.yaml
# books/[生成されたスラッグ]/チャプターファイル.md

# 3. プレビューで確認
npx zenn preview
```

## 公開について

- **published: true** にすることで Zenn.dev で公開されます
- Git にプッシュすると自動的に Zenn.dev に反映されます
- 下書き状態（**published: false**）でも Git 管理は可能です

## 注意事項

1. **ファイル名**: 記事ファイル名は自動生成されたスラッグを変更しないことを推奨
2. **画像**: 画像は Zenn.dev にアップロードするか、外部 URL を使用
3. **リンク**: 記事間のリンクは相対パスではなく、Zenn の URL を使用
4. **プレビュー**: 本番環境と若干の差異がある場合があるため、最終確認は Zenn.dev で行う

## トラブルシューティング

### コマンドが見つからない場合

```bash
npm install
```

### プレビューが起動しない場合

- ポート 8000 が使用されていないか確認
- 他の zenn preview プロセスが起動していないか確認

## 参考リンク

- [Zenn CLI 公式ガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Zenn の Markdown 記法](https://zenn.dev/zenn/articles/markdown-guide)
