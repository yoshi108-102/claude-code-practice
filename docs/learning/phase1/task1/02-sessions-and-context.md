# 02. セッション・コンテキスト管理・checkpoint

出典: [how-claude-code-works.md](https://code.claude.com/docs/en/how-claude-code-works), [best-practices.md](https://code.claude.com/docs/en/best-practices)

## セッション = ディレクトリに紐付く会話ログ

- 保存先: `~/.claude/projects/` 以下に JSONL
- 新しいセッションは **fresh context** で始まる（前のセッションの履歴はない）
- 永続化したいルールは `CLAUDE.md`、学習した事実は auto memory (`MEMORY.md`)
- `claude --continue` で最新セッション再開 / `claude --resume` で一覧から選択
- `claude --continue --fork-session` で会話履歴を保ったまま別ブランチで実験

### worktree と並列セッション

セッションはディレクトリに紐付くので、git worktree で branch ごとに独立ディレクトリを作れば Claude セッションも並列に走らせられる。

## context window はすぐ埋まる

context window に載るもの:
- 会話履歴（全メッセージ）
- 読んだファイルの内容
- tool 実行結果（特に大きい）
- CLAUDE.md / MEMORY.md / loaded skills / system instructions

**「LLM は context が埋まるほど性能が落ちる」** が最重要原則。best-practices の多くはこれへの対策。

### context 管理のツール

| コマンド / 機能 | 役割 |
|---|---|
| `/context` | 現在の context 使用量を可視化 |
| `/compact` | 会話を要約して圧縮（`/compact focus on X` で観点指定可） |
| `/clear` | context 完全リセット |
| `/btw` | 「ちょっとした質問」を履歴に残さず聞く |
| `/rewind` / Esc×2 | 任意地点までロールバック（会話のみ / コードのみ / 両方） |
| **subagents** | 別の context window で作業させ、要約だけ返す |
| **skills** | description のみ常時ロード、本体は on-demand |

auto-compaction が効かないほど巨大な単一ファイル/出力がある場合は "thrashing error" が出る。

## Checkpointing：file 編集の自動スナップショット

- Claude がファイルを編集する**前に**毎回スナップショットを取る
- `Esc + Esc` または `/rewind` で、会話のみ/コードのみ/両方を任意地点に戻せる
- checkpoint は session-local、**git の代替ではない**
- 外部副作用（DB, API, deploy）は checkpoint できないので、そういうコマンドには個別に許可を求める

→ **"危険そうなことを試させて、ダメなら戻す"** という使い方が推奨される。
