# 01. context window の解剖 — 何が、いつロードされるか

> 出典: [Explore the context window](https://code.claude.com/docs/en/context-window)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「Explore the context window」（インタラクティブシミュレーション + 解説）を起点に Claude が自動生成した教材です。

## 概要

context window は「Claude がこのセッションについて知っていること**すべて**」を保持する固定サイズの入れ物（イラスト上は 200K tokens）。あなたの指示・読んだファイル・Claude 自身の応答に加えて、**ターミナルには一切表示されない**コンテンツも大量に含む。

docs はこの中身を「`claude` 起動からの時間軸」でシミュレーションし、「何がいつロードされ、いくらコストするか」を見せる。重要な認識:**自分が打ったプロンプトは context のごく一部**で、大半はプロジェクト知識（CLAUDE.md・メモリ・ツール定義・ファイル読み込み）である。

## 公式docsに沿った解説

### Before you type anything（最初のプロンプト前にロードされるもの）

`claude` を起動して**あなたが1文字も打つ前に**、以下が順に context へ入る。多くは**ターミナルに表示されない（invisible）**。

| ロードされるもの | 役割 | 表示 |
|---|---|---|
| **System prompt** | 振る舞い・ツール使用・応答整形の中核指示。常に最初にロード | 不可視 |
| **Auto memory (MEMORY.md)** | 前セッションからの Claude の自分用メモ。**先頭 200 行 or 25KB**（先に達した方）まで | 不可視 |
| **Environment info** | cwd / platform / shell / OS / git か否か。git の branch・status・直近コミットは system prompt 末尾に別ブロック | 不可視 |
| **MCP tools（deferred）** | MCP ツール**名**のみ列挙。スキーマは既定で遅延ロード（必要時に tool search で取得）。`ENABLE_TOOL_SEARCH` で挙動変更 | 不可視 |
| **Skill descriptions** | 利用可能 skill の1行説明。本体は invoke 時のみロード。`disable-model-invocation: true` の skill はこの一覧に出ない | 不可視 |
| **~/.claude/CLAUDE.md** | 全プロジェクト共通のユーザー設定 | 不可視 |
| **Project CLAUDE.md** | プロジェクト規約・ビルドコマンド・設計メモ。**最重要ファイル** | 不可視 |

> docs の Tip: Project CLAUDE.md は**200 行以内**に保ち、参照系の内容は skill や path-scoped rule に逃がして「必要なときだけロード」させると context を節約できる。

### As Claude works（Claude が作業する中で増えるもの）

あなたがプロンプトを送ると、そこから先は実作業に応じて context が増える。

- **Your prompt**: 実は非常に小さい（イラストでは 45 tokens）。「すでにロード済みのプロジェクト知識」と比べると微々たるもの
- **File reads**: Claude がファイルを読むたびに加算。**context 使用量を最も食うのはファイル読み込み**。ターミナルには `Read auth.ts` の1行しか出ないが、内部では数千 tokens が入っている
- **Path-scoped rules**: `.claude/rules/` 配下で `paths:` パターンを持つ rule は、**マッチするファイルを読んだ瞬間に自動ロード**される（例: `src/api/**` にマッチする rule）
- **Hooks**: `PostToolUse` 等の hook が編集のたびに発火。hook の出力は**`hookSpecificOutput.additionalContext` 経由でのみ context に入る**（exit 0 の素の stdout は debug log 行きで context には入らない）

> docs の Tip（ファイル読み込み）: プロンプトを具体的に（「auth.ts のバグを直して」）すると Claude が読むファイルが減る。調査が重いタスクは **subagent に委譲**して、大量のファイル読み込みをメイン context の外に出す。

### Subagent — 別の context window

フォローアップで「subagent を使って調べて」と頼むと、Claude は**まっさらな別 context window**を持つ subagent に委譲する。

- subagent は**自分用の短い system prompt** + CLAUDE.md（自前のコピー）+ 同じ MCP/skill セットアップを持つ
- ただし**メインの会話履歴・auto memory は引き継がない**（組み込み Explore/Plan は CLAUDE.md すらスキップしてさらに軽量）
- subagent が 6,100 tokens 分のファイルを読んでも、**メインに返るのは最終要約（イラストでは 420 tokens）+ 小さなメタデータだけ**。これが context 節約の本質

> これは Phase 1 / Task 1・Task 2 で繰り返し出た「subagent = 別 context で広く読んで要約だけ返す」と完全に一致する。context window の図で見ると効果が一目瞭然。

### `!`（bang/shell mode）と user-only skill

- **`!git status`**: shell mode で実行したコマンドと出力が**あなたのメッセージの一部として context に入る**。Claude にコマンドを実行させずに、結果を context へ持ち込みたいときに有用（Task 2 の shell mode と直結）
- **user-only skill（`disable-model-invocation: true`）**: 起動時の skill 一覧に説明が**載らない**ので、invoke する瞬間まで context コスト**ゼロ**。コミット・デプロイ・送信など副作用のある skill に付けるべき設定

## 重要ポイント

- context window は「会話」だけでなく、**起動時に大量に積まれる不可視コンテンツ（system prompt / memory / CLAUDE.md / tool 定義 / skill 説明）** を含む
- **自分のプロンプトは context のごく一部**。大半はプロジェクト知識とファイル読み込み
- **ファイル読み込みが最大の消費源** → プロンプトを具体的にする / 重い調査は subagent へ
- **表示（ターミナル）≠ context**。`Read auth.ts` の1行裏で数千 tokens が入っている。「見えないコスト」を意識する
- path-scoped rule は「マッチするファイルを読んだとき」だけ自動ロードされる（常時ではない）
- hook 出力が context に入るのは `additionalContext` 経由のときだけ
- 起動時ロードを丸ごと省きたいなら Task 2 で出た `--bare` フラグ（hook/skill/plugin/MCP/memory/CLAUDE.md を全部スキップ）

## コード例 / 図

### context を埋める順序（起動 → 作業 → compact）

```text
[起動時・不可視]
 System prompt → Auto memory → Env info → MCP tool 名 → Skill 説明
 → ~/.claude/CLAUDE.md → Project CLAUDE.md
[あなたのプロンプト（小）]
 → Claude のファイル読み込み（大）→ path-scoped rule 自動ロード → hook
[フォローアップ]
 → subagent（別 context で読みまくる）→ 要約だけメインに返る
[!git status] → コマンド+出力が context へ
[/compact] → 会話を構造化要約に置換（次の教材 02）
```

### 自分の context を確認するコマンド

```text
/context   # カテゴリ別のライブ内訳 + 最適化提案
/memory    # 起動時にどの CLAUDE.md / auto memory がロードされたか
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 次の教材: [02-compaction-and-context-tools.md](02-compaction-and-context-tools.md) — /compact で何が残るか・/context / /memory
- 関連 docs: [Memory](https://code.claude.com/docs/en/memory) / [Subagents](https://code.claude.com/docs/en/sub-agents) / [Hooks guide](https://code.claude.com/docs/en/hooks-guide)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_
