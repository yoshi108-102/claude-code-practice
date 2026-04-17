# 01. agentic loop と 5 つの tool カテゴリ

出典: [how-claude-code-works.md](https://code.claude.com/docs/en/how-claude-code-works)

## agentic loop の 3 フェーズ

Claude Code は **gather context → take action → verify results** の3フェーズをブレンドしながらループする。

- **gather context**: ファイルを読む、grep/glob で探す、web 検索する
- **take action**: ファイル編集、コマンド実行、テスト実行、git 操作
- **verify results**: テストを走らせる、ビルドする、出力を確認する

フェーズは固定ではなく、**タスクが何を必要とするかに応じて Claude 自身が決める**。例: バグ修正は 3 フェーズを繰り返す。質問応答なら gather だけで終わる。

ユーザーは loop の途中で `Esc` で割り込んで方向転換できる（"人間もループの一部"）。

## 「agentic harness」という考え方

Claude Code は、Claude モデルに **agency（行動能力）を与える外殻**。モデル単体はテキストしか出せないが、Claude Code が tools と実行環境を提供することで、コードを読み・書き・動かせるエージェントになる。

## 5 つの tool カテゴリ

| カテゴリ | 例 |
|---|---|
| **File operations** | Read, Edit, Write, ファイル名変更 |
| **Search** | Glob（パターン検索）, Grep（内容検索） |
| **Execution** | Bash（シェル実行）, テスト・git |
| **Web** | WebSearch, WebFetch（ドキュメント参照・エラーメッセージ検索） |
| **Code intelligence** | 型エラー検出、定義ジャンプ、参照検索（**plugin 必要**） |

これに加えて subagents を spawn する、ユーザーに質問する（AskUserQuestion）などのオーケストレーション tool もある。

拡張性: skills / MCP / hooks / subagents は **この 5 カテゴリの上に乗る層**。Phase 3 以降で扱う。

## Claude が各セッションでアクセスできるもの

- 現在のディレクトリとサブディレクトリのファイル
- ターミナル（あなたがコマンドラインで実行できること全て）
- git 状態（branch, uncommitted changes, 履歴）
- `CLAUDE.md`（プロジェクト固有指示）
- `MEMORY.md` の先頭 200 行または 25KB（auto memory）
- 設定した MCP servers / skills / subagents / Chrome 拡張

→ "single-file assistant" ではなく **プロジェクト横断** で動ける点が最大の違い。
