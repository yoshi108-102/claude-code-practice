# Skills 詳解 — 再利用可能な知識とワークフロー

> 出典: [Extend Claude with skills](https://code.claude.com/docs/en/skills)（閲覧日 2026-06-12）
> このノートは公式ドキュメントの「skills」を起点に、Claudeが自動生成した教材です。

## 概要

Skills は Claude の能力を拡張する。`SKILL.md` ファイルに指示を書くと、Claude のツールキットに追加される。Claude は関連する Skill を自動的に使うか、`/skill-name` で直接呼び出せる。CLAUDE.md が"いつも知っておくべきこと"なら、Skill は"必要なときだけ読み込む詳細コンテンツ"。Skill の本文はそれが使われるまでコンテキストを消費しない。

Custom commands と Skills は統合済み。`.claude/commands/deploy.md` と `.claude/skills/deploy/SKILL.md` は同じ `/deploy` を生成し、動作は同じ。Skills は追加機能（ディレクトリ構造・フロントマター・自動ロード）が使える。

Claude Code の Skills は [Agent Skills](https://agentskills.io) オープンスタンダードに準拠し、他の AI ツールとの互換性も持つ。

## 公式 docs に沿った解説

### バンドル済み Skills

Claude Code には最初から以下の Skills が含まれる（`disableBundledSkills` 設定で無効化可能）:

| Skill | 目的 |
|---|---|
| `/run` | アプリを起動して変更が動作することを確認 |
| `/verify` | コード変更をテスト/型チェックなしで確認 |
| `/run-skill-generator` | `/run` と `/verify` 向けにプロジェクト起動方法を記録 |
| `/code-review` | コードレビュー |
| `/batch` | 複数タスクの並列実行 |
| `/debug` | デバッグ支援 |
| `/loop` | 繰り返しタスクのループ実行 |

`/run` と `/verify` は設定なしで動くが、非標準の起動手順（DB・envファイル・マルチステップビルド等）には `/run-skill-generator` で記録が必要。

### 最初の Skill を作る

```bash
# ディレクトリ作成
mkdir -p ~/.claude/skills/summarize-changes
```

`~/.claude/skills/summarize-changes/SKILL.md` に保存:

```yaml
---
description: Summarizes uncommitted changes and flags anything risky. Use when the user asks what changed, wants a commit message, or asks to review their diff.
---

## Current changes

!`git diff HEAD`

## Instructions

Summarize the changes above in two or three bullet points, then list any risks you notice such as missing error handling, hardcoded values, or tests that need updating. If the diff is empty, say there are no uncommitted changes.
```

`` !`git diff HEAD` `` は**ダイナミックコンテキストインジェクション**: Claude Code がコマンドを実行し、出力を Skill 本文に埋め込んでから Claude に渡す。

### Skills の置き場所とスコープ

| 場所 | パス | 適用範囲 |
|---|---|---|
| Enterprise (Managed) | managed settings 経由 | 組織の全ユーザー |
| Personal | `~/.claude/skills/<name>/SKILL.md` | 全プロジェクト |
| Project | `.claude/skills/<name>/SKILL.md` | このプロジェクトのみ |
| Plugin | `<plugin>/skills/<name>/SKILL.md` | プラグインが有効な場所 |

同名が複数スコープにある場合: enterprise > personal > project。Plugin Skills は `plugin-name:skill-name` のように名前空間で区別されるため衝突しない。

#### ライブ変更検出

セッション中に Skills ディレクトリを編集（追加・変更・削除）すると、再起動なしで反映される。ただし最初から存在しなかったトップレベル skills ディレクトリを新規作成した場合は再起動が必要。

#### モノレポ対応

プロジェクト Skills は作業ディレクトリから上にさかのぼってリポジトリルートまで探索する。サブディレクトリの `.claude/skills/` も作業中に動的に発見される。例: `packages/frontend/` で作業中は `packages/frontend/.claude/skills/` も探索対象になる。

### SKILL.md の構造

```
my-skill/
├── SKILL.md           # メイン指示（必須）
├── template.md        # テンプレート
├── examples/
│   └── sample.md      # 期待する出力例
└── scripts/
    └── validate.sh    # Claude が実行できるスクリプト
```

### フロントマター リファレンス

| フィールド | 必須 | 説明 |
|---|---|---|
| `name` | No | 表示名（コマンド名はディレクトリ名から取得） |
| `description` | 推奨 | 用途。Claude がいつ使うかを判断するために使用。1,536文字に切り詰め |
| `when_to_use` | No | 追加のトリガー条件。`description` に連結される |
| `argument-hint` | No | オートコンプリート時のヒント |
| `arguments` | No | 位置引数名（`$name` で参照） |
| `disable-model-invocation` | No | `true` にすると Claude が自動ロードしない（手動呼び出し専用）。`false` がデフォルト |
| `user-invocable` | No | `false` にすると `/` メニューから隠す（Claude が自動的に使うバックグラウンドナレッジ向け）。`true` がデフォルト |
| `allowed-tools` | No | この Skill がアクティブな間、承認なしで使えるツール |
| `disallowed-tools` | No | この Skill がアクティブな間、使えないツール（次のメッセージで解除） |
| `model` | No | この Skill 実行中のモデル（ターン中のみ有効） |
| `effort` | No | 努力レベル（`low`/`medium`/`high`/`xhigh`/`max`） |
| `context` | No | `fork` でサブエージェントとして実行 |
| `agent` | No | `context: fork` 時に使うサブエージェントタイプ |
| `hooks` | No | この Skill のライフサイクルフック |
| `paths` | No | このパターンにマッチするファイルを操作するときだけ自動ロード |
| `shell` | No | `` !`command` `` ブロックのシェル（`bash` or `powershell`） |

#### コマンド名の決まり方

| Skill の置き場所 | コマンド名の元 |
|---|---|
| `~/.claude/skills/` または `.claude/skills/` 配下のディレクトリ | ディレクトリ名 |
| `.claude/commands/` 配下のファイル | ファイル名（拡張子なし） |
| Plugin の `skills/` サブディレクトリ | ディレクトリ名（プラグインで名前空間化） |
| Plugin ルートの `SKILL.md` | フロントマターの `name`（なければプラグインのディレクトリ名） |

#### 文字列置換

| 変数 | 説明 |
|---|---|
| `$ARGUMENTS` | 呼び出し時に渡された全引数 |
| `$ARGUMENTS[N]` | N 番目の引数（0-indexed） |
| `$N` | `$ARGUMENTS[N]` の短縮形 |
| `$name` | `arguments` フロントマターで宣言した名前付き引数 |
| `${CLAUDE_SESSION_ID}` | 現在のセッション ID |
| `${CLAUDE_EFFORT}` | 現在の努力レベル |
| `${CLAUDE_SKILL_DIR}` | SKILL.md があるディレクトリ（パス参照に使用） |

### Skill コンテンツのタイプ

**リファレンス型**: API 規約・スタイルガイドなど、会話中に参照する知識

```yaml
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
```

**タスク型**: デプロイ・コミットなど、特定アクションの手順。副作用があるため `disable-model-invocation: true` が多い

```yaml
---
name: deploy
description: Deploy the application to production
context: fork
disable-model-invocation: true
---

Deploy the application:
1. Run the test suite
2. Build the application
3. Push to the deployment target
```

### 誰が呼び出すかを制御する

| フロントマター | ユーザーが呼べる | Claude が自動ロード | コンテキストへのロード |
|---|---|---|---|
| （デフォルト） | Yes | Yes | 説明は常時、本文は呼び出し時 |
| `disable-model-invocation: true` | Yes | No | 説明はなし、本文は呼び出し時 |
| `user-invocable: false` | No | Yes | 説明は常時、本文は呼び出し時 |

### Skill コンテンツのライフサイクル

Skill が呼び出されると、レンダリングされた SKILL.md の内容が1つのメッセージとしてセッションに追加され、**残りのセッション中ずっと保持**される。Auto-compaction 時は各 Skill の最初の 5,000 トークンを再アタッチ（全体で 25,000 トークン上限）。

### ツールの事前承認

`allowed-tools` フィールドで、Skill がアクティブな間に承認なしで使えるツールを設定:

```yaml
---
name: commit
description: Stage and commit the current changes
disable-model-invocation: true
allowed-tools: Bash(git add *) Bash(git commit *) Bash(git status *)
---
```

### 引数の渡し方

```yaml
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.
```

`/fix-issue 123` → Claude は "Fix GitHub issue 123 following our coding standards..." を受け取る。

複数引数のパターン:

```yaml
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

`/migrate-component SearchBar React Vue` → `$0=SearchBar`, `$1=React`, `$2=Vue`

## 高度なパターン

### ダイナミックコンテキストインジェクション

`` !`<command>` `` 構文でシェルコマンドを実行し、出力を Skill 本文に埋め込む。Claude が受け取る前に実行されるため「前処理」として機能する:

```yaml
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`
```

複数行コマンドはフェンスコードブロック（ ` ```! ` で開始）を使用。

`"disableSkillShellExecution": true` を settings に設定すると、user/project/plugin Skill のシェル実行を無効化できる（managed Skill は影響なし）。

### サブエージェントとして実行する

`context: fork` でサブエージェントとして実行。Skill コンテンツがそのサブエージェントへのプロンプトになる:

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:
1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

`agent` フィールドで実行環境を指定: `Explore`, `Plan`, `general-purpose`, またはカスタムサブエージェント。

| アプローチ | システムプロンプト | タスク | 追加ロード |
|---|---|---|---|
| `context: fork` の Skill | エージェントタイプから | SKILL.md の内容 | CLAUDE.md（Explore/Plan は除く） |
| `skills` フィールドを持つ Subagent | Subagent の本文 | Claude の委任メッセージ | プリロード Skills + CLAUDE.md |

### Claude の Skill アクセスを制限する

**全 Skill を無効化**:
```
# deny ルールに追加:
Skill
```

**特定の Skill を許可/拒否**:
```
# 特定の Skill のみ許可
Skill(commit)
Skill(review-pr *)

# 特定の Skill を拒否
Skill(deploy *)
```

### skillOverrides で Skill の可視性を上書き

settings からフロントマターを編集せずに Skill の可視性を制御。`/skills` メニューでスペースキーで状態を切り替え、Enter で保存（`.claude/settings.local.json` に書き込まれる）。

| 値 | Claude への表示 | `/` メニュー |
|---|---|---|
| `"on"` | 名前と説明 | あり |
| `"name-only"` | 名前のみ | あり |
| `"user-invocable-only"` | 非表示 | あり |
| `"off"` | 非表示 | 非表示 |

## 重要ポイント

- Skills の本文は**使われるまでコンテキストを消費しない**。大きなリファレンスドキュメントを安心して入れられる
- `disable-model-invocation: true` は副作用のある Skill（デプロイ・コミット等）に必ず付ける
- コマンド名はディレクトリ名から決まる（`name` フロントマターとは別）
- `allowed-tools` は制限ではなく「事前承認」。他のツールも使える（承認が必要になるだけ）
- `disallowed-tools` は Skill アクティブ中のみ有効。次のメッセージで解除される
- Skill を `context: fork` で実行するとサブエージェントが使われ、メインコンテキストから隔離される
- `${CLAUDE_SKILL_DIR}` を使うとスクリプト参照がスコープに依存しなくなる

## コード例 / 図

### 視覚的出力を生成する Skill（コードベースビジュアライザ）

```yaml
---
name: codebase-visualizer
description: Generate an interactive collapsible tree visualization of your codebase.
allowed-tools: Bash(python3 *)
---

# Codebase Visualizer

Run the visualization script from your project root:

```bash
python3 ${CLAUDE_SKILL_DIR}/scripts/visualize.py .
```
```

このパターンを応用すれば、依存グラフ・テストカバレッジレポート・API ドキュメント・DB スキーマ可視化など様々な視覚的出力が生成可能。

### トラブルシューティング

**Skill がトリガーされない場合**:
1. `description` にユーザーが自然に使いそうなキーワードを含める
2. `What skills are available?` で Skill が表示されるか確認
3. リクエストを `description` に近い言い回しに変える
4. 直接 `/skill-name` で呼び出す

**Skill が多すぎてトリガーされる場合**:
1. `description` をより具体的にする
2. 手動のみにするなら `disable-model-invocation: true` を追加

**Skill の説明が切り捨てられる場合**: `/doctor` でバジェット状況を確認。`skillListingBudgetFraction` 設定で拡張可能。

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## 次のトピック

- [03-hooks-fundamentals/](../03-hooks-fundamentals/index.md) — Hooks の基礎（概念・イベント・設定）

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
