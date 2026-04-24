# learning-flow

段階的な学習プロジェクト（Phase/Task構成）の進行を支援する Claude Code プラグイン。

「勝手に進めず、解説 → 確認 → 実装 → 振り返り → 次へ」のサイクルを守りながら、学習ノート（Q&A・リファレンス）を自動で整理する。

## 提供スキル

| スキル | 種類 | 役割 |
|---|---|---|
| `learning-flow-rules` | 自動発動 | 学習プロジェクト検出時に基本ルールを適用 |
| `/learning-flow:init` | スラッシュ | 新規学習プロジェクトの初期化 |
| `/learning-flow:status` | スラッシュ | 現在の Phase/Task 進捗を表示 |
| `/learning-flow:topic <name>` | スラッシュ | 会話を学習トピック md として保存 |
| `/learning-flow:reference <theme>` | スラッシュ | Web 検索込みの深掘り reference md を生成 |
| `/learning-flow:next` | スラッシュ | 次タスクへ進む（事前チェック込み） |

## インストール

このプラグインは **user scope のローカルプラグイン** として配置されている想定。

設置パス: `~/.claude/plugins/local/learning-flow/`

Claude Code の設定で local plugin を有効化している場合は、起動時に自動ロードされる。
手動で有効化するには以下のいずれかを行う:

```bash
# 方法A: プラグインディレクトリ指定で Claude Code を起動
claude --plugin-dir ~/.claude/plugins/local/learning-flow

# 方法B: settings.json の plugins フィールドに追加（詳細は Claude Code ドキュメント参照）
```

## 使い方

### 1. 新規学習プロジェクトの初期化

学習したいテーマのディレクトリに移動してから:

```
/learning-flow:init
```

対話でテーマ・Phase 数・各 Phase の内容を聞かれる。完了すると以下が生成される:

- `CLAUDE.md`（プロジェクトコンテキスト）
- `docs/LEARNING_CONTEXT.md`（学習フロー全体像）
- `plans/{theme}-learning-phases-design.md`（Phase 設計書）
- `docs/learning/`（学習ノート置き場）

### 2. 学習サイクル

1. 解説 → 確認 → 実装 → 振り返り の各ステップで Claude が勝手に先へ進まない
2. トピックが一区切りついたら `/learning-flow:topic <name>` で保存
3. 深掘り調査が必要なら `/learning-flow:reference <theme>` で web 検索込みで保存
4. Task 完了時、`main.md` の振り返りを書いて `/learning-flow:next` で次タスクへ

### 3. 進捗確認

いつでも:

```
/learning-flow:status
```

## ディレクトリ構造

プラグインが扱う学習プロジェクトのレイアウト:

```
project-root/
├── CLAUDE.md
├── plans/
│   └── {theme}-learning-phases-design.md
└── docs/
    ├── LEARNING_CONTEXT.md
    └── learning/
        ├── phase1/
        │   ├── task1/
        │   │   ├── main.md             ← 目次 + 振り返り
        │   │   ├── 01-topic-a.md       ← トピック別 Q&A
        │   │   ├── 02-topic-b.md
        │   │   └── reference/
        │   │       └── deep-dive.md    ← web 検索込み深掘り
        │   └── task2/
        └── phase2/
```

## 設計方針

- **1 Phase = 新規概念 1〜2 個**: 詰め込みすぎず、段階的に
- **解説は会話で、記録は docs で**: 会話は流れるが、ノートは残る
- **referenceは深掘り専用**: トピック md は要約・Q&A、reference md はソース付き詳細
- **コードにも docs リンク**: 該当コード上に 1 行コメント + 学習ノートへのリンク

## ライセンス

MIT
