<!-- DEPRECATED: 新版は ../07-env-vars-and-model-config/index.md（HTML 版は index.html）-->
# 04. 環境変数とモデル設定

> 出典:
> - [Environment variables](https://code.claude.com/docs/en/env-vars)
> - [Model configuration](https://code.claude.com/docs/en/model-config)
> （閲覧日 2026-05-22）
> このノートは公式ドキュメント「env-vars」「model-config」を起点に Claude が自動生成した教材です。

## 概要

settings.json の中でも特に使う2系統 — **環境変数（env）** と **モデル設定** をまとめて扱う。どちらも「セッションの挙動を外から差し込む」手段で、優先順位の感覚が実務で効く。

## 公式docsに沿った解説

### 環境変数の設定方法と優先順位

3つの設定経路:
1. シェルで `export VAR=value`
2. settings.json の `env` キー（全セッションに適用、スコープ階層あり）
3. （一部）`.env`

**優先順位**: 環境変数は settings ファイルを上書き。ただし CLI フラグや `/model` 等のセッション内コマンドが一部機能で env を上書きする。

### よく使う環境変数（目的別）

| 目的 | 変数 | 用途 |
|---|---|---|
| 認証 | `ANTHROPIC_API_KEY` | 直接認証の API キー |
| プロバイダ | `CLAUDE_CODE_USE_BEDROCK` / `..._VERTEX` | Bedrock/Vertex 経由（Phase 4） |
| エンドポイント | `ANTHROPIC_BASE_URL` | プロキシ/ゲートウェイへルーティング（**モデルは変わらない**） |
| モデル | `ANTHROPIC_MODEL` | デフォルトモデル名 |
| 推論 | `CLAUDE_CODE_EFFORT_LEVEL` | effort（low〜max/auto） |
| 記憶 | `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | auto memory 無効化（Task 1） |
| 記憶 | `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | CLAUDE.md ロード無効化 |
| ツール | `ENABLE_TOOL_SEARCH` | MCP tool search（Phase 1 Task 3 で既出） |
| 背景処理 | `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | background bash 無効化（Phase 1 Task 2） |
| context | `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 自動 compact の発火%（Phase 1 Task 3） |
| 安全 | `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | /rewind 無効化（Phase 1 Task 3） |
| テレメトリ | `DISABLE_TELEMETRY` / `CLAUDE_CODE_ENABLE_TELEMETRY` | 計測の OFF/ON |

> ここまでの Phase で出てきた機能の多くに「env で殺すスイッチ」がある、という地図として押さえる。

### モデルの選び方（優先順位順）

1. **セッション中**: `/model <alias|name>`（引数なしで picker）。`d` で user settings に既定として保存
2. **起動時**: `claude --model <alias|name>`
3. **環境変数**: `ANTHROPIC_MODEL=<alias|name>`
4. **settings**: `model` フィールド（恒久）

`--model` と `ANTHROPIC_MODEL` は**そのセッション限定**。resume したセッションは保存時のモデルを維持する。

### モデルエイリアス

| エイリアス | 意味 |
|---|---|
| `default` | アカウント種別の推奨モデルに戻す（オーバーライド解除） |
| `best` | 最も高性能（現状 `opus` 相当） |
| `sonnet` | 日常コーディング向け最新 Sonnet |
| `opus` | 複雑な推論向け最新 Opus |
| `haiku` | 高速・軽量 |
| `sonnet[1m]` / `opus[1m]` | 1M トークンコンテキスト版 |
| `opusplan` | **plan モード中は opus、実行に移ると sonnet** に自動切替 |

> API では `opus`=Opus 4.7 / `sonnet`=Sonnet 4.6。Bedrock/Vertex/Foundry では古めに解決されるので、ピン留めには full model name か `ANTHROPIC_DEFAULT_*_MODEL` を使う。

### effort level（推論の深さ）

`/effort`、`--effort`、`CLAUDE_CODE_EFFORT_LEVEL`、settings の `effortLevel` で設定。Opus 4.7 は `low/medium/high/xhigh/max`、デフォルト `xhigh`。`max` はセッション限定。プロンプトに `ultrathink` と書くとそのターンだけ深く考える。

### subagent / モデル制御（Phase 1 Task 2 reference と接続）

- `CLAUDE_CODE_SUBAGENT_MODEL` で全 subagent のモデルを強制（呼び出し時 model 引数や frontmatter より優先、`inherit` で通常解決）
- `availableModels`（managed/policy）で選べるモデルを allowlist 制限。組織で強制したいときに

> Phase 1 Task 2 reference「subagent のモデル上書き（呼び出し時 > frontmatter > 親継承）」の上位に、この env による全体強制が乗る、という階層。

### モデル設定の組織制御（3点セット）

完全に制御するには3つを組み合わせる:
- `availableModels`: 選べるモデルを制限
- `model`: 起動時の初期選択
- `ANTHROPIC_DEFAULT_*_MODEL`: Default やエイリアスの解決先を固定

## 重要ポイント

- env は3経路（shell / settings.env / 一部 .env）。**env は settings を上書き、CLI/`/model` が一部 env を上書き**
- これまでの Phase の機能の多くに **env のキルスイッチ**がある（auto memory / background / checkpoint / tool search 等）
- モデル選択の優先順位: **`/model`・`--model`・`ANTHROPIC_MODEL`（セッション限定）> settings.model（恒久）**
- エイリアス `opusplan` = **plan で opus、実行で sonnet** に自動切替（コスト最適の定番）
- effort は推論の深さ。Opus 4.7 デフォルト `xhigh`、`ultrathink` でそのターンだけ深く
- `availableModels`（managed）で組織のモデル allowlist 強制、`CLAUDE_CODE_SUBAGENT_MODEL` で subagent モデル全体強制
- `ANTHROPIC_BASE_URL` は**送り先を変えるだけでモデルは変えない**（混同注意）

## コード例 / 図

### モデル指定の優先順位

```text
/model sonnet（セッション中・即時）
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

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 前の教材: [03-permission-modes.md](03-permission-modes.md)
- 関連 docs: [Model config](https://code.claude.com/docs/en/model-config) / [Env vars](https://code.claude.com/docs/en/env-vars) / [Subagents](https://code.claude.com/docs/en/sub-agents)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_
