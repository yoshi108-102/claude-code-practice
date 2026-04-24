---
name: topic
description: このスキルは、ユーザーが `/learning-flow:topic <topic-name>` を実行した時、または「今の話を学習ノートに保存して」「このトピックをdocsに残して」「Q&Aをまとめて保存」と言った時に発動する。直前までの会話の解説・質疑応答をまとめ、docs/learning/phase{N}/task{M}/NN-{topic}.md として保存する。
argument-hint: <topic-name> (例: dynamodb-keys, iam-basics)
allowed-tools: Read, Write, Edit, Glob, Bash, AskUserQuestion
---

# 学習トピックmdへの保存

直前までの会話（解説パート + Q&A）を学習ノートとして保存する。保存先は現在のPhase/Taskから自動判定する。

## 実行フロー

### ステップ1: 引数の取得

引数で渡された `<topic-name>` を使う。引数がない場合はユーザーに聞く:

```
トピック名を指定してください（kebab-case推奨、例: dynamodb-keys, terraform-modules）
```

### ステップ2: 現在のPhase/Taskを特定

`docs/LEARNING_CONTEXT.md` を読み、進行中のPhase番号を取得。

次に `docs/learning/phase{N}/` を `Glob` で検索し、進行中のTaskを特定:

1. Taskディレクトリを列挙（`task1/`, `task2/` ...）
2. 各Taskの `main.md` を読み「振り返り」セクションが未記入のものを「進行中」とみなす
3. 全Task完了 or 新規Task開始が必要な場合、ユーザーに確認:
   ```
   Phase {N} の進行中Taskが見つかりません。新しいTaskを作成しますか？
   - 新規作成する場合、Task番号（例: 3）を指定してください
   ```

### ステップ3: 保存先パスの決定

```
docs/learning/phase{N}/task{M}/{NN}-{topic-name}.md
```

`{NN}` は既存ファイル数 + 1 の2桁ゼロパディング:

- Task内の `[0-9][0-9]-*.md` を `Glob` で列挙
- 最大番号を取得し +1（初回なら 01）

ディレクトリが存在しない場合は `Bash` で `mkdir -p` する。

### ステップ4: トピックmdの生成

以下の構造で保存:

```markdown
# {{TOPIC_TITLE}}

## 概要

{{会話から抽出した1〜2段落の要約}}

## 解説

{{会話中の解説パート}}

### {{サブセクション1}}

...

## Q&A

**Q: {{質問1}}**

{{回答1}}

**Q: {{質問2}}**

{{回答2}}

## 関連

- 深掘り: [reference/{{theme}}.md](reference/{{theme}}.md)
- 次のトピック: （未作成なら空欄）

---

_Saved at {{date}} via /learning-flow:topic_
```

**`{{TOPIC_TITLE}}`**: topic-name を人間可読なタイトルに変換（kebab-case → 日本語/英語の自然な表現）。判断に迷ったらユーザーに確認。

### ステップ5: main.md の更新

`docs/learning/phase{N}/task{M}/main.md` が存在する場合、目次に今作ったトピックへのリンクを `Edit` で追加する。例えば既存目次が以下の場合:

```markdown
## 目次

- [01-dynamodb-keys.md](01-dynamodb-keys.md) - DynamoDBのキー設計
- [02-terraform-basics.md](02-terraform-basics.md) - Terraform入門
```

このリストの末尾に新しい行を追加する（矢印や "追加" などの注釈コメントを本文に含めない）:

```markdown
- [{{NN}}-{{topic}}.md]({{NN}}-{{topic}}.md) - {{TOPIC_TITLE}}
```

`main.md` が存在しない場合は新規作成:

```markdown
# Phase {N} / Task {M}: {{Task title}}

## 目次

- [{{NN}}-{{topic}}.md]({{NN}}-{{topic}}.md) - {{TOPIC_TITLE}}

## 振り返り

（Task完了時に記入）
```

### ステップ6: 完了案内

```
学習トピックを保存しました:
  docs/learning/phase{N}/task{M}/{NN}-{topic-name}.md

目次を更新しました:
  docs/learning/phase{N}/task{M}/main.md

次のアクション:
  - 深掘りが必要なら /learning-flow:reference で詳細mdを追加
  - Task完了したら /learning-flow:next で次へ進む
```

## 重要な注意点

### 会話から抽出すべきもの

- **解説**: ユーザーの質問に答えた内容、ユーザーが理解を確認した結論
- **Q&A**: ユーザーの質問と、それに対する回答のペア
- **図やコード例**: 会話中に示した mermaid 図、コードブロック、表
- **Claude自身の推測で補った詳細**: 会話になかった補足情報は**追加しない**。保存するのはあくまで「会話に出た内容」

### 保存すべきでないもの

- ユーザーが明示的に「保存不要」と言った雑談
- 他のタスクの話題
- 冗長な前振り・相槌

### 冪等性

同じ `<topic-name>` で2回呼ばれた場合:

- 既存ファイルを検出して「既存ファイルがあります。追記(a)/上書き(o)/中止(c)？」を `AskUserQuestion` で聞く
- 追記の場合、既存セクションの末尾にQ&Aを追加

### タイトルの日本語化

トピック名が kebab-case の場合、日本語タイトルを推測して `{{TOPIC_TITLE}}` に使う:

- `dynamodb-keys` → "DynamoDBのキー設計"
- `terraform-basics` → "Terraformの基本"
- `iam-overview` → "IAM概論"

推測に自信がない場合はユーザーに確認。
