---
title: "claude-memプラグインの導入と使い方"
emoji: "🧠"
type: "tech"
topics: ["claudecode", "ai", "プラグイン", "開発効率化"]
published: true
---

## claude-mem とは

[claude-mem](https://github.com/thedotmack/claude-mem) は、Claude Codeの会話履歴やメモリを管理するプラグインです。

### 主な機能

- **セッション履歴の永続化** - 過去のセッションを参照可能
- **コンテキストの自動要約** - 重要な情報を圧縮して保持
- **セマンティック検索** - 過去の会話から関連情報を検索
- **プロジェクト単位の管理** - プロジェクトごとにコンテキストを分離

### なぜ必要か

Claude Codeの標準機能では、セッションをクリア（`/clear`）すると会話履歴が失われます。
claude-memを使うと、過去のセッションの内容を自動的に記録・検索できるようになります。

---

## インストール

### 前提条件

- Node.js 18以上
- Claude Code がインストール済み

### インストール手順

```bash
# Claude Code のプラグインとしてインストール
claude plugins install claude-mem
```

または、GitHubから直接インストール：

```bash
# リポジトリをクローン
git clone https://github.com/thedotmack/claude-mem.git

# インストール
cd claude-mem
npm install
npm run build
```

---

## ファイル構成

インストール後、以下のディレクトリ構成になります：

```
~/.local/share/claude-mem/
├── plugin/
│   └── scripts/
│       ├── worker-service.cjs   # ワーカーサービス
│       └── worker-cli.js        # ワーカーCLI
└── data/                        # 保存されたメモリデータ

~/.claude/plugins/marketplaces/thedotmack/
└── package.json                 # プラグインメタデータ
```

---

## ワーカーサービスの起動

claude-memはバックグラウンドワーカーとして動作します。

### 起動

```bash
node ~/.local/share/claude-mem/plugin/scripts/worker-cli.js start
```

### 状態確認

```bash
curl -s http://127.0.0.1:37777/api/readiness
```

正常時のレスポンス：
```json
{"status":"ready","mcpReady":true}
```

### 停止

```bash
node ~/.local/share/claude-mem/plugin/scripts/worker-cli.js stop
```

### デフォルトポート

- **37777** - ワーカーサービスのポート

---

## アーキテクチャ：MCPサーバーを使わない理由

claude-memはMCPサーバーとしても設定可能ですが、**hooksベース**での運用を推奨します。

### MCPサーバー vs Hooks

| 方式 | メリット | デメリット |
|------|----------|------------|
| **MCPサーバー** | 標準的なツール統合 | 設定が複雑、オーバーヘッド大 |
| **Hooks（推奨）** | 軽量、自動実行、設定シンプル | カスタマイズに知識が必要 |

### Hooksベースの仕組み

claude-memは `~/.claude/settings.json` にhooksを設定して動作します：

```json
{
  "hooks": {
    "SessionStart": [...],      // セッション開始時にコンテキスト読み込み
    "UserPromptSubmit": [...],  // ユーザー入力時の処理
    "PostToolUse": [...],       // ツール使用後に観察を記録
    "Stop": [...]               // セッション終了時にサマリー生成
  }
}
```

この方式なら：
- **自動実行** - ユーザーが意識せずにメモリが管理される
- **軽量** - 必要なときだけワーカーと通信
- **シンプル** - MCP設定ファイルが不要

:::message
MCPサーバー設定ファイル（`~/.local/share/claude-mem/.mcp.json`）は削除しても問題ありません。
:::

---

## 使い方

### 基本的な使い方

claude-memをインストールすると、Claude Codeのセッション開始時に自動的にコンテキストが読み込まれます。

```
# セッション開始時（自動）
→ 過去のセッションから関連コンテキストを読み込み

# セッション中
→ 会話内容を自動的に記録

# セッション終了時
→ 重要な情報を自動的に保存
```

### 検索機能の利用方法

claude-memの検索機能を使うには、以下の3つの方法があります：

#### 方法1: カスタムコマンドを作成（推奨）

`~/.claude/commands/mem-search.md` を作成して、スラッシュコマンドとして利用できるようにします：

```bash
mkdir -p ~/.claude/commands
```

```markdown:~/.claude/commands/mem-search.md
# mem-search コマンド

claude-memの過去の記録を検索します。

## 使用方法

検索クエリ: `$ARGUMENTS`

## 実行手順

1. 以下のコマンドで検索APIを呼び出してください:

\`\`\`bash
curl -s "http://127.0.0.1:37777/api/search?query=$ARGUMENTS"
\`\`\`

2. 結果をユーザーに分かりやすく表示してください。
```

作成後、**セッションを再起動**すると `/mem-search "検索語"` で利用できます。

#### 方法2: APIを直接呼び出す

Claude Codeのセッション内で、Bashツールを使ってAPIを直接呼び出せます：

```bash
curl -s "http://127.0.0.1:37777/api/search?query=認証フロー"
```

#### 方法3: Web UIを使う

ブラウザで `http://127.0.0.1:37777/` にアクセスすると、GUIで検索・閲覧できます。

### 主要なAPIエンドポイント

| エンドポイント | 説明 |
|---------------|------|
| `GET /api/search?query=検索語` | 統合検索 |
| `GET /api/observations` | 観察一覧 |
| `GET /api/observation/:id` | 特定の観察を取得 |
| `GET /api/readiness` | ワーカー状態確認 |
| `GET /api/stats` | 統計情報 |

### セマンティック検索の例

```bash
# 認証関連の過去の作業を検索
curl -s "http://127.0.0.1:37777/api/search?query=認証フロー"

# ファイル単位で検索
curl -s "http://127.0.0.1:37777/api/search/by-file?file=auth.ts"

# タイプ別に検索（bugfix, feature, decision など）
curl -s "http://127.0.0.1:37777/api/search/by-type?type=bugfix"
```

---

## 設定

### 設定ファイル

`~/.claude-mem/config.json` で設定をカスタマイズできます：

```json
{
  "port": 37777,
  "autoSave": true,
  "maxHistory": 100,
  "projectIsolation": true
}
```

| 設定 | 説明 | デフォルト |
|------|------|-----------|
| `port` | ワーカーサービスのポート | 37777 |
| `autoSave` | 自動保存の有効化 | true |
| `maxHistory` | 保存する履歴の最大数 | 100 |
| `projectIsolation` | プロジェクト単位でメモリを分離 | true |

---

## トラブルシューティング

### エラー1: 「Worker did not become ready within 15 seconds」

ワーカーサービスが起動していません。

**対処法:**
```bash
# ワーカーを起動
node ~/.local/share/claude-mem/plugin/scripts/worker-cli.js start

# 状態確認
curl -s http://127.0.0.1:37777/api/readiness
```

### エラー2: 「ENOENT: no such file or directory」

プラグインファイルが見つかりません。

**対処法:**
- claude-memを再インストールする
- ファイルパスが正しいか確認する

### エラー3: 「thedotmack/package.json が見つからない」

```
ENOENT: no such file or directory, open '/Users/xxx/.claude/plugins/marketplaces/thedotmack/package.json'
```

claude-memがバージョン確認のために参照するファイルが存在しません。

**対処法:**
```bash
# 不足ディレクトリとpackage.jsonを作成
mkdir -p ~/.claude/plugins/marketplaces/thedotmack
echo '{"name": "claude-mem", "version": "8.5.1"}' > ~/.claude/plugins/marketplaces/thedotmack/package.json

# ワーカーを再起動
node ~/.local/share/claude-mem/plugin/scripts/worker-cli.js start
```

### エラー4: ポートが使用中

```bash
# ポートの使用状況確認
lsof -i :37777

# 既存プロセスを終了
kill <PID>

# ワーカーを再起動
node ~/.local/share/claude-mem/plugin/scripts/worker-cli.js start
```

---

## デバッグ

### プロセスの確認

```bash
# ワーカープロセスの確認
ps aux | grep worker-service

# ポートの使用状況確認
lsof -i :37777
```

### ログの確認

```bash
# ログディレクトリの確認
ls -la ~/.local/share/claude-mem/

# ログファイルがあれば確認
cat ~/.local/share/claude-mem/logs/worker.log
```

### 手動でAPIを叩く

```bash
# ヘルスチェック
curl -s http://127.0.0.1:37777/api/health

# 準備状態確認
curl -s http://127.0.0.1:37777/api/readiness
```

---

## 自動起動の設定

毎回手動でワーカーを起動するのは面倒なので、自動起動を設定します。

### macOS (launchd)

```bash
# plistファイルを作成
cat > ~/Library/LaunchAgents/com.claude-mem.worker.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.claude-mem.worker</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/node</string>
        <string>/Users/YOUR_USERNAME/.local/share/claude-mem/plugin/scripts/worker-cli.js</string>
        <string>start</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
EOF

# YOUR_USERNAME を自分のユーザー名に置換
sed -i '' "s/YOUR_USERNAME/$(whoami)/g" ~/Library/LaunchAgents/com.claude-mem.worker.plist

# サービスを登録・起動
launchctl load ~/Library/LaunchAgents/com.claude-mem.worker.plist
```

### Linux (systemd)

```bash
# サービスファイルを作成
sudo cat > /etc/systemd/user/claude-mem.service << 'EOF'
[Unit]
Description=Claude Mem Worker Service
After=network.target

[Service]
ExecStart=/usr/bin/node /home/YOUR_USERNAME/.local/share/claude-mem/plugin/scripts/worker-cli.js start
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
EOF

# サービスを有効化・起動
systemctl --user enable claude-mem
systemctl --user start claude-mem
```

---

## シェルエイリアスの設定

便利なエイリアスを `.zshrc` や `.bashrc` に追加：

```bash
###################### claude-mem プラグイン ######################
# ワーカーサービス直接起動（bun使用）
alias claude-mem='bun "~/.claude/plugins/marketplaces/thedotmack/plugin/scripts/worker-service.cjs"'

# ワーカーCLI経由の操作
alias cmem-start='node ~/.local/share/claude-mem/plugin/scripts/worker-cli.js start'
alias cmem-stop='node ~/.local/share/claude-mem/plugin/scripts/worker-cli.js stop'
alias cmem-status='curl -s http://127.0.0.1:37777/api/readiness'
alias cmem-ps='ps aux | grep worker-service | grep -v grep'
```

| エイリアス | 用途 |
|-----------|------|
| `claude-mem` | ワーカーサービス直接起動 |
| `cmem-start` | ワーカー起動（CLI経由） |
| `cmem-stop` | ワーカー停止 |
| `cmem-status` | 状態確認 |
| `cmem-ps` | プロセス確認 |

設定後は `source ~/.zshrc` で反映してください。

---

## まとめ

| 項目 | 内容 |
|------|------|
| 用途 | セッション履歴の永続化・検索 |
| 動作方式 | Hooksベース（MCPサーバー不要） |
| ワーカーポート | 37777 |
| 起動コマンド | `node ~/.local/share/claude-mem/plugin/scripts/worker-cli.js start` |
| 状態確認 | `curl -s http://127.0.0.1:37777/api/readiness` |
| 検索API | `curl -s "http://127.0.0.1:37777/api/search?query=検索語"` |
| カスタムコマンド | `~/.claude/commands/mem-search.md` を作成 |

**メリット:**
- 過去のセッションを参照できる
- コンテキストの自動管理（Hooksで自動実行）
- セマンティック検索で関連情報を素早く取得
- MCPサーバー設定不要で軽量に動作

**カスタムコマンドのポイント:**
- `~/.claude/commands/` にMarkdownファイルを作成
- `$ARGUMENTS` で引数を受け取れる
- セッション再起動後に `/コマンド名` で利用可能

**注意点:**
- ワーカーサービスが起動している必要がある
- 初回設定時にエラーが出やすいので、トラブルシューティングを参照
- カスタムコマンドは新しいセッションで反映される

---

## 参考リンク

- [claude-mem GitHub リポジトリ](https://github.com/thedotmack/claude-mem)
- [Claude Code 公式ドキュメント](https://docs.anthropic.com/claude-code)

