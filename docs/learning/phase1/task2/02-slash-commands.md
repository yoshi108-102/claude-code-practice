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

## 振り返りクイズ

回答は各問の `**回答**:` 行の下に記入してください。
全問記入後に `/learning-flow:grade` を実行すると、Claude が採点して進捗を更新します。

範囲: 02-slash-commands.md / reference (subagent-model-override, customizing-bundled-skills, plan-mode-vs-plan-subagent, claude-agents-cli-and-sources, worktree-usage-and-tradeoffs)

---

### Q1. 組み込みコマンドとバンドルスキルの本質的な違い

`/help` の出力には `[Skill]` マーク付きのコマンドと無いものが混在している。これら 2 種類は **「実装位置」と「実行の確実性」の観点でどう違う**か。説明したうえで、**`/clear` を `/init` で代用しようとすると何が問題になる**か（あるいはその逆も）の例を挙げよ。

実装位置というか、実装レイヤーとしては、AIを通さないのが/clearのような組み込みコマンドで、AIに指示してやらせるのが/initのようなバンドルスキル。
AIは不確実性があるのでstaticにコンテキストが常に消去されるとは限らないし、逆をやると自由度が不足する（clearとinitはそもそもやることが少し違うので代用について記載するのは少し難しいが...）

**参考**:
- [Commands](https://code.claude.com/docs/en/commands)
- [Skills - Bundled skills](https://code.claude.com/docs/en/skills#bundled-skills)

**関連ノート**: [02-slash-commands.md](02-slash-commands.md)

**回答**:

---

### Q2. `/clear` と `/compact` の使い分け

`/clear` と `/compact` は両方とも「コンテキストを軽くする」コマンドだが、性質が大きく異なる。**それぞれ何をするか**、**どんな場面で使うべきか**、そして **「タスク切替時に `/compact` を使うのは何が問題か」** を説明せよ。

/clearは組み込みコマンドとして、セッションにおけるコンテキストをまっさらにする一方、/compactは要約して軽くする。前者は明らかにハルシネーションがひどくなる（2回やってもバグが治らないなど）か、話を変える時。後者は同じタスクを続けたいが会話が深くなった時に使用する。そのため、タスク切り替え時に/compactを使うと無駄なコンテキストによりAIが混乱するのでやめた方がいい

**参考**:
- [Commands - /clear and /compact](https://code.claude.com/docs/en/commands)

**関連ノート**: [02-slash-commands.md](02-slash-commands.md) (セッション管理セクション)

**回答**:

---

### Q3. 引数表記のルール

公式 docs のコマンド表で、コマンドの引数は `<arg>` または `[arg]` の 2 種類で表記される。**それぞれの意味は何か**。また、ユーザーが実際に `/agents <command>` と打ったときに**何が起こるか**（コマンドとして正しく動くか、何か文字列として渡されるか）を説明せよ。
<arg>は必須引数で[arg]はオプション引数。

**参考**:
- [Commands - argument notation](https://code.claude.com/docs/en/commands)

**回答**:

---

### Q4. `/context` で何が分かるか / 何のために使うか

`/context` を実行すると **「カテゴリ別のトークン使用量」** が表示される。表示される代表的な 5 カテゴリ（System prompt / System tools / Memory files / Skills / Messages 等）のうち、**ユーザーが「軽くしたい」と思ったときに介入できる**のはどれで、どのコマンドで対処するか。逆に**介入できない**カテゴリは何で、それはなぜか。

**参考**:
- [Commands - /context](https://code.claude.com/docs/en/commands)
- [How Claude Code Works - context window](https://code.claude.com/docs/en/how-claude-code-works)

**関連ノート**: [02-slash-commands.md](02-slash-commands.md)

**回答**:

---

### Q5. subagent を別モデルで動かす方法と優先順位

親セッションを Opus 4.7 のまま、subagent だけ Sonnet で動かしたい。**3 つの設定レイヤー**があり、それぞれの優先順位がある。**3 つを優先順位順に挙げ**、**「親セッションが Opus でも subagent が必ず Opus で動いてしまう設定」** はどれか説明せよ。

**参考**:
- [Subagents](https://code.claude.com/docs/en/sub-agents)

**関連ノート**: [reference/subagent-model-override.md](reference/subagent-model-override.md)

**回答**:

---

### Q6. bundled skill のカスタマイズ — 「コーディング規約」を実装する 4 つの道

チームで `/review` 相当の独自レビュールールを実装したい場合、**4 つのカスタマイズ手段（A: CLAUDE.md / B: 別名 skill / C: 同名上書き / D: plugin）** がある。それぞれの**メリット・デメリット**を整理し、**チームで本格運用するときに最も推奨される**のはどれで、なぜか。**同名上書き (C) が推奨されない理由**も併せて説明せよ。

**参考**:
- [Skills - Where skills live](https://code.claude.com/docs/en/skills)
- [Skills - precedence](https://code.claude.com/docs/en/skills)

**関連ノート**: [reference/customizing-bundled-skills.md](reference/customizing-bundled-skills.md)

**回答**:

---

### Q7. `/permissions` の 3 階層とルール衝突

`/permissions` のルールは 3 つのスコープに分かれて保存される（**user global / project shared / project local**）。**それぞれの保存先ファイルパス**を答え、**同じツールに対して `allow` ルールと `deny` ルールが両方ある場合、どちらが勝つか**、また**その設計理由（なぜそうなっているのが望ましいか）**を説明せよ。

**参考**:
- [Permissions](https://code.claude.com/docs/en/permissions)
- [Settings](https://code.claude.com/docs/en/settings)

**関連ノート**: [02-slash-commands.md](02-slash-commands.md) (権限カテゴリ)

**回答**:

---

### Q8. `/undo` の限界 — Bash 副作用と git の役割分担

`/undo` は「Claude が直前のターンで行った編集を取り消す」コマンドだが、**取り消せるもの / 取り消せないもの** がはっきり分かれている。**取り消せない代表例を 2 つ**挙げ、**それぞれをロールバックしたい場合に何を使うべきか**（git / 手動 / 諦める など）を説明せよ。さらに、**「3 ターン前の編集を取り消したい」場合に `/undo` を 3 回連打してはいけない理由**も答えよ。

**参考**:
- [Commands - /undo / /rewind](https://code.claude.com/docs/en/commands)
- [How Claude Code Works - checkpoints](https://code.claude.com/docs/en/how-claude-code-works)

**関連ノート**: [02-slash-commands.md](02-slash-commands.md) (編集レビュー)

**回答**:

---
