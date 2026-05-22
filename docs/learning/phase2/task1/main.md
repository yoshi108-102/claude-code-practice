# Phase 2 / Task 1: 記憶系（CLAUDE.md / auto memory / .claude）

## 今回読む docs

- [memory](https://code.claude.com/docs/en/memory) — CLAUDE.md 階層・rules・auto memory
- [claude-directory](https://code.claude.com/docs/en/claude-directory) — `.claude` ディレクトリの全体像

## 進め方

**ハンズオン中心**（ユーザー希望）。教材は実践寄りのリファレンスとして置き、メインはこのリポジトリで実際に手を動かす:
- このプロジェクトの `CLAUDE.md` を実用水準に監査・改善
- `.claude/` の commit / gitignore の線引きを実地で点検
- auto memory（`~/.claude/projects/<project>/memory/`）を実物で確認（復習キュー Task1-Q1 の回収）

## 目次

- [01-claude-md-hierarchy-and-rules.md](01-claude-md-hierarchy-and-rules.md) — CLAUDE.md の階層と `.claude/rules/` [done][quiz][graded]
- [02-auto-memory.md](02-auto-memory.md) — auto memory（MEMORY.md）の仕組み [done][quiz][graded]
- [03-claude-directory-map.md](03-claude-directory-map.md) — `.claude` ディレクトリの全体地図 [done][quiz][graded]

> 解説形式: トップダウン講義（ハンズオンは途中で中止）。お題①（auto memory 実物確認）・②（gitignore 修正）は実施済み。

## reference（深掘り Q&A）

（lesson 中の Q&A から自動生成されます）

## 振り返り（Task まとめクイズ）

（採点済み: 2026-05-22 — Topic: ✅4/⚠️1/❌1、まとめ: ✅1/⚠️1。02-Q1・03-Q2・まとめQ1 を復習キューへ）

各小単元のクイズは各教材 md 内（`## 振り返りクイズ`）にあります。ここには **Topic を横断する論点**だけを置きます。
回答は `**回答**:` 行の下に記入し、`/learning-flow:grade --summary` で採点します。

---

### Q1. 「どこに書くか」の統合判断

次の4つの指示を、それぞれ最適な置き場所（CLAUDE.md / path-scoped rule / skill / hook）に振り分け、理由を一言添えよ。これは記憶系の全 Topic（CLAUDE.md・rules・hook の役割分担）を横断する問い。

- (あ) 「コミット前に必ず `make lint` を走らせる」
- (い) 「このプロジェクトは 2-space インデント」
- (う) 「`src/api/` 配下では全エンドポイントに入力バリデーション必須」
- (え) 「リリースノートを生成する10ステップの手順」

**参考**:
- [Memory > When to add / where to put](https://code.claude.com/docs/en/memory)
- [Hooks guide](https://code.claude.com/docs/en/hooks-guide)

**関連ノート**: [01-claude-md-hierarchy-and-rules.md](01-claude-md-hierarchy-and-rules.md)

**回答**:
あ: 必ずコマンドを実行したいのでhookに書く必要がある。
い: あにそういうlint ruleを追記すればいいわけなので、同様にしてhook
う: これについてはpath-scoped ruleに記載する
え: ルールベースの話ではないのでclaude.mdに書くべき。もちろんimportとかしてfileを分けてもいいが
---

### Q2. CLAUDE.md と auto memory の使い分け（横断）

同じ「Claude に覚えておいてほしいこと」でも、CLAUDE.md に書くべきものと auto memory に委ねるべきものは異なる。両者を「誰が書くか」「チーム共有されるか」の2軸で対比し、「チーム全員に効かせたい恒久ルール」と「自分のこのマシンでの発見」がそれぞれどちらに行くべきか説明せよ。

**参考**:
- [Memory > CLAUDE.md vs auto memory](https://code.claude.com/docs/en/memory#claude-md-vs-auto-memory)

**関連ノート**: [01-claude-md-hierarchy-and-rules.md](01-claude-md-hierarchy-and-rules.md), [02-auto-memory.md](02-auto-memory.md)

**回答**:
CLAUDE.mdに書くべきなのはチーム全体のルールであり共有されるべきルールである。なので1つ目のチーム全体ルールはこっちに書くのが良い
auto memoryについては自分のマシンでの発見を描くものなので個人管理、共有はしない