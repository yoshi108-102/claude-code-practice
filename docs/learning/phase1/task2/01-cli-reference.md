# 01. CLI リファレンス — 起動コマンドとフラグ

> 出典: [CLI reference](https://code.claude.com/docs/en/cli-reference)（閲覧日 2026-05-08）
> このノートは公式ドキュメント「CLI reference」の構造を起点に Claude が自動生成した教材です。

## 概要

Claude Code を**ターミナルから起動・制御する方法**のリファレンス。`claude` 単独実行で対話セッションが始まるほか、サブコマンド（`claude -p`, `claude -c`, `claude update` 等）と豊富な CLI フラグでセッションの振る舞いを細かく指定できる。

`claude --help` には全フラグは出ない（公式 docs 注記あり）ので、フラグを探す場合は本ページが一次情報。

## 公式docsに沿った解説

### CLI commands（起動・セッション制御サブコマンド）

セッション開始や継続、認証、メンテナンス系の動作を行う。代表例:

| コマンド | 役割 |
|---|---|
| `claude` | 対話セッションを開始 |
| `claude "query"` | 初期プロンプト付きで対話開始 |
| `claude -p "query"` | **非対話（print）モード**で 1 回応答して終了。SDK 経由のスクリプトで使う想定 |
| `cat file \| claude -p "query"` | パイプ入力を流し込んで非対話実行 |
| `claude -c` | カレントディレクトリで**最後の会話を継続** |
| `claude -r "<session>" "query"` | ID または名前で**特定セッションを再開** |
| `claude update` | 最新版にアップデート |
| `claude install [version]` | 指定バージョン (`2.1.118` / `stable` / `latest`) のネイティブバイナリをインストール |
| `claude auth login` / `logout` / `status` | Anthropic アカウントへのサインイン状態を管理。`auth status` は JSON 出力。0 = ログイン済み、1 = 未ログイン |
| `claude agents` | 設定済み subagent 一覧 |
| `claude auto-mode defaults` | Auto mode の組み込み classifier ルールを JSON でダンプ |
| `claude mcp` | MCP サーバー設定 |
| `claude plugin` (alias `plugins`) | プラグイン管理 |
| `claude project purge [path]` | プロジェクト関連のローカル状態（transcript、task list、debug log、編集履歴、`~/.claude.json` のエントリ）を**まとめて削除**。`--dry-run` でプレビュー、`-y` で確認スキップ |
| `claude remote-control` | リモコンサーバーモードで起動（claude.ai / Claude app から操作） |
| `claude setup-token` | CI / スクリプト用の長寿命 OAuth トークン発行（`subscription` プラン必要） |
| `claude ultrareview [target]` | ultrareview を非対話で実行。`--json` / `--timeout <minutes>` をサポート |

サブコマンドのタイプミスは「Did you mean ... ?」と提案され、誤起動はしない（`claude udpate` → `claude update` の提案）。

### CLI flags（セッションの振る舞いをチューニング）

特に押さえておきたい主要フラグを目的別に整理:

#### 起動モード
- `--print` / `-p` … **非対話モード**。応答後に終了。`-p` は Agent SDK 的なスクリプト連携向け
- `--continue` / `-c` … カレントディレクトリの直近会話を継続
- `--resume` / `-r` … セッション ID/名前指定で再開、または対話的に選択
- `--fork-session` … `--resume` / `--continue` 時に**別の session ID で開始**して元を保持
- `--from-pr` … PR 番号 / URL から、その PR に紐づく会話を再開
- `--worktree` / `-w` … `.claude/worktrees/<name>` に隔離 git worktree を作って開始。`-w '#123'` で PR ブランチを取得して worktree 化
- `--session-id <UUID>` … 任意の UUID をセッション ID として使う

#### モデル / エフォート
- `--model <name|alias>` … `sonnet` / `opus` / フルモデル ID 指定
- `--effort <low|medium|high|xhigh|max>` … この session の effort level
- `--fallback-model <name>` … 既定モデル過負荷時の自動フォールバック（**print モード限定**）

#### システムプロンプト（4 種）
- `--system-prompt "..."` … デフォルトを**完全に置換**
- `--system-prompt-file <path>` … ファイルでデフォルトを置換
- `--append-system-prompt "..."` … デフォルトに**追記**
- `--append-system-prompt-file <path>` … ファイル内容を追記

`--system-prompt(-file)` と `--append-system-prompt(-file)` は併用可（置換版どちらか一方 + 追記版）。**通常は append を推奨**（Claude Code の組み込み能力を保ったまま要件追加できる）。

#### 権限・許可
- `--permission-mode <default|acceptEdits|plan|auto|dontAsk|bypassPermissions>` … **開始時の permission mode** を指定
- `--dangerously-skip-permissions` … `--permission-mode bypassPermissions` と同等
- `--allow-dangerously-skip-permissions` … `Shift+Tab` の循環に bypassPermissions を追加するだけで、開始は別モード（plan 等）
- `--allowedTools` / `--disallowedTools` … 個別 tool の許可・拒否リスト
- `--tools` … 使用可能ツールを制限（`""` で全無効、`"default"` で全許可、`"Bash,Edit,Read"` のように指定）
- `--permission-prompt-tool` … 非対話モードで permission を扱う MCP tool を指定

#### 入出力 / 自動化
- `--output-format <text|json|stream-json>` … print モードの出力フォーマット
- `--input-format <text|stream-json>` … print モードの入力フォーマット
- `--include-partial-messages` / `--include-hook-events` / `--replay-user-messages` … `stream-json` で詳細イベントを取得
- `--max-turns <N>` … print モードの最大ターン数
- `--max-budget-usd <amount>` … print モードのコスト上限
- `--json-schema '<schema>'` … 完了時に schema 検証された JSON を返す（structured outputs）

#### 設定・拡張
- `--add-dir <path...>` … 作業ディレクトリ追加（**ファイルアクセス権だけ**付与、`.claude/` 設定は基本ロードしない）
- `--mcp-config <json...>` / `--strict-mcp-config` … MCP 設定の上書き
- `--plugin-dir <path>` / `--plugin-url <url>` … セッション限定でプラグインを読み込む
- `--settings <path|json>` / `--setting-sources <user,project,local>` … 設定ファイルを切り替え
- `--bare` … hook / skill / plugin / MCP / auto memory / CLAUDE.md を**全部スキップ**して高速起動。`Bash` / read / edit のみ使えるシンプルモード（スクリプト用）
- `--betas <name>` … API ベータヘッダ（API key ユーザーのみ）

#### IDE / 表示
- `--ide` … 有効な IDE が 1 つだけある場合は自動接続
- `--chrome` / `--no-chrome` … Chrome 連携の ON/OFF
- `--remote` / `--remote-control` / `--teleport` … claude.ai 上の Web セッションとの連動
- `--name` / `-n` … セッションに表示名を付与（`/resume` から再開できる）
- `--verbose` … ターン単位の詳細ログ
- `--debug "<categories>"` / `--debug-file <path>` … デバッグログのカテゴリ絞り込み

#### Hook 系
- `--init` … session 開始前に Setup hook の `init` matcher を実行（print モードのみ）
- `--init-only` … Setup + SessionStart hook を回して**会話せずに終了**
- `--maintenance` … `maintenance` matcher の Setup hook 実行（print モードのみ）

#### その他
- `--exclude-dynamic-system-prompt-sections` … マシン依存情報（cwd / env / git）をシステムプロンプトから外し、最初のユーザーメッセージへ移動。**マルチユーザー prompt cache の hit rate を上げる**用途
- `--disable-slash-commands` … skill / command を全部無効化
- `--no-session-persistence` … セッション永続化を切る（print モード限定）
- `--agents '<json>'` / `--agent <name>` … subagent をその場で定義 / 既存 agent を選択

### システムプロンプトフラグの整理

| フラグ | 振る舞い |
|---|---|
| `--system-prompt` | デフォルトを完全に置換（テキスト） |
| `--system-prompt-file` | デフォルトをファイル内容で置換 |
| `--append-system-prompt` | デフォルトに**追記**（テキスト） |
| `--append-system-prompt-file` | デフォルトに**追記**（ファイル） |

- `--system-prompt` と `--system-prompt-file` は**排他**
- append 系は置換系どちらかと併用可
- 多くの場合 **append** を使う方が安全（Claude Code 標準機能が保たれる）

## 重要ポイント

- `claude` 1 コマンドで対話、`-p` 付けて非対話、`-c`/`-r` でセッション再開、という**3 系統の起動形態**を覚えるとほぼカバーできる
- 非対話 (print) 限定のフラグ（`--max-turns`, `--max-budget-usd`, `--fallback-model`, `--no-session-persistence` 等）は対話セッションでは使えない
- セキュリティ系フラグ（`--dangerously-skip-permissions`, `--allow-dangerously-skip-permissions`）は名前通り「危険」。CI でも基本は permission 設定や Auto mode を活用すべき
- `--bare` はスクリプトから高速起動したいときの選択肢。skill / hook / plugin の load を全部省くため、軽量だが**機能制限が大きい**
- `claude project purge` は**ローカル状態を破壊的に消す**サブコマンド。Claude が勝手に呼んではいけないし、ユーザー側でも `--dry-run` で確認するのが安全
- `auth status` は JSON で `exit code` も含めて返るので、CI で「ログイン済みか」を判定するのに使える

## コード例

### 非対話で 1 回だけ質問してすぐ終了（CI 想定）

```bash
# テンプレ: 出力を JSON で受け取り、ターン数 3 まで、コスト上限 $0.50
claude -p "Run tests and summarize failures" \
  --output-format json \
  --max-turns 3 \
  --max-budget-usd 0.50
```

### 既存セッション再開 + plan モードで開始

```bash
claude --resume auth-refactor --permission-mode plan
```

### worktree を切って PR ブランチで作業

```bash
# PR #123 を fetch して隔離 worktree で作業開始
claude -w '#123'
```

### システムプロンプトに追加要件だけ足す

```bash
claude --append-system-prompt "Always respond in Japanese. Use TypeScript for examples."
```

## 関連

- 議論・Q&A:
  - [reference/claude-agents-cli-and-sources.md](reference/claude-agents-cli-and-sources.md) — `claude agents` CLI と subagent の source 階層
  - [reference/plan-mode-vs-plan-subagent.md](reference/plan-mode-vs-plan-subagent.md) — plan mode と Plan subagent の区別
  - [reference/worktree-usage-and-tradeoffs.md](reference/worktree-usage-and-tradeoffs.md) — worktree の必須条件 / merge / branch 切替との比較
- 次の教材: [02-slash-commands.md](02-slash-commands.md) — セッション内のスラッシュコマンド
- 関連 docs: [Settings](https://code.claude.com/docs/en/settings) / [Permission modes](https://code.claude.com/docs/en/permission-modes) / [Headless mode](https://code.claude.com/docs/en/headless)

---

_Auto-generated at 2026-05-08 via /learning-flow:material（公式docs駆動）_
