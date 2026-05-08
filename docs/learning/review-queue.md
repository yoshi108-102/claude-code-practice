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

### [2026-05-08] Phase 1 / Task 1 — Q1 CLAUDE.md vs MEMORY.md の保管場所と性質

**問題**: CLAUDE.md と MEMORY.md を「誰が書くか」「目的」「保管場所と共有範囲」の 3 軸で比較して説明せよ。

**当時の回答**: CLAUDE.md はユーザが書く振る舞い定義ファイル、MEMORY.md は Claude が書く過去のやり取り要約。保管場所は作業ディレクトリ直下、プロジェクト単位で共有。`~/.claude/` 以下に置けば全プロジェクトで共有。

**2026-05-08 再出題時**: 「誰が書くか」「目的」は OK。保管場所について「`.claude` 配下であらゆるプロジェクト共通」と回答 → 実際は `~/.claude/projects/<project-hash>/memory/` で**プロジェクト単位**で分離されている点が抜けていた。

**模範解答の要点**:
- MEMORY.md の保管場所は **`~/.claude/projects/<project-hash>/memory/`** 固定（ユーザー環境側）。作業ディレクトリ直下ではない
- **プロジェクトハッシュごと**にディレクトリが分かれる → 「全プロジェクト共通」ではなく「プロジェクト個別」
- git 管理外。ユーザーは通常直接編集しない（が、実態はただの md なので編集自体は可能）
- 中身は「**会話ログの要約**」ではなく「ユーザーの人物像・好み・フィードバック・プロジェクト文脈の**個別の気づき**の蓄積」
- `/compact` の会話要約とは別物

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/reference/auto-memory.md`

---

### [2026-05-08] Phase 1 / Task 1 — Q4 subagent の context window

**問題**: subagent を使うべき場面と理由を、context window の観点から説明せよ。

**当時の回答**: バグ発生時に subagent を使う。subagent は main より多くの context を持てるため、より多くのファイルを調査できる。

**2026-05-08 再出題時**: 「main の context を汚さない」という Why は掴めた。ただし用途を「コーディング全般・実装の丸投げ」と答えた → 典型ユースケースは**広範な調査タスク**であり、コードを書く工程は main で進める方が良い。

**模範解答の要点**:
- 「多くの context を持てる」は誤り → **context window のサイズは同じ**（モデル依存）
- 正しくは「**別の** context window を持つ」→ main の context を汚さない
- subagent は調査結果の**要約だけ返す**のが本質（例: 20 ファイル読んでも main には 5 行だけ戻る）
- 使うべき場面の典型: 「auth system 全体を調べて」のような**広範な調査で main の context を温存したいとき**
- **コーディング全般を subagent に丸投げするのは推奨されない**。`Explore` agent が read-only なのも象徴的で「コードを書く」より「コードを探して要約する」が本来の役回り

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/02-sessions-and-context.md`, `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/04-best-practices-essentials.md`

---

### [2026-05-08] Phase 1 / Task 1 — Q6 Auto モードの仕組み

**問題**: Auto モードの動作と、Plan モードとの使い分けは？

**当時の回答**: Auto モードはあらゆる permission を許可するモード。skill でフローを定めきっている場合などの信頼できるタスクで使うべき。

**2026-05-08 再出題時**: Plan モードと Plan → Auto のワークフロー提案は OK。ただし Auto モードを「オール自動承認」と捉えている点が前回から変わっていない → 別 classifier モデルによる二重チェックの仕組みを掴むのが要点。

**模範解答の要点**:
- Auto モードは「あらゆる permission 許可」ではない
- **別の classifier モデル（判定用 Claude）が危険行動をブロック**する仕組み → "野放し" ではなく "**AI による二重チェック**"
- `claude -p`（非対話）でも使える。classifier が繰り返しブロックしたら **abort** するフォールバック付き
- 用途: 長時間の自動実行、CI での利用など、人間が逐次確認できない場面
- Plan モード: 「人間が事前に承認」、Auto モード: 「**別 AI が逐次承認**」という対比で捉える

**関連ノート**: `/Users/yoshi/claude-code-practice/docs/learning/phase1/task1/03-permissions-and-safety.md`

---

## 完了

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
