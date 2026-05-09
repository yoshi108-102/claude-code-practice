---
title: bundled skill（/review, /simplify 等）のカスタマイズ方法
phase: 1
task: 2
topic: bundled skill / customization / team rules / coding standards
created: 2026-05-08
---

# bundled skill（`/review`, `/simplify` 等）のカスタマイズ方法

出典:
- [Skills](https://code.claude.com/docs/en/skills) — bundled skills, where skills live, precedence, restrict skill access
- [Commands](https://code.claude.com/docs/en/commands) — `[Skill]` マークの説明

---

## 質問

> `/simplify` とか `/review` は実際の開発ではレビュールールがあるが（コーディング規約と一緒）、**カスタマイズは可能か?**

## 結論

**可能。コーディング規約のアナロジーはそのまま当てはまる。**
4 つの道があり、効果と手間で使い分ける。

```
[手間: 小]                                              [手間: 大]
   │                                                       │
   ▼                                                       ▼
A. CLAUDE.md にルール追加 → B. 別名 skill 自作 → C. 同名上書き → D. plugin 配布
```

## A. CLAUDE.md にチームルールを書く（最速）

skill 実行中も CLAUDE.md は context に乗る（fork 実行でも CLAUDE.md は load される。docs の "Skill with `context: fork`" テーブル参照）。
→ bundled skill は CLAUDE.md のルールを踏まえて動く。

```markdown
# Code Review Rules
- 必ず JSDoc を確認
- console.log 残しは指摘
- import はアルファベット順
```

| Pros | Cons |
|---|---|
| コスト最小 | bundled skill 本体は変えられない |
| git 共有で全員に伝わる | CLAUDE.md が肥大すると常時 context 圧迫 |

→ **8 割のケースはこれで足りる**。ESLint config を `.eslintrc` に書くのと同じ発想。

## B. 別名のカスタム skill を作る（推奨）

```bash
mkdir -p .claude/skills/team-review
```

`.claude/skills/team-review/SKILL.md`:

```yaml
---
name: team-review
description: チーム独自のレビュールールで PR をチェック
---

## チームのレビュー観点
1. セキュリティ: env 変数のハードコード、SQL 直書き
2. DB アクセス: N+1 クエリ、トランザクション境界
3. テスト: 新機能には integration test 必須
4. UI: アクセシビリティ属性

## 実行手順
PR diff を読み、上記の観点で issue を列挙してください。
重大度（high/medium/low）も付けて。
```

呼び出し: `/team-review`。bundled `/review` とは別物。

| Pros | Cons |
|---|---|
| bundled skill を壊さない | 新コマンド名を覚える必要 |
| 完全に独自の挙動 | |
| git 共有 | |
| `allowed-tools`, `paths`, `model` も指定可能 | |
| **invoke 時のみ load** されるので context 効率良い | |

→ **最も健全な道**。CLAUDE.md だと常時 context に乗るが、skill は invoke 時だけ load されるから、詳細ルールを書いても context 圧迫しない。

## C. 同名で bundled skill を上書き（注意）

docs:
> When skills share the same name across levels, enterprise overrides personal, and personal overrides project.

順位:
```
Enterprise > Personal (~/.claude/skills/) > Project (.claude/skills/) > [bundled]
```

`.claude/skills/review/SKILL.md` を作れば bundled `/review` を上書き可能（bundled は precedence table に明記されてないが、最下位と推察）。

```yaml
---
name: review
description: PR をうちの team rule でレビュー
---
[完全に独自の review ロジック]
```

| Pros | Cons |
|---|---|
| 既存ユーザーが `/review` で team rule を使える | bundled の改善が反映されなくなる |
| | 「`/review` が普通と違う」混乱の元 |
| | bundled の挙動を完全置換 |

→ **B の方が無難**。同名上書きは「team-wide で必ず統一強制したい」最終手段。

## D. plugin として配布（チーム / OSS スケール）

```
my-team-plugin/
├── plugin.json
└── skills/
    └── review/SKILL.md
```

git repo にして各メンバー install。
plugin skill は `plugin-name:skill-name` で namespace 分離されるので bundled とぶつからない:

```text
/my-team-plugin:review    ← plugin
/review                   ← bundled
```

| Pros | Cons |
|---|---|
| バージョン管理可能 | セットアップコスト |
| enable/disable 切替可 | |
| 複数 repo で共有 | |

## コーディング規約のアナロジー

| 規約の世界 | Claude Code の世界 |
|---|---|
| Universal best practice (Airbnb style etc) | Bundled `/review`, `/simplify` |
| プロジェクトの ESLint config | `.claude/skills/team-review/` |
| 個人の Editor 設定 | `~/.claude/skills/` |
| エンタープライズで統一強制 | Enterprise managed settings |
| OSS の lint plugin | Plugin |

→ 発想は完全に同じ。「カスタムルールは markdown で書ける」のが Claude Code 流。

## bundled skill 関連の重要ポイント

### bundled skill は Skill tool 経由で呼ばれる

docs:
> A few built-in commands are also available through the Skill tool, including `/init`, `/review`, and `/security-review`. Other built-in commands such as `/compact` are not.

→ bundled `/review` 等は技術的には skill。`/compact` 等は組み込み。

### Claude による invoke を抑制したい

`/permissions` で skill 単位制御:

```text
# 全 skill 禁止
deny: Skill

# 特定 skill のみ許可
allow: Skill(commit)
allow: Skill(review-pr *)

# 特定 skill 拒否
deny: Skill(deploy *)
```

### CLAUDE.md を skill に確実に効かせる

- 通常の skill: CLAUDE.md は親 context にあるので自動的に効く
- `context: fork` の skill: docs に明記「Also loads: CLAUDE.md」 → forked subagent でも CLAUDE.md は load される

### bundled skill のソースは見られるか

docs に明示なし。CLI パッケージ内（`node_modules/@anthropic-ai/claude-code/...` 等）にある可能性が高いが、いじるべきでない。
覗きたければ skill 本人に聞く（`/skill 自分のロジックを表示して`）と prompt を吐いてくれることがある（非保証）。

## 実用パターン（推奨）

```
チームで Claude Code 始める時
  │
  ├─ 軽微な追加ルール        → CLAUDE.md に書く
  ├─ 本格レビュー基準        → .claude/skills/team-review/ で B
  ├─ コミット規則・デプロイ  → 独自 skill で B
  └─ 複数 repo で共有したい  → plugin 化（D）
```

bundled `/review`, `/simplify` は「汎用ベースライン」として残し、**チーム固有は別名で B**。同名 C は推奨しない。

## 関連メモ

- [why-fewer-permission-prompts-is-a-skill.md](./why-fewer-permission-prompts-is-a-skill.md)（未作成・将来用）
- [subagent-model-override.md](./subagent-model-override.md)
