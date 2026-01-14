# Zenn記事執筆ルール

## 記事フォーマット

- フロントマターには必ず `title`, `emoji`, `type`, `topics`, `published` を含める
- `type` は `tech`（技術記事）または `idea`（アイデア）
- `topics` は最大5つまで
- `published: false` で下書き保存

## Markdown記法

- Zenn独自記法を活用する:
  - `:::message` でメッセージボックス
  - `:::details タイトル` で折りたたみ
  - `@[youtube](動画ID)` でYouTube埋め込み
  - `@[github](リポジトリURL)` でGitHubリポジトリカード

## コードブロック

- 言語指定を必ず行う（```typescript など）
- ファイル名表示: ```typescript:src/example.ts
- diff表示: ```diff typescript

## 画像

- 画像は `/images/記事スラッグ/` ディレクトリに配置
- 画像パスは `/images/記事スラッグ/image.png` 形式で指定
