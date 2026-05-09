# Plan mode と Plan subagent — 別物だが補完関係

出典:
- [Permission modes > Analyze before you edit with plan mode](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)
- [Sub-agents > Built-in subagents](https://code.claude.com/docs/en/sub-agents#built-in-subagents)

**Phase 1 / Task 2 の lesson 中、ユーザー Q「plan mode というのは『コード変えんなよ』と言われた agent がいるだけ？」から派生。**

## 議論のきっかけ

`claude agents` の解説でビルトイン subagent に **Plan** があると分かった直後、ユーザーが「じゃあ plan mode って Plan subagent のこと？」と直感的な仮説を立てた。半分正解で半分ずれていたので、両者を docs ベースで分離整理した。

## Q&A

### Q1. plan mode と Plan subagent は同じものか？

**A.** **別物**。レイヤーが違う。

| | Plan **mode** | Plan **subagent** |
|---|---|---|
| レイヤー | session 全体に課す permission のレベル | Claude が呼び出すビルトイン subagent の 1 つ |
| 単位 | session 全体 | 1 回の delegate（別 context） |
| 効果 | **メイン agent 自身**が読み取り系しかできなくなる | 呼び出された subagent が読み取り系しかできなくなる |
| 出力 | **計画 (plan) を提示 → 承認 UI** | 通常の subagent 結果（要約） |
| 入り方 | `Shift+Tab` 循環 / `--permission-mode plan` / `/plan` | Claude が plan mode 中に**自動で delegate** |

### Q2. どう補完しているのか？

**A.** plan mode が**枠組み**、Plan subagent が**その枠内のヘルパー**。

```
[ユーザー]
  ↓ Shift+Tab で plan mode へ
[メイン Claude] ─── permission mode = plan ───┐
  読み取り・コマンド実行は OK                     │
  ファイル編集は禁止                              │
  ↓ コードベース調査が必要と判断                  │
  ↓ delegate                                   │
[Plan subagent] ─── 別 context, 読み取り限定 ──┘
  ファイル群を読んで要約だけ返す
  ↓
[メイン Claude]
  ↓ 計画を生成
[ユーザー] ← 計画を見せられて承認 UI
   - Approve & auto
   - Approve & acceptEdits
   - Approve & manual
   - Keep planning
   - Refine with Ultraplan
```

公式 docs から関連箇所:

**Plan mode**（[permission-modes#plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)）:

> Plan mode tells Claude to research and propose changes without making them. Claude reads files, runs shell commands to explore, and writes a plan, but does not edit your source.

**Plan subagent**（[sub-agents#built-in-subagents](https://code.claude.com/docs/en/sub-agents#built-in-subagents)）:

> A research agent used during plan mode to gather context before presenting a plan.
> - Tools: Read-only tools (denied access to Write and Edit tools)
> - Purpose: Codebase research for planning
> When you're in plan mode and Claude needs to understand your codebase, it delegates research to the Plan subagent. **This prevents infinite nesting** (subagents cannot spawn other subagents) **while still gathering necessary context**.

### Q3. なぜ「plan mode = メインを縛るだけ」では不十分なのか？

**A.** メイン Claude の context が膨らんでしまうから。

- メインだけで広範な調査をすると、計画提示の頃には context が一杯になりかねない
- 「広く読む役」を別 context の subagent に投げて、メインには**要約だけ戻す**設計が context 効率良
- これは復習キュー Q4 (subagent の本質) と直結: subagent = 別 context で読みまくって要約を返す

### Q4. 関連: 他のビルトイン subagent

`claude agents` で見える組み込み 3 種:

- **Explore**: read-only 調査専門。plan mode 関係なく、Claude が「広く調べたい」時に使う
- **Plan**: plan mode 中の調査専門
- **general-purpose**: 全 tool 使える汎用（exploration + modification 両方）

3 つとも「メインの context を温存する」目的の道具立て。

## 結論 / 押さえるポイント

- **plan mode** は **session 全体の permission ルール**で、メイン Claude を縛る
- **Plan subagent** は plan mode 中に呼ばれる**別 context のヘルパー**
- 両者は同じ層に並ぶものではなく、**枠組みとヘルパー**の関係
- subagent の本質「別 context で広く読んで要約だけ返す」が、plan mode の設計にもそのまま現れている
- 詳細な subagent の作り方は Phase 3 で本格扱い

## 関連

- 教材: [01-cli-reference.md](../01-cli-reference.md)（`--permission-mode` フラグ）
- 関連 reference: [claude-agents-cli-and-sources.md](claude-agents-cli-and-sources.md)
- 復習キュー: Phase 1 / Task 1 — Q4「subagent の context window」, Q6「Auto モードの仕組み」
