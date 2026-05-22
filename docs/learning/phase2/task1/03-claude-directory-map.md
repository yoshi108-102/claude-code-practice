# 03. `.claude` ディレクトリの全体地図

> 出典: [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「claude-directory」の構造を起点に Claude が自動生成した教材です。

## 概要

`.claude/` は Claude Code が設定・ルール・拡張を読む場所。**プロジェクト直下の `.claude/`**（チーム共有）と **ホームの `~/.claude/`**（自分の全プロジェクト）の2系統がある。実務で大事なのは「**どれを git に commit し、どれを gitignore するか**」の線引き。

## 公式docsに沿った解説

### プロジェクト直下（リポジトリルート）

| パス | 役割 | git |
|---|---|---|
| `CLAUDE.md` | 毎セッション読まれるプロジェクト指示 | **committed** |
| `.mcp.json` | プロジェクトスコープの MCP サーバー（チーム共有） | **committed** |
| `.worktreeinclude` | 新 worktree にコピーする gitignored ファイル指定 | **committed** |
| `.claude/settings.json` | 権限・hook・設定 | **committed** |
| `.claude/settings.local.json` | 個人の設定上書き | **gitignored** |
| `.claude/rules/*.md` | トピック別指示（`paths:` で局所化可） | **committed** |
| `.claude/skills/<name>/SKILL.md` | 名前で呼ぶ再利用プロンプト（補助ファイルも同梱可） | **committed** |
| `.claude/commands/*.md` | 旧コマンド（新規は skills 推奨。同じ `/name` 起動） | committed |
| `.claude/output-styles/` | プロジェクト共有の output style | committed |
| `.claude/agents/<name>.md` | 専用 subagent（別 context window） | **committed** |
| `.claude/agents/agent-memory/<name>/MEMORY.md` | subagent の永続メモリ（メイン auto memory とは別） | committed |

> docs の注記: **commands と skills は同じ仕組みになった**。新規ワークフローは `skills/` を使う（同じ `/name` 起動 + 補助ファイル同梱可）。

### ホーム `~/.claude/`（ユーザーレベル）

- `~/.claude/CLAUDE.md` — 全プロジェクト共通の個人指示
- `~/.claude/settings.json` — 個人のグローバル設定（`autoMemoryDirectory` 等はここだけ受理）
- `~/.claude/rules/`, `~/.claude/skills/`, `~/.claude/agents/` — 全プロジェクトに効く個人版
- `~/.claude/projects/<project>/memory/` — auto memory（リポジトリ単位、教材02）
- `~/.claude.json` — アプリ状態・UI 設定（local、commit しない）

### commit / gitignore の線引き(実務の肝)

- **commit**: `CLAUDE.md`, `.claude/settings.json`, `.claude/rules/`, `.claude/skills/`, `.claude/agents/`, `.mcp.json` — チームで共有したい設定・拡張
- **gitignore**: `CLAUDE.local.md`, `.claude/settings.local.json`, `.claude/worktrees/` — 個人・マシン固有・一時的なもの
- 原則:「**チーム全員に効かせたいか / 自分だけか**」で分ける。`*.local.*` と `worktrees/` は基本 gitignore

## 重要ポイント

- `.claude/` は2系統:**project（チーム共有・git）** と **`~/.claude/`（自分の全プロジェクト）**
- **committed と gitignored を意識して分ける**。`settings.json`=共有 / `settings.local.json`=個人、`CLAUDE.md`=共有 / `CLAUDE.local.md`=個人
- commands は旧式、**新規は skills**（同じ `/name`、補助ファイル同梱可）
- subagent は `.claude/agents/`、その永続メモリは `agent-memory/<name>/MEMORY.md`（メイン auto memory と別物）
- `.claude/worktrees/` と `*.local.*` は gitignore が原則(Task 2 の worktree 学習と接続)

## コード例 / 図

### 典型的な `.claude/` レイアウト

```text
your-project/
├── CLAUDE.md                      # committed
├── CLAUDE.local.md                # gitignored（個人）
├── .mcp.json                      # committed
└── .claude/
    ├── settings.json              # committed（権限・hook）
    ├── settings.local.json        # gitignored（個人上書き）
    ├── rules/
    │   ├── testing.md             # committed（paths: でテストファイルに限定）
    │   └── api-design.md          # committed
    ├── skills/<name>/SKILL.md     # committed
    ├── agents/<name>.md           # committed
    └── worktrees/                 # gitignored
```

### .gitignore の最小テンプレ

```gitignore
CLAUDE.local.md
.claude/settings.local.json
.claude/worktrees/
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 前の教材: [02-auto-memory.md](02-auto-memory.md)
- 関連 docs: [Settings](https://code.claude.com/docs/en/settings) / [Skills](https://code.claude.com/docs/en/skills) / [Subagents](https://code.claude.com/docs/en/sub-agents)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_

## 振り返りクイズ

回答は各問の `**回答**:` 行の下に記入してください。
全問記入後に `/learning-flow:grade 03-claude-directory-map` で採点します。

---

### Q1. commit / gitignore の線引き

`.claude/` まわりで **git に commit すべきファイル**と **gitignore すべきファイル**を、それぞれ2つずつ挙げよ。また、その線引きを決める原則を一言で述べよ。

**参考**:
- [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory)

**回答**:
commitすべきなのはプロジェクトレベルでのclaude.mdやsettings.jsonなど、チーム単位で共有すべきもので、
ignoreすべきなのはClaude.local.jsonとかsettings.local.jsonみたいな個人で使うもの
---

### Q2. project の `.claude/` と user の `~/.claude/` の違い

同じ `settings.json` でも `プロジェクト/.claude/settings.json` と `~/.claude/settings.json` では役割が違う。それぞれのスコープ（誰に効くか）を説明し、`autoMemoryDirectory` のような設定が**なぜ user settings からしか受け付けられないのか**（project/local 不可の理由）を述べよ。

**参考**:
- [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory)
- [Memory > Storage location](https://code.claude.com/docs/en/memory#storage-location)

**回答**:
前者はプロジェクト単位で後者はユーザ単位で効果を発揮する。
autoMemorydirectoryはプロジェクト単位で記憶を跨ぐためのものなので、当然それよりマネージメントの範囲の広いユーザレベルの設定で担当するしかない