---
title: "ghq + GitHub CLI + fzf で快適なリポジトリ管理環境を構築する"
emoji: "📦"
type: "tech"
topics: ["ghq", "githubcli", "git", "fzf", "開発環境"]
published: false
---

## はじめに

複数のGitリポジトリを扱う開発者にとって、リポジトリの管理は重要な課題です。この記事では、以下のツールを組み合わせた効率的なワークフローを紹介します：

- **ghq** - リポジトリを統一的なディレクトリ構造で管理
- **GitHub CLI (gh)** - GitHubとの連携をCLIで完結
- **fzf** - ファジー検索でリポジトリを素早く選択
- **zoxide** - 頻繁にアクセスするディレクトリを記憶

---

## ツールの概要

### ghq とは

[ghq](https://github.com/x-motemen/ghq) は、Gitリポジトリを統一的なディレクトリ構造で管理するツールです。

```
~/develop/
├── github.com/
│   ├── your-username/
│   │   ├── project-a/
│   │   └── project-b/
│   └── other-user/
│       └── awesome-lib/
└── gitlab.com/
    └── company/
        └── internal-tool/
```

### GitHub CLI (gh) とは

[GitHub CLI](https://cli.github.com/) は、GitHub公式のコマンドラインツールです。リポジトリの作成、PR、Issue管理などをターミナルから実行できます。

### なぜこの組み合わせが良いのか

| 従来の方法 | ghq + gh の場合 |
|-----------|----------------|
| `git clone` でバラバラの場所に配置 | 統一されたディレクトリ構造 |
| ブラウザでGitHub操作 | CLIで完結 |
| `cd ~/projects/...` と長いパス入力 | `ffg` でfzf検索→即移動 |

---

## インストール

### macOS (Homebrew)

```bash
# ghq
brew install ghq

# GitHub CLI
brew install gh

# fzf（ファジー検索）
brew install fzf

# zoxide（スマートcd）
brew install zoxide
```

### バージョン確認

```bash
ghq --version   # ghq version 1.7.1
gh --version    # gh version 2.81.0
fzf --version   # 0.x.x
```

---

## 初期設定

### 1. ghq のルートディレクトリ設定

`~/.gitconfig` に以下を追加：

```ini
[ghq]
    root = ~/develop
```

または、コマンドで設定：

```bash
git config --global ghq.root ~/develop
```

:::message
`~/develop` は任意のパスに変更可能です。`~/src` や `~/repos` なども一般的です。
:::

### 2. GitHub CLI の認証

```bash
gh auth login
```

対話形式で設定：
1. `GitHub.com` を選択
2. `HTTPS` を選択
3. 認証方法を選択（ブラウザ認証が簡単）

認証状態の確認：

```bash
gh auth status
```

```
github.com
  ✓ Logged in to github.com account your-username (keyring)
  - Active account: true
  - Git operations protocol: https
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo', 'workflow'
```

### 3. Git の基本設定

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

---

## シェル設定（.zshrc）

以下の設定を `~/.zshrc` に追加します：

### fzf + ghq 連携

```bash
#■■■■■■■■■■■ fzfの設定 ■■■■■■■■■■■■
# プレビューをbatで表示（batがない場合は削除）
export FZF_DEFAULT_OPTS="--preview 'bat --color=always --style=numbers --line-range=:500 {}'"

# ghqで管理しているリポジトリをfzfで選択
alias gfqfzf='ghq list --full-path | fzf --reverse'

# 選択したリポジトリに移動
alias ffg='cd $(gfqfzf)'

# 従来のpecoユーザー向けエイリアス
alias pecog='cd $(gfqfzf)'
```

### Git ブランチ選択

```bash
# fzfでブランチを選択してcheckout
alias git-fbranch='git checkout "$(git branch -a | grep -v HEAD | sed "s/^[* ] //" | fzf | sed "s#remotes/[^/]*/##")"'
```

### zoxide（スマートcd）

```bash
# zoxideを有効化（cdコマンドを拡張）
eval "$(zoxide init zsh --cmd cd)"
```

### 便利なGitエイリアス

```bash
# gitプロジェクトのルートディレクトリに移動
alias groot='cd $(git rev-parse --show-toplevel)'
```

設定後は以下で反映：

```bash
source ~/.zshrc
```

---

## 基本的なワークフロー

### リポジトリをクローン

```bash
# ghqでクローン（自動的に適切なディレクトリに配置）
ghq get https://github.com/username/repository

# 短縮形（GitHubの場合）
ghq get username/repository
```

### リポジトリに移動

```bash
# fzfで検索して移動
ffg

# または
cd $(ghq list --full-path | fzf)
```

### リポジトリ一覧表示

```bash
# 全リポジトリをリスト表示
ghq list

# フルパスで表示
ghq list --full-path
```

### 新規リポジトリ作成

```bash
# ディレクトリを作成
mkdir -p ~/develop/github.com/your-username/new-project
cd ~/develop/github.com/your-username/new-project

# Gitリポジトリ初期化
git init

# GitHubにリポジトリ作成 + リモート設定
gh repo create new-project --private --source=. --push
```

---

## GitHub CLI の便利な使い方

### リポジトリ操作

```bash
# リポジトリ情報を表示
gh repo view

# ブラウザでリポジトリを開く
gh repo view --web

# リポジトリをクローン（ghqと組み合わせる場合は ghq get を推奨）
gh repo clone username/repository
```

### Pull Request

```bash
# PR一覧
gh pr list

# PRを作成
gh pr create --title "タイトル" --body "説明"

# PRをチェックアウト
gh pr checkout 123

# PRをマージ
gh pr merge 123
```

### Issue

```bash
# Issue一覧
gh issue list

# Issueを作成
gh issue create --title "バグ報告" --body "詳細"

# Issueを閲覧
gh issue view 123
```

### ワークフロー（GitHub Actions）

```bash
# ワークフロー一覧
gh workflow list

# ワークフロー実行
gh workflow run build.yml

# 実行履歴
gh run list
```

---

## 実践的なワークフロー例

### 1. 新しいプロジェクトを始める

```bash
# 1. ディレクトリ作成
mkdir -p ~/develop/github.com/your-username/my-new-app
cd ~/develop/github.com/your-username/my-new-app

# 2. プロジェクト初期化（例：Node.js）
npm init -y
git init
echo "node_modules/" > .gitignore

# 3. 初回コミット
git add .
git commit -m "Initial commit"

# 4. GitHubにリポジトリ作成 & プッシュ
gh repo create my-new-app --private --source=. --push
```

### 2. 既存リポジトリで作業開始

```bash
# 1. fzfでリポジトリを選択して移動
ffg
# → fzfで "my-app" と入力して選択

# 2. 最新を取得
git pull

# 3. 作業ブランチ作成
git checkout -b feature/new-feature

# 4. 作業...

# 5. コミット & プッシュ
git add .
git commit -m "Add new feature"
git push -u origin feature/new-feature

# 6. PRを作成
gh pr create --fill
```

### 3. 他人のリポジトリにコントリビュート

```bash
# 1. フォーク & クローン
gh repo fork username/awesome-project --clone

# 2. 変更を加える
cd awesome-project
git checkout -b fix/typo
# 編集...

# 3. コミット & プッシュ
git add .
git commit -m "Fix typo in README"
git push -u origin fix/typo

# 4. PRを作成（元リポジトリへ）
gh pr create --fill
```

---

## ディレクトリ構造の例

実際の運用では以下のような構造になります：

```
~/develop/
├── github.com/
│   ├── your-username/          # 自分のリポジトリ
│   │   ├── my-app/
│   │   ├── dotfiles/
│   │   └── blog/
│   ├── facebook/               # 他ユーザー/組織
│   │   └── react/
│   └── microsoft/
│       └── vscode/
├── gitlab.com/                 # 別のホスティング
│   └── company/
│       └── internal-tool/
└── bitbucket.org/
    └── team/
        └── legacy-project/
```

---

## トラブルシューティング

### ghq get でエラーが出る

```bash
# SSH設定の確認
ssh -T git@github.com

# HTTPS認証の確認
gh auth status
```

### GitHub CLIの認証エラー

```bash
# 再認証
gh auth logout
gh auth login
```

### fzfでプレビューが表示されない

```bash
# batがインストールされているか確認
which bat

# なければインストール
brew install bat
```

---

## まとめ

| ツール | 役割 |
|--------|------|
| **ghq** | リポジトリを統一的なディレクトリ構造で管理 |
| **gh** | GitHub操作をCLIで完結 |
| **fzf** | ファジー検索でリポジトリを素早く選択 |
| **zoxide** | 頻繁にアクセスするディレクトリを記憶 |

**便利なエイリアス一覧:**

| エイリアス | 説明 |
|-----------|------|
| `ffg` | fzfでリポジトリ選択→移動 |
| `git-fbranch` | fzfでブランチ選択→checkout |
| `groot` | Gitルートディレクトリに移動 |

この組み合わせにより：
- リポジトリが散らばらない
- 素早くプロジェクト間を移動できる
- GitHub操作がターミナルで完結する

ぜひ試してみてください。

---

## 参考リンク

- [ghq GitHub リポジトリ](https://github.com/x-motemen/ghq)
- [GitHub CLI 公式ドキュメント](https://cli.github.com/manual/)
- [fzf GitHub リポジトリ](https://github.com/junegunn/fzf)
- [zoxide GitHub リポジトリ](https://github.com/ajeetdsouza/zoxide)
