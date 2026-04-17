# claude-code-practice - プロジェクトコンテキスト

## 目的

Claude Code 学習プロジェクト。公式 docs/reference（全114ページ）を網羅しながら、Phase ごとに成果物を作って基礎から統合までを 5 Phase で学ぶ。

## 最重要ルール（IMPORTANT: 毎セッション守ること）

- **YOU MUST: 勝手に実装を進めない**。各ステップで「解説 → ユーザー確認 → 実装 → 振り返り」の順を守る。ユーザーが「進めて」と明示するまで次に進まない
- **公式 docs/reference に沿って進める**。各 Task の冒頭で該当ページを WebFetch で読み、解説の末尾に出典 URL を必ず書く
- **脱線トピックには Phase ラベルを明示**。例「これは Phase 4 で本格扱いです」
- **学習終了時に全114ページに触れる**。読了した URL は `plans/claude-code-learning-phases-design.md` の checklist で `[x]` に更新

## docs 取得時の URL 形式

- 人間向け canonical: `https://code.claude.com/docs/en/<slug>`（**`.md` を付けない**）
- LLM 用 raw markdown: `https://code.claude.com/docs/en/<slug>.md`（CSS なし、リンクでユーザーに渡すと壊れる。WebFetch 用途のみ）
- 全ページインデックス: `https://code.claude.com/docs/llms.txt`

## 学習ノートの保存規則

- トピック別ノート: `docs/learning/phase{N}/task{M}/NN-<topic>.md`
- 深掘りリファレンス: `docs/learning/phase{N}/task{M}/reference/<topic>.md`
- Task ごとの目次: `docs/learning/phase{N}/task{M}/main.md`（読んだ docs の checklist + 振り返りを含む）
- 各ノートの冒頭に `出典: [page-name](URL)` を必ず書く

## 関連ドキュメント

- `plans/claude-code-learning-phases-design.md` — 全 Phase の設計 + URL チェックリスト
- `docs/LEARNING_CONTEXT.md` — 学習フローの全体像、Phase 進捗テーブル
- `plans/phase1-implementation.md` 等 — 各 Phase の Task 分割
