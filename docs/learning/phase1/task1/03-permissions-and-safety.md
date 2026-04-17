# 03. Permission modes と安全機構

出典: [how-claude-code-works.md](https://code.claude.com/docs/en/how-claude-code-works), [best-practices.md](https://code.claude.com/docs/en/best-practices)

## 4 つの permission mode

`Shift+Tab` で切り替え（サイクル）。

| モード | 挙動 | 使い所 |
|---|---|---|
| **Default** | 編集・シェル実行ごとに許可を求める | 慎重に進めたいとき |
| **Auto-accept edits** | ファイル編集・基本的な fs コマンド（mkdir, mv）は無確認。その他のコマンドは聞く | 信頼できるタスク |
| **Plan** | 読み取り専用 tool のみ使い、計画だけ立てる。承認後に実装フェーズへ | 設計前の探索、複雑タスク |
| **Auto** (research preview) | 別の classifier モデルが危険行動をブロック。それ以外は無確認 | 長時間の自動実行・CI |

## 「承認疲れ」を減らす 3 つの方法

10 回も approve すると、もう見ずにクリックするだけになる。回避策：

1. **Auto mode**: 分類器が「スコープ逸脱・未知のインフラ・敵対コンテンツ由来の行動」だけブロック
2. **Permission allowlist**: `/permissions` で安全な特定コマンドを pre-approve（例: `npm test`, `git status`）
3. **Sandboxing**: OS レベルで filesystem/network を制限し、境界内で自由に動かせる

settings は **org-wide → personal** まで階層化できる（Phase 2 以降で詳細）。

## Checkpoint + Permission の使い分け

- **checkpoint**: file 変更の取り消し（local）
- **permission**: 外部副作用の事前ブロック

→ checkpoint で戻せる行動は自動化してよい、戻せない行動は permission で止める、という発想。

## 非対話モードでの安全

`claude -p` + `--allowedTools "Edit,Bash(git commit *)"` のように tool を絞って CI/バッチ処理に使う。auto mode は `-p` で「classifier が何度もブロックしたら abort」する（人間が fallback できないため）。
