---
name: init
description: このスキルは、ユーザーが `/learning-flow:init` を実行した時、または「新しい学習プロジェクトを始めたい」「Phase設計書を作って」「学習フローの雛形を作って」と言った時に発動する。対話形式で学習テーマとPhase構成を聞き出し、CLAUDE.md、docs/LEARNING_CONTEXT.md、plans/*-design.md の骨組みを生成する。
argument-hint: (引数なし、対話で収集)
allowed-tools: Read, Write, Edit, Bash, AskUserQuestion, Glob
---

# 新規学習プロジェクトの初期化

このスキルを実行すると、構造化された学習プロジェクトの土台が整う。対話でユーザーから情報を集め、テンプレートから4ファイルを生成する。

## 実行フロー

### ステップ1: 対話で情報収集

`AskUserQuestion` を使って以下を順に聞く（1メッセージで全部ではなく、適宜分ける）:

1. **学習テーマ**（自由記述）
   - 例: "Kubernetes", "Rust", "関数型プログラミング", "量子コンピューティング"

2. **プロジェクトの成果物**（自由記述、省略可）
   - 「学習の過程で何を作るか」例: "シンプルなTODOアプリ", "CLIツール", "なし（理論学習のみ）"

3. **Phase数**（選択）
   - 3 / 5 / 7 / 10 / その他

4. **各Phaseのテーマと新規概念**
   - Phase数が決まったら、各Phaseで「何を学ぶか / どの概念を新規導入するか」を聞く
   - ユーザーが「おまかせ」と言った場合、テーマから推論して提案

### ステップ2: 既存ファイル確認

生成前に以下をチェック:

```bash
ls CLAUDE.md docs/LEARNING_CONTEXT.md plans/ 2>/dev/null
```

- `CLAUDE.md` が既にある → 上書きせず、追記提案（diff見せてから判断をユーザーに任せる）
- `docs/LEARNING_CONTEXT.md` が既にある → 中断し「既存プロジェクトを検出した。新規作成しますか、追記しますか？」と確認
- `plans/` がない → 新規作成

### ステップ3: ファイル生成

以下の4ファイルを生成する。テンプレートは `templates/` 配下。

| ファイル | 内容 | テンプレート |
|---|---|---|
| `CLAUDE.md` | プロジェクトコンテキスト、最重要ルール、ドキュメント構成 | `templates/CLAUDE.md.template` |
| `docs/LEARNING_CONTEXT.md` | 学習フロー全体像、進捗テーブル、運用ルール | `templates/LEARNING_CONTEXT.md.template` |
| `plans/{theme-slug}-learning-phases-design.md` | Phase 1〜N の設計書（テーマ、ゴール、新規概念） | `templates/design.md.template` |
| `docs/learning/.gitkeep` | 空ファイル | - |

**プレースホルダ**:

テンプレート内の `{{PLACEHOLDER}}` を対話で集めた値に置換する:

- `{{PROJECT_NAME}}` - リポジトリ名（`basename $PWD`）
- `{{THEME}}` - 学習テーマ
- `{{THEME_SLUG}}` - テーマのkebab-case版（Kubernetes → kubernetes）
- `{{DELIVERABLE}}` - 成果物（未指定なら「（なし）」）
- `{{PHASE_COUNT}}` - Phase数
- `{{PHASE_TABLE}}` - 「| Phase | テーマ | 新規概念 | 状態 |」表の中身
- `{{PHASE_SECTIONS}}` - 各Phaseの詳細セクション（## Phase 1: ... など）
- `{{GENERATED_AT}}` - 生成日時（`date +%Y-%m-%d`）

### ステップ4: 生成後の案内

全ファイル生成完了後、ユーザーに以下を案内する:

```
学習プロジェクトの土台を作成しました:
  - CLAUDE.md
  - docs/LEARNING_CONTEXT.md
  - plans/{theme-slug}-learning-phases-design.md

次のステップ:
  1. plans/{theme-slug}-learning-phases-design.md を開いて、各Phaseの詳細を肉付けする
  2. Phase 1 のTask分割を plans/phase1-implementation.md に書く
  3. /learning-flow:status で進捗を確認できます
```

## 重要な注意点

### 既存ファイルを破壊しない

`Write` の前に必ず `Read` で存在チェック。既存あれば:
- `CLAUDE.md`: 追記内容をdiff表示してユーザー判断を仰ぐ
- それ以外: 「上書きしますか？（y/n）」と `AskUserQuestion` で確認

### テーマスラグの生成

`{{THEME_SLUG}}` はテーマを以下で変換:
- 小文字化
- スペース→ハイフン
- 日本語は英訳またはローマ字（迷ったらユーザーに確認）
- 特殊文字除去

例:
- "AWS学習" → `aws`（ユーザーに `aws` でいいか確認）
- "関数型プログラミング" → `functional-programming`
- "Rust" → `rust`

### Phaseが決まらない時

ユーザーが「テーマ決まってないけどとりあえず始めたい」と言った場合:

- テーマ: 仮で「general-learning」
- Phase数: デフォルト5
- Phase内容: 空欄のまま生成し、「後で埋めてください」と案内

## テンプレート詳細

テンプレートファイルは以下を参照:

- `templates/CLAUDE.md.template` - プロジェクトコンテキスト雛形
- `templates/LEARNING_CONTEXT.md.template` - 学習フロー全体像雛形
- `templates/design.md.template` - Phase設計書雛形

テンプレートは意図的にミニマルに保つ。ユーザーがあとで書き足すことを前提にする。
