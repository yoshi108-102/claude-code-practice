# Hooks 基礎 — ライフサイクルイベントと設定

> 出典: [Automate actions with hooks](https://code.claude.com/docs/en/hooks-guide)（閲覧日 2026-06-12）、および [Hooks reference](https://code.claude.com/docs/en/hooks)（閲覧日 2026-06-12）
> このノートは公式ドキュメントの hooks-guide（前半）と hooks リファレンスの概念・設定部分を起点に、Claudeが自動生成した教材です。

## 概要

Hooks は Claude Code のライフサイクルの特定ポイントで実行されるユーザー定義のシェルコマンド。LLM に「実行してほしい」と頼むのではなく、**決定論的に確実に実行される**制御を提供する。ファイル編集後の自動フォーマット、コマンド実行前の検証、セッション開始時のコンテキスト注入、通知送信などに使用する。

判断が必要な場合は `type: "prompt"` / `type: "agent"` フックも使える（Claude モデルが条件を評価する）。

## 公式 docs に沿った解説

### Hook の仕組み（ライフサイクル）

イベントが発火 → matcher でフィルタ → `if` 条件チェック → Hook ハンドラ実行 → Claude Code が結果に基づいて動作

この 4 ステップを通過したフックのみ実際に実行される。

### 最初の Hook を作る — デスクトップ通知

`~/.claude/settings.json` に追加:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

`/hooks` を実行して設定が反映されているか確認。Notification イベントを選択すると詳細（イベント・matcher・タイプ・ソースファイル・コマンド）が確認できる。

### 何を自動化できるか

- クロードが入力を待つ時に通知を受け取る
- 編集後にコードを自動フォーマット
- 保護ファイルへの編集をブロック
- コンパクション後にコンテキストを再注入
- 設定変更の監査
- ディレクトリ変更時に環境を再読み込み
- 特定のパーミッションプロンプトを自動承認

### 代表的なパターン例

#### コード自動フォーマット

`.claude/settings.json` に追加:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

#### 保護ファイルへの編集をブロック

`.claude/hooks/protect-files.sh` を作成して `chmod +x`:

```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED_PATTERNS=(".env" "package-lock.json" ".git/")

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH matches protected pattern '$pattern'" >&2
    exit 2
  fi
done

exit 0
```

`.claude/settings.json` に登録:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

#### コンパクション後のコンテキスト再注入

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Reminder: use Bun, not npm. Run bun test before committing.'"
          }
        ]
      }
    ]
  }
}
```

### 全 Hook イベント一覧

| イベント | 発火タイミング |
|---|---|
| `SessionStart` | セッション開始・再開時 |
| `Setup` | `--init-only` / `--init` / `--maintenance` で起動時（CI・スクリプト向け） |
| `UserPromptSubmit` | プロンプト送信時（Claude が処理する前） |
| `UserPromptExpansion` | ユーザーコマンドがプロンプトに展開される時 |
| `PreToolUse` | ツール呼び出し実行前（ブロック可） |
| `PermissionRequest` | パーミッションダイアログが表示される時 |
| `PermissionDenied` | auto モード分類器がツール呼び出しを拒否した時 |
| `PostToolUse` | ツール呼び出し成功後 |
| `PostToolUseFailure` | ツール呼び出し失敗後 |
| `PostToolBatch` | 並列ツール呼び出しのバッチ完了後（次のモデル呼び出し前） |
| `Notification` | Claude Code が通知を送る時 |
| `MessageDisplay` | アシスタントのメッセージテキスト表示中 |
| `SubagentStart` | サブエージェントがスポーンされた時 |
| `SubagentStop` | サブエージェントが終了した時 |
| `TaskCreated` | `TaskCreate` でタスクが作成される時 |
| `TaskCompleted` | タスクが完了としてマークされる時 |
| `Stop` | Claude が応答を終えた時 |
| `StopFailure` | API エラーでターンが終了した時（出力・終了コードは無視） |
| `TeammateIdle` | Agent team のチームメイトがアイドル状態になる時 |
| `InstructionsLoaded` | CLAUDE.md / `.claude/rules/*.md` がコンテキストに読み込まれた時 |
| `ConfigChange` | セッション中に設定ファイルが変更された時 |
| `CwdChanged` | 作業ディレクトリが変わった時（direnv 連携等） |
| `FileChanged` | 監視ファイルがディスク上で変更された時 |
| `WorktreeCreate` | Worktree が作成される時（デフォルトの git 動作を置き換える） |
| `WorktreeRemove` | Worktree が削除される時 |
| `PreCompact` | コンテキストコンパクション前 |
| `PostCompact` | コンテキストコンパクション完了後 |
| `Elicitation` | MCP サーバーがツール呼び出し中にユーザー入力を要求した時 |
| `ElicitationResult` | ユーザーが MCP elicitation に応答した後（サーバーに送信前） |
| `SessionEnd` | セッション終了時 |

### Hook タイプ

| タイプ | 説明 |
|---|---|
| `"type": "command"` | シェルコマンドを実行（最も一般的） |
| `"type": "http"` | URL に POST |
| `"type": "mcp_tool"` | 接続済み MCP サーバーのツールを呼び出す |
| `"type": "prompt"` | 単一ターンの LLM 評価（判断が必要な場合） |
| `"type": "agent"` | ツールアクセスを持つマルチターン検証（実験的） |

### 設定場所（スコープ）

| 場所 | スコープ | 共有可 |
|---|---|---|
| `~/.claude/settings.json` | 全プロジェクト | No（ローカルマシン） |
| `.claude/settings.json` | このプロジェクト | Yes（リポジトリにコミット可） |
| `.claude/settings.local.json` | このプロジェクト | No（gitignore 推奨） |
| Managed policy settings | 組織全体 | Yes（管理者制御） |
| Plugin `hooks/hooks.json` | プラグインが有効な場所 | Yes（プラグインにバンドル） |
| Skill / Agent フロントマター | Skill/Agent がアクティブな間 | Yes（ファイル内で定義） |

### Matcher パターン

Matcher なし（`""` または省略）は全イベントで発火。Matcher を付けることで絞り込める。

| Matcher 値 | 評価方法 | 例 |
|---|---|---|
| 単一ツール名 / 列挙 | 完全一致または `\|` で OR | `Bash`, `Edit\|Write` |
| 正規表現 | 先頭が `^` または `.*` を含む、または `?` / `+` / `{` / `[` を含む | `mcp__.*`, `^Bash$` |
| 空文字列 | 全イベントにマッチ | `""` |

#### イベント別 Matcher が照合するフィールド

| イベント | Matcher が照合するもの | 値の例 |
|---|---|---|
| `PreToolUse`, `PostToolUse`, `PermissionRequest` 等 | ツール名 | `Bash`, `Edit\|Write`, `mcp__.*` |
| `SessionStart` | セッションの開始方法 | `startup`, `resume`, `clear`, `compact` |
| `SessionEnd` | セッション終了の理由 | `clear`, `resume`, `logout` 等 |
| `Notification` | 通知タイプ | `permission_prompt`, `idle_prompt` 等 |
| `SubagentStart`, `SubagentStop` | エージェントタイプ名 | `general-purpose`, `Explore`, カスタム名 |
| `PreCompact`, `PostCompact` | コンパクションのトリガー | `manual`, `auto` |
| `ConfigChange` | 設定ソース | `user_settings`, `project_settings` 等 |
| `FileChanged` | 監視するファイル名（リテラル） | `.envrc\|.env` |

### `if` フィールドによる細粒度フィルタ

`if` フィールドはツール名と引数を合わせてフィルタする。`matcher` は グループレベルでツール名のみフィルタするのに対し、`if` はフック個別レベルで動作する:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(git *)",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check-git-policy.sh"
          }
        ]
      }
    ]
  }
}
```

`if` フィールドはツールイベント（`PreToolUse`, `PostToolUse` 等）にのみ有効。他のイベントに付けるとフックが実行されない。

### Hook ハンドラの共通フィールド

| フィールド | 必須 | 説明 |
|---|---|---|
| `type` | Yes | `command` / `http` / `mcp_tool` / `prompt` / `agent` |
| `if` | No | ツール名と引数でフィルタ |
| `timeout` | No | タイムアウト秒数（デフォルトはタイプ・イベントによる） |
| `statusMessage` | No | フック実行中に表示するメッセージ |
| `once` | No | `true` でセッション中に 1 回だけ実行 |

#### command フック固有フィールド

| フィールド | 必須 | 説明 |
|---|---|---|
| `command` | Yes | 実行するシェルコマンド |
| `args` | No | exec 形式で引数を配列で渡す（シェルを経由しない） |
| `async` | No | `true` で非同期実行（結果を待たない） |
| `asyncRewake` | No | 非同期フックが終了したら Claude を再起動 |
| `shell` | No | `powershell` を指定すると PowerShell で実行 |

exec 形式（`args` を使用）はシェルを経由しないため引用符の問題が発生しない。パスに空白がある場合に有効:

```json
{
  "type": "command",
  "command": "node",
  "args": ["${CLAUDE_PLUGIN_ROOT}/scripts/format.js", "--fix"]
}
```

### Skill / Agent のフロントマターでフックを定義

Skill や Agent の YAML フロントマター内に直接 Hook を書くことができる。そのコンポーネントがアクティブな間だけ適用される:

```yaml
---
name: secure-coder
description: Coding with security validation
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

### `/hooks` メニュー

Claude Code 内で `/hooks` を実行するとイベント別に設定済みのフック一覧が表示される。読み取り専用（変更は settings JSON を直接編集するか Claude に頼む）。

## 重要ポイント

- Hooks は「頼む」ではなく「強制する」手段。`PreToolUse` で `.env` 編集をブロックすれば確実に防げる
- 同一イベントの複数 Hook は**並列実行**。deny が 1 つでもあればツール呼び出しをブロック
- `type: "command"` はシェルフォームとexec フォームを選べる。パスに空白があるなら exec 形式を使う
- Matcher は大文字小文字を区別する。`Bash` と `bash` は別もの
- `async: true` で非同期実行できるが、ブロック判定はできない
- Skill/Agent フロントマター内の Hook はそのコンポーネントがアクティブな間だけ有効

## コード例 / 図

### 複数フックを同じイベントに登録（並列実行）

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r .tool_input.command >> ~/.claude/bash.log"
          },
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm-rf.sh"
          }
        ]
      }
    ]
  }
}
```

両フックが並列実行される。ロギングフックは exit 0（決定なし）、ガードレールフックは exit 2（deny）→ deny が勝ちコマンドをブロック。ログエントリはすでに書き込まれている。

### Notification イベントの Matcher 値

| Matcher | 発火タイミング |
|---|---|
| `permission_prompt` | ツール使用の承認が必要 |
| `idle_prompt` | 応答完了、次のプロンプト待ち |
| `auth_success` | 認証完了 |
| `elicitation_dialog` | MCP サーバーが入力フォームを開く |
| `elicitation_complete` | MCP elicitation フォームが送信/却下された |

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## 次のトピック

- [04-hooks-io-and-patterns/](../04-hooks-io-and-patterns/index.md) — Hooks の I/O・デシジョン制御・高度なパターン

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
