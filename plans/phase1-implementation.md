# Phase 1 — 基礎: Claude Code の世界観を掴む / 実装計画

## Phase 1 の目的

Claude Code の「何ができるのか」「どう動くのか」を公式 docs に沿って理解し、CLI / IDE / Desktop / Web の各 Surface で基本タスクを自力で実行できる状態になる。Phase 1 終了時に担当 32 ページを全て読み終わっている。

## 最終成果物

- 自分用の最小 CLAUDE.md（プロジェクトの好みと基本ルールを書いたもの）
- 各 Surface（最低 CLI + VS Code、可能なら Desktop / Web / JetBrains / Chrome も）で同じタスクを実行した比較メモ
- checkpoint 巻き戻しの体験ノート
- よく使う slash commands リスト（自分用）
- 直近 3 週間の What's new に目を通した週次キャッチアップ習慣の起点

## Task 分割

読むだけでは退屈なので、各 Task は **「docs を読む → 手を動かす → Q&A を残す」** のサイクル。1 Task あたり 30分〜2時間の粒度。

---

### Task 1: 概論 & クイックスタート

**読む docs (4):**
- overview.md
- how-claude-code-works.md
- quickstart.md
- best-practices.md

**やること:**
- Claude Code で簡単なタスクを 1 個実行（例: このリポジトリの README を作る、簡単なスクリプトを書かせる）
- agentic loop / 4つのコアツール / permission の関係を自分の言葉で説明できるか確認

**学習アウトプット:**
- `docs/learning/phase1/task1/main.md`
  - Claude Code の "agentic loop" とは？
  - permission model の 3 モードの違い
  - best-practices で印象に残ったこと

---

### Task 2: CLI の基本操作

**読む docs (4):**
- cli-reference.md
- commands.md
- interactive-mode.md
- keybindings.md

**やること:**
- 主要 slash commands を一通り試す（`/help`, `/model`, `/context`, `/resume`, `/clear`, `/compact`, `/cost` など）
- キーバインドを確認し、気に入ったものは覚える／カスタマイズする
- 非対話モード（`claude -p "..."`）と対話モードの違いを体験

**学習アウトプット:**
- よく使う slash commands の個人用チートシート
- 自分用キーバインドのメモ（必要ならカスタム設定）

---

### Task 3: セッション・コンテキスト管理

**読む docs (2):**
- context-query-window.md もとい context-window.md
- checkpointing.md

**やること:**
- `/context` で現在のコンテキスト使用量を見る
- 長めのセッションを `/compact` で圧縮してみる
- 意図的に変更を加えてから checkpoint で巻き戻す体験（安全網の確認）

**学習アウトプット:**
- context window の中身（system prompt / tools / conversation / files）の理解
- checkpoint が使える場面・使えない場面の整理

---

### Task 4: 入力と表示のバリエーション

**読む docs (3):**
- voice-dictation.md
- fast-mode.md
- fullscreen.md

**やること:**
- fast mode (`/fast`) を有効にして体感の違いを見る
- fullscreen レンダリングを試す
- voice dictation は環境が許せば試す（任意）

**学習アウトプット:**
- fast mode を使うべき場面／避ける場面

---

### Task 5: セットアップ深掘り & ターミナル最適化

**読む docs (3):**
- setup.md
- authentication.md
- terminal-config.md

**やること:**
- 今使っているインストール方法の位置付けを確認（native / Homebrew / WinGet）
- ログイン情報の管理方法を確認
- terminal-config.md に沿ってターミナル設定を見直し（通知、タブタイトル、iTerm/WezTerm等）

**学習アウトプット:**
- 自分の環境で最適化した設定の記録

---

### Task 6: IDE / Surface 体験

**読む docs (7):**
- vs-code.md
- jetbrains.md
- desktop.md / desktop-quickstart.md
- claude-code-on-the-web.md / web-quickstart.md
- chrome.md

**やること:**
- VS Code 拡張を入れて、CLI と同じタスクを実行して比較
- Desktop app を触って sessions の並列管理を体験（任意）
- Web 版 / Chrome (beta) を試す（任意）

**学習アウトプット:**
- Surface 比較表（得意な作業 / 不得意な作業）
- 自分がメインで使う Surface の選定理由

---

### Task 7: 共通ワークフロー & トラブルシュート

**読む docs (4):**
- common-workflows.md
- platforms.md
- troubleshooting.md
- errors.md

**やること:**
- common-workflows の 2-3 個のパターンを自分のプロジェクトで実践（テスト自動生成、PR メッセージ生成など）
- troubleshooting / errors は「困ったら見る索引」として構造を把握

**学習アウトプット:**
- 自分のよくやる作業に当てはまるワークフローの整理
- トラブル発生時の確認順序メモ

---

### Task 8: 最新情報のキャッチアップ習慣

**読む docs (5):**
- changelog.md
- whats-new/index.md
- whats-new/2026-w13.md
- whats-new/2026-w14.md
- whats-new/2026-w15.md

**やること:**
- 最近 3 週間のアップデートに目を通す
- 今後 weekly で what's new を確認する習慣の起点を作る（例: 毎週月曜に確認）

**学習アウトプット:**
- 最近の注目機能 Top 3
- 週次チェックの運用ルール

---

## Phase 1 完了判定

- [ ] 担当 32 ページを全てチェック（`plans/claude-code-learning-phases-design.md` の Phase 1 セクション）
- [ ] 成果物 5 点（CLAUDE.md / Surface 比較 / checkpoint 体験 / slash cheatsheet / whats-new 習慣）ができている
- [ ] 主要概念を自分の言葉で説明できる: agentic loop / context window / checkpointing / permission modes / 各 Surface の違い

## 次 Phase への接続

Phase 1 で「使う側」の基礎が固まったら、Phase 2 で「CLAUDE.md / settings / permissions」を深掘りし、Claude Code を**自分仕様に仕立てる**段階に入る。
