# Hooks I/O・デシジョン制御・高度なパターン

> 出典: [Hooks reference](https://code.claude.com/docs/en/hooks)（後半）および [Automate actions with hooks](https://code.claude.com/docs/en/hooks-guide)（後半）（閲覧日 2026-06-12）
> このノートは hooks リファレンスの I/O フォーマット・デシジョン制御・prompt/agent/MCP ツールフック・実例パターン・安全考察を対象に、Claudeが自動生成した教材です。

## 概要

Hook の「入出力」と「デシジョン制御」は、単なるシェルコマンド実行を超えた高度な制御を可能にする。stdin でイベントデータを受け取り、stdout と終了コードで Claude Code の動作を制御する。JSON 出力でより細かい決定（allow/deny/defer/ask）や追加コンテキストの注入もできる。

## 公式 docs に沿った解説

### Common Input Fields（共通入力フィールド）

全 Hook イベントで stdin から以下の JSON が渡される:

| フィールド | 説明 |
|---|---|
| `session_id` | 現在のセッション ID |
| `transcript_path` | セッション transcript ファイルのパス |
| `cwd` | イベント発火時の作業ディレクトリ |
| `permission_mode` | 現在のパーミッションモード |
| `effort` | 努力レベル |
| `hook_event_name` | フックを起動したイベント名 |
| `agent_id` | サブエージェントコンテキストの場合、サブエージェント ID |
| `agent_type` | サブエージェントコンテキストの場合、エージェントタイプ |

各イベントにはイベント固有のフィールドが追加される。例: `PreToolUse` では `tool_name` と `tool_input`。

### Exit Code Output（終了コードによる出力）

Hook の終了コードが Claude Code の動作を決める:

| 終了コード | 意味 |
|---|---|
| **Exit 0** | 成功・異議なし。アクションは通常通り進む。`UserPromptSubmit` / `SessionStart` では stdout のテキストが Claude のコンテキストに追加される |
| **Exit 2** | アクションをブロック。stderr に理由を書くと Claude にフィードバックとして返る |
| **その他** | 非ブロックエラー。アクションは進む。transcript に `<hook name> hook error` が表示される |

#### イベント別 Exit 2 の挙動

`SessionStart`, `Setup`, `Notification` 等の**インフォメーショナルイベント**では exit 2 はブロックできない（stderr をユーザーに表示して続行）。`PreToolUse`, `UserPromptSubmit` 等では exit 2 でブロックできる。

### JSON Output（構造化出力）

exit 2 はシンプルなブロックしかできない。より細かい制御には exit 0 + JSON を stdout に出力する:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Use rg instead of grep for better performance"
  }
}
```

> **注意**: JSON 出力を使う場合は exit 0 にする。exit 2 のときは JSON が無視される。

#### PreToolUse の permissionDecision 値

| 値 | 挙動 |
|---|---|
| `"allow"` | インタラクティブなパーミッションプロンプトをスキップ（deny/ask ルールは引き続き適用） |
| `"deny"` | ツール呼び出しをキャンセルし、reason を Claude に返す |
| `"ask"` | ユーザーにパーミッションプロンプトを通常通り表示 |
| `"defer"` | 非インタラクティブモード（`-p`）でプロセスを終了し、Agent SDK ラッパーが入力を収集して再開できるようにする |

`"allow"` はインタラクティブプロンプトをスキップするが、settings の deny ルールは覆せない。managed settings の deny リストも常に優先される。

#### デシジョンコントロール サマリー

| イベント | デシジョンパターン | 主要フィールド |
|---|---|---|
| `UserPromptSubmit`, `Stop` 等 | トップレベル `decision: "block"` | `{"decision": "block", "reason": "..."}` |
| `PreToolUse`, `PermissionRequest` 等 | `hookSpecificOutput` パターン | `hookSpecificOutput.permissionDecision` 等 |
| `WorktreeCreate` | パス返却 | `worktreePath` を返す |
| `SessionStart`, `Setup` 等 | コンテキストのみ | stdout テキストが Claude のコンテキストに追加 |
| `WorktreeRemove`, `Notification`, `SessionEnd` 等 | なし（ブロック不可） | — |

#### 端末通知を出す（terminalSequence）

```json
{
  "terminalSequence": "]777;notify;Claude Code;done"
}
```

stdout に出力することで OSC 777 形式の端末通知を発火できる。

#### Claude にコンテキストを注入する（additionalContext）

```json
{
  "additionalContext": "Branch: feature/auth\nUncommitted changes: 3 files\nActive issue: AUTH-1234"
}
```

`additionalContext` で返したテキストは Claude が読むシステムリマインダーとして注入される。

#### SessionStart での環境変数永続化

```bash
# CLAUDE_ENV_FILE に書き込むと、以降の Bash コマンドすべてのプリアンブルとして実行される
echo "export DB_HOST=localhost" >> "$CLAUDE_ENV_FILE"
```

### HTTP フック

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/hooks/tool-use",
            "headers": {
              "Authorization": "Bearer $MY_TOKEN"
            },
            "allowedEnvVars": ["MY_TOKEN"]
          }
        ]
      }
    ]
  }
}
```

エンドポイントは command フックと同じ JSON 出力フォーマットでレスポンスを返す。HTTP ステータスコードだけではアクションをブロックできない。ヘッダー値の `$VAR_NAME` / `${VAR_NAME}` は `allowedEnvVars` リストにある変数だけが展開される。

### MCP ツールフック

接続済み MCP サーバーのツールを直接呼び出せる:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "mcp_tool",
            "server": "my_server",
            "tool": "security_scan",
            "input": {
              "file_path": "$tool_input.file_path"
            }
          }
        ]
      }
    ]
  }
}
```

### プロンプトベースフック（type: "prompt"）

判断が必要な場合に使う。Claude モデル（デフォルト Haiku）が yes/no 決定を返す:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if all tasks are complete. If not, respond with {\"ok\": false, \"reason\": \"what remains to be done\"}."
          }
        ]
      }
    ]
  }
}
```

モデルが `"ok": false` を返した場合:
- `Stop` / `SubagentStop`: `reason` が Claude へのフィードバックとなり作業を継続
- `PreToolUse`: ツール呼び出しが拒否され、`reason` がツールエラーとして返る
- `PostToolUse`, `PostToolBatch`, `UserPromptSubmit`: ターンが終了し `reason` が警告行として表示

### エージェントベースフック（type: "agent"、実験的）

ファイルを読んだりコマンドを実行したりして条件を検証する場合に使う。最大 50 ターンのツール使用:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verify that all unit tests pass. Run the test suite and check the results.",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

prompt フックはフック入力データだけで判断できる場合に使用。agent フックはコードベースの実際の状態を確認する必要がある場合に使用。

### 実例パターン

#### 自動承認（PermissionRequest + ExitPlanMode）

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "ExitPlanMode",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\": {\"hookEventName\": \"PermissionRequest\", \"decision\": {\"behavior\": \"allow\"}}}'"
          }
        ]
      }
    ]
  }
}
```

特定ツール（`ExitPlanMode` 等）への承認ダイアログを自動承認。matcher を `.*` や空にすると**全てのパーミッションプロンプトを自動承認**してしまうので matcher を絞ること。

#### direnv 連携（CwdChanged + CLAUDE_ENV_FILE）

```json
{
  "hooks": {
    "SessionStart": [
      { "hooks": [{ "type": "command", "command": "direnv export bash > \"$CLAUDE_ENV_FILE\"" }] }
    ],
    "CwdChanged": [
      { "hooks": [{ "type": "command", "command": "direnv export bash > \"$CLAUDE_ENV_FILE\"" }] }
    ]
  }
}
```

#### 設定変更の監査ログ（ConfigChange）

```json
{
  "hooks": {
    "ConfigChange": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "jq -c '{timestamp: now | todate, source: .source, file: .file_path}' >> ~/claude-config-audit.log"
          }
        ]
      }
    ]
  }
}
```

#### Stop フックの無限ループ防止

```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0  # Claude を止める
fi
# ... フックのメインロジック
```

Stop フックが 8 回連続でブロックすると Claude Code が強制的にオーバーライドする。`stop_hook_active` フィールドで検出してループを防ぐ。

### セキュリティ考察

- Hook のスクリプトはシェルで実行されるため、入力に悪意のあるデータが含まれる可能性を考慮すること
- `PreToolUse` フックは `bypassPermissions` モードでも発火する（bypass を使っていてもブロックできる）
- Hook が `"allow"` を返しても settings の deny ルールは覆せない。Hook はルールを厳しくはできるが緩めることはできない
- `PermissionRequest` フックは非インタラクティブモード（`-p`）では発火しない。自動 CI では `PreToolUse` を使う
- 複数の `PreToolUse` フックが `updatedInput` でツール引数を書き換える場合、フックは並列実行されるため最後に終わったものが勝つ（非決定論的）

### デバッグ方法

1. `Ctrl+O` でトランスクリプト表示（成功は無音、ブロックエラーは stderr、非ブロックエラーは `<hook name> hook error` 通知）
2. `claude --debug-file /tmp/claude.log` でデバッグログを記録
3. `tail -f /tmp/claude.log` で別端末から監視
4. セッション中からは `/debug` で有効化

### よくあるトラブルシューティング

**JSON 検証失敗**: シェルプロファイルに unconditional な `echo` があると、hook の JSON 出力の前に余分なテキストが付く。`$-` フラグを確認してインタラクティブシェルでのみ echo する。

**hook が発火しない**: matcher は大文字小文字を区別する。`Bash` と `bash` は別。

**Stop フックの上限**: `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` 環境変数で上限を変更できる。

## 重要ポイント

- **exit 0 + JSON** で細かい制御。**exit 2** でシンプルなブロック。両方使うと JSON が無視される
- `"allow"` は prompt をスキップするが deny ルールは優先される。Hook はルールを「緩める」ことはできない
- `additionalContext` で Claude のコンテキストにテキストを注入。`terminalSequence` で端末通知
- prompt フック（Haiku）は ok/not-ok の判断を出すだけ。agent フック（50 ターン）は実際にコードを調べられる
- `CLAUDE_ENV_FILE` に書き込むと環境変数を Bash コマンドをまたいで永続化できる
- Stop フックは 8 回連続ブロックでオーバーライドされる。`stop_hook_active` フラグを確認してループを防ぐ

## コード例 / 図

### PreToolUse で破壊的コマンドをブロック

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q "rm -rf"; then
  echo "Blocked: rm -rf is not allowed" >&2
  exit 2
fi

exit 0
```

### 権限決定の決め方フロー

```
PreToolUse フックが発火
    ↓
permissionDecision を返すか?
  - "deny" → ツール呼び出しをキャンセル + reason を Claude に返す
  - "allow" → プロンプトをスキップ（deny ルールは引き続き適用）
  - "ask" → 通常通りユーザーにプロンプト表示
  - なし(exit 0) → 通常のパーミッションフローに進む
```

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## 次のトピック

- [05-subagents/](../05-subagents/index.md) — カスタムサブエージェントの作成と設定

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
