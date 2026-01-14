# zen-docs

Zenn記事・本を管理するリポジトリです。

## セットアップ

```bash
npm install
```

## コマンド

| コマンド | 説明 |
|---------|------|
| `npm run preview` | ローカルプレビューサーバーを起動 (http://localhost:8000) |
| `npm run new:article` | 新しい記事を作成 |
| `npm run new:book` | 新しい本を作成 |

## ディレクトリ構成

```
zen-docs/
├── articles/       # 記事（.md ファイル）
└── books/          # 本（ディレクトリ単位で管理）
```

## 記事の作成

```bash
npm run new:article
# または
npx zenn new:article --slug 記事のスラッグ --title "記事タイトル" --type tech
```

### 記事のフロントマター

```yaml
---
title: "記事タイトル"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["topic1", "topic2"]
published: true
---
```

## 本の作成

```bash
npm run new:book
# または
npx zenn new:book --slug 本のスラッグ
```

## プレビュー

```bash
npm run preview
```

ブラウザで http://localhost:8000 にアクセスしてプレビューを確認できます。

## デプロイ

このリポジトリをZennと連携することで、mainブランチへのpush時に自動的に記事が公開されます。

1. [Zenn デプロイ設定](https://zenn.dev/dashboard/deploys)にアクセス
2. GitHubリポジトリ連携で `Nenene01/zen-docs` を選択

## 参考リンク

- [Zenn CLIの使い方](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Zenn記事の書き方](https://zenn.dev/zenn/articles/markdown-guide)
