# 02. compaction で何が残るか / context を見る道具

> 出典: [Explore the context window > What survives compaction / Check your own session](https://code.claude.com/docs/en/context-window#what-survives-compaction)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「Explore the context window」の後半（compaction とセルフチェック）を起点に Claude が自動生成した教材です。

## 概要

長いセッションで context が埋まってくると、`/compact` が会話履歴を**構造化された要約に置き換えて**空きを作る。重要なのは「**何が要約で消え、何がディスクから再注入されるか**」がロード方法ごとに違うこと。ここを理解しないと「compact したら CLAUDE.md のルールが効かなくなった？」のような誤解が生まれる。

## 公式docsに沿った解説

### `/compact` が何をするか

`/compact` は会話履歴を AI 生成の**構造化要約**で置換する。要約が保持するもの:

- あなたの要求・意図
- 主要な技術概念
- 調査/変更したファイルと重要なコードスニペット
- 発生したエラーとその修正
- 保留中のタスク・現在の作業

逆に**失われるもの**: ツール出力の全文・途中の推論。Claude は作業内容を参照できるが、「earlier に読んだ正確なコード」は持っていない状態になる。

> ターミナルには「Conversation compacted」と出るだけ。要約処理自体は表示されない。

### What survives compaction（compaction を生き残るもの）

ロード方法ごとに compaction 後の運命が違う。**これが本章の核心**。

| メカニズム | compaction 後 |
|---|---|
| System prompt / output style | **変わらず**（そもそもメッセージ履歴の外） |
| プロジェクトルート CLAUDE.md / 無スコープ rule | **ディスクから再注入** |
| Auto memory | **ディスクから再注入** |
| `paths:` frontmatter 付き rule | **失われる**（マッチするファイルを再度読むまで） |
| サブディレクトリのネスト CLAUDE.md | **失われる**（そのディレクトリのファイルを再度読むまで） |
| invoke 済み skill 本体 | **再注入**（skill あたり 5,000 tokens / 合計 25,000 tokens 上限。超過分は古いものから削除） |
| Hooks | 該当なし（hook はコードとして実行、context ではない） |

ポイント:

- **「常に効いてほしい」ルールは `paths:` を外す or ルートの CLAUDE.md に置く**。path-scoped rule は便利だが compaction で消え、対象ファイルを読み直すまで復活しない
- skill 本体は再注入されるが**先頭優先で切り詰め**られる → 重要な指示は `SKILL.md` の**先頭に**置く（Phase 3 の skill 作成で効いてくる）

### Skill descriptions は例外（生き残らない）

起動時にロードされる中で **skill の1行説明だけは compaction 後に再注入されない**。実際に invoke した skill のみが保持される。「compact 後に Claude が使える skill が減ったように見える」のはこのため。

### Check your own session（自分のセッションを確認する）

シミュレーションの数値は代表値。実際の使用量は次で確認:

| コマンド | 何が見える |
|---|---|
| `/context` | カテゴリ別のライブ内訳 + **最適化提案** |
| `/memory` | 起動時にどの CLAUDE.md / auto memory がロードされたか |

## 重要ポイント

- `/compact` = 会話を**構造化要約に置換**（`/clear` は捨てる、の対比は Task 2 で既習）
- compaction 後の運命は**ロード方法依存**:system prompt は不変、ルート CLAUDE.md / auto memory は再注入、**path-scoped rule とネスト CLAUDE.md は消える**
- 「常時効かせたいルール」は `paths:` を外すかルート CLAUDE.md へ
- skill 本体は再注入だが **5K/25K token 上限 + 先頭優先で切り詰め** → 重要指示は先頭に
- **skill の説明一覧は compaction を生き残らない**（invoke 済みのみ保持）
- 困ったら `/context`（内訳 + 提案）と `/memory`（ロード確認）

## コード例 / 図

### compaction を生き残るかどうかの早見

```text
生き残る（再注入される）:
  System prompt / output style   … そもそも履歴外
  ルート CLAUDE.md / 無スコープ rule … ディスクから再注入
  Auto memory                    … ディスクから再注入
  invoke 済み skill 本体          … 再注入（5K/25K 上限・古いもの削除）

消える（条件付き復活）:
  paths: 付き rule               … マッチするファイル再読込まで
  ネスト CLAUDE.md               … そのディレクトリのファイル再読込まで
  skill の説明一覧               … 復活しない（invoke 済みのみ残る）
```

### context が重くなってきたときの手順

```text
1. /context        … 何が context を食っているか内訳を見る
2. （重い調査が原因なら）subagent に委譲
3. /compact focus on the schema decisions  … 要約して空きを作る（フォーカス指示も可）
4. タスクが完全に切り替わるなら /clear
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 前の教材: [01-context-window-anatomy.md](01-context-window-anatomy.md) — 何が context を埋めるか
- 次の教材: [03-checkpointing.md](03-checkpointing.md) — checkpoint で巻き戻す
- 関連 docs: [Memory > path-specific rules](https://code.claude.com/docs/en/memory#path-specific-rules) / [Reduce token usage](https://code.claude.com/docs/en/costs#reduce-token-usage) / [Prompt caching](https://code.claude.com/docs/en/prompt-caching)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_
