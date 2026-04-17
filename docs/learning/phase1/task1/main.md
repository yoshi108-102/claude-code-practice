# Phase 1 - Task 1: 概論 & クイックスタート

## 今回読んだ docs

- [overview.md](https://code.claude.com/docs/en/overview) — Claude Code の全体像
- [how-claude-code-works.md](https://code.claude.com/docs/en/how-claude-code-works) — agentic loop、ツール、セッション、権限
- [quickstart.md](https://code.claude.com/docs/en/quickstart) — 初回セットアップから最初のコード変更まで
- [best-practices.md](https://code.claude.com/docs/en/best-practices) — 効果的に使うためのパターン集

## トピック別ノート

- [01-agentic-loop-and-tools.md](./01-agentic-loop-and-tools.md) — agentic loop の 3 フェーズと tool カテゴリ
- [02-sessions-and-context.md](./02-sessions-and-context.md) — セッション・コンテキスト管理・checkpoint
- [03-permissions-and-safety.md](./03-permissions-and-safety.md) — 4 つの permission mode と安全機構
- [04-best-practices-essentials.md](./04-best-practices-essentials.md) — 効果的なプロンプトと CLAUDE.md の書き方

## リファレンス（深掘り / 脱線トピック）

- [reference/remote-control.md](./reference/remote-control.md) — 別デバイスからローカルセッション操縦（Phase 4 で本格扱い）
- [reference/auto-memory.md](./reference/auto-memory.md) — Claude が自分で書き貯める memory（Phase 2 で本格扱い）
- [reference/subagent-memory.md](./reference/subagent-memory.md) — subagent ごとの独立 memory と context 分離（Phase 3 で本格扱い）

## 振り返り（Task 1 一旦まとめ）

### 主要概念の整理

- **agentic loop**: gather context → take action → verify results のブレンド。Claude 自身が次手を決めてチェーンする
- **5 tool カテゴリ**: File ops / Search / Execution / Web / Code intelligence。skills/MCP/hooks/subagents はこの上の拡張層
- **context window = 最重要制約**: 埋まるほど性能が落ちる。`/context`, `/compact`, `/clear`, subagents, skills で管理
- **2 つの安全機構**: checkpoint（file 編集の自動スナップショット、Esc×2 or `/rewind`）と permission（4 モード、`Shift+Tab`）
- **黄金ルール**: 探索(Plan mode) → 計画 → 実装 → コミット。小さい変更は直接、複雑変更は Plan mode で

### 印象に残ったベストプラクティス

1. **Claude に検証手段を渡せ**（テスト、スクショ、期待出力）— "最も高レバレッジ"
2. **subagent で context を汚さず調査**させる
3. **2 回修正しても直らなければ `/clear` して再スタート**、長セッションにこだわらない
4. **CLAUDE.md は "消してミスが出るか？" で判断**。長くすると無視されるので剪定が命

### 今日学んだ脱線トピック（reference に保存済み）

| 脱線 | 本格扱い |
|---|---|
| Remote Control（スマホから続き） | Phase 4 |
| auto memory（Claude が書くメモ） | Phase 2 |
| subagent memory（agent ごとの独立メモ） | Phase 3 |

### 次 Task に持ち越す疑問・未実施の演習

- **Task 1 の "hands-on" パート未実施**: `/init` で CLAUDE.md を育てる、小タスク実行、checkpoint 巻き戻し体験 — 次回セッションで実施
- Task 2（CLI 基本操作）で試したい: `/context` を眺めながら長セッションの context がどう増えるか観察

### 読んだ公式 docs（Phase 1 Task 1 担当分）

- [x] [overview](https://code.claude.com/docs/en/overview)
- [x] [how-claude-code-works](https://code.claude.com/docs/en/how-claude-code-works)
- [x] [quickstart](https://code.claude.com/docs/en/quickstart)
- [x] [best-practices](https://code.claude.com/docs/en/best-practices)

### 読んだ公式 docs（脱線で先取り / 軽く触れた分）

- [ ] [remote-control](https://code.claude.com/docs/en/remote-control) — 要点のみ、Phase 4 で再読
- [ ] [memory](https://code.claude.com/docs/en/memory) — auto-memory セクションのみ、Phase 2 で全体再読
- [ ] [sub-agents](https://code.claude.com/docs/en/sub-agents) — memory 部分のみ、Phase 3 で全体再読
