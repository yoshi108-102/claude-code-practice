---
title: subagent のモデルを親セッションと別にする方法
phase: 1
task: 2
topic: subagent / model override / cost optimization
created: 2026-05-08
---

# subagent のモデルを親セッションと別にする方法

出典:
- [Subagents](https://code.claude.com/docs/en/sub-agents) — agent 定義 frontmatter の `model` フィールド
- 本セッションの Agent ツール schema（`model` パラメータが `sonnet | opus | haiku` の enum）

---

## 質問

> subagent だけ sonnet とかできないの?
> （親セッションが Opus 4.7 のまま、サブエージェントだけ軽いモデルで動かしたい）

## 結論

**できる。むしろコスト最適化の王道パターン。**

subagent のモデルは **3 段階の優先順位**で決まる:

```
[最優先] 1. Agent tool 呼び出し時の model 引数
            ↓ 指定なし
         2. agent 定義ファイル frontmatter の model
            ↓ 指定なし
         3. 親セッションのモデルを継承
```

## 設定方法（3 レイヤー）

### ① 呼び出し時に上書き — 最強で柔軟

Agent ツール schema に `model` パラメータがある:

```json
{
  "subagent_type": "Explore",
  "description": "rg ベースで symbol を探す",
  "prompt": "...",
  "model": "sonnet"
}
```

選択肢: `sonnet | opus | haiku` の 3 択。
**Opus セッションから sonnet subagent を呼ぶ**ことが可能。

### ② Agent 定義の frontmatter

`~/.claude/agents/<name>.md` または plugin 配下:

```yaml
---
name: my-agent
description: ...
model: sonnet
tools: [Read, Grep, Bash]
---
```

その agent を呼ぶときは何も指定しなくても sonnet で動く。

### ③ 何も書かないと親モデル継承

Opus セッション → subagent も Opus（高くつく）。
**意図しない高コストの最大原因はこれ。**

## 実用パターン

```
[親セッション: Opus 4.7] ← 設計判断・複雑な実装
   │
   ├─ Explore subagent (sonnet)     ← grep/find は安いモデルで十分
   ├─ Plan subagent (opus)          ← 設計判断は親と同じ高品質で
   ├─ general-purpose (haiku)       ← 単純な調査タスク
   └─ code-reviewer (sonnet)        ← レビューは sonnet で十分
```

## なぜこれがコスト的に効くか

- **Opus は Sonnet の数倍単価**（公式料金表参照）
- **subagent は並列実行可能** — 独立した複数の subagent を 1 メッセージで投げると同時に走る
- **subagent の中間結果は親 context に流入しない** — subagent 内で 100 ファイル読んでも親に返るのは最終要約だけ。**親の context 圧迫を防げる**

→ 「親は Opus、subagent は sonnet/haiku」が定石。

## 落とし穴

| 落とし穴 | 内容 |
|---|---|
| **デフォルト継承** | 明示しないと親モデルになる。Opus セッションで subagent 大量発射 = 高額 |
| **agent 定義で固定された model は `/model` に従わない** | frontmatter `model: opus` の agent は、親が sonnet でも opus で走る |
| **subagent_type ごとにデフォルトが違う** | 例: `claude-code-guide` は haiku、`Plan` は親継承、など agent 定義しだい |
| **`Explore` 等の組み込み subagent_type も model 上書き可** | 一覧で見えなくても schema 上は受け付ける |

## 確認方法

- `/agents` でエージェント一覧と各 agent の model 設定を確認
- agent 定義ファイル（`~/.claude/agents/*.md` や plugin 内）を直接読む
- Agent 呼び出し時に `model` を明示するのが最も確実

## 関連メモ

- [plan-mode vs Plan subagent](./plan-mode-vs-plan-subagent.md)
- [claude-agents CLI and sources](./claude-agents-cli-and-sources.md)
