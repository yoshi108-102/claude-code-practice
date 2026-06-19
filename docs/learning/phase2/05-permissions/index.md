# 権限ルール — allow / ask / deny

> 出典: [Configure permissions](https://code.claude.com/docs/en/permissions)（閲覧日 2026-06-12）
> このノートは公式ドキュメントの「Configure permissions」を起点に、Claudeが自動生成した教材です。

## 概要

permission は「Claude Code が**何をしてよく、何をしてはいけないか**」を細かく決める仕組み。CLAUDE.md はガイド（強制ではない）だが、permission は **Claude Code（クライアント）が強制する** 実行制御の本体。プロンプトや CLAUDE.md で何を言われても、permission が許さなければ実行されない。

## 公式 docs に沿った解説

### 3 種類のルールと評価順

| ルール | 効果 |
|---|---|
| **allow** | 確認なしで実行を許可 |
| **ask** | 毎回確認プロンプトを出す |
| **deny** | 実行を禁止 |

評価順は **deny → ask → allow**、最初にマッチしたルールが勝つ。**deny は常に最優先**。

ツール呼び出しの大まかな自動承認規則（参考）:

| ツール種別 | 例 | 承認要否 |
|---|---|---|
| Read-only | ファイル読み取り、Grep | 不要 |
| Bash コマンド | シェル実行 | 必要（初回は "Yes, don't ask again" で恒久許可） |
| ファイル編集 | Edit/Write | 必要（セッション終了まで有効） |

#### deny の 2 種類の挙動

- **裸のツール名**（`Bash`）→ ツールごと Claude の context から消える（Claude が存在を知らない）
- **スコープ付き**（`Bash(rm *)`）→ ツールは使えるが、マッチする呼び出しだけブロック

### permission ルール構文

ルール書式: `Tool` または `Tool(specifier)`

#### ツール全体をマッチ

| ルール | 効果 |
|---|---|
| `Bash` | 全 Bash コマンドにマッチ |
| `WebFetch` | 全 WebFetch にマッチ |
| `Read` | 全ファイル読み取りにマッチ |

`Bash(*)` は `Bash` と等価（全 Bash コマンドにマッチ）。

#### 指定子で細かく制御

| ルール | 効果 |
|---|---|
| `Bash(npm run build)` | 完全一致 `npm run build` のみ |
| `Read(./.env)` | cwd 直下の `.env` ファイルの読み取り |
| `WebFetch(domain:example.com)` | example.com へのフェッチのみ |

### Bash のワイルドカード詳細

`*` は任意位置に置ける。**スペースの有無が語境界を決める**:

- `Bash(ls *)` → `ls -la` にマッチするが `lsof` には**しない**（語境界あり）
- `Bash(ls*)` → `lsof` にも**マッチ**（語境界なし）
- `Bash(ls:*)` は `Bash(ls *)` と等価（末尾限定の書き方）
- `*` 1 つで複数の引数をまたいでマッチする。`Bash(git *)` は `git log --oneline --all` にもマッチ

#### 複合コマンドのルール

Claude Code はシェル演算子（`&&`/`||`/`;`/`|`/`|&`/`&`/改行）を認識し、各サブコマンドを**独立に**評価する。`Bash(safe-cmd *)` は `safe-cmd && other-cmd` を許可しない（各サブコマンドが別々にマッチ必須）。

#### プロセスラッパーの剥がし

`timeout`/`time`/`nice`/`nohup`/`stdbuf`/bare `xargs` は照合前に剥がされる。`Bash(npm test *)` は `timeout 30 npm test` にもマッチ。ただし `devbox run`/`npx`/`docker exec` 等の**環境ランナーは剥がされない** → `Bash(devbox run *)` は `devbox run rm -rf .` まで許可してしまう。ランナー+内部コマンドを明示するのが安全: `Bash(devbox run npm test)`。

#### read-only コマンドの自動許可

`ls`/`cat`/`echo`/`grep`/`find`/`cd`/`head`/`tail` 等の read-only コマンドは全モードで自動承認される（設定不要）。

### Read/Edit のパスパターン（gitignore 仕様）

| パターン | 意味 | 例 |
|---|---|---|
| `//path` | **絶対**パス（FS ルート） | `Read(//Users/alice/secrets/**)` |
| `~/path` | **ホーム**から | `Read(~/Documents/*.pdf)` |
| `/path` | **プロジェクトルート**から | `Edit(/src/**/*.ts)` |
| `path` / `./path` | **cwd**から | `Read(*.env)` |

> ⚠️ **`/Users/alice/file` は絶対パスではない**。プロジェクトルート基準になる。絶対パスは `//Users/alice/file`。

- `*` は単一ディレクトリ内のみ、`**` はディレクトリをまたいで再帰マッチ
- シンボリックリンク: allow は symlink とその解決先の**両方が**マッチ必要。deny は**いずれか**がマッチで適用

### ツール別ルール

#### WebFetch

- `WebFetch(domain:example.com)` → example.com へのフェッチのみ許可

#### MCP ツール

- `mcp__puppeteer` → `puppeteer` サーバーのツール全体
- `mcp__puppeteer__*` → ワイルドカードで全ツール（同等）
- `mcp__puppeteer__navigate` → 特定ツールのみ

allow でツール名にワイルドカードを使う場合は `mcp__<server>__` のリテラルプレフィックスが必須（サーバー名はワイルドカード不可）。deny/ask はツール名全体でのワイルドカード（`mcp__*` 等）が使える。

#### Agent（subagent）

- `Agent(Explore)` → Explore subagent
- `Agent(Plan)` → Plan subagent
- `Agent(my-custom-agent)` → カスタム subagent

deny でサブエージェントを無効化: `{"deny": ["Agent(Explore)"]}`

#### Cd（/cd コマンド）

`Cd` ルールは `/cd` コマンド（モデルが直接呼ぶ tool ではない）の対象ディレクトリを制御する。deny で `/cd` 自体を無効化、または特定パスへの移動を禁止できる。

### hooks による permission 拡張

`PreToolUse` hook は permission プロンプトの**前**に走り、deny/allow を動的に判定できる。ただし hook は deny ルールを**迂回しない**（deny-first は保たれる）。exit code 2 でブロックすると allow ルールよりも優先してツール呼び出しを止められる。

### 作業ディレクトリの拡張

デフォルトは Claude 起動ディレクトリのみアクセス可能。以下で拡張できる:

- 起動時: `--add-dir <path>`
- セッション中: `/add-dir`
- 永続設定: settings の `additionalDirectories`

ディレクトリ変更（設定ルートごと切り替え）は `/cd`（v2.1.169+）。

### 優先順位とマージ

permission は全スコープで**マージ**:

```
Managed > CLI 引数 > Local > Project > User
```

**どこかで deny されたら、他のどのレベルでも allow できない**。managed の deny は `--allowedTools` でも覆せない。

### Managed 設定での強制

| 設定 | 効果 |
|---|---|
| `allowManagedPermissionRulesOnly: true` | user/project の allow/ask/deny を無効化 |
| `permissions.disableBypassPermissionsMode: "disable"` | bypassPermissions モードを組織封印 |
| `permissions.disableAutoMode: "disable"` | auto モードを組織封印 |

### パーミッションとサンドボックスの関係

- **permission**: どのツールを使えるか・どのファイル/ドメインにアクセスできるかの制御（全ツール対象）
- **sandboxing**: Bash ツールとその子プロセスへの OS レベルの FS/ネットワーク制限

両方を組み合わせて多層防御になる。

## 重要ポイント

- **permission はクライアントが強制**。CLAUDE.md/プロンプトでは覆せない
- 評価順 **deny → ask → allow**、最初のマッチが勝つ。**deny 最優先**
- 裸の `Bash`（deny）= context から消す / `Bash(rm *)`（deny）= 使えるが該当だけブロック
- Bash ワイルドは**スペースで語境界**、複合コマンドは各部マッチ必須
- 環境ランナー（devbox/npx/docker）は剥がされず危険 → ランナー+内部コマンドを明示
- Read/Edit パスは gitignore 仕様。**`/path` はプロジェクトルート、絶対は `//path`**
- 全スコープ**マージ + deny 最優先** → 「一度 deny したら誰も allow できない」
- URL 引数で curl を制限するのは脆い → `Bash(curl *)` を deny して `WebFetch(domain:...)` を使う

## コード例 / 図

### npm/git commit は許可、git push は禁止

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)", "Bash(git commit *)", "Bash(git * main)"],
    "deny":  ["Bash(git push *)"]
  }
}
```

### .env や secrets を読ませない

```json
{
  "permissions": {
    "deny": ["Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]
  }
}
```

### MCP ツール全体を無効化

```json
{
  "permissions": {
    "deny": ["mcp__*"]
  }
}
```

### eval/exec 系は都度確認（ask）、それ以外は許可

```json
{
  "permissions": {
    "ask":  ["Bash(eval *)", "Bash(exec *)"],
    "allow": ["Bash(npm run *)"]
  }
}
```

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## 次のトピック

- [06-permission-modes/](../06-permission-modes/index.md) — permission モード（6 種類）

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
