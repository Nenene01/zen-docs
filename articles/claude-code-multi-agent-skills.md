---
title: "Claude Codeでマルチエージェント環境を構築する - Codexとサブエージェントの活用"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "openai", "codex", "ai", "マルチエージェント"]
published: true
---

## はじめに

Claude Codeには「スキル」という拡張機能があり、独自のコマンドを作成できます。本記事では、このスキル機能を活用して**マルチエージェント環境**を構築する方法を解説します。

具体的には以下の2つのスキルを作成し、複数のAIエージェントを連携させます：

1. **`/codex`** - OpenAI Codex CLIを呼び出してセカンドオピニオンを得る
2. **`/ask-agent`** - Claude Codeのサブエージェント（Task tool）を活用する

:::message
本記事は [@owayo](https://zenn.dev/owayo) さんの記事「[Claude CodeのSkills機能で、独自のスラッシュコマンドを作成する](https://zenn.dev/owayo/articles/63d325934ba0de)」を参考にしています。
:::

:::message
**カスタムスラッシュコマンドはスキルにマージされました**

`.claude/commands/review.md` のファイルと `.claude/skills/review/SKILL.md` のスキルは、どちらも `/review` として機能します。既存の `.claude/commands/` ファイルは引き続き動作しますが、スキルの方が以下の追加機能をサポートするため推奨されます：

- サポートファイル用のディレクトリ構成
- フロントマターによる呼び出し制御
- 関連する場合にClaudeが自動的にスキルをロードする機能
:::

## マルチエージェント構成の全体像

```
┌─────────────────────────────────────────────────────┐
│  ユーザー                                            │
└─────────────────┬───────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────────────────┐
│  Claude Code（メインエージェント）                   │
│  - 統括・判断・実装                                  │
│  - ファイル編集・コマンド実行                        │
├─────────────────┬───────────────────────────────────┤
│                 ↓                                   │
│  ┌─────────────────────┐  ┌─────────────────────┐  │
│  │ サブエージェント     │  │ Codex CLI           │  │
│  │ (Anthropic)         │  │ (OpenAI)            │  │
│  │ - 探索・調査        │  │ - レビュー・相談     │  │
│  │ - 計画立案          │  │ - セカンドオピニオン │  │
│  └─────────────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

この構成により、**Anthropic（Claude）とOpenAI（Codex）の両方**を活用した複数観点でのプランニングが可能になります。

## 前提条件

- Claude Codeがインストールされていること
- OpenAI Codex CLIがインストールされていること（`/codex`スキル用）

```bash
# Codex CLIのインストール
npm install -g @openai/codex
```

## スキルの作成

### ディレクトリ構成

スキルは以下のいずれかに配置できます：

| スコープ | 配置場所 | 用途 |
|---------|---------|------|
| グローバル | `~/.claude/skills/スキル名/SKILL.md` | 全プロジェクトで使用 |
| プロジェクト | `.claude/skills/スキル名.md` | 特定プロジェクトのみ |

本記事ではグローバルスキルとして作成します。

```bash
# スキル用ディレクトリを作成
mkdir -p ~/.claude/skills/codex
mkdir -p ~/.claude/skills/ask-agent
```

### 1. Codex相談スキル（`/codex`）

OpenAI Codex CLIを呼び出して、コードや文章についてセカンドオピニオンを得るスキルです。

```bash
touch ~/.claude/skills/codex/SKILL.md
```

以下の内容を記述します：

```markdown:~/.claude/skills/codex/SKILL.md
# Codex相談スキル

OpenAI Codex CLIを使用して、コードや文章についてセカンドオピニオンを得ます。

## 実行方法

ユーザーの依頼内容に応じて、以下のコマンドをBashツールで実行してください：

\`\`\`bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" "<相談内容>"
\`\`\`

## パラメータ説明

| パラメータ | 役割 |
|-----------|------|
| `--full-auto` | 完全自動モード（確認なしで実行） |
| `--sandbox read-only` | 読み取り専用サンドボックス（安全な分析環境） |
| `--cd <dir>` | 対象ディレクトリを指定 |

## 主な用途

### 1. コードレビュー
\`\`\`bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" "このコードをレビューして、改善点を指摘してください: <ファイルパス>"
\`\`\`

### 2. 設計相談
\`\`\`bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" "この設計について意見をください: <設計内容の説明>"
\`\`\`

### 3. バグ調査
\`\`\`bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" "このエラーの原因を調査してください: <エラー内容>"
\`\`\`

### 4. 文章校閲
\`\`\`bash
codex exec --full-auto --sandbox read-only --cd "$(pwd)" "この文章を校閲して改善点を教えてください: <ファイルパス>"
\`\`\`
```

### 2. サブエージェント相談スキル（`/ask-agent`）

Claude CodeのTaskツール（サブエージェント）を使用して、専門的な調査・分析を依頼するスキルです。

```bash
touch ~/.claude/skills/ask-agent/SKILL.md
```

以下の内容を記述します：

```markdown:~/.claude/skills/ask-agent/SKILL.md
# サブエージェント相談スキル

Claude CodeのTaskツール（サブエージェント）を使用して、専門的な調査・分析を依頼します。

## 利用可能なエージェントタイプ

| タイプ | 用途 | 使用場面 |
|--------|------|----------|
| `Explore` | コードベース探索 | ファイル検索、コード理解、構造把握 |
| `Plan` | 計画立案 | 実装戦略、アーキテクチャ設計 |
| `Bash` | コマンド実行 | Git操作、ビルド、ターミナル作業 |
| `general-purpose` | 汎用調査 | 複雑なマルチステップタスク |

## 使用方法

ユーザーの依頼内容に応じて、Taskツールで適切なサブエージェントを起動してください。

### 1. コードベース探索（Explore）

\`\`\`
Taskツールを使用:
- subagent_type: "Explore"
- description: "短い説明（3-5語）"
- prompt: "探索内容の詳細"
\`\`\`

### 2. 計画立案（Plan）

\`\`\`
Taskツールを使用:
- subagent_type: "Plan"
- description: "短い説明（3-5語）"
- prompt: "計画したい内容の詳細"
\`\`\`

### 3. 汎用調査（general-purpose）

\`\`\`
Taskツールを使用:
- subagent_type: "general-purpose"
- description: "短い説明（3-5語）"
- prompt: "調査内容の詳細"
\`\`\`

## 並列実行

複数の独立した調査が必要な場合は、複数のTaskツールを同時に呼び出して並列実行できます。
```

### 3. settings.jsonへの登録

作成したスキルを`~/.claude/settings.json`に登録します。

:::message alert
**グローバルスコープへの追加について**

`~/.claude/settings.json`への登録は**全プロジェクトに影響**します。以下の点を考慮して、自身の判断で適切に行ってください：

- 本当にすべてのプロジェクトで必要なスキルか
- 既存の設定と競合しないか
- チーム開発の場合、他のメンバーへの影響はないか

特定のプロジェクトでのみ使用する場合は、プロジェクトローカル（`.claude/settings.json`）への登録を検討してください。
:::

```json:~/.claude/settings.json
{
  "skills": [
    {
      "path": "/Users/あなたのユーザー名/.claude/skills/codex",
      "triggers": ["codex", "codexと相談", "codexに聞いて", "レビューして"]
    },
    {
      "path": "/Users/あなたのユーザー名/.claude/skills/ask-agent",
      "triggers": ["ask-agent", "エージェントに聞いて", "調査して", "探索して"]
    }
  ]
}
```

## 動作確認

Claude Codeを再起動後、スキルが使えるようになります。

### Codexへの接続テスト

```
❯ /codex 接続テストをお願いします

⏺ Bash(codex exec --full-auto --sandbox read-only --cd "$(pwd)" "こんにちは。接続テストです。")
  ⎿  OpenAI Codex v0.44.0 (research preview)
     workdir: /Users/hoge/project
     model: gpt-5.2-codex
     ...
     こんにちは！Codex CLI上で動作するコーディング支援AIです。

⏺ 接続確認完了です。
  ┌────────────────┬─────────────────────────────────┐
  │      項目      │              状態               │
  ├────────────────┼─────────────────────────────────┤
  │ Codex CLI      │ ✅ インストール済み             │
  │ モデル         │ gpt-5.2-codex                   │
  │ 接続テスト     │ ✅ 正常に応答                   │
  └────────────────┴─────────────────────────────────┘
```

### サブエージェントの動作確認

```
❯ /ask-agent 自己紹介してください

⏺ Task(サブエージェント自己紹介)
  ⎿  Done (0 tool uses · 14.2k tokens · 10s)

⏺ サブエージェントからの自己紹介

  こんにちは。私はClaude Codeのサブエージェント（Task tool経由で呼び出される
  調査・分析担当）です。

  メインのClaude Codeが複雑なタスクや調査が必要な場面で、私を呼び出して作業を
  委譲します。私は独立したスレッドで動作し、調査結果をメインエージェントに
  報告します。
```

## 用途例

### 用途1: 並列プランニング

新機能の実装計画を立てる際、複数の観点から同時に検討できます。

```
❯ 新機能Xの実装計画を立てて

⏺ Claude Code（メイン）
  ├─→ Plan エージェント: 実装戦略を検討      ← 並列実行
  ├─→ Explore エージェント: 既存コード調査   ← 並列実行
  └─→ Codex: 設計のセカンドオピニオン        ← 並列実行
              ↓
      結果を統合して最終計画を提示
```

### 用途2: コードレビューのセカンドオピニオン

Claude Codeで書いたコードをCodexにレビューしてもらう：

```
❯ /codex src/utils/helper.ts をレビューして改善点を教えてください
```

### 用途3: 段階的な調査と実装

1. `/ask-agent` でコードベースを探索
2. 探索結果をもとにClaude Codeで実装
3. `/codex` で実装をレビュー

```
❯ /ask-agent 認証機能がどこに実装されているか調べて
⏺ Explore エージェントが src/auth/ を発見

❯ 見つかった場所に新しい認証方式を追加して
⏺ Claude Codeが実装

❯ /codex 今の変更をセキュリティの観点でレビューして
⏺ Codexがセキュリティレビューを実施
```

### 用途4: 複雑なバグ調査

```
❯ /ask-agent このエラーの原因を調査して：TypeError: Cannot read property 'id' of undefined

⏺ general-purpose エージェントが以下を調査：
  - エラー発生箇所の特定
  - 関連するデータフローの追跡
  - 考えられる原因の列挙
```

## まとめ

| スキル | プロバイダー | 主な用途 |
|--------|-------------|---------|
| `/codex` | OpenAI | レビュー、設計相談、セカンドオピニオン |
| `/ask-agent` | Anthropic | 探索、計画立案、調査 |

Claude Codeのスキル機能を活用することで、**AnthropicとOpenAIの両方のAI**を組み合わせたマルチエージェント環境を構築できます。それぞれの得意分野を活かして、より質の高い開発体験を実現しましょう。

---

## 関連記事

### プロジェクト固有のCodexスキル

本記事ではグローバルスコープの汎用スキルを紹介しましたが、**特定プロジェクトに特化したCodexスキル**を作成する方法は以下の記事で解説しています：

👉 **[記事執筆を加速！Claude CodeとCodexのSkill連携で校閲・画像生成を自動化](https://zenn.dev/nenene01/articles/claude-code-codex-skill)**

Zenn記事執筆に特化した例として、校閲や画像生成プロンプト作成などの具体的なワークフローを紹介しています。

:::message
**スコープの使い分け**

| スコープ | 配置場所 | 本記事 | 関連記事 |
|---------|---------|--------|---------|
| グローバル | `~/.claude/skills/` | ✅ 汎用Codex | - |
| プロジェクト | `.claude/skills/` | - | ✅ Zenn特化Codex |

プロジェクトスコープのスキルは、同名のグローバルスキルをオーバーライドします。
:::

## 参考

- [Claude をスキルで拡張する - 公式ドキュメント](https://code.claude.com/docs/ja/skills)
- [Claude CodeのSkills機能で、独自のスラッシュコマンドを作成する](https://zenn.dev/owayo/articles/63d325934ba0de) - @owayo
- [OpenAI Codex CLI](https://github.com/openai/codex)
