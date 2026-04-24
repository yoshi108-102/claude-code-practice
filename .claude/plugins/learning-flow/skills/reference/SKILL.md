---
name: reference
description: このスキルは、ユーザーが `/learning-flow:reference <theme>` を実行した時、または「深掘りリファレンスを作って」「Web検索してまとめて」「この技術の詳細をdocsに残して」と言った時に発動する。指定テーマについてWeb検索を含む深掘り調査を行い、ソース付きのリファレンスmdを docs/learning/phase{N}/task{M}/reference/{theme}.md に保存する。
argument-hint: <theme> [--no-web]  (例: dynamodb-vs-rdb, terraform-state)
allowed-tools: Read, Write, Edit, Glob, Bash, WebSearch, WebFetch, AskUserQuestion
---

# 深掘りリファレンスmdの生成

指定テーマについて、Web検索を含む詳細調査を行い、ソース付きリファレンスmdを `reference/` 配下に保存する。

## 実行フロー

### ステップ1: 引数の解釈

`<theme>`: 必須、kebab-case。例: `dynamodb-vs-rdb`, `terraform-state-management`
`--no-web`: 任意、Web検索を使わない（既存知識のみ）

引数なしならユーザーに聞く:

```
リファレンスのテーマを指定してください（kebab-case、例: dynamodb-vs-rdb）
Web検索を使いますか？（y/n、デフォルト: y）
```

### ステップ2: 現在のPhase/Taskを特定

`learning-flow-topic` スキルと同じロジック:

1. `docs/LEARNING_CONTEXT.md` から進行中Phaseを取得
2. `docs/learning/phase{N}/task*/` から進行中Task（`main.md` の振り返りが未記入）を特定
3. 保存先: `docs/learning/phase{N}/task{M}/reference/{theme}.md`

### ステップ3: Web検索（`--no-web` なしの場合）

テーマを検索クエリに変換し、`WebSearch` で3〜5件の信頼できるソースを取得:

- 公式ドキュメント（AWS Docs, Terraform Registry, MDN 等）を優先
- 日付が新しいものを優先
- 検索結果のURLから `WebFetch` で本文取得（必要に応じて）

### ステップ4: リファレンスmdの生成

以下の構造で保存:

```markdown
# {{THEME_TITLE}}

## 概要

{{テーマの2〜3段落の要約}}

## 詳細

### {{サブテーマ1}}

{{詳細説明、表、コードブロック}}

### {{サブテーマ2}}

...

## 比較表 / 具体例

{{テーブル形式での比較、または具体的なコード例}}

## よくある誤解

- **誤解: ...**
  - 実際: ...

## まとめ

- ポイント1
- ポイント2

## 参考文献

- [{{title1}}]({{url1}}) - {{簡単な説明、閲覧日}}
- [{{title2}}]({{url2}}) - {{簡単な説明、閲覧日}}
- [{{title3}}]({{url3}}) - {{簡単な説明、閲覧日}}

---

_Saved at {{date}} via /learning-flow:reference_
```

**書式のガイドライン**:

- **トピックmdとの差別化**: トピックmdは「Q&A中心」、リファレンスは「体系的な情報整理 + ソース」
- **比較表を積極的に使う**: A vs B、旧 vs 新 など、違いを明確にする
- **コード例は短く**: 長いコードは避け、要点を示す最小例
- **参考文献は必須**: Web検索を使った場合、必ずソースを記載。URL、タイトル、閲覧日を揃える

### ステップ5: 関連トピックmdへのリンク追加

直前に `/learning-flow:topic` で保存したトピックmd（または同Task内で最新のトピックmd）を検出し、「関連」セクションに今作ったリファレンスへのリンクを `Edit` で追加:

```markdown
## 関連

- 深掘り: [reference/{{theme}}.md](reference/{{theme}}.md)  ← 追加
```

既に同じリンクがあればスキップ。

### ステップ6: 完了案内

```
リファレンスを保存しました:
  docs/learning/phase{N}/task{M}/reference/{theme}.md

参考文献 {N}件:
  - {url1}
  - {url2}
  ...

関連トピック {topic}.md にリンクを追加しました。
```

## 重要な注意点

### Web検索の使い分け

- **使うべき**: 新しい技術、バージョン差異、ベストプラクティス、仕様の正確性が重要
- **使わなくてもよい**: 一般的な概念、既知のパターン、ユーザーが既にソースを示した内容

判断に迷う場合は使う（正確性を優先）。

### ソースの信頼性

以下の優先順位で選ぶ:

1. 公式ドキュメント（例: docs.aws.amazon.com, developer.mozilla.org）
2. 公式ブログ / 公式アナウンス
3. 著名技術者の記事（Martin Fowler, Adrian Cockcroft 等）
4. 信頼できる技術メディア（The Register, InfoQ 等）
5. Stack Overflow（評価の高い回答のみ）

避けるもの:

- 日付のない個人ブログ
- 翻訳・転載記事（原文を辿れない）
- 古い情報（3年以上前のフレームワーク情報など）

### 冪等性

既存ファイルがある場合:

- `AskUserQuestion` で「追記(a)/上書き(o)/中止(c)？」を聞く
- 追記の場合、新しい調査結果を「追補」セクションとして末尾に追加

### リファレンス1ファイル1テーマ

1つのリファレンスに複数テーマを詰め込まない。テーマが広すぎると感じたら、分割提案:

```
{theme} は範囲が広いので、以下に分けませんか？
- {theme}-basics
- {theme}-advanced
どうしますか？
```
