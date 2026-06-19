# Agent Teams — 複数セッションの並列協調

> 出典: [Orchestrate teams of Claude Code sessions](https://code.claude.com/docs/en/agent-teams)（閲覧日 2026-06-12）
> このノートは公式ドキュメントの「agent-teams」を起点に、Claudeが自動生成した教材です。

## 概要

Agent teams は複数の Claude Code インスタンスを協調して動かす仕組み。1 つのセッションがチームリードとして機能し、作業を調整・タスク割り当て・結果を統合する。チームメイトは独立して作業しながら、互いに直接メッセージを送り合える。

> **注意**: Agent teams は実験的機能でデフォルト無効。`settings.json` または環境変数に `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` を設定して有効化する。既知の制限事項（セッション再開・タスク協調・シャットダウン動作）がある。Claude Code v2.1.32 以降が必要。

サブエージェントとの違い: サブエージェントは 1 つのセッション内で動作し結果を親に返すだけ。Agent teams はチームメイト同士が直接通信できる独立したセッション群。

## 公式 docs に沿った解説

### いつ Agent teams を使うか

最も効果的なユースケース:
- **調査とレビュー**: 複数チームメイトが問題の異なる側面を同時に調査
- **新機能・モジュール**: 各チームメイトが別々のピースを担当
- **競合仮説でのデバッグ**: 異なる理論を並列テストして早期収束
- **クロスレイヤー対応**: フロントエンド・バックエンド・テストを別々のチームメイトが担当

Agent teams は協調オーバーヘッドが発生し、トークン消費も大幅に増える。逐次タスク・同一ファイルの編集・依存度の高い作業では単一セッションや Subagents の方が効率的。

### サブエージェントとの比較

| | Subagents | Agent teams |
|---|---|---|
| **コンテキスト** | 独自コンテキスト、結果を呼び出し元に返す | 独自コンテキスト、完全独立 |
| **通信** | 親エージェントにのみ結果を返す | チームメイト同士が直接メッセージ |
| **協調** | 親が全ての仕事を管理 | 共有タスクリストで自律調整 |
| **トークンコスト** | 低（サマリーとして返す） | 高（各メンバーが独立した Claude インスタンス） |

### 有効化と開始

`settings.json` で有効化:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

自然言語でチームを作成:

```
I'm designing a CLI tool. Create an agent team to explore this from different angles:
one teammate on UX, one on technical architecture, one playing devil's advocate.
```

Claude がチームを作成し、チームメイトをスポーン、協調して作業。

### アーキテクチャ

| コンポーネント | 役割 |
|---|---|
| **Team lead** | チームを作成し、チームメイトをスポーン・調整するメインセッション |
| **Teammates** | 割り当てられたタスクをこなす独立した Claude Code インスタンス |
| **Task list** | チームメイトがクレームして完了する共有タスクリスト |
| **Mailbox** | エージェント間通信メッセージングシステム |

チームとタスクはローカルに保存:
- チーム設定: `~/.claude/teams/{team-name}/config.json`
- タスクリスト: `~/.claude/tasks/{team-name}/`

これらはチームが active な間だけ存在し、クリーンアップ時またはセッション終了時に削除される。手動編集は不可（次の状態更新で上書きされる）。

### 表示モード

- **In-process**: チームメイトは全員メインターミナル内で動く。`Shift+Down` でチームメイトを切り替え。全端末で動作
- **Split panes**: 各チームメイトが独自ペインを持つ。tmux または iTerm2 が必要

デフォルトは `"auto"`（tmux セッション内または iTerm2 なら split panes、それ以外は in-process）。

`~/.claude/settings.json` で設定:
```json
{ "teammateMode": "in-process" }
```

または 1 セッションだけ:
```bash
claude --teammate-mode in-process
```

### チームの制御

**チームメイトのモデル指定**:
```
Create a team with 4 teammates. Use Sonnet for each teammate.
```

**プラン承認を要求**:
```
Spawn an architect teammate to refactor auth module. Require plan approval before they make changes.
```

リードがプランを承認するまでチームメイトは read-only モードで待機。リードが拒否するとチームメイトは修正して再提出。

**チームメイトに直接話しかける**:
- In-process: `Shift+Down` でチームメイトに移動してタイプ
- Split pane: チームメイトのペインをクリック

**タスクの管理**:
- **リードが割り当て**: どのチームメイトにどのタスクを渡すか明示
- **自律クレーム**: タスク完了後、チームメイトが次の未割り当て・ブロックなしタスクを自動でクレーム

タスクには状態（pending / in progress / completed）と依存関係がある。ファイルロックで同一タスクへの競合クレームを防ぐ。

**チームメイトのシャットダウン**:
```
Ask the researcher teammate to shut down
```

**チームのクリーンアップ**:
```
Clean up the team
```

チームメイトがまだ running な場合はリードがクリーンアップに失敗する（先にシャットダウン必要）。クリーンアップは必ずリードから実行すること。

### Hooks でクオリティゲートを実施

| Hook | 用途 |
|---|---|
| `TeammateIdle` | チームメイトがアイドルになる時。exit 2 でフィードバックを送り作業継続 |
| `TaskCreated` | タスク作成時。exit 2 で作成を防いでフィードバック |
| `TaskCompleted` | タスク完了時。exit 2 で完了を防いでフィードバック |

### サブエージェント定義をチームメイトに使う

サブエージェントタイプをチームメイトとして参照できる:

```
Spawn a teammate using the security-reviewer agent type to audit the auth module.
```

`tools` allowlist と `model` はその定義から継承される。定義の本文はチームメイトのシステムプロンプトへの追加指示として付与される。

> **注**: サブエージェント定義の `skills` と `mcpServers` フロントマターフィールドは、チームメイトとして動作する場合には適用されない。

### コンテキストと通信

各チームメイトは独自のコンテキストウィンドウを持つ。スポーン時に通常セッションと同じプロジェクトコンテキスト（CLAUDE.md・MCP サーバー・Skills）を読み込む。リードの会話履歴は引き継がない。

**情報共有の仕組み**:
- **自動メッセージ配信**: チームメイトのメッセージはリードに自動的に届く
- **アイドル通知**: チームメイトが作業完了してアイドルになるとリードに通知
- **共有タスクリスト**: 全エージェントがタスク状態を確認してクレームできる
- **チームメイトメッセージング**: 名前で特定チームメイトにメッセージ

### ベストプラクティス

**十分なコンテキストを渡す**: チームメイトはリードの会話履歴を継承しない。スポーンプロンプトにタスク固有の詳細を含める。

**適切なチームサイズ**: 3〜5 人が多くのワークフローで最適。トークンコストはチームメイト数に比例する。チームメイト 1 人あたり 5〜6 タスクで生産性が高い。

**タスクサイズ**: 小さすぎる（協調オーバーヘッドが大きい）・大きすぎる（チェックインなしに長く作業しすぎる）を避ける。明確な成果物を持つ自己完結した単位が理想。

**ファイル競合を避ける**: 同じファイルを 2 人が編集すると上書きが起きる。各チームメイトが別々のファイルセットを担当するよう分割。

**監視とステアリング**: チームメイトの進捗をチェックし、うまくいっていないアプローチをリダイレクト。

## 重要ポイント

- Agent teams は実験的機能。`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` で有効化が必要
- サブエージェントと違い、チームメイト同士が直接メッセージを送れる
- トークン消費が大幅に増える（各メンバーが独立した Claude インスタンス）。研究・レビュー・新機能開発のように並列探索が真に価値を持つ場合に使う
- クリーンアップは必ずリードから実行する。チームメイトからのクリーンアップは状態不整合を引き起こす可能性がある
- 既知の制限: `/resume` と `/rewind` は in-process チームメイトを復元しない

## コード例 / 図

### 並列コードレビューの例

```
Create an agent team to review PR #142. Spawn three reviewers:
- One focused on security implications
- One checking performance impact
- One validating test coverage
Have them each review and report findings.
```

各レビュアーが同じ PR に対し異なる視点でフィルタをかけ、リードが最後に統合。

### 競合仮説によるデバッグ

```
Users report the app exits after one message instead of staying connected.
Spawn 5 agent teammates to investigate different hypotheses. Have them talk to
each other to try to disprove each other's theories, like a scientific debate.
```

この「議論」構造が重要: 複数が独立して調査しつつ互いの理論を否定しようとすることで、1 つが生き残った場合の信頼度が高くなる。

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

## 次のトピック

- [07-tools-reference/](../07-tools-reference/index.md) — 組み込みツールのリファレンス

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
