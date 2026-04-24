---
name: status
description: このスキルは、ユーザーが `/learning-flow:status` を実行した時、または「学習の進捗を教えて」「今どのPhase/Taskにいるか見せて」「学習状況を確認」と言った時に発動する。docs/LEARNING_CONTEXT.md から進捗を読み取り、Phase一覧・現在地・保存済みトピックを表示する。
argument-hint: (引数なし)
allowed-tools: Read, Glob, Bash
---

# 学習進捗の表示

現在の学習プロジェクトの状態を一覧表示する。書き換えは行わない読み取り専用コマンド。

## 実行フロー

### ステップ1: 前提ファイルの確認

```bash
test -f docs/LEARNING_CONTEXT.md
```

存在しない場合は以下を表示して終了:

```
学習プロジェクトが未初期化です。
/learning-flow:init で初期化してください。
```

### ステップ2: LEARNING_CONTEXT.md から進捗を読む

`docs/LEARNING_CONTEXT.md` を読み、以下を抽出:

- Phase構成テーブル（| Phase | テーマ | 新規概念 | 状態 |）
- 現在の進捗セクション（「現在の進捗」見出し以下）
- 学習ノート構成（docs/learning/ のパス）

### ステップ3: 現在のPhase/Taskを特定

進捗テーブルから「進行中」の行を探す。行の `Phase N` を現在のPhaseとする。

次に、`docs/learning/phase{N}/` 配下のTaskディレクトリを `Glob` で列挙:

```
docs/learning/phase{N}/task*/
```

各Taskディレクトリで `main.md` を読み、「振り返り」セクションの有無から完了/進行中を判定する（振り返りセクションが書かれている → 完了）。

### ステップ4: 保存済みトピックの列挙

現在進行中のTaskディレクトリ内の `*.md` ファイル（`main.md` 除く）を `Glob` で取得:

```
docs/learning/phase{N}/task{M}/[0-9]*-*.md
```

同様に `reference/*.md` も取得。

### ステップ5: 表示フォーマット

以下の形式で出力:

```markdown
# 学習進捗

## Phase一覧

| Phase | テーマ | 新規概念 | 状態 |
|---|---|---|---|
| 1 | MVP | Cognito, API Gateway, Lambda, DynamoDB | **進行中** |
| 2 | ストレージ分離 | S3 | 未着手 |
...

## 現在地: Phase 1 / Task 2

### 保存済みトピック

- docs/learning/phase1/task2/01-dynamodb-keys.md
- docs/learning/phase1/task2/02-terraform-basics.md

### リファレンス

- docs/learning/phase1/task2/reference/iam-overview.md

### 振り返り

main.md の「振り返り」セクション: **未記入**

## 復習キュー

- 待機中: {N} 件（最古: YYYY-MM-DD）
- 次回セッション開始時に再出題されます
  （空なら「待機中の復習問題はありません」と表示）

## 次のアクション候補

- 解説をトピックmdに保存: `/learning-flow:topic <name>`
- 深掘りリファレンス作成: `/learning-flow:reference <theme>`
- 次タスクへ進む: `/learning-flow:next`
```

### ステップ6: 復習キューの状態表示

1. `docs/learning/review-queue.md` が存在するかチェック
2. 存在すれば、「## 待機中」セクション以下の `### [YYYY-MM-DD]` 見出しを `Grep` で列挙
3. 件数と最古の日付を表示
4. ファイルが無い or 待機中が0件なら「待機中の復習問題はありません」と表示

## 特殊ケース

### 進行中Phaseが複数ある場合

警告を出し、最初に見つかった1つを表示:

```
警告: 進行中のPhaseが複数検出されました（Phase 1, Phase 3）。
LEARNING_CONTEXT.md を見直してください。
```

### 進行中が0の場合

全Phase未着手 or 全Phase完了を区別して表示:

```
全Phaseが未着手です。Phase 1から始めるには最初のタスクの解説を開始してください。
```

または:

```
全{N} Phase完了。お疲れさまでした。
```

### Taskディレクトリが空の場合

```
Phase {N} のTaskディレクトリがまだありません。
Task 1を始める場合は、plans/ の設計書を参照して解説を開始してください。
```

## 重要な注意点

- **書き換え禁止**: このスキルは読み取り専用。`Write` / `Edit` は使わない
- **状態の解釈を誤らない**: テーブル記法の微妙な違い（`**進行中**` vs `進行中` vs `🚧`）に対応できるよう、"進行中" というキーワードを含む行を検出する正規表現にする
- **推測で進捗を書き換えない**: 表示したLEARNING_CONTEXT.md の状態が実態と違うとユーザーが気づいた場合、`/learning-flow:next` で更新するよう案内
