# 03. Permission モード — 6つの動作モード

> 出典: [Choose a permission mode](https://code.claude.com/docs/en/permission-modes)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「permission-modes」の構造を起点に Claude が自動生成した教材です。

## 概要

permission ルール（教材02）が「個別の許可/禁止」なら、permission モードは「**そのセッション全体で、どれくらい確認を挟むか**」のベースライン。ルールはモードの上に重ねる。Phase 1 Task 1 の復習キュー Q6（Auto モード）の本格版がここ。

## 公式docsに沿った解説

### 6つのモード

| モード | 確認なしで動くもの | 向き |
|---|---|---|
| `default` | 読み取りのみ | 開始時・慎重な作業 |
| `acceptEdits` | 読み取り + ファイル編集 + 一般的な FS コマンド（mkdir/touch/mv/cp/rm/sed） | レビュー前提で実装を回す |
| `plan` | 読み取りのみ（編集しない、計画を出す） | 変更前のコードベース調査 |
| `auto` | ほぼ全部（**別 classifier が背後で安全チェック**） | 長時間タスク・プロンプト疲れ軽減 |
| `dontAsk` | 事前承認済みツールのみ（それ以外は自動 deny） | CI・ロックダウン環境 |
| `bypassPermissions` | 全部（チェックなし） | 隔離コンテナ/VM のみ |

**`bypassPermissions` 以外の全モードで、protected paths への書き込みは自動承認されない**（後述）。

### モードの切り替え

- **`Shift+Tab`** で循環:`default → acceptEdits → plan`（Phase 1 Task 2 で既習）
- **起動時**: `claude --permission-mode plan`
- **デフォルト**: settings の `permissions.defaultMode`
- `auto`/`bypassPermissions`/`dontAsk` は通常の循環に出ない（opt-in や専用フラグが必要）

### plan モード（教材で重要）

- Claude は**読む・調べる・計画を書く**が**ソースを編集しない**。permission プロンプトは default と同じ
- `Shift+Tab` か `/plan` で入る。`--permission-mode plan` でも
- 計画提示後の選択肢:Auto で開始 / acceptEdits で承認 / 各編集を手動レビュー / 計画を続ける / Ultraplan で精緻化
- **承認するとモードが切り替わって編集開始**

> Phase 1 Task 2 の reference「plan mode と Plan subagent は別物」と接続。plan **mode** は session 全体の permission、Plan **subagent** は調査ヘルパー。

### auto モード（復習キュー Phase1-Q6 の本格版）

- **別の classifier モデル**が各アクションを実行前にレビューし、「リクエストを逸脱する/未知のインフラを触る/敵対的コンテンツ由来」のものをブロック
- **要件**: v2.1.83+、Sonnet 4.6 / Opus 4.6 / Opus 4.7、**Anthropic API のみ**（Bedrock/Vertex/Foundry 不可）、Team/Enterprise は管理者の有効化が必要
- **デフォルトでブロック**:`curl|bash`、機密データの外部送信、本番デプロイ/マイグレーション、大量削除、IAM 付与、force push、main への直 push
- **デフォルトで許可**:作業ディレクトリ内のローカル操作、lock/manifest 記載の依存インストール、read-only HTTP、開始ブランチへの push
- **会話で述べた境界を尊重**:「push するな」と言えば classifier がブロック。ただし**compaction で境界メッセージが消えると失われる** → 確実にするなら deny ルール
- **フォールバック**:3連続 or 累計20回ブロックでプロンプトに戻る。`-p`（非対話）では abort
- auto 中は **`Bash(*)` 等の広すぎる allow ルールは一時無効化**される（`Bash(npm test)` のような狭いルールは有効）

> Phase1-Q6 で正答した「classifier による二重チェック / Plan は人間承認・Auto は別AI承認」がそのままここ。

### dontAsk モード

プロンプトすべき呼び出しを**全部自動 deny**。`permissions.allow` 記載と read-only Bash だけ実行。`ask` ルールも deny になる。**CI で「事前定義したことだけやらせる」**用途。

### bypassPermissions モード

全プロンプト・安全チェックをスキップ。**`rm -rf /` や `rm -rf ~` だけはサーキットブレーカーで確認**。隔離コンテナ/VM 専用。root/sudo では起動拒否。**prompt injection への保護はゼロ** → プロンプトなしで安全チェックが欲しいなら auto を使え、というのが docs の指針。

### protected paths（全モード共通の最後の砦）

`bypassPermissions` 以外の全モードで、以下への書き込みは**自動承認されない**:

- ディレクトリ:`.git`, `.vscode`, `.idea`, `.husky`, `.claude`（ただし `.claude/commands` `agents` `skills` `worktrees` は除く）
- ファイル:`.gitconfig`, `.gitmodules`, `.bashrc`/`.zshrc` 等, `.mcp.json`, `.claude.json`

default/acceptEdits/plan ではプロンプト、auto では classifier 行き、dontAsk では deny、bypass では許可。

## 重要ポイント

- モードは「セッション全体の確認頻度のベースライン」、その上に permission ルールを重ねる
- `Shift+Tab` 循環は `default → acceptEdits → plan` の3つだけ。auto/dontAsk/bypass は別途
- **auto = 別 classifier の二重チェック**（野放しではない、Phase1-Q6 の正解）。Anthropic API 限定・モデル要件あり
- **会話で述べた境界は compaction で消えうる** → 確実にしたいなら deny ルール（境界 ⇄ ルールの使い分け）
- `bypassPermissions` は prompt injection 無防備 → 隔離環境専用。「プロンプトなし + 安全」が欲しいなら auto
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

### plan → 実行 の安全な動線（Phase 1 Task 2 と同じ）

```text
Shift+Tab で plan → 計画を見る → 承認時に acceptEdits/auto を選ぶ → 編集開始
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 前の教材: [02-permissions.md](02-permissions.md)
- 次の教材: [04-env-vars-and-model-config.md](04-env-vars-and-model-config.md) — 環境変数とモデル設定
- 関連 docs: [Permissions](https://code.claude.com/docs/en/permissions) / [Configure auto mode](https://code.claude.com/docs/en/auto-mode-config) / [Sandboxing](https://code.claude.com/docs/en/sandboxing)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_
