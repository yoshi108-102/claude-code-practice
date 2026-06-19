<!-- DEPRECATED: 新版は ../05-permissions/index.md（HTML 版は index.html）-->
# 02. 権限ルール — allow / ask / deny

> 出典: [Configure permissions](https://code.claude.com/docs/en/permissions)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「permissions」の構造を起点に Claude が自動生成した教材です。

## 概要

permission は「Claude Code が**何をしてよく、何をしてはいけないか**」を細かく決める仕組み。Task 1 で繰り返した「CLAUDE.md はガイド（強制ではない）、強制したいなら permission/hook」の**強制側の本体**がこれ。

最重要の性質:**permission ルールはモデルではなく Claude Code（クライアント）が強制する**。プロンプトや CLAUDE.md で何を言われても、permission が許さなければ実行されない。

## 公式docsに沿った解説

### 3種類のルールと評価順

| ルール | 効果 |
|---|---|
| **allow** | 確認なしで実行を許可 |
| **ask** | 毎回確認プロンプトを出す |
| **deny** | 実行を禁止 |

評価順は **deny → ask → allow**、**最初にマッチしたルールが勝つ**。つまり **deny は常に最優先**。

> deny の挙動には2種類:
> - **裸のツール名**（`Bash`）→ ツールごと context から消える（Claude が存在を知らない）
> - **スコープ付き**（`Bash(rm *)`）→ ツールは使えるが、マッチする呼び出しだけブロック

### ルール構文 `Tool` / `Tool(specifier)`

- ツール名だけ → 全使用にマッチ（`Bash`, `Read`, `WebFetch`）
- 指定子付き → 特定の使用だけ（`Bash(npm run build)`, `Read(./.env)`, `WebFetch(domain:example.com)`）

### Bash のワイルドカード（ハマりどころ多数）

`*` は任意位置に置ける。**スペースの有無が決定的**:

- `Bash(ls *)` → `ls -la` にマッチするが `lsof` には**しない**（語境界）
- `Bash(ls*)` → `lsof` にも**マッチする**(語境界なし)
- `Bash(ls:*)` は `Bash(ls *)` と等価（末尾限定の書き方）

実務で効く挙動:

- **複合コマンドは各サブコマンドが独立にマッチ必須**。`Bash(safe-cmd *)` は `safe-cmd && other-cmd` を許可しない（`&&`, `||`, `;`, `|`, `&` 等を認識）
- **プロセスラッパーは剥がされる**:`timeout`, `time`, `nice`, `nohup`, `stdbuf`, 無フラグ `xargs` → `Bash(npm test *)` は `timeout 30 npm test` にもマッチ
- ただし `devbox run`, `npx`, `docker exec` 等の**環境ランナーは剥がされない** → `Bash(devbox run *)` は `devbox run rm -rf .` まで許してしまう。**ランナー+内部コマンドを明示**（`Bash(devbox run npm test)`）
- **read-only コマンド**（`ls`, `cat`, `grep`, `find`, `cd`, read-only な `git` 等）は全モードでプロンプトなし実行

> ⚠️ **引数で URL を縛るのは脆い**。`Bash(curl http://github.com/ *)` は `curl -X GET ...`、`https://`、リダイレクト、変数展開で簡単にすり抜ける。URL 制限したいなら **curl/wget を deny → WebFetch(domain:...) を使う** か **PreToolUse hook** で。

### Read / Edit のパスパターン（gitignore 仕様）

| パターン | 意味 | 例 |
|---|---|---|
| `//path` | **絶対**パス（FS ルート） | `Read(//Users/alice/secrets/**)` |
| `~/path` | **ホーム**から | `Read(~/Documents/*.pdf)` |
| `/path` | **プロジェクトルート**から | `Edit(/src/**/*.ts)` |
| `path` / `./path` | **cwd**から | `Read(*.env)` |

> ⚠️ **`/Users/alice/file` は絶対パスではない**（プロジェクトルート基準）。絶対パスは `//Users/alice/file`。ここは事故りやすい。
> Read/Edit deny は Claude の組み込みファイルツールと `cat`/`head`/`sed` 等には効くが、**Python/Node スクリプトが自前で開くファイルには効かない**。OS レベルで止めたいなら sandbox。

### ツール別ルール

- **MCP**: `mcp__puppeteer`（サーバー全体）/ `mcp__puppeteer__*` / `mcp__puppeteer__navigate`（特定ツール）
- **Agent**: `Agent(Explore)`, `Agent(Plan)`, `Agent(my-custom-agent)` で subagent を制御（deny で無効化）

### 優先順位とマージ（教材01の続き）

permission は全スコープで**マージ**され、**deny がどのスコープでも最優先**:

```
Managed > CLI 引数 > Local > Project > User
```

**どこかで deny されたら、他のどのレベルでも allow できない**。managed の deny は `--allowedTools` でも覆せない。

### hook で permission を拡張

`PreToolUse` hook は permission プロンプトの前に走り、deny/ask/allow を動的に判定できる。ただし**hook は deny/ask ルールを迂回しない**(deny-first は保たれる)。「Bash 全部 allow にしつつ、特定だけ hook でブロック」が定番パターン。

### managed 設定での強制

`allowManagedPermissionRulesOnly: true` で user/project の権限ルールを無効化し managed だけ有効に。`disableBypassPermissionsMode` / `disableAutoMode` で危険モードを封印（→ 教材03）。

## 重要ポイント

- **permission はクライアントが強制**。CLAUDE.md/プロンプトでは覆せない（Task 1 の「強制は permission へ」の本体）
- 評価順 **deny → ask → allow**、最初のマッチが勝つ。**deny 最優先**
- 裸の `Bash`（deny）= context から消す / `Bash(rm *)`（deny）= 使えるが該当だけブロック
- Bash ワイルドは**スペースで語境界**、複合コマンドは各部マッチ必須、環境ランナー（devbox/npx/docker）は剥がされず危険
- Read/Edit パスは gitignore 仕様。**`/path` はプロジェクトルート、絶対は `//path`**
- 全スコープ**マージ + deny 最優先** → 「一度 deny したら誰も allow できない」
- 引数での URL 制限は脆い → deny + WebFetch or hook で

## コード例 / 図

### npm/git commit は許可、git push は禁止

```json
{
  "permissions": {
    "allow": ["Bash(npm run *)", "Bash(git commit *)"],
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

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 前の教材: [01-settings-files-and-precedence.md](01-settings-files-and-precedence.md)
- 次の教材: [03-permission-modes.md](03-permission-modes.md) — permission モード
- 関連 docs: [Permission modes](https://code.claude.com/docs/en/permission-modes) / [Sandboxing](https://code.claude.com/docs/en/sandboxing) / [Hooks](https://code.claude.com/docs/en/hooks-guide)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_
