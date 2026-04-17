# Remote Control — ローカルセッションを別デバイスから操作する

出典: [remote-control](https://code.claude.com/docs/en/remote-control)

**Phase 1 Task 1 で overview を読んだ際に興味を持った機能。本格的な扱いは Phase 4。**

## 本質

ローカルで走っている Claude Code セッションを、スマホ / 別ブラウザから操作する。コード実行は**自分のマシンのまま**で、UI だけ遠隔。

```
[自宅 Mac で] claude remote-control
              ↓ QR コード表示
[電車内 iPhone] QR スキャン → 続きから会話
```

## できること

- デスクで始めた作業をソファ / 移動中のスマホで継続
- **複数デバイス同時接続**（Mac ターミナル / ブラウザ / スマホで会話同期）
- ローカル環境そのまま活用（MCP servers、filesystem、project config）
- ネットワーク切断 → 復旧で自動再接続
- **Push 通知**: Claude が「テスト終わった」「判断が必要」で自動プッシュ。プロンプトで `notify me when tests finish` のように明示指示も可

## 起動 3 パターン

| コマンド | 用途 |
|---|---|
| `claude remote-control` | **server mode**。プロセス常駐、複数セッション受け付け。spacebar で QR トグル |
| `claude --remote-control` (`--rc`) | 通常の対話セッション + Remote Control 有効 |
| `/remote-control` (セッション内) | 既存会話を途中から遠隔化 |

VS Code 拡張では `/remote-control` を prompt box で実行。

### server mode の主要フラグ

- `--name "My Project"` セッション名
- `--spawn same-dir` / `worktree` / `session` — 複数接続時の分離方法
- `--capacity 32` 同時セッション数上限
- `--sandbox` / `--no-sandbox` — OS レベル隔離

## 接続方法（別デバイスから）

1. QR コードスキャン（Claude モバイルアプリで開く）
2. URL をブラウザで開く（claude.ai/code）
3. claude.ai/code or Claude app のセッション一覧から選ぶ（緑ステータス + PC アイコン）

`/mobile` でアプリの DL QR が出る。

## 全セッションで自動有効にする

`/config` → `Enable Remote Control for all sessions` を `true`。各 claude プロセスが1セッションずつ登録される。

## 類似機能との違い

| 機能 | 実行場所 | トリガー | 用途 |
|---|---|---|---|
| **Remote Control** | 自分のマシン（CLI/VS Code） | 既存セッションをデバイス間で引き継ぎ | 進行中の作業を別デバイスから操縦 |
| [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web) | Anthropic クラウド | ブラウザで新規起動 | ローカルにない repo、並列タスク |
| [Dispatch](https://code.claude.com/docs/en/desktop) | Desktop app（自マシン） | スマホから1shot タスク投げる | 外出先から委譲、最小セットアップ |
| [Channels](https://code.claude.com/docs/en/channels) | 自分のマシン（CLI） | Telegram/Discord/独自 webhook | 外部イベントに反応 |
| [Slack](https://code.claude.com/docs/en/slack) | Anthropic クラウド | `@Claude` メンション | チームチャットから PR |
| [Scheduled tasks](https://code.claude.com/docs/en/scheduled-tasks) | CLI/Desktop/cloud | 時刻スケジュール | 定期自動化 |

## セキュリティ

- **Outbound HTTPS のみ**、マシンで inbound ポートを開かない
- Anthropic API 経由で streaming connection、全 TLS
- 複数の短命クレデンシャル（目的ごとにスコープ・期限別）

## 条件・制約

- **サブスク必須**: Pro / Max / Team / Enterprise のみ。API key 不可、claude.ai OAuth のみ
- Team / Enterprise は admin が Remote Control トグルを enable する必要あり
- Claude Code **v2.1.51 以降**
- Push 通知は v2.1.110 以降
- **ローカルプロセスが生きている間のみ**（terminal 閉じたら終わる）
- ネットワーク断 10 分以上でタイムアウト・プロセス終了
- ultraplan 起動すると Remote Control は切断される（同じ UI を奪い合う）
- ローカル専用コマンド: `/mcp`, `/plugin`, `/resume`（インタラクティブピッカー使うやつ）

## トラブル対処（頻出）

- **"requires a claude.ai subscription"** → `ANTHROPIC_API_KEY` をunset して `claude auth login`
- **"full-scope login token"** → `claude setup-token` の長寿命トークンは不可、`claude auth login` で full-scope に
- **"disabled by your organization's policy"** → Team/Enterprise admin が toggle を有効化
- **通知届かない** → iOS の Focus mode、Android のバッテリー最適化除外を確認

## 今後の学習フック

- Phase 4 で `/routines` や `/channels` と併せて「Claude Code を常駐化」する運用を学ぶ
- Phase 4 の Dispatch / Slack 統合と併せて「場所を問わず開発」のパターンを理解
