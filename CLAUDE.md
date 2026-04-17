# claude-code-practice - プロジェクトコンテキスト

## 目的

Claude Code 学習プロジェクト。公式 docs/reference（全114ページ）を網羅しつつ、Phase ごとに実際の成果物（CLAUDE.md → skill → hook → subagent → plugin → MCP サーバー → Agent SDK エージェント）を作りながら、基礎から高度な統合までを5 Phaseで学ぶ。

## 最重要ルール

- **公式 docs/reference に沿って進める**: 各 Task は公式ページを根拠にする。Phase 終了時にそのPhaseの担当ページを全て読み終わっている状態にする
- **学習終了時に全114ページに触れる**: `plans/claude-code-learning-phases-design.md` の URL チェックリストで進捗管理
- **勝手に実装を進めない**: 各ステップで 解説 → ユーザー確認 → 実装 の順を守る
- **質疑応答は docs に残す**: `docs/learning/phase{N}/task{M}/` にトピック別 md で記録
- **reference は別 md**: 深掘り情報は `reference/` ディレクトリに分離

## 現在の進捗

Phase 1 の学習準備段階。詳細は `docs/LEARNING_CONTEXT.md` を参照。

## ドキュメント構成

- `plans/` — Phase 設計書、Task 実装計画
- `docs/learning/` — 学習ノート（Phase/Task 別）
- `docs/LEARNING_CONTEXT.md` — 学習フローの全体像、進捗、ドキュメントルール

## 公式 docs の参照元

- 一次ソース: https://code.claude.com/docs/en/overview
- 全ページインデックス: https://code.claude.com/docs/llms.txt
