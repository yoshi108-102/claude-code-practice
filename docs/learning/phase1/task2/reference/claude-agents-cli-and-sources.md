# `claude agents` CLI と subagent の source 階層

出典:
- [CLI reference](https://code.claude.com/docs/en/cli-reference)
- [Sub-agents](https://code.claude.com/docs/en/sub-agents)

**Phase 1 / Task 2 の lesson 中、ユーザー Q「`claude agents` について教えてほしい」から派生。**
**subagent の本格扱いは Phase 3。ここでは「`claude agents` CLI が何を見ているか」だけ押さえる目的。**

## 議論のきっかけ

cli-reference 教材の主要サブコマンド一覧で `claude agents` が登場。「何をするコマンド？」「`/agents` スラッシュコマンドと何が違う？」を整理した。

## Q&A

### Q1. `claude agents` は何をする？

**A.** インタラクティブセッションを起動せずに、設定済みの subagent を**一覧表示**する読み取り専用 CLI サブコマンド。

公式 docs（[Sub-agents > Use the /agents command](https://code.claude.com/docs/en/sub-agents)）の定義:

> To list all configured subagents from the command line without starting an interactive session, run `claude agents`. This shows agents grouped by source and indicates which are overridden by higher-priority definitions.

→ **CI / スクリプトから「どんな subagent が見えてるか」を確認する用途**にハマる。

### Q2. 「source」とは何か。どんな種類がある？

**A.** subagent を定義できる場所が複数あり、それぞれ source として認識される。`claude agents` はこの source ごとに**グループ化**して表示する。

| 優先度 | Source | パス / 経路 | スコープ |
|---|---|---|---|
| 0（特別） | **Built-in** | Claude Code 同梱（`Explore` / `Plan` / `general-purpose`） | 常に利用可 |
| 1（最高） | Managed settings | `.claude/agents/`（[managed settings](https://code.claude.com/docs/en/settings#settings-files) 内、組織管理） | エンタープライズ全体 |
| 2 | `--agents` CLI flag | 起動時に JSON で渡す | このセッションのみ |
| 3 | `.claude/agents/` | プロジェクトリポジトリ内 | このプロジェクト |
| 4 | `~/.claude/agents/` | ホーム | 自分の全プロジェクト |
| 5（最低） | Plugin の `agents/` | プラグイン同梱 | プラグイン有効時 |

**衝突時のルール**: 同名なら**優先度の高い定義が勝つ**。`claude agents` は何が override されているかも表示する。

### Q3. `claude agents`（CLI）と `/agents`（スラッシュコマンド）の違いは？

**A.** 役割が分離している。

| 観点 | `claude agents` | `/agents` |
|---|---|---|
| 実行場所 | ターミナル（**セッション外**） | セッション内 |
| 機能 | **読み取り専用の一覧表示** | **管理 UI**（Running タブ + Library タブ） |
| 用途 | 監査・スクリプトでの確認 | 対話的に作る・編集・止める |

→ **CLI 版 = 「外から見る窓」、スラッシュ版 = 「中で操作するパネル」**。

`/agents` の Library タブは（公式 docs より）:

> View all available subagents (built-in, user, project, and plugin)
> Create new subagents with guided setup or Claude generation

## 結論 / 押さえるポイント

- `claude agents` は「**source ごとにグルーピングされた subagent 一覧**」を見るための非対話 CLI
- subagent の定義場所は **Built-in を除いて 5 階層**、衝突は優先度で解決
- 一覧操作は CLI、作成・編集・削除はセッション内 `/agents`
- subagent の作り方や frontmatter は **Phase 3 で本格扱い**。今は「どこに置けば誰に見えるか」だけ理解すれば十分

## 関連

- 教材: [01-cli-reference.md](../01-cli-reference.md)
- 関連 reference: [plan-mode-vs-plan-subagent.md](plan-mode-vs-plan-subagent.md) — plan mode と Plan subagent の区別
- 復習キュー: Phase 1 / Task 1 — Q4「subagent の context window」
