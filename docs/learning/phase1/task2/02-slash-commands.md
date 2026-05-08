# 02. スラッシュコマンドとバンドルスキル

> 出典: [Commands](https://code.claude.com/docs/en/commands)（閲覧日 2026-05-08）
> このノートは公式ドキュメント「Commands」の構造を起点に Claude が自動生成した教材です。

## 概要

セッション内で `/` から始まる入力で呼び出せる**コマンドの一覧**。モデル切り替え・権限管理・コンテキストクリア・ワークフロー実行などを行う。`/` 単独入力で全コマンドメニューが出て、文字を続ければフィルタされる。

コマンドは **メッセージの先頭でのみ認識**される。コマンド名のあとに続く文字列はそのコマンドへの引数として渡る（例 `/btw 質問内容` の「質問内容」が `/btw` の引数）。

ユーザーごとにメニューに出るコマンドは違う。**プラットフォーム / プラン / 環境**で可用性が変わる（`/desktop` は macOS / Windows のみ、`/upgrade` は Pro / Max のみ等）。

## 公式docsに沿った解説

### 「組み込みコマンド」と「バンドルスキル」の違い

公式表で **[Skill]** マーク付きのものは**バンドルスキル**で、自分で書く skill と同じ仕組み（プロンプトを Claude に渡して、関連時には Claude 自身が自動 invoke できる）。マーク無しは **CLI に直接実装された組み込みコマンド**で、コードで挙動が固定。

| 種別 | 実装場所 | 自動 invoke | カスタマイズ |
|---|---|---|---|
| 組み込みコマンド | CLI 本体のコード | 不可 | 不可 |
| バンドルスキル | プロンプト + `SKILL.md` 同等の構造 | 可 | 自作スキルで上書き可 |

→ 自作 skill は `/skill-name` で同じく呼べるので、**バンドルスキルと自作スキルは同じレールに乗っている**。

### 引数の表記

公式表では:
- `<arg>` … 必須
- `[arg]` … 省略可

例: `/copy [N]` は引数なしで最後の応答コピー、`N` を渡すと N 番目を遡る。

### 主要コマンド（カテゴリ別ピックアップ）

#### セッション管理 / コンテキスト
- `/clear` (alias: `/reset`, `/new`) … 空のコンテキストで新会話を開始（前会話は `/resume` で残る）
- `/compact [instructions]` … 会話を要約してコンテキストを解放。フォーカス指示も渡せる
- `/context` … 現在のコンテキスト使用量を**色付きグリッド**で可視化、最適化提案を表示
- `/rewind` (alias: `/checkpoint`, `/undo`) … 会話 / コードを過去地点へ巻き戻し（checkpointing）
- `/resume [session]` (alias: `/continue`) … セッション ID / 名前で再開、または picker
- `/branch [name]` (alias: `/fork`) … 現会話の枝を作成（元会話は `/resume` で戻れる）
- `/copy [N]` … 直近の応答をクリップボードへ（コードブロックがあれば picker、`w` でファイル書き出し）
- `/export [filename]` … 会話をテキストでエクスポート

#### モデル / 動作モード
- `/model [model]` … モデル切り替え（途中変更は確認が出る、応答完了を待たず即適用）
- `/effort [level|auto]` … effort level (`low|medium|high|xhigh|max|auto`)
- `/fast [on|off]` … fast mode の ON/OFF
- `/plan [description]` … plan mode に入る、引数で初期タスクも指定
- `/sandbox` … sandbox mode の toggle（対応プラットフォームのみ）

#### 権限・拡張
- `/permissions` (alias: `/allowed-tools`) … allow / ask / deny ルールの管理 UI
- `/agents` … subagent 設定の管理
- `/skills` … skill 一覧、`Space` で表示制御、`t` で token 順ソート
- `/plugin` / `/reload-plugins` … プラグイン管理 / 再読み込み
- `/mcp` … MCP サーバー接続・OAuth
- `/hooks` … hook 設定の確認

#### 編集 / レビュー
- `/diff` … 未コミットの diff + ターンごとの diff を viewer で表示
- `/review [PR]` … ローカルで PR レビュー（深い分析は `/ultrareview`）
- `/ultrareview [PR]` … クラウド sandbox での多エージェント review
- `/ultraplan <prompt>` … ultraplan セッションで plan を作って browser で確認
- `/security-review` … 現ブランチの未コミット差分のセキュリティレビュー

#### バンドルスキル（**[Skill]** マーク付き）
| コマンド | 役割 |
|---|---|
| `/batch <instruction>` | コードベース全体に並列で大規模変更（5〜30 unit に分解 → 各 unit を background agent + worktree） |
| `/claude-api [migrate\|managed-agents-onboard]` | Claude API/SDK ドキュメントを読み込む。`migrate` で既存コードのモデル ID 等を新版へ移行 |
| `/debug [description]` | デバッグログを ON にして session log を読み解析 |
| `/fewer-permission-prompts` | transcript を走査して頻出 read-only Bash/MCP を allowlist 化 |
| `/loop [interval] [prompt]` (alias: `/proactive`) | プロンプトを定期実行 |
| `/simplify [focus]` | 直近変更を 3 つの review agent で並列レビュー → 修正適用 |

#### 設定 / メモリ
- `/config` (alias: `/settings`) … テーマ / モデル / output style 等
- `/memory` … `CLAUDE.md` 編集、auto-memory の toggle と中身確認
- `/init` … プロジェクトに `CLAUDE.md` を生成（`CLAUDE_CODE_NEW_INIT=1` で skill / hook / personal memory も walkthrough）
- `/keybindings` … キーバインド設定ファイルを開く / 作る
- `/theme` … カラーテーマ
- `/statusline` … status line 設定
- `/terminal-setup` … `Shift+Enter` 等の terminal キーバインド設定

#### 認証 / 課金
- `/login` / `/logout` … Anthropic アカウント
- `/usage` (alias: `/cost`, `/stats`) … セッションコスト / 利用枠 / 統計
- `/extra-usage` … rate limit 超過時に extra usage を有効化
- `/upgrade` … プラン変更
- `/passes` … 友人に 1 週間共有（対象アカウントのみ）
- `/privacy-settings` … プライバシー設定（Pro/Max のみ）

#### 連携・配布
- `/install-github-app` … Claude GitHub Actions の設定
- `/install-slack-app` … Claude Slack app
- `/team-onboarding` … 過去 30 日のセッションから onboarding ガイド生成
- `/teleport` (alias: `/tp`) … claude.ai web セッションをローカルに引き込む
- `/remote-control` (alias: `/rc`) … claude.ai からリモコン可能にする
- `/remote-env` … `--remote` で起動する Web セッションのデフォルト環境
- `/web-setup` … `gh` 経由で claude.ai web に GitHub 接続
- `/desktop` (alias: `/app`) … Desktop app に引き継ぎ（macOS/Windows のみ）

#### モバイル / アプリ
- `/mobile` (alias: `/ios`, `/android`) … モバイルアプリの QR コード
- `/chrome` … Chrome 連携設定
- `/ide` … IDE 連携状態の表示

#### バックグラウンド / 自動化
- `/tasks` (alias: `/bashes`) … バックグラウンドタスク管理
- `/schedule [description]` (alias: `/routines`) … routine の作成・実行
- `/btw <question>` … サイド質問（履歴に残らない）
- `/recap` … セッションリキャップを手動生成

#### ヘルスチェック / 診断
- `/doctor` … インストール / 設定の診断、`f` で Claude に修正させる
- `/insights` … セッションの利用パターン分析
- `/heapdump` … メモリ使用量解析用にヒープスナップショット
- `/release-notes` … バージョン別リリースノート
- `/feedback [report]` (alias: `/bug`) … フィードバック / バグ報告
- `/status` … 設定 UI の Status タブ（応答中でも開ける）

#### Bedrock / Vertex 専用 wizard
- `/setup-bedrock` … `CLAUDE_CODE_USE_BEDROCK=1` 時のみ表示
- `/setup-vertex` … `CLAUDE_CODE_USE_VERTEX=1` 時のみ表示

#### その他
- `/color [color|default]` … プロンプトバーの色
- `/focus` … focus view の toggle（fullscreen 限定）
- `/tui [default|fullscreen]` … TUI レンダラ切り替え
- `/voice [hold|tap|off]` … voice dictation
- `/rename [name]` … セッション名の変更
- `/autofix-pr [prompt]` … 現ブランチ PR を Web で監視 + CI 失敗 / レビューコメント時に自動修正 push
- `/exit` (alias: `/quit`) … 終了
- `/help` … ヘルプ
- `/stickers` … ステッカー注文（ジョーク的だが本物）
- `/setup-token`（CLI 側のみ） … 長寿命 OAuth トークン

#### 削除されたコマンド（参考）
- `/vim` … v2.1.92 で削除。代わりに `/config` → Editor mode で切り替え
- `/pr-comments` … v2.1.91 で削除。代わりに「PR コメント見て」と Claude に直接指示

### MCP prompts

MCP サーバーが prompt を公開していると `/mcp__<server>__<prompt>` という形で**動的に**コマンドメニューに出る。組み込みコマンドではなくサーバー側の機能（Phase 5 で扱う）。

## 重要ポイント

- **コマンド = 文頭のみ**。文中の `/` は普通のテキスト
- **alias** が多い（`/clear` = `/reset` = `/new`、`/usage` = `/cost` = `/stats` 等）。覚えやすい方を 1 つ知っていれば十分
- **`/clear` と `/compact` は別物**。clear は会話を捨てる、compact は要約して残す
- **`/context` は超有用**。コンテキスト使用量を可視化して、何が context を食っているかが見える
- **バンドルスキルは「Claude が自分で呼ぶ」可能性がある**。同名で自作 skill を作れば上書きできる
- **`/btw` は履歴に残らないサイド質問**。長時間タスクの最中でも割り込みなしで使える（Phase 1 / Task 4 で再登場）
- 危険系: `/sandbox` / `/permissions` / `/clear` を間違えて使うと意図せず作業が消えうる。Claude が勝手に叩かないこと
- プラットフォーム依存: `/desktop` は Mac/Win 限定、`/upgrade` は有償プラン限定、`/sandbox` は対応 OS のみ

## コード例

### コンテキストを節約する典型ワークフロー

```text
# 1. 重い作業中、要約で context を解放
/compact focus on what we decided about the schema

# 2. 次のタスクへの切替時
/clear

# 3. context が増えてきたら可視化
/context
```

### 過去のセッションに戻る

```text
# 名前で再開
/resume auth-refactor

# あるいは picker
/resume
```

### バンドルスキルを試す

```text
# 直近変更をレビュー → 修正適用（並列 review agent 3 つ）
/simplify focus on memory efficiency

# コードベースを 5〜30 unit に分けて並列改修
/batch migrate src/ from Solid to React
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 次の教材: [03-keyboard-shortcuts.md](03-keyboard-shortcuts.md) — 対話モードのキーボードショートカット
- 関連 docs: [Skills](https://code.claude.com/docs/en/skills) / [Plugins](https://code.claude.com/docs/en/plugins) / [Interactive mode](https://code.claude.com/docs/en/interactive-mode)

---

_Auto-generated at 2026-05-08 via /learning-flow:material（公式docs駆動）_
