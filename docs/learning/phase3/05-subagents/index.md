# カスタムサブエージェント — 設定と活用

> 出典: [Create custom subagents](https://code.claude.com/docs/en/sub-agents)（閲覧日 2026-06-12）
> このノートは公式ドキュメントの「sub-agents」を起点に、Claudeが自動生成した教材です。

## 概要

サブエージェントは特定タスクを担う専門 AI アシスタント。サイドタスクが検索結果・ログ・ファイル内容で会話を埋めてしまう場合、サブエージェントは独自コンテキストでそれを処理し、サマリーだけを返す。同じ種類のワーカーを繰り返しスポーンしているなら、カスタムサブエージェントを定義する価値がある。

各サブエージェントは独自のコンテキストウィンドウ・カスタムシステムプロンプト・特定ツールアクセス・独立パーミッションを持つ。Claude はタスクの説明にマッチするサブエージェントに自動的に委任する。

> **注**: サブエージェントは 1 つのセッション内で動作する。複数の独立セッションを並行して監視するには [background agents](/en/agent-view)、セッション同士が通信する場合は [agent teams](/en/agent-teams) を参照。

## 公式 docs に沿った解説

### ビルトインサブエージェント

| エージェント | モデル | ツール | 用途 |
|---|---|---|---|
| **Explore** | Haiku（高速） | 読み取り専用（Write/Edit 拒否） | ファイル発見・コード検索・コードベース探索 |
| **Plan** | 親会話を継承 | 読み取り専用 | Plan モード中のコードベース調査 |
| **general-purpose** | 親会話を継承 | 全ツール | 複雑な調査・マルチステップ操作・コード修正 |

Explore と Plan は CLAUDE.md とgit status をスキップする（コンテキストを小さく保つため）。

ビルトインをブロックするには `permissions.deny` に `Agent(Explore)` のように追加。`Agent` ツール自体を deny すれば全サブエージェント委任を禁止できる。

### 最初のサブエージェントを作る

`/agents` コマンドでタブ付きインターフェースが開く（Running タブ: 実行中サブエージェント, Library タブ: 管理）。またはサブエージェント定義ファイルを手動で作成:

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

サブエージェントは YAML フロントマター（設定）+ Markdown 本文（システムプロンプト）で定義。

### サブエージェントのスコープと優先度

| 場所 | スコープ | 優先度 |
|---|---|---|
| Managed settings | 組織全体 | 1（最高） |
| `--agents` CLI フラグ | 現在のセッション | 2 |
| `.claude/agents/` | このプロジェクト | 3 |
| `~/.claude/agents/` | 全プロジェクト | 4 |
| Plugin の `agents/` | プラグインが有効な場所 | 5（最低） |

同名の場合は優先度が高いものが勝つ。`.claude/agents/` はプロジェクトルートから上にさかのぼって探索。`--add-dir` で追加したディレクトリはファイルアクセスのみで、サブエージェントはスキャンされない。

サブエージェントはセッション開始時にロードされる。`/agents` インターフェース経由での作成は即時反映されるが、ファイルを直接編集した場合は再起動が必要。

### サポートされるフロントマターフィールド

| フィールド | 必須 | 説明 |
|---|---|---|
| `name` | Yes | 小文字・ハイフン使用の一意識別子。Hooks は `agent_type` としてこの値を受け取る |
| `description` | Yes | Claude がいつ委任するかを判断するための説明 |
| `tools` | No | 使えるツールのリスト（省略で全ツールを継承） |
| `disallowedTools` | No | 拒否するツール（継承または指定リストから除去） |
| `model` | No | `sonnet`/`opus`/`haiku`/`fable`/フルモデルID/`inherit`。デフォルトは `inherit` |
| `permissionMode` | No | `default`/`acceptEdits`/`auto`/`dontAsk`/`bypassPermissions`/`plan` |
| `maxTurns` | No | エージェントが停止するまでの最大ターン数 |
| `skills` | No | 起動時にコンテキストにプリロードする Skills |
| `mcpServers` | No | このサブエージェントが利用できる MCP サーバー |
| `hooks` | No | このサブエージェントのライフサイクルフック |
| `memory` | No | 永続メモリスコープ: `user`/`project`/`local` |
| `background` | No | `true` で常にバックグラウンドタスクとして実行 |
| `effort` | No | 努力レベル（`low`/`medium`/`high`/`xhigh`/`max`） |
| `isolation` | No | `worktree` で一時 git worktree で実行 |
| `color` | No | UI 表示色（`red`/`blue`/`green`/`yellow`/`purple`/`orange`/`pink`/`cyan`） |
| `initialPrompt` | No | `--agent` や `agent` 設定で main セッションとして起動した時に自動送信される最初のプロンプト |

### モデルの解決順序

1. `CLAUDE_CODE_SUBAGENT_MODEL` 環境変数
2. 呼び出し時の `model` パラメータ
3. サブエージェント定義の `model` フロントマター
4. main 会話のモデル

### ツールアクセスの制御

```yaml
# allowlist（これだけ使える）
---
name: safe-researcher
tools: Read, Grep, Glob, Bash
---

# denylist（これ以外全部使える）
---
name: no-writes
disallowedTools: Write, Edit
---
```

両方設定した場合: `disallowedTools` が先に適用され、その後 `tools` が解決される。両方にあるツールは除去される。

#### サブエージェントが使えないツール（UIや状態に依存するため）

- `Agent`（サブエージェントは他のサブエージェントをスポーンできない）
- `AskUserQuestion`
- `EnterPlanMode`
- `ExitPlanMode`（`permissionMode: plan` の場合は除く）
- `ScheduleWakeup`
- `WaitForMcpServers`

### パーミッションモード

| モード | 挙動 |
|---|---|
| `default` | 標準パーミッションチェック（プロンプトあり） |
| `acceptEdits` | ファイル編集と一般的なファイルシステムコマンドを自動受諾 |
| `auto` | バックグラウンド分類器がコマンドとパスを評価 |
| `dontAsk` | パーミッションプロンプトを自動拒否（明示的に許可されたツールは動く） |
| `bypassPermissions` | パーミッションプロンプトをスキップ（要注意） |
| `plan` | Plan モード（読み取り専用探索） |

親が `bypassPermissions` または `acceptEdits` の場合はそれが優先され、サブエージェントの `permissionMode` は無視される。

### Skills をサブエージェントにプリロード

```yaml
---
name: api-developer
description: Implement API endpoints following team conventions
skills:
  - api-conventions
  - error-handling-patterns
---

Implement API endpoints. Follow the conventions and patterns from the preloaded skills.
```

`skills` フィールドにリストした Skill の全内容は起動時にコンテキストに注入される（説明だけでなく本文全体）。`disable-model-invocation: true` の Skill はプリロードできない。

### 永続メモリ

```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---

Update your agent memory as you discover patterns, conventions, and recurring issues.
```

| スコープ | 場所 | 使いどころ |
|---|---|---|
| `user` | `~/.claude/agent-memory/<name>/` | 全プロジェクトで共有する学習 |
| `project` | `.claude/agent-memory/<name>/` | プロジェクト固有（バージョン管理可） |
| `local` | `.claude/agent-memory-local/<name>/` | プロジェクト固有（バージョン管理しない） |

メモリが有効な場合: システムプロンプトにメモリディレクトリの読み書き指示が追加。`MEMORY.md` の最初の 200 行 or 25KB が自動的に含まれる。Read/Write/Edit ツールが自動有効化される。

### フックをサブエージェントに定義する

サブエージェントのフロントマター内で直接フックを定義（そのサブエージェントがアクティブな間だけ適用）:

```yaml
---
name: code-reviewer
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

サブエージェントとして呼び出された場合、フロントマター内の `Stop` フックは自動的に `SubagentStop` イベントに変換される。

settings.json でサブエージェントのライフサイクルイベントに応答することもできる（`SubagentStart` / `SubagentStop`）。

### サブエージェントの呼び出し方法

**自然言語**: 名前をプロンプトに含める（Claude が判断して委任）
```
Use the code-reviewer subagent to look at the auth changes
```

**@-メンション**: 特定サブエージェントを保証して 1 タスク実行
```
@"code-reviewer (agent)" look at the auth changes
```

**セッション全体**: `--agent` フラグで起動時にサブエージェントのシステムプロンプトを使用
```bash
claude --agent code-reviewer
```

または `.claude/settings.json` に:
```json
{ "agent": "code-reviewer" }
```

### フォアグラウンド vs バックグラウンド

**フォアグラウンド**: 完了まで main 会話をブロック。パーミッションプロンプトがリアルタイムで表示される。

**バックグラウンド**: 並行して実行。セッション内の許可済みパーミッションで動作。プロンプトが必要なツール呼び出しは自動拒否される。

Claude が自動判断するほか、`Ctrl+B` で実行中タスクをバックグラウンドに送れる。

### サブエージェントのコンテキスト（起動時のロード内容）

| コンテンツ | ロードされるか |
|---|---|
| エージェント自身のシステムプロンプト（本文 or `prompt` フィールド） | Yes（常に） |
| タスクメッセージ（Claude が書く委任プロンプト） | Yes |
| CLAUDE.md / メモリ階層 | Yes（Explore と Plan を除く） |
| git status | Yes（Explore と Plan を除く） |
| プリロード Skills（`skills` フィールド） | Yes（列挙されたもの） |
| 親会話のコンテキスト | No（fork の場合のみ Yes） |

### Fork — 現在の会話を引き継ぐ

Fork は全会話を継承するサブエージェント（通常のサブエージェントは新鮮なコンテキストで開始）:

```
/fork draft unit tests for the parser changes so far
```

Fork はメインセッションと同じシステムプロンプト・ツール・モデル・メッセージ履歴を持つ。自身のツール呼び出しはメイン会話に見えず、最終結果だけが返ってくる。背景: `CLAUDE_CODE_FORK_SUBAGENT=1` で有効化（v2.1.161 以降はデフォルト有効）。

| | Fork | Named subagent |
|---|---|---|
| コンテキスト | 全会話履歴 | 新鮮なコンテキスト |
| システムプロンプト | メインセッションと同じ | 定義ファイルから |
| プロンプトキャッシュ | 親と共有（安い） | 独立キャッシュ |

## 重要ポイント

- サブエージェントを使う主な目的は**コンテキスト隔離**。詳細はサブエージェントで処理し、サマリーだけを受け取る
- `description` フィールドが Claude の委任判断に使われる。「use proactively」など積極的な委任を促す文言を含めると有効
- `tools` は allowlist、`disallowedTools` は denylist。両方設定した場合は後者が先に適用される
- サブエージェントは他のサブエージェントをスポーンできない（ネスト不可）
- Fork はコンテキストを引き継ぐため、名前付きサブエージェントが必要とする背景説明が多い場合に有効
- `memory` フィールドで会話をまたいだ知識の蓄積が可能。`project` スコープがデフォルト推奨
- `isolation: worktree` でサブエージェントが一時 git worktree で分離された状態で実行される

## コード例 / 図

### コードレビュー専門サブエージェント

```markdown
---
name: code-reviewer
description: Expert code review specialist. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files

Review checklist:
- Code clarity and readability
- Error handling
- No exposed secrets
- Test coverage

Provide feedback by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)
```

### DB 読み取り専用サブエージェント（Hook 連携）

```markdown
---
name: db-reader
description: Execute read-only database queries
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access.
If asked to modify data, explain you only have read access.
```

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## 次のトピック

- [06-agent-teams/](../06-agent-teams/index.md) — Agent teams による並列協調

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
