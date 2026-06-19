# Permission モード — 6 つの動作モード

> 出典: [Choose a permission mode](https://code.claude.com/docs/en/permission-modes)（閲覧日 2026-06-12）
> このノートは公式ドキュメントの「Choose a permission mode」を起点に、Claudeが自動生成した教材です。

## 概要

permission ルール（トピック 05）が「個別の許可/禁止」なら、permission モードは「**そのセッション全体で、どれくらい確認を挟むか**のベースライン」。ルールはモードの上に重ねる。モードを選ぶことで、細かいレビューを都度行うか、長時間タスクを邪魔なく回すかを制御できる。

## 公式 docs に沿った解説

### 6 つのモード一覧

| モード | 確認なしで動くもの | 向き |
|---|---|---|
| `default` | 読み取りのみ | 開始時・慎重な作業 |
| `acceptEdits` | 読み取り + ファイル編集 + 一般的な FS コマンド（mkdir/touch/mv/cp/rm/sed） | レビュー前提で実装を回す |
| `plan` | 読み取りのみ（編集しない・計画を出す） | 変更前のコードベース調査 |
| `auto` | ほぼ全部（**背後で classifier が安全チェック**） | 長時間タスク・プロンプト疲れ軽減 |
| `dontAsk` | 事前承認済みツールのみ（それ以外は自動 deny） | CI・ロックダウン環境 |
| `bypassPermissions` | 全部（`rm -rf /` や `rm -rf ~` は例外的に確認） | 隔離コンテナ/VM のみ |

**`bypassPermissions` 以外の全モードで、protected paths への書き込みは自動承認されない**（後述）。

### モードの切り替え方法

- **`Shift+Tab`** で循環: `default → acceptEdits → plan`（3 つのみ）
- **起動時**: `claude --permission-mode plan`
- **デフォルト**: settings の `permissions.defaultMode`
- `auto` / `dontAsk` / `bypassPermissions` は通常の循環に出ない（opt-in や専用フラグが必要）

VS Code では画面下部のモードインジケーターから切り替え可能。Desktop でも専用セレクターがある。

### acceptEdits モード

`acceptEdits` モードでは作業ディレクトリ内のファイル作成・編集が確認なしで実行される。追加で自動承認される Bash コマンド: `mkdir`/`touch`/`rm`/`rmdir`/`mv`/`cp`/`sed`（作業ディレクトリ内のパスのみ）。

`git diff` や エディタで事後レビューする開発スタイルに向く。`Shift+Tab` 1 回で入れる。

### plan モード

plan モードでは Claude は**読む・調べる・計画を書く**が**ソースを編集しない**。permission プロンプトは `default` と同じ。

計画提示後の選択肢:
- Auto で開始
- acceptEdits で承認（各編集を手動レビュー）
- 計画を続ける / フィードバックを与える
- Ultraplan でブラウザベースのレビューに移行

承認するとモードが切り替わって編集開始。`Shift+Tab` か `/plan` で入る。`Ctrl+G` で提案プランをエディタで直接編集してから承認できる。

`showClearContextOnPlanAccept: true` を settings に入れると、承認時にプランニングコンテキストのクリアオプションが表示される。

### auto モード（要件あり）

auto モードは別の **classifier モデル** が各アクションを実行前にレビューし、「リクエストを逸脱する/未知のインフラを触る/敵対的コンテンツ由来」のものをブロック。

#### 利用要件

- Claude Code v2.1.83+
- モデル: Anthropic API では Opus 4.6 以降 or Sonnet 4.6。Bedrock/Vertex AI/Foundry では Opus 4.7 以降のみ
- Anthropic API ではデフォルト有効。Bedrock/Vertex/Foundry では `CLAUDE_CODE_ENABLE_AUTO_MODE=1` が必要（v2.1.158+）
- Team/Enterprise では管理者の有効化が必要

#### classifier がデフォルトでブロックするもの

- `curl | bash` などのダウンロード+実行
- 機密データの外部エンドポイントへの送信
- 本番デプロイ/マイグレーション
- クラウドストレージの大量削除
- IAM 付与、shared インフラへの変更
- セッション開始前から存在したファイルの不可逆削除
- force push、main への直接 push

#### デフォルトで許可するもの

- 作業ディレクトリ内のローカルファイル操作
- lock ファイル/マニフェスト記載の依存インストール
- `.env` を読んで対応する API に送信
- read-only HTTP リクエスト
- 開始ブランチ、または Claude が作成したブランチへの push

#### 会話で述べた境界

「push するな」のように会話で述べた境界を classifier が尊重する。ただし **context compaction で境界メッセージが消えると失われる** → 確実に守りたいなら deny ルールを使う。

#### フォールバック

3 連続 or 累計 20 回ブロックでプロンプトに戻る（`-p` 非対話では abort）。`/permissions` の「Recently denied」タブから `r` でリトライできる。

#### auto モード中の広すぎる allow ルールの自動無効化

`Bash(*)` や `Bash(python*)` 等の広すぎる allow ルールは auto モード中一時的に無効化される。`Bash(npm test)` のような狭いルールは有効のまま。auto を抜けると復元。

### dontAsk モード

プロンプトが必要な全呼び出しを**自動 deny**。`permissions.allow` 記載と read-only Bash だけ実行。`ask` ルールも deny 扱いになる。**CI で「事前定義したことだけやらせる」**用途。`claude --permission-mode dontAsk` で起動。

### bypassPermissions モード

全プロンプト・安全チェックをスキップ。`rm -rf /` や `rm -rf ~` だけはサーキットブレーカーで確認。**隔離コンテナ/VM 専用**。

- root/sudo では起動拒否（recognized sandbox 内は自動スキップ）
- **prompt injection への保護はゼロ** → 「プロンプトなし + 安全」なら auto を使え
- `--permission-mode bypassPermissions` または `--dangerously-skip-permissions` で起動
- 一度 bypass なしで起動したセッションから bypass には入れない（再起動が必要）
- `disableBypassPermissionsMode: "disable"` で組織封印（managed settings）

### protected paths（全モード共通の最後の砦）

`bypassPermissions` 以外の全モードで、以下への書き込みは**自動承認されない**:

| モード | 保護パスへの書き込み |
|---|---|
| `default` / `acceptEdits` / `plan` | プロンプト |
| `auto` | classifier に送る |
| `dontAsk` | 自動 deny |
| `bypassPermissions` | 許可 |

保護ディレクトリ例: `.git`/`.vscode`/`.idea`/`.husky`/`.cargo`/`.devcontainer`/`.yarn`/`.mvn`/`.claude`（ただし `.claude/worktrees` は除く）

保護ファイル例: `.gitconfig`/`.gitmodules`/`.bashrc`/`.zshrc`/`.mcp.json`/`.claude.json` など多数

`allow` ルールを settings に書いても protected paths は事前承認されない（安全チェックが allow ルールより先に走るため）。

## 重要ポイント

- モードは「セッション全体の確認頻度のベースライン」。その上に permission ルールを重ねる
- `Shift+Tab` 循環は `default → acceptEdits → plan` の 3 つのみ。auto/dontAsk/bypass は別途
- **auto = 別 classifier の二重チェック**（野放しではない）。Anthropic API 限定・モデル要件あり
- **会話で述べた境界は compaction で消えうる** → 確実にしたいなら deny ルール
- `bypassPermissions` は prompt injection 無防備 → 隔離環境専用。「プロンプトなし + 安全」なら auto
- **protected paths**（`.git`/`.claude` 等）は bypass 以外で自動承認されない最後の砦
- managed の `disableBypassPermissionsMode` / `disableAutoMode` で危険モードを組織封印

## コード例 / 図

### モードと「確認なしで動く範囲」

```text
default            読むだけ
acceptEdits        読む + 編集 + mkdir/mv/cp/rm（作業ディレクトリ内）
plan               読むだけ（編集禁止・計画を出す）
auto               ほぼ全部（classifier が逸脱をブロック）
dontAsk            allow 済みのみ（他は自動 deny）
bypassPermissions  全部（rm -rf / ~ だけ確認）
```

### plan → 実行 の安全な動線

```text
Shift+Tab で plan モード入場
  → Claude が調べて計画を出す
  → 承認 → acceptEdits/auto でモード切替 → 編集開始
```

### defaultMode を settings に永続化

```json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

`defaultMode: "auto"` は `~/.claude/settings.json`（ユーザー設定）にのみ有効。project/local settings に書くと無視される（v2.1.142+）。

### Bedrock/Vertex で auto モードを有効化

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_AUTO_MODE": "1"
  }
}
```

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## 次のトピック

- [07-env-vars-and-model-config/](../07-env-vars-and-model-config/index.md) — 環境変数とモデル設定

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
