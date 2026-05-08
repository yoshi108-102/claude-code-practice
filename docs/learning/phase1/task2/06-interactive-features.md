# 06. 対話モード補助機能 — プロンプト提案・/btw・タスクリスト・リキャップ・PR ステータス

> 出典:
> - [Interactive mode > Prompt suggestions](https://code.claude.com/docs/en/interactive-mode#prompt-suggestions)
> - [Interactive mode > Side questions with /btw](https://code.claude.com/docs/en/interactive-mode#side-questions-with-%2Fbtw)
> - [Interactive mode > Task list](https://code.claude.com/docs/en/interactive-mode#task-list)
> - [Interactive mode > Session recap](https://code.claude.com/docs/en/interactive-mode#session-recap)
> - [Interactive mode > PR review status](https://code.claude.com/docs/en/interactive-mode#pr-review-status)
> （閲覧日 2026-05-08）

このノートは「Interactive mode」のうち、ショートカット系以外の**対話セッション補助機能**を起点に Claude が自動生成した教材です。

## 概要

ユーザー体験を補助する 5 機能をまとめて扱う:

| 機能 | 目的 |
|---|---|
| **Prompt suggestions** | プロンプト入力欄に「次のひと言」候補をグレー表示 |
| **`/btw`** | サイド質問。会話履歴に残らないクイック Q&A |
| **Task list** | 複雑な作業を進捗付きでステータスエリアに表示 |
| **Session recap** | 席を外して戻ってきた時の 1 行サマリ |
| **PR review status** | フッターに PR レビュー状態の色付きリンク |

## 公式docsに沿った解説

### Prompt suggestions（プロンプト提案）

セッション最初は、入力欄に**プロジェクトの git 履歴**から推測した提案がグレーで出る。最近触ったファイルを反映する。

応答後も会話履歴をベースに次のステップ候補が出続ける（複数 step 依頼の続きや自然な後続）。

#### 操作

| 操作 | 動作 |
|---|---|
| `Tab` または `→` | 提案を採用 |
| `Enter` | 採用 + 即送信 |
| 何か文字を打つ | 提案を破棄 |

#### 仕組みと無効化

- 親会話の **prompt cache を再利用**するバックグラウンド呼び出し → 追加コストは小さい
- cache が cold な時は提案生成自体をスキップ
- 会話の最初のターン後、**非対話モード**、**plan モード**では自動的にスキップ
- 完全 OFF にしたい場合:

```bash
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
```

または `/config` から toggle。

### Side questions with `/btw`（サイド質問）

「いまの作業に関係ないけど、ちょっと聞きたい」を**会話履歴を汚さずに**処理する仕組み。

```text
/btw what was the name of that config file again?
```

#### 特徴

- **現会話の全コンテキストにアクセス可能**（Claude が直近で読んだコード / 決定 / 議論内容を踏まえて答えられる）
- 質問と回答は **dismissible overlay** に表示。**会話履歴には絶対残らない**
- **Claude が処理中でも**走らせられる。サイド質問は独立して動き、メインターンを邪魔しない
- **Tool アクセスは無し**: ファイル読み・コマンド実行・検索はできない。**今コンテキストにある情報だけ**で答える
- **1 ターンのみ**。フォローアップ不可（必要なら通常プロンプトで）
- prompt cache 再利用なので低コスト

#### 操作

`Space` / `Enter` / `Escape` で回答を閉じる。

#### `/btw` と subagent の対比

> `/btw` is the inverse of a subagent

| 項目 | `/btw` | subagent |
|---|---|---|
| コンテキスト | **メインの全部見える** | **空** |
| ツール | **無し** | **全部** |
| 用途 | 「今の Claude が知ってること」を聞く | 「外に何があるか」を調べに行かせる |

### Task list（タスクリスト）

複雑な多段作業を Claude が**自分で task list 化**して進捗管理する。`Ctrl+T` でターミナルのステータスエリアに表示。

#### 特徴

- 表示は**最大 5 件**
- 全 task を見たい / クリアしたい時は Claude に直接「全 task 見せて」「全部クリアして」と頼む
- **コンテキスト圧縮（compact）を超えても保持**される → 大プロジェクトで Claude を組織化
- セッション間でタスクを共有したい時:

```bash
CLAUDE_CODE_TASK_LIST_ID=my-project claude
```

`~/.claude/tasks/<my-project>/` に task が共有保存される。

### Session recap（セッションリキャップ）

席を外して戻ってきた時、ターミナルに**直近セッションの 1 行サマリ**が出る機能。

#### 動作条件

- **3 分以上**最終ターンから経過
- ターミナルが**フォーカスを外している**間にバックグラウンド生成
- セッションが**少なくとも 3 ターン**進んでいる
- 連続 2 回は出さない

#### 手動 / 無効化

- `/recap` でいつでも生成
- `/config` の **Session recap** で OFF 可能
- **非対話モードでは常にスキップ**
- 全プラン / 全プロバイダで既定 ON

### PR review status

現ブランチに**オープンな PR がある**時、フッターに `PR #123` のような**クリック可能なリンク**が出る。アンダーラインの色が状態を表す。

| 色 | 状態 |
|---|---|
| 緑 | approved |
| 黄 | pending review |
| 赤 | changes requested |
| グレー | draft |
| 紫 | merged |

#### 操作

- `Cmd+click`（macOS）または `Ctrl+click`（Win/Linux）でブラウザで PR を開く
- ステータスは **60 秒ごとに自動更新**

#### 必須条件

- `gh` CLI がインストール済み
- `gh auth login` で認証済み

## 重要ポイント

- **`/btw` の最大の価値は「履歴を汚さない」こと**。長時間タスクの最中でも、副次的な質問を投げて消える
- `/btw` は **tool 使えない**のを忘れない。「ファイル読み直して」と頼んでも答えられない（subagent や通常プロンプトを使う）
- task list は**自動生成**される。Claude が自分で細分化して進捗を可視化する仕組みなので、ユーザーは見るだけ
- task list は compact を超えて生き残るので、長期戦の道しるべになる
- Session recap は AFK 復帰の体験を整える機能。デフォルト ON だが、recap が邪魔だと感じたら `/config` で切れる
- PR review status は `gh` 認証必須。CI / リモートでなくローカル開発機の `gh` を使う
- prompt suggestions は cache hit を狙った最適化なので、**料金的には軽い**ことを理解しておく

## コード例 / 図

### `/btw` の使いどころ

```text
[Claude が大きい refactor を進行中]
ユーザー: /btw この PR のレビュアー誰だっけ？
[overlay 表示]
Claude: 直近の git log と PR ステータスから、レビュアーは alice と bob です
[Space で閉じる、refactor は中断されない]
```

### 共有 task list で複数セッションを跨ぐ

```bash
# プロジェクト A 用の共有 task list
CLAUDE_CODE_TASK_LIST_ID=migration-alpha claude

# 別ターミナルで再開しても同じ task list
CLAUDE_CODE_TASK_LIST_ID=migration-alpha claude
```

### PR ステータスの活用

```text
フッター表示例:
[main]  PR #446 (yellow underline)  ←  pending review
       Cmd+click → ブラウザで PR を開く
[60 秒後]
[main]  PR #446 (red underline)     ←  changes requested
```

### prompt suggestion を切る

```bash
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
claude
```

または `/config` の **Prompt suggestions** trigger をオフ。

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 次の教材: [07-keybindings-config.md](07-keybindings-config.md) — キーバインドのカスタマイズ
- 関連 docs: [Subagents](https://code.claude.com/docs/en/sub-agents) / [Environment variables](https://code.claude.com/docs/en/env-vars) / [GitHub CLI](https://cli.github.com/)

---

_Auto-generated at 2026-05-08 via /learning-flow:material（公式docs駆動）_
