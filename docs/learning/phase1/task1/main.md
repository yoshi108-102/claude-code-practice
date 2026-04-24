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
- [05-auto-mode-deep-dive.md](./05-auto-mode-deep-dive.md) — Auto モード深掘り（classifier の挙動 + 復習キュー Q6 の補足）

## リファレンス（深掘り / 脱線トピック）

- [reference/remote-control.md](./reference/remote-control.md) — 別デバイスからローカルセッション操縦（Phase 4 で本格扱い）
- [reference/auto-memory.md](./reference/auto-memory.md) — Claude が自分で書き貯める memory（Phase 2 で本格扱い）
- [reference/subagent-memory.md](./reference/subagent-memory.md) — subagent ごとの独立 memory と context 分離（Phase 3 で本格扱い）

## 学習サマリ（Task 1 時点）

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

## 次 Task に持ち越す疑問・未実施の演習

- **Task 1 の "hands-on" パート未実施**: `/init` で CLAUDE.md を育てる、小タスク実行、checkpoint 巻き戻し体験 — 次回セッションで実施
- Task 2（CLI 基本操作）で試したい: `/context` を眺めながら長セッションの context がどう増えるか観察

## 読んだ公式 docs

### Phase 1 Task 1 担当分

- [x] [overview](https://code.claude.com/docs/en/overview)
- [x] [how-claude-code-works](https://code.claude.com/docs/en/how-claude-code-works)
- [x] [quickstart](https://code.claude.com/docs/en/quickstart)
- [x] [best-practices](https://code.claude.com/docs/en/best-practices)

### 脱線で先取り / 軽く触れた分

- [ ] [remote-control](https://code.claude.com/docs/en/remote-control) — 要点のみ、Phase 4 で再読
- [ ] [memory](https://code.claude.com/docs/en/memory) — auto-memory セクションのみ、Phase 2 で全体再読
- [ ] [sub-agents](https://code.claude.com/docs/en/sub-agents) — memory 部分のみ、Phase 3 で全体再読

## 振り返り（クイズ形式）

回答は各問の下に記入。採点・補足は Claude が `/learning-flow:next` 実行時に行う。
分からない問題は「分からない」「要復習」と書けば採点対象になる（復習キューに積まれる）。

### Q1. CLAUDE.md と MEMORY.md の違い

CLAUDE.md と MEMORY.md を、「**誰が書くか**」「**目的**」「**保管場所と共有範囲**」の 3 軸で比較して説明せよ。

**回答**:
CLAUDE.md はユーザ自身が記載する、claudeに対する振る舞いを定義するファイルで、MEMORY.md はclaudeが自動的に記載する、過去のやり取りをまとめたファイルである。保管場所は作業ディレクトリの直下で、共有範囲はプロジェクト単位。ただ、ホームディレクトリの.claude以下に作成した場合はすべてのプロジェクトで共有される。

### Q2. agentic loop の 3 フェーズ

agentic loop の 3 フェーズを列挙し、「ログイン機能にバグが出た」タスクで各フェーズが具体的にどう回るか、1 アクションずつ例を挙げよ。

**回答**:
gather content -> take action -> verify resultsの3つのフェーズを繰り返す。
1. gather content: 関連ファイルやエラーメッセージなどを収集する。
2. take action: コードの修正やテストの実行などを行う。
3. verify results: 修正結果を確認する。

### Q3. 5 tool カテゴリ

Claude Code の 5 tool カテゴリを全て挙げよ。また、このうち利用に **plugin が必要なもの** はどれか。

**回答**:
1. File ops: ファイルの読み書き、作成などを行う。Opsとは操作という意味。
2. Search: コードやドキュメントを検索する。
3. Execution: コードの実行やテストを行う。
4. Web: Webサイトの情報を収集する。
5. Code intelligence: コードの解析やリファクタリングを行う。

pluginが必要なものは、WebとCode intelligence。


### Q4. subagent を使う合理性

"LLM は context が埋まるほど性能が落ちる" という原則から、**subagent を使うべき場面と、その理由** を説明せよ。main session で直接調査するのと比べて何が変わるか。

**回答**:
コードにバグが発生した場合、subagent を使うべきである。なぜなら、subagent は main session よりも多くの context を持つことができるため、より多くのファイルを調査することができるからである。main session で直接調査すると、context が埋まってしまい、性能が落ちてしまう可能性がある。

### Q5. checkpoint と permission の役割の違い

checkpoint と permission は両方「安全機構」だが役割が違う。

- checkpoint は何を守るか
checkpointは会話の途中の状態を保存するもので、
- permission は何を守るか
permissionはFile ops等の操作を許可するかどうかを制御する。
- **checkpoint で戻せないもの** の例を 1 つ挙げよ

**回答**:
checkpointで戻せないものは、外部副作用を伴う操作。例えば、terraform applyを実行した場合、リモート側の状態はcheckpointでは戻せない。
### Q6. Plan モードと Auto モードの使い分け

4 つの permission mode のうち、**Plan モード** と **Auto モード** は両方「承認疲れを減らす」仕組みだが、設計思想が正反対である。

- それぞれの動作の違いは？
- どういうタスクで Plan を、どういうタスクで Auto を使うべきか？

**回答**:
Planモードは、読み取り専用toolのみ使って計画を立てることだけ行い、Autoモードは、あらゆるpermissionを許可するモードである。Planモードは、設計前の探索や、複雑なタスクで使うべきである。Autoモードは、skillでフローを定め切っている場合など、信頼できるタスクのみで使用されるべきである。

### Q7. /clear が推奨される理由

"2 回同じ修正をしても直らなければ `/clear` して再スタートせよ" が best-practices で勧められている。この助言を **context window の性能特性** から説明せよ。長セッションに粘らない方が速いのはなぜか。

**回答**:
context windowが長くなると、Claudeの性能が落ちるため。
### Q8. CLAUDE.md に何を書くべきか（判断問題）

以下の 3 つのうち、CLAUDE.md に書くべきものはどれか。理由も述べよ（"消すと Claude がミスを起こすか？" の観点で）。

1. 「テストは `npm test` で走る」
2. 「React は宣言的 UI ライブラリです」
3. 「本番 DB への直接接続は禁止。必ず staging 経由で」

**回答**:
1,3はclaude.mdに書くべきであるが、2は書くべきではない。なぜなら、2はclaudeが知っている情報だからである。1はtestの実行方法、3は本番環境への接続方法という、claudeが知らない可能性のある情報だからである。
### Q9. 本番 DB マイグレーションに何で備えるか

あなたが Claude に「本番 DB にマイグレーションを走らせる」作業を任せるとしたら、**checkpoint と permission のどちらに頼るべきか**。理由も述べよ。

**回答**:
permissionに頼るべきである。なぜなら、上にも書いたが、外部副作用のある処理はcheckpointでは戻せないからである。
### Q10. `--continue` と `--fork-session` の違い

`claude --continue` と `claude --continue --fork-session` の違いは何か。**後者を使いたくなる具体的な場面** を 1 つ挙げよ。

**回答**:
--continueは、現在のセッションを継続する。--fork-sessionは、現在のセッションを継続するが、新しいセッションとして保存される。後者を使いたくなる場面は、現在のセッションを継続するが、新しいセッションとして保存したい場合である。
具体的な例としては、例えばFastAPI + Next.jsのような構成でやっていたが、バックエンドのapiもnextのrestapiで書けるんじゃないかと感じたので、forkして試してみる。と言ったような場合である。
