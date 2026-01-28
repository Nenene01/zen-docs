# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Zenn記事・本を管理するコンテンツリポジトリ。Zenn CLIを使用してMarkdownファイルを管理し、GitHubリポジトリ連携でZennに自動デプロイされる。

## Commands

```bash
npm run preview       # ローカルプレビュー (http://localhost:8000)
npm run new:article   # 新規記事作成
npm run new:book      # 新規本作成
```

記事作成時のオプション指定:
```bash
npx zenn new:article --slug スラッグ名 --title "タイトル" --type tech
```

## Content Structure

- `articles/` - 記事ファイル（`*.md`）
- `books/` - 本（ディレクトリ単位、各章が`*.md`）

## Article Frontmatter

```yaml
---
title: "記事タイトル"
emoji: "😸"
type: "tech"  # tech または idea
topics: ["topic1", "topic2"]
published: true  # false で下書き
---
```

## Available Skills

### `/codex` - Codex連携スキル

OpenAI Codex CLIと連携して、記事の品質向上をサポートします。

**主な用途:**
- 記事の校閲（誤字脱字、文法、技術的正確性）
- 文章改善の提案
- 画像生成プロンプト作成（DALL-E用）
- 記事構成のレビュー

**使用例:**
```
/codex この記事を校閲して
/codex 画像生成プロンプトを作って
```

記事作成後は `/codex` で校閲を依頼し、品質向上に活用してください。

## Article Creation Rules

### クレジット表記

Claude Codeで作成・編集した記事には、記事末尾に以下のクレジットを追加してください：

```markdown
---

*この記事は [Claude Code](https://claude.ai/code) を使用して作成されました。*
```

校閲にCodexを使用した場合は以下のように記載：

```markdown
---

*この記事は [Claude Code](https://claude.ai/code) を使用して作成され、[OpenAI Codex CLI](https://github.com/openai/codex) による校閲を経ています。*
```
