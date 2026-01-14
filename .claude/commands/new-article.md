# 新規記事作成

新しいZenn記事を作成します。

## 手順

1. ユーザーに記事のトピックを確認
2. 適切なスラッグを生成（英数字とハイフン）
3. `npx zenn new:article --slug {slug}` で記事ファイルを作成
4. フロントマターを設定
5. 記事の骨子を作成

## 使用例

```bash
npx zenn new:article --slug my-first-article --title "はじめての記事" --type tech
```
