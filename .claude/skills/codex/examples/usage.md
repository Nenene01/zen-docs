# Codexスキル使用例

## 1. 記事の校閲

```bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" \
  "articles/記事名.md を校閲して、誤字脱字、文法の誤り、技術的な不正確さを指摘してください"
```

**ユーザーの入力例:**
- 「この記事を校閲して」
- 「文章をチェックして」
- 「誤字脱字がないか確認して」

## 2. 文章の改善提案

```bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" \
  "articles/記事名.md の読みやすさを向上させる改善案を提案してください"
```

**ユーザーの入力例:**
- 「もっと読みやすくして」
- 「文章を改善して」
- 「表現を見直して」

## 3. 画像生成プロンプト作成

```bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" \
  "articles/記事名.md の内容を図解するための画像生成プロンプト（DALL-E用）を作成してください"
```

**ユーザーの入力例:**
- 「画像を作りたい」
- 「図解用のプロンプトを作って」
- 「アイキャッチ画像のアイデアが欲しい」

## 4. 記事構成のレビュー

```bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" \
  "articles/記事名.md の構成を分析して、読者にとってより分かりやすい構成を提案してください"
```

**ユーザーの入力例:**
- 「構成を見直したい」
- 「章立てを改善して」
- 「流れを確認して」

## 5. セカンドオピニオン

```bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" \
  "articles/記事名.md の内容について、技術的な観点からセカンドオピニオンをください"
```

**ユーザーの入力例:**
- 「別の視点から見て」
- 「レビューして」
- 「意見を聞かせて」
