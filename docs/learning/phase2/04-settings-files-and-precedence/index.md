# settings.json の階層と優先順位

> 出典: [Claude Code settings](https://code.claude.com/docs/en/settings)（閲覧日 2026-06-12）
> このノートは公式ドキュメントの「Claude Code settings」を起点に、Claudeが自動生成した教材です。

## 概要

`settings.json` は Claude Code の挙動を設定するファイル。CLAUDE.md が「振る舞いのガイド（強制ではない）」であるのに対し、settings は **permission・env・model など技術的な設定（強制レイヤー寄り）** を担う。設定には **4 つのスコープ** があり、それぞれ「誰に効くか」「チームと共有するか」が異なる。

## 公式 docs に沿った解説

### 4 つのスコープと保管場所

| スコープ | 場所 | 効く範囲 | git 共有 |
|---|---|---|---|
| **Managed** | macOS: `/Library/Application Support/ClaudeCode/` / Linux: `/etc/claude-code/` / Windows: レジストリ or `C:\Program Files\ClaudeCode\` | マシン上の全ユーザー | IT が MDM/plist/registry で配布 |
| **User** | `~/.claude/settings.json` | 自分の全プロジェクト | × |
| **Project** | `.claude/settings.json` | リポジトリの全員 | ○（commit） |
| **Local** | `.claude/settings.local.json` | 自分・このリポのみ | × （`.gitignore` 推奨） |

別枠として `~/.claude.json` があり、OAuth セッション・MCP 設定・プロジェクト状態・キャッシュが入る。また `.mcp.json` はプロジェクト別 MCP 設定ファイル。

#### Managed 設定の配布方法

組織管理者は以下の3経路で managed settings を配布できる:

1. **MDM/OS ポリシー**（macOS は plist/Jamf、Windows は レジストリ/Group Policy）
2. **ファイルベース配布**（上記のパスに `managed-settings.json` を設置）
3. **Drop-in ディレクトリ**（`managed-settings.d/*.json` をアルファベット順でマージ）

### 優先順位（precedence）

同じ設定キーが複数スコープにある場合、**上位が下位を上書き**:

```
1. Managed（最強・CLI 引数でも覆せない）
2. CLI 引数（--model 等。一時的なセッション上書き）
3. Local（.claude/settings.local.json）
4. Project（.claude/settings.json）
5. User（~/.claude/settings.json）
```

**重要な例外**: permission ルール（allow/ask/deny）は「上書き」ではなく**全スコープでマージ**される。deny は最優先で適用されるため、「どこかで deny されたら他のどのレベルでも allow できない」。

### 主要な設定キー（カテゴリ別）

#### モデル・AI 動作

| キー | 用途 |
|---|---|
| `model` | デフォルトモデル（alias または full model name） |
| `availableModels` | ユーザーが選択できるモデルの制限（managed 推奨） |
| `fallbackModel` | 過負荷時のフォールバックモデル列 |
| `effortLevel` | 努力レベルを永続化（`low`/`medium`/`high`/`xhigh`） |
| `alwaysThinkingEnabled` | 拡張思考をデフォルト有効 |

#### パーミッション・セキュリティ

| キー | 用途 |
|---|---|
| `permissions.allow/ask/deny` | 権限ルール（→ トピック 05） |
| `permissions.defaultMode` | 起動時の permission モード（→ トピック 06） |
| `allowManagedPermissionRulesOnly` | user/project の権限ルールを無効化（managed のみ） |

#### MCP サーバー

| キー | 用途 |
|---|---|
| `allowedMcpServers` | MCP サーバーのホワイトリスト（managed のみ） |
| `deniedMcpServers` | MCP サーバーのブラックリスト |
| `allowManagedMcpServersOnly` | managed 定義の MCP のみ許可 |

#### メモリ・CLAUDE.md

| キー | 用途 |
|---|---|
| `autoMemoryEnabled` | auto memory の ON/OFF（Phase 2 Task 1） |
| `claudeMd` | 組織管理の CLAUDE.md 内容（managed 限定） |
| `claudeMdExcludes` | 除外する CLAUDE.md パターン |

#### Hooks・拡張

| キー | 用途 |
|---|---|
| `hooks` | ライフサイクルイベントでのコマンド（Phase 3） |
| `allowManagedHooksOnly` | managed 定義のフックのみ許可 |

#### 環境変数

| キー | 用途 |
|---|---|
| `env` | 全セッションに適用する環境変数（→ トピック 07） |

#### UI・スタイル

| キー | 用途 |
|---|---|
| `language` | 応答言語・音声入力言語 |
| `outputStyle` | システムプロンプト調整 |
| `editorMode` | キーバインド（`normal`/`vim`） |

### 設定変更の反映タイミング

**リアルタイム反映**（セッション再開不要）:
- `permissions`、`hooks`、credential helpers

**セッション再開 or 別操作が必要**:
- `model` → `/model` コマンドで途中切替可能
- `outputStyle` → `/clear` で再構築

### 無効エントリの処理（Managed 設定）

managed settings に無効な値が入っても、Claude Code は無効エントリをスキップして警告を記録し、残りの有効なポリシーを継続する。ただし `allowedMcpServers` のようなセキュリティ系フィールドは例外処理が厳しく、無効時は空のホワイトリスト（= MCP 不許可）として機能する。

## 重要ポイント

- settings は 4 スコープ（managed > CLI > local > project > user）。**managed は何があっても覆せない**
- CLAUDE.md が「ガイド」なら settings は「**技術的設定・強制寄り**」—役割分担を意識
- **permission ルールだけは上書きでなくマージ**。deny は全スコープ最優先 → 「どこかで deny したら誰も allow できない」
- `settings.local.json` は個人用 → `.gitignore` に追加（Phase 2 Task 1 で実施済み）
- managed-only のキー（`claudeMd`, `allowedMcpServers`, `allowManagedPermissionRulesOnly` 等）は user/project に書いても無効
- `managed-settings.d/` によるドロップイン配布で複数チームが独立してポリシーを追加できる

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

`$schema` を書くと VS Code/Cursor でオートコンプリートとインライン検証が有効になる。

### 「user で許可、project で拒否」→ どちらが勝つ?

```text
user settings:    allow: Bash(git push *)
project settings: deny:  Bash(git push *)
→ deny が勝つ（permission はマージ + deny 最優先）
```

### managed settings の Drop-in 配布例

```
managed-settings.d/
├── 10-telemetry.json      # 先に適用
├── 20-security.json       # 後に適用（スカラーは上書き、配列はマージ+重複排除）
└── 30-policies.json
```

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## 次のトピック

- [05-permissions/](../05-permissions/index.md) — 権限ルール（allow/ask/deny）の詳細

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
