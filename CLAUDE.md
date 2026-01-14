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
