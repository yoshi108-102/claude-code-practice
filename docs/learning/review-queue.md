<!--
復習キュー。各 Task の振り返りクイズで ❌誤答 / ⚠️一部正解 / 「要復習」と判定された問題を蓄積する。
次回学習セッション開始時に待機中のエントリを再出題し、答えられたものは「完了」セクションへ移す。

フォーマット:

### [YYYY-MM-DD] Phase N / Task M — Q{番号} {短いタイトル}

**問題**: ...
**当時の回答**: ...
**模範解答の要点**:
- ...
**関連ノート**: [path](path)
-->

# 復習キュー

## 待機中

### [2026-05-22] Phase 2 / Task 1 — 02-Q1 auto memory はチーム共有されない（マシンローカル）

**問題**: auto memory（MEMORY.md）の保管場所と、「リポジトリ単位」「マシンローカル」が実務で意味すること（チーム共有・worktree 間共有）を説明せよ。

**当時の回答 (2026-05-22)**: 保管パスは正解だが「リポジトリ単位 = チーム共有される(git)」と誤答。

**模範解答の要点**:
- auto memory は **git に乗らない・チーム共有されない・マシンローカル**（`~/.claude/projects/<id>/memory/` は git 管理外）
- 「リポジトリ単位」= **同一リポの全 worktree / サブディレクトリで1つの memory ディレクトリを共有**（このマシン上で）。"チーム共有" ではない
- worktree 間では共有される（同一リポなので）
- ※ 同セッションの まとめQ2 では「auto memory は共有しない」と正答しており、そちらが正しい

**参考**:
- [Memory > Auto memory > Storage location](https://code.claude.com/docs/en/memory#storage-location)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase2/task1/02-auto-memory.md`

---

### [2026-05-22] Phase 2 / Task 1 — 03-Q2 autoMemoryDirectory が user 限定な理由（セキュリティ）

**問題**: project の `.claude/` と user の `~/.claude/` のスコープの違い、`autoMemoryDirectory` がなぜ user settings からしか受け付けられないか。

**当時の回答 (2026-05-22)**: スコープの違いは正解。理由を「範囲が広いから user で担当」とした（惜しい）。

**模範解答の要点**:
- 理由は**セキュリティ**: project/local の settings ファイルは**プロジェクトディレクトリ内**にある → clone させた悪意あるリポが settings を仕込み、**memory 書き込み先を機密フォルダに誘導**する攻撃が可能
- それを防ぐため project/local からは受理せず、リポ外の user/policy settings からのみ受理する

**参考**:
- [Memory > Auto memory > Storage location](https://code.claude.com/docs/en/memory#storage-location)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase2/task1/03-claude-directory-map.md`

---

### [2026-05-22] Phase 2 / Task 1 — まとめQ1 「多段の手順」は skill（CLAUDE.md ではない）

**問題**: 4つの指示を CLAUDE.md / path-scoped rule / skill / hook に振り分けよ（あ:コミット前 lint / い:2-space indent / う:api バリデーション / え:10ステップ手順）。

**当時の回答 (2026-05-22)**: あ=hook✅ う=path-scoped rule✅。い=hook（本来 CLAUDE.md）、**え=CLAUDE.md（誤り、正しくは skill）**。

**模範解答の要点**:
- あ（必ず実行）→ **hook** / う（領域限定）→ **path-scoped rule**
- い（スタイル規約）→ **CLAUDE.md**（hook で強制も実務的には可だが、設問意図はガイド）
- え（**多段の手順**）→ **skill**（invoke 時のみロード）。CLAUDE.md に手順を書くと毎セッション context を食う。「手順 → skill」は記憶系の核原則

**参考**:
- [Memory > When to add](https://code.claude.com/docs/en/memory)
- [Skills](https://code.claude.com/docs/en/skills)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase2/task1/main.md`

---

### [2026-05-22] Phase 1 / Task 2 — Q3 引数表記のルール（`<arg>` / `[arg]` と引数なしコマンド）

**問題**: コマンド表の `<arg>` / `[arg]` の意味を答え、`/agents <command>` と打ったとき何が起こるかを説明せよ。

**当時の回答 (2026-05-22)**: 前半「`<arg>`=必須、`[arg]`=省略可」は正解。後半「`/agents <command>` で何が起こるか」が書きかけで未完。

**模範解答の要点**:
- `<arg>` = 必須引数 / `[arg]` = 省略可能な引数
- `/agents` は表でどちらの記法も付かない = **引数を取らないコマンド**
- 後ろに文字列を付けても引数として意味を持たず、コマンドは subagent 管理 UI を開くだけ
- 「引数表記が無い = 余計な文字列は無視される（コマンドが起動するのみ）」と捉える

**参考**:
- [Commands - argument notation](https://code.claude.com/docs/en/commands)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task2/02-slash-commands.md`

---

### [2026-05-22] Phase 1 / Task 2（まとめ）— Q1 /btw・subagent・Plan subagent の context×tool 軸

**問題**: 3 つを「メイン context が見えるか」「ツールが使えるか」で整理し、「読んだコードを踏まえた質問を履歴を汚さず投げたい」時にどれを使うか、subagent で代用できない理由を説明せよ。

**当時の回答 (2026-05-22)**: 選択（/btw を使う、subagent は main context を持たないので不可）と表は概ね正解だが、**`/btw` のツールを「読み取りのみ」と誤答**。

**模範解答の要点**:
- `/btw` = メイン context 見える / **ツールは一切なし**（読み取りすら不可）
- Plan subagent = context 見えない / read-only、subagent = context 見えない / 全ツール
- `/btw` は subagent の完全な鏡像（記憶は全部あるが外に手を出せない）

**参考**:
- [Interactive mode > Side questions with /btw](https://code.claude.com/docs/en/interactive-mode#side-questions-with-%2Fbtw)
- [Sub-agents > Built-in subagents](https://code.claude.com/docs/en/sub-agents#built-in-subagents)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task2/06-interactive-features.md`

---

### [2026-05-22] Phase 1 / Task 2（まとめ）— Q3 worktree が効く場面 / `/sandbox` との違い

**問題**: worktree が branch 切替より価値を出す場面を 2 つ。worktree が隔離するもの/しないもの。本物の隔離を担う別機能の名前。

**当時の回答 (2026-05-22)**: 「わからない」。

**模範解答の要点**:
- 効く場面: ①複数 Claude セッション並行（同一ブランチ2箇所 checkout 不可の制約を回避）②subagent の並列 edit（`isolation: worktree`）。他にメイン非汚染の実験、PR 隔離レビュー
- 隔離するのは**ファイル並びだけ**。`.git`/remote は共有、FS 的閉じ込め・権限・ネットワーク制限はなし
- 本物の隔離 = **`/sandbox`（sandboxing、Phase 2）**

**参考**:
- [Run parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task2/reference/worktree-usage-and-tradeoffs.md`

---

### [2026-05-22] Phase 1 / Task 2（まとめ）— Q4 シェル実行3経路の context への影響

**問題**: (A) shell mode `!` / (B) Claude に Bash 委譲 / (C) background bash。A と B の permission 差、A 多用の問題（context）、C が解決する問題。

**当時の回答 (2026-05-22)**: A/B の permission 差と C の非同期性は正解。**「A の出力も context を食う」という観点が抜けた**。

**模範解答の要点**:
- A = permission 経由せず user 権限そのまま / B = Claude の permission に縛られる
- **A 多用の問題: shell mode の出力もコンテキストを消費する**（`! ls` 連発で context 膨張）
- C = 長時間処理を非同期化し、待たずに次の作業へ進める

**参考**:
- [Interactive mode > Shell mode with ! prefix](https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix)
- [Interactive mode > Background bash commands](https://code.claude.com/docs/en/interactive-mode#background-bash-commands)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task2/05-history-and-shell-mode.md`

---

### [2026-06-12] Phase 1 / Task 2（まとめ）— Q5 subagent モデルの優先順位とコストの罠

**問題**: subagent のモデルを決める 3 レイヤーを優先順位順に。意図しない高コストの最大原因。`model:opus` 固定が `/model` に従わない理由を subagent の本質と絡めて。

**当時の回答 (2026-05-22)**: 「覚えていない」。

**2026-06-12 再出題時**: 「わからない」。本セッションで [reference/subagent-model-override.md](../phase1/task2/reference/subagent-model-override.md) を 30 分前に深掘りした直後だったが定着せず。記憶への定着段階に課題あり（理解 → 言語化のリハーサル不足）。

**模範解答の要点**:
- 優先順位: ①呼び出し時の `model` 引数 → ②agent 定義 frontmatter の `model` → ③親モデル継承
- 高コストの最大原因 = ③の暗黙継承（Opus セッションで subagent 大量発射 = 全部 Opus）
- `model:opus` 固定が親 `/model` に従わない = subagent は**別の独立した実行単位**だから（別枠＝context window が別、という本質と整合）

**参考**:
- [Sub-agents > model field](https://code.claude.com/docs/en/sub-agents)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task2/reference/subagent-model-override.md`

---

### [2026-06-12] Phase 1 / Task 2（まとめ）— Q6 bundled skill カスタマイズ4経路

**問題**: (B) 別名 skill が最も健全な理由を context 効率で (A) CLAUDE.md と対比。(C) 同名上書きが非推奨な理由を 2 つ。

**当時の回答 (2026-05-22)**: 「わからん」。

**2026-06-12 再出題時**: 「わからない」。本セッションで [reference/customizing-bundled-skills.md](../phase1/task2/reference/customizing-bundled-skills.md) を深掘りした直後だったが定着せず。Q5 と同様、理解 → 言語化のリハーサル不足。

**模範解答の要点**:
- (B) が健全な理由: CLAUDE.md(A) は**常時 context に乗る**ので詳細ルールで圧迫。別名 skill(B) は**invoke 時のみ load**で context 効率良 + bundled を壊さない
- (C) 非推奨の理由2つ: ①bundled の改善が反映されなくなる ②「`/review` が普通と挙動が違う」混乱の元

**参考**:
- [Skills - where skills live / precedence](https://code.claude.com/docs/en/skills)

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task2/reference/customizing-bundled-skills.md`

---

## 完了

### [取得: 2026-05-22] Phase 1 / Task 1 — Q6 Auto モードの仕組み

**問題**: Auto モードの動作と、Plan モードとの使い分けは？

**初回回答 (2026-05-08)**: Auto を「あらゆる permission を許可するモード」と誤答。
**再出題回答 (2026-05-22)**: classifier というガードレールが AI の自律実行を監視、Plan で詰めて Auto に任せる、と正答。✅ 正解

**要点**:
- 別の classifier モデルが危険行動をブロック → "野放し" ではなく "AI による二重チェック"
- Plan = 人間が事前承認 / Auto = 別 AI が逐次承認 の対比
- classifier が繰り返しブロックすれば abort。CI・長時間自動実行が用途

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/03-permissions-and-safety.md`

---

### [取得: 2026-05-22] Phase 1 / Task 1 — Q1 CLAUDE.md vs MEMORY.md の保管場所と性質

**問題**: CLAUDE.md と MEMORY.md を「誰が書くか」「目的」「保管場所と共有範囲」の 3 軸で比較して説明せよ。

**初回回答 (2026-05-08)**: 保管場所を「`.claude` 配下であらゆるプロジェクト共通」と誤答。
**再出題回答 (2026-05-22)**: MEMORY.md は `~/.claude/projects/` 配下でプロジェクト単位、CLAUDE.md はプロジェクトルート + `~/.claude/CLAUDE.md` のユーザーレベル二層、と正答。✅ 正解

**要点**:
- MEMORY.md の保管は `~/.claude/projects/<project-hash>/memory/`（**プロジェクト個別**、全プロジェクト共通ではない）
- 中身は会話ログ要約ではなく、人物像・好み・フィードバック・文脈の個別の気づきの蓄積

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/reference/auto-memory.md`

---

### [取得: 2026-05-22] Phase 1 / Task 1 — Q4 subagent の context window

**問題**: subagent を使うべき場面と理由を、context window の観点から説明せよ。

**初回回答 (2026-05-08)**: 「subagent は main より多くの context を持てる」「コーディング丸投げ用途」と誤答。
**再出題回答 (2026-05-22)**: 「別の context window を立てて調査をやらせ、main の context 消費を抑える」と正答。✅ 正解

**要点**:
- サイズは同じ。違うのは「別枠の context window を持てる」点
- 調査結果の要約だけ main に返す → 広範な調査が典型用途、コーディング丸投げは非推奨

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/02-sessions-and-context.md`, `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/04-best-practices-essentials.md`

---

### [取得: 2026-05-08] Phase 1 / Task 1 — Q3 plugin が必要な tool カテゴリ

**問題**: 5 tool カテゴリのうち plugin 追加が必要なものはどれか。

**初回回答 (2026-04-24)**: Web と Code intelligence。
**再出題回答 (2026-05-08)**: Code intelligence。✅ 正解

**要点**:
- plugin が必要なのは **Code intelligence のみ**
- Web（WebSearch, WebFetch）は**標準装備**で plugin 不要

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/01-agentic-loop-and-tools.md`

---

### [取得: 2026-05-08] Phase 1 / Task 1 — Q5 checkpoint が保存するもの

**問題**: checkpoint は何を守るか。

**初回回答 (2026-04-24)**: 会話の途中の状態を保存するもの。
**再出題回答 (2026-05-08)**: ローカルにある会話とかファイルは守られる。外部のDBとかは守られない。✅ 正解（会話 + ファイル の両方を含めた点で出題意図クリア）

**要点**:
- checkpoint は「会話の状態」だけでなく「**ファイル編集の自動スナップショット**」も含む
- Claude がファイルを編集する**直前**に毎回スナップショットを取る
- `Esc+Esc` または `/rewind` で「**会話のみ / コードのみ / 両方**」を選んでロールバックできる
- session-local、**git の代替ではない**
- 外部 DB / 外部 API への送信済みリクエスト等は対象外

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/02-sessions-and-context.md`, `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/03-permissions-and-safety.md`
