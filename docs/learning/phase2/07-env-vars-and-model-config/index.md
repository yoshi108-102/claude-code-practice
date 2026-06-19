# 環境変数とモデル設定

> 出典:
> - [Environment variables](https://code.claude.com/docs/en/env-vars)（閲覧日 2026-06-12）
> - [Model configuration](https://code.claude.com/docs/en/model-config)（閲覧日 2026-06-12）
> このノートは公式ドキュメント「env-vars」「model-config」を起点に、Claudeが自動生成した教材です。

## 概要

settings.json の中でも特によく使う 2 系統 — **環境変数（env）** と **モデル設定** をまとめて扱う。どちらも「セッションの挙動を外から差し込む」手段。優先順位の感覚と、それぞれのキルスイッチ・ピン留め方法が実務で効く。

## 公式 docs に沿った解説

### 環境変数の設定方法

3 つの経路:

1. **シェルで直接**（macOS/Linux: `export VAR=value`、PowerShell: `$env:VAR = "value"`）
2. **settings.json の `env` キー**（スコープ階層あり）
3. 一部は **`.env`** ファイルから読み込まれる

```json
{
  "env": {
    "API_TIMEOUT_MS": "1200000",
    "BASH_DEFAULT_TIMEOUT_MS": "300000"
  }
}
```

設定ファイルの `env` スコープは settings と同じ（user/project/local/managed）。

### 優先順位

```
環境変数（シェル）> CLI フラグ/セッションコマンド > 設定ファイル
```

例: `ANTHROPIC_MODEL` は `model` 設定を上書きするが、`--model` や `/model` が `ANTHROPIC_MODEL` を上書きする。

### よく使う環境変数（目的別）

#### 認証

| 変数 | 用途 |
|---|---|
| `ANTHROPIC_API_KEY` | 直接認証の API キー（サブスクリプションより優先） |
| `ANTHROPIC_AUTH_TOKEN` | `Authorization` ヘッダーのカスタム値 |

#### API/ネットワーク

| 変数 | 用途 |
|---|---|
| `ANTHROPIC_BASE_URL` | プロキシ/ゲートウェイへルーティング（**モデルは変わらない**） |
| `API_TIMEOUT_MS` | タイムアウト（デフォルト 600000 = 10 分） |
| `ANTHROPIC_CUSTOM_HEADERS` | カスタムヘッダー（`Name: Value` 形式） |

#### モデル選択

| 変数 | 用途 |
|---|---|
| `ANTHROPIC_MODEL` | デフォルトモデル名（セッション限定） |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | `opus` エイリアスの解決先をピン留め |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | `sonnet` エイリアスの解決先をピン留め |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `haiku` エイリアスの解決先をピン留め |
| `ANTHROPIC_DEFAULT_FABLE_MODEL` | `fable` エイリアスの解決先をピン留め |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 全 subagent のモデルを強制（`inherit` で通常解決） |

#### 推論・努力度

| 変数 | 用途 |
|---|---|
| `CLAUDE_CODE_EFFORT_LEVEL` | 努力度（`low`/`medium`/`high`/`xhigh`/`max`/`auto`）。`/effort` を上書き |
| `MAX_THINKING_TOKENS` | 固定思考予算（Opus 4.6/Sonnet 4.6 の固定バジェットモード用） |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | 適応的推論を無効化（Opus 4.6/Sonnet 4.6 のみ） |

#### 機能の ON/OFF（キルスイッチ）

| 変数 | 用途 |
|---|---|
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | auto memory 無効化（Phase 2 Task 1） |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | CLAUDE.md ロード無効化 |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | バックグラウンドタスク全体を無効化 |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | `/rewind` 無効化 |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | 1M コンテキストウィンドウ無効化 |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | fast mode 無効化 |
| `CLAUDE_CODE_ENABLE_AUTO_MODE` | Bedrock/Vertex/Foundry で auto mode 有効化（v2.1.158+） |
| `ENABLE_TOOL_SEARCH` | MCP tool search 有効化（Phase 1 Task 3 で既出） |

#### コンテキスト管理

| 変数 | 用途 |
|---|---|
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 自動 compact の発火%（1〜100、デフォルト 95） |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 自動圧縮計算用コンテキスト容量（トークン） |

#### Bash タイムアウト

| 変数 | 用途 |
|---|---|
| `BASH_DEFAULT_TIMEOUT_MS` | 長時間実行コマンドのデフォルトタイムアウト（デフォルト 120000） |
| `BASH_MAX_TIMEOUT_MS` | モデルが設定可能な最大タイムアウト（デフォルト 600000） |
| `BASH_MAX_OUTPUT_LENGTH` | Bash 出力の最大文字数（超過時はファイルに保存） |

#### テレメトリ

| 変数 | 用途 |
|---|---|
| `DISABLE_TELEMETRY` | テレメトリ全体を無効化 |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | OpenTelemetry データ収集を有効化 |
| `DO_NOT_TRACK` | トラッキング無効化 |

### モデルの選び方（優先順位順）

1. **セッション中**: `/model <alias|name>`（引数なしでピッカー）
2. **起動時**: `claude --model <alias|name>`
3. **環境変数**: `ANTHROPIC_MODEL=<alias|name>`
4. **settings**: `model` フィールド（恒久）

v2.1.153 以降、`/model` でモデルを選ぶとユーザー設定の `model` フィールドに保存されデフォルトになる（セッションのみに留めるには picker 内で `s` を押す）。

`--model` フラグと `ANTHROPIC_MODEL` は起動したセッションにのみ有効。`--resume` 時は保存時のモデルを維持する。

### モデルエイリアス

| エイリアス | 意味 |
|---|---|
| `default` | オーバーライドを解除してアカウント種別の推奨モデルに戻す |
| `best` | 最も高性能（Fable 5 が利用可能ならそちら、なければ最新 Opus） |
| `fable` | Claude Fable 5（最長・最高性能。v2.1.170+） |
| `sonnet` | 日常コーディング向け最新 Sonnet |
| `opus` | 複雑な推論向け最新 Opus |
| `haiku` | 高速・軽量 |
| `sonnet[1m]` / `opus[1m]` | 1M トークンコンテキスト版 |
| `opusplan` | **plan モード中は opus、実行に移ると sonnet** に自動切替（コスト最適） |

Anthropic API では `opus` = Opus 4.8、`sonnet` = Sonnet 4.6 に解決。Bedrock/Vertex/Foundry では少し古いバージョンに解決されるため、ピン留めには full model name か `ANTHROPIC_DEFAULT_*_MODEL` を使うのが安全。

#### デフォルトモデル（アカウント種別）

| アカウント種別 | デフォルト |
|---|---|
| Max / Team Premium / Enterprise pay-as-you-go / Anthropic API | Opus 4.8 |
| Claude Platform on AWS | Opus 4.7 |
| Pro / Team Standard / Enterprise subscription seats | Sonnet 4.6 |
| Bedrock / Vertex / Foundry | Sonnet 4.5 |

### 努力度（effort level）

`/effort`、`--effort`、`CLAUDE_CODE_EFFORT_LEVEL`、settings の `effortLevel` で設定。

| レベル | いつ使う |
|---|---|
| `low` | 短く範囲が狭く、速度優先のタスク |
| `medium` | コスト重視で多少の性能ダウンを許容 |
| `high` | バランス。Fable 5/Opus 4.8/Opus 4.6/Sonnet 4.6 のデフォルト |
| `xhigh` | 深い推論。Opus 4.7 のデフォルト |
| `max` | 最大推論（セッション限定。settings には設定不可） |
| `ultracode` | xhigh + dynamic workflow（セッション限定） |

モデルによって対応レベルが異なる（例: Fable 5 は `low`〜`max`、Opus 4.6 は `xhigh` 非対応）。

`ultrathink` をプロンプトに書くとそのターンだけ深く考える（API には `xhigh` 相当が送られる。`think harder` 等は通常テキストとして扱われ認識されない）。

### モデル設定の組織制御（3 点セット）

完全に制御するには 3 つを組み合わせる:

| 設定 | 役割 |
|---|---|
| `availableModels` | ユーザーが選べるモデルを制限 |
| `model` | 起動時の初期モデル選択 |
| `ANTHROPIC_DEFAULT_*_MODEL` | エイリアスの解決先を固定（Default 選択時も制御） |

`model` を設定してもユーザーが `/model` で `default` を選ぶと最新版に戻る → 完全固定には `ANTHROPIC_DEFAULT_*_MODEL` との組み合わせが必要。

### フォールバックモデル

過負荷/不可用時のフォールバック設定:

```json
{
  "fallbackModel": ["claude-sonnet-4-6", "claude-haiku-4-5"]
}
```

起動時フラグ: `claude --fallback-model sonnet,haiku`。チェーンは最大 3 モデルまで。

### 1M コンテキストウィンドウ

Fable 5、Opus 4.6+、Sonnet 4.6 が対応。`[1m]` サフィックスで明示指定:

```bash
/model opus[1m]
/model claude-opus-4-8[1m]
```

Max/Team/Enterprise では Opus の 1M コンテキストは subscription 込み。Sonnet の 1M は全プランで usage credits 必要。無効化: `CLAUDE_CODE_DISABLE_1M_CONTEXT=1`。

### Bedrock/Vertex/Foundry でのモデルピン留め

エイリアスは built-in のデフォルトに解決されるが、プロバイダーによって古い版に解決される場合がある。初期セットアップ時にピン留めを推奨:

```bash
export ANTHROPIC_DEFAULT_OPUS_MODEL='us.anthropic.claude-opus-4-8'      # Bedrock
export ANTHROPIC_DEFAULT_SONNET_MODEL='claude-sonnet-4-6'                # Vertex/Foundry
```

## 重要ポイント

- env は 3 経路（shell / settings.env / `.env`）。**env は settings を上書き、CLI/`/model` が env を上書き**
- Phase 1〜2 で出てきた機能の多くに **env のキルスイッチ**がある（auto memory / background / checkpoint / tool search 等）
- モデル選択の優先順位: **`/model`・`--model`・`ANTHROPIC_MODEL`（セッション限定）> settings.model（恒久）**
- エイリアス `opusplan` = **plan で opus、実行で sonnet** に自動切替（コスト最適の定番）
- effort は推論の深さ。`ultrathink` でそのターンだけ深く考える
- `availableModels`（managed）で組織のモデル allowlist を強制、`CLAUDE_CODE_SUBAGENT_MODEL` で subagent モデル全体を強制
- `ANTHROPIC_BASE_URL` は**送り先を変えるだけでモデルは変えない**（混同注意）

## コード例 / 図

### モデル指定の優先順位

```text
/model sonnet（セッション中・即時・ユーザー設定に保存）
  ↑ 上書き
claude --model opus（起動時・セッション限定）
ANTHROPIC_MODEL=opus（env・セッション限定）
  ↑ 上書き
settings.json "model": "opus"（恒久デフォルト）
```

### opusplan で計画は賢く・実行は安く

```json
{ "model": "opusplan" }
```

```text
plan モード   → opus（設計・推論）
実行モード    → sonnet（コード生成）
```

### 組織でモデルを固定する（3 点セット）

```json
{
  "model": "claude-sonnet-4-5",
  "availableModels": ["claude-sonnet-4-5", "haiku"],
  "env": {
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "claude-sonnet-4-5"
  }
}
```

→ `env` がないと、ユーザーが `/model` で `default` を選んだときに最新版に逃げられる。

## 関連・深掘り（reference）

- （`lesson` 中に Q&A が発生したら、同ディレクトリ `reference/` 配下に md+html で追加され、ここにリンクされます）

---

_Auto-generated at 2026-06-12 via /learning-flow:material（公式docs駆動）_
