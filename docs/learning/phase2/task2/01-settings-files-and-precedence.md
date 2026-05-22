# 01. settings.json の階層と優先順位

> 出典: [Claude Code settings](https://code.claude.com/docs/en/settings)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「settings」の構造を起点に Claude が自動生成した教材です。

## 概要

`settings.json` は Claude Code の挙動を設定するファイル。Task 1 の CLAUDE.md と同じく**スコープが複数**あり、それぞれ「誰に効くか」が違う。CLAUDE.md が「振る舞いのガイド」だったのに対し、settings は **permission・env・model など技術的な設定（強制レイヤー寄り）** を扱う。

## 公式docsに沿った解説

### 4つのスコープと保管場所

| スコープ | 場所 | 効く範囲 | git 共有 |
|---|---|---|---|
| **User** | `~/.claude/settings.json` | 自分の全プロジェクト | × |
| **Project** | `.claude/settings.json` | リポジトリの全員 | ○（commit） |
| **Local** | `.claude/settings.local.json` | 自分・このリポのみ | ×（gitignore） |
| **Managed** | macOS `/Library/Application Support/ClaudeCode/` / Linux `/etc/claude-code/` / Windows レジストリ or `C:\Program Files\ClaudeCode\` | マシン全ユーザー | IT が配布 |

> Task 1 の CLAUDE.md 4階層と**同じ構造**（managed/user/project/local）。「誰と共有するか」で選ぶ、も同じ。Task 1 で `.gitignore` に `settings.local.json` を足したのはこの local スコープのため。
> 別枠で `~/.claude.json`（global config storage）に OAuth セッション・MCP 設定・プロジェクト状態・キャッシュが入る。

### 優先順位（precedence）— ここが実務の肝

同じ設定が複数スコープにあるとき、**高い方が勝つ**:

```
1. Managed（最強・上書き不可。CLI 引数でも覆せない）
2. CLI 引数（--model 等。一時的なセッション上書き）
3. Local（.claude/settings.local.json）
4. Project（.claude/settings.json）
5. User（~/.claude/settings.json）
```

**重要な例外**:
- **permission ルールは「上書き」ではなく「マージ」**される（全スコープの allow/ask/deny が合算）。しかも deny は最優先（→ 教材02）
- だから「user で許可、project で拒否」なら**拒否が勝つ**

### 主要な設定キー（よく使うもの）

| キー | 用途 |
|---|---|
| `permissions.allow/ask/deny` | 権限ルール（→ 教材02） |
| `permissions.defaultMode` | 起動時の permission モード（→ 教材03） |
| `env` | 全セッションに適用する環境変数（→ 教材04） |
| `model` | デフォルトモデル（→ 教材04） |
| `autoMemoryEnabled` | auto memory の ON/OFF（Task 1） |
| `hooks` | ライフサイクルイベントでのコマンド（Phase 3） |
| `outputStyle` | output style（Phase 2 後半） |
| `sandbox.enabled` | bash サンドボックス（Phase 2 後半） |
| `claudeMd` | 組織管理の CLAUDE.md（managed 限定） |
| `availableModels` | 選択可能モデルの allowlist（managed で強制） |

### 反映タイミング

ほとんどの設定はファイル変更で**自動リロード**。例外（再起動 or 別操作が要る）:
- `model` → `/model` で途中切替
- `outputStyle` → system prompt の一部なので `/clear` で再構築

## 重要ポイント

- settings は4スコープ（managed > CLI > local > project > user）。**managed は何があっても覆せない**
- CLAUDE.md が「ガイド」なら settings は「**技術的設定・強制寄り**」。役割分担を意識（Task 1 の結論の続き）
- **permission ルールだけは上書きでなくマージ**。deny は全スコープ最優先 → 「どこかで deny されたら誰も allow できない」
- `settings.local.json` は個人用 → gitignore（Task 1 で実施済み）
- managed-only のキー（`claudeMd`, `availableModels` の強制 等）は user/project に書いても無効

## コード例 / 図

### 最小の settings.json

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(npm run test *)", "Read(~/.zshrc)"],
    "deny":  ["Bash(curl *)", "Read(./.env)", "Read(./secrets/**)"]
  },
  "env": { "CLAUDE_CODE_ENABLE_TELEMETRY": "1" },
  "model": "claude-sonnet-4-6",
  "autoMemoryEnabled": true
}
```

### 「user で許可、project で拒否」→ どちらが勝つ?

```text
user settings:    allow: Bash(git push *)
project settings: deny:  Bash(git push *)
→ deny が勝つ（permission はマージ + deny 最優先）
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 次の教材: [02-permissions.md](02-permissions.md) — 権限ルール（allow/ask/deny）
- 関連 docs: [Permissions](https://code.claude.com/docs/en/permissions) / [Memory](https://code.claude.com/docs/en/memory)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_
