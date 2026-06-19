# ツールリファレンス — 組み込みツール一覧と動作

> 出典: [Tools reference](https://code.claude.com/docs/en/tools-reference)（閲覧日 2026-06-12）
> このノートは公式ドキュメントの「tools-reference」を起点に、Claudeが自動生成した教材です。

## 概要

Claude Code が使える組み込みツールのリファレンス。ツール名は permissions ルール・サブエージェント tool リスト・hook matcher で使う正確な文字列。ツールを完全無効化するには permission settings の `deny` 配列に名前を追加する。カスタムツールを追加するには MCP サーバーを接続する。

## 公式 docs に沿った解説

### ツール一覧

| ツール | 説明 | パーミッション必要 |
|---|---|---|
| `Agent` | サブエージェントをスポーン（独立コンテキストで処理、結果だけを返す） | No |
| `AskUserQuestion` | 複数選択の質問でユーザーに確認 | No |
| `Bash` | シェルコマンドを実行 | Yes |
| `CronCreate` | セッション内で定期・ワンショットプロンプトをスケジュール | No |
| `CronDelete` | スケジュールタスクをキャンセル | No |
| `CronList` | セッションの全スケジュールタスクをリスト | No |
| `Edit` | ファイルへのターゲット編集 | Yes |
| `EnterPlanMode` | Plan モードに切り替え（コーディング前のアプローチ設計） | No |
| `EnterWorktree` | 隔離 git worktree を作成して切り替え | No |
| `ExitPlanMode` | プランを提示して Plan モードを終了 | Yes |
| `ExitWorktree` | Worktree セッションを終了してもとのディレクトリに戻る | No |
| `Glob` | パターンマッチングでファイルを検索 | No |
| `Grep` | ファイル内容でパターンを検索 | No |
| `ListMcpResourcesTool` | 接続中 MCP サーバーのリソースをリスト | No |
| `LSP` | 言語サーバー経由のコードインテリジェンス（定義ジャンプ・参照・型エラー） | No |
| `Monitor` | バックグラウンドでコマンドを実行し、各出力行を Claude に返す | Yes |
| `NotebookEdit` | Jupyter notebook セルを変更 | Yes |
| `PowerShell` | PowerShell コマンドをネイティブ実行 | Yes |
| `PushNotification` | デスクトップ通知を送信（Remote Control 接続時はスマートフォン通知も） | No |
| `Read` | ファイル内容を読み取る | No |
| `ReadMcpResourceTool` | URI で特定の MCP リソースを読み取る | No |
| `RemoteTrigger` | claude.ai 上で Routines を作成・実行 | No |
| `ScheduleWakeup` | 自己ペース `/loop` の次のイテレーションをスケジュール | No |
| `SendMessage` | Agent team のチームメイトにメッセージ送信、またはサブエージェントを再開 | No |
| `ShareOnboardingGuide` | `ONBOARDING.md` をアップロードして共有リンクを返す | Yes |
| `Skill` | Skill を main 会話内で実行 | Yes |
| `TaskCreate` | タスクリストに新しいタスクを作成 | No |
| `TaskGet` | 特定タスクの詳細を取得 | No |
| `TaskList` | 全タスクとその状態をリスト | No |
| `TaskStop` | 実行中のバックグラウンドタスクを停止 | No |
| `TaskUpdate` | タスクのステータス・依存関係・詳細を更新 | No |
| `TeamCreate` | Agent team を複数チームメイトで作成 | No |
| `TeamDelete` | Agent team を解散してチームメイトプロセスをクリーンアップ | No |
| `ToolSearch` | tool search 有効時に遅延ツールを検索・ロード | No |
| `WebFetch` | URL からコンテンツを取得 | Yes |
| `WebSearch` | Web 検索を実行 | Yes |
| `Workflow` | 動的ワークフローを実行（複数サブエージェントをオーケストレーションして1つの結果を返す） | Yes |
| `Write` | ファイルを作成または上書き | Yes |

### パーミッションルールと Hook での参照形式

ツール名を使う場面:
- `permissions.allow` / `permissions.deny` の settings
- `--allowedTools` / `--disallowedTools` CLI フラグ
- Agent SDK の `allowedTools` / `disallowedTools` オプション
- サブエージェント定義の `tools` / `disallowedTools` フロントマター
- Skill の `allowed-tools` フロントマター
- Hook の `if` 条件

全てで同じルール形式 `ToolName(specifier)` が使える:

| ルール形式 | 適用先 | 詳細 |
|---|---|---|
| `Bash(npm run *)` | Bash, Monitor | コマンドパターンマッチング |
| `PowerShell(Get-ChildItem *)` | PowerShell | コマンドパターンマッチング |
| `Read(~/secrets/**)` | Read, Grep, Glob, LSP | パスパターンマッチング |
| `Edit(/src/**)` | Edit, Write, NotebookEdit | パスパターンマッチング |
| `Skill(deploy *)` | Skill | Skill 名マッチング |
| `Agent(Explore)` | Agent | サブエージェントタイプマッチング |
| `WebFetch(domain:example.com)` | WebFetch | ドメインマッチング |
| `WebSearch` | WebSearch | 指定子なし（ツール全体を allow/deny） |

> Hook の `matcher` フィールドはベアのツール名を使う（括弧付き形式は使わない）。

> `Edit(...)` の allow ルールは同じパスへの `Read(...)` も自動的に許可するため、別途 `Read(...)` ルールは不要。

### 各ツールの動作詳細

#### Agent ツール

サブエージェントを独立したコンテキストウィンドウでスポーン。中間ツール呼び出しや出力は親会話に見えず、最終結果だけが返る。

`tools` / `disallowedTools` フィールドによるツール制御:
- **どちらも未設定**: 親で利用可能な全ツールを継承
- **`tools` のみ**: リストされたツールだけ使える
- **`disallowedTools` のみ**: リストされたツール以外全部使える
- **両方設定**: `disallowedTools` が優先。両方にあるツールは除去

**フォアグラウンド vs バックグラウンド**:
- フォアグラウンド: 同じパーミッションプロンプトが表示される
- バックグラウンド: プロンプトは表示しない。許可済みパーミッションで動作し、未許可は自動拒否してから続行

#### Bash ツール

各コマンドは別プロセスで実行される。いくつかの永続性ルール:

- `cd` でのディレクトリ変更は main セッションでは後続 Bash コマンドに引き継がれる（プロジェクトディレクトリ内またはadditional directories 内のみ）。サブエージェントセッションでは引き継がれない
- 環境変数は引き継がれない。次の `export` は次のコマンドに引き継がれない
- シェルの alias と関数はセッション開始時にソースされ、全 Bash コマンドで利用可能

**制限値**:
- タイムアウト: デフォルト 2 分（最大 10 分）。`BASH_DEFAULT_TIMEOUT_MS` と `BASH_MAX_TIMEOUT_MS` で変更可
- 出力長: デフォルト 30,000 文字。超過したら全出力をファイルに保存してパスを通知。`BASH_MAX_OUTPUT_LENGTH` で変更可（上限 150,000 文字）

#### Edit ツール

正確な文字列置換。`old_string` を `new_string` に置換（regex やファジーマッチなし）。

3 つのチェックを通過する必要がある:
1. **Read-before-edit**: 現在の会話でファイルを読んでいること。かつ読んだ後にファイルが変更されていないこと
2. **Match**: `old_string` がファイルにそのままの形で存在すること
3. **Uniqueness**: `old_string` がファイルに 1 回だけ登場すること（複数回ある場合は周辺コンテキストを追加するか `replace_all: true`）

`cat`, `head`, `tail`, `sed -n 'X,Yp'`, `grep` 等で Bash を使ったファイル表示も Read-before-edit の要件を満たす（ただしパイプやリダイレクトがない場合のみ）。

#### Glob ツール

名前パターンでファイルを検索。結果は変更日時でソートされ最大 100 ファイルまで。

- `**/*.js`: 全深度の .js ファイル
- `src/**/*.ts`: src/ 以下の .ts ファイル
- `*.{json,yaml}`: カレントディレクトリの .json と .yaml

デフォルトで `.gitignore` を尊重しない（gitignore されたファイルも見つかる）。`.gitignore` を尊重させるには `CLAUDE_CODE_GLOB_NO_IGNORE=false` を設定。

#### Grep ツール

ファイル内容でパターンを検索（[ripgrep](https://github.com/BurntSushi/ripgrep) ベース、正規表現構文）。

3 つの出力モード:
- `files_with_matches`: ファイルパスのみ（デフォルト）
- `content`: マッチした行（ファイル名・行番号付き）
- `count`: ファイルごとのマッチ数

`glob` パラメータでファイルスコープ、`type` パラメータで言語スコープ（`py`, `rust` 等）。`multiline: true` で複数行マッチ。デフォルトで `.gitignore` を尊重する（gitignore されたファイルはスキップ）。

#### LSP ツール

コードインテリジェンス（言語サーバー経由）。ファイル編集後に型エラー・警告を自動報告。また以下のオンデマンドナビゲーション:
- シンボルの定義へジャンプ
- シンボルの全参照を検索
- 型情報の取得
- ファイル内シンボルのリスト
- ワークスペース全体でシンボルを名前で検索
- インターフェースの実装を検索
- 呼び出し階層のトレース

コードインテリジェンスプラグインをインストールしないと非アクティブ。

#### Monitor ツール

バックグラウンドでコマンドを実行し、各出力行を Claude に返す。ログファイルの tail・CI ジョブの状態監視・ディレクトリの変更監視などに使用。

Claude Code v2.1.98 以降が必要。Bash と同じパーミッションルールを使用。Amazon Bedrock, Google Vertex AI, Microsoft Foundry では利用不可。

#### Read ツール

ファイルパスを受け取り、行番号付きでコンテンツを返す。

特殊ファイルタイプのサポート:
- **画像**: PNG, JPG 等は視覚的コンテンツとして返す（大きな画像はリサイズ・再圧縮される）
- **PDF**: 短いものは全体、10 ページ超は `pages` パラメータで範囲指定（最大 20 ページ）
- **Jupyter notebooks**: `.ipynb` は全セル（コード・マークダウン・可視化）を返す

#### WebFetch ツール

URL とプロンプトを受け取る。HTML は Markdown に変換してから小型・高速モデルでプロンプトを処理し、その結果を返す（ロッシーな設計）。

- HTTP URL は自動的に HTTPS にアップグレード
- 大きなページは固定文字数に切り詰め
- 15 分間キャッシュ
- 異なるホストへのリダイレクトはフォローしない（新しい URL での再フェッチを促す）

デフォルトモードと `acceptEdits` モードでは最初にドメインに到達した時に確認プロンプトが出る（事前承認済みドキュメントドメインは除く）。`WebFetch(domain:example.com)` ルールで事前承認可能。

#### WebSearch ツール

Anthropic の Web Search バックエンドで検索を実行し、タイトルと URL を返す。結果ページ自体は取得しない（WebFetch でフォローアップ）。

1 回のコールで最大 8 回のバックエンド検索を内部で実行。`allowed_domains` で特定ホストに絞れるが `blocked_domains` との同時指定は不可。

Amazon Bedrock では利用不可。

#### Write ツール

新規ファイル作成または既存ファイルの完全上書き（追記・マージなし）。

既存ファイルを上書きする場合は、現在の会話でそのファイルを少なくとも 1 回読んでいる必要がある。新規ファイルにはこの制約なし。部分的な変更には Edit を使う。

## 重要ポイント

- ツール名は正確な文字列（大文字小文字を区別）。permissions・Hook matcher・サブエージェント定義で使用
- `Edit(...)` の allow ルールは同じパスへの `Read(...)` も自動許可する
- Hook の `matcher` はベアのツール名（括弧なし）。permissions ルールとは形式が異なる
- Bash での `cd` はメインセッションでは引き継がれるが、サブエージェントでは引き継がれない
- Read-before-edit ルール: Edit も Write も事前に Read していないと失敗する
- Glob は `.gitignore` を無視（デフォルト）、Grep は `.gitignore` を尊重（デフォルト）
- WebFetch はロッシーな設計。raw データが必要なら `curl` via Bash を使う

## コード例 / 図

### 現在利用可能なツールを確認する方法

Claude に直接聞く:
```
What tools do you have access to?
```

MCP ツールを確認するには `/mcp` コマンドを使う。

### ツール制御の組み合わせ例

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Read(~/projects/**)",
      "WebFetch(domain:docs.example.com)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(~/.ssh/**)",
      "Write(~/.claude/**)"
    ]
  }
}
```

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## Phase 3 完了

Phase 3（拡張機能）の全 7 トピックが完了しました。

- [Phase 3 トップへ](../main.md)

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
