# Phase 1 / Task 2: CLI の基本操作

## 今回読む docs

- [cli-reference](https://code.claude.com/docs/en/cli-reference) — CLI フラグ・オプション一覧
- [commands](https://code.claude.com/docs/en/commands) — slash commands 一覧
- [interactive-mode](https://code.claude.com/docs/en/interactive-mode) — 対話モードの挙動
- [keybindings](https://code.claude.com/docs/en/keybindings) — キーバインド

## 目次

- [01-cli-reference.md](01-cli-reference.md) — CLI 起動コマンドとフラグ [done]
- [02-slash-commands.md](02-slash-commands.md) — スラッシュコマンドとバンドルスキル
- [03-keyboard-shortcuts.md](03-keyboard-shortcuts.md) — 対話モードのキーボードショートカット [done]
- [04-vim-editor-mode.md](04-vim-editor-mode.md) — Vim エディタモード [done]
- [05-history-and-shell-mode.md](05-history-and-shell-mode.md) — コマンド履歴とシェル統合（reverse search / background bash / shell mode `!`） [done]
- [06-interactive-features.md](06-interactive-features.md) — プロンプト提案・/btw・タスクリスト・リキャップ・PR ステータス [done]
- [07-keybindings-config.md](07-keybindings-config.md) — キーバインドのカスタマイズ [skipped: ユーザー判断。要点=`~/.claude/keybindings.json` でキー操作を再割り当て可能]

## reference（深掘り Q&A）

- [reference/claude-agents-cli-and-sources.md](reference/claude-agents-cli-and-sources.md)
- [reference/plan-mode-vs-plan-subagent.md](reference/plan-mode-vs-plan-subagent.md)
- [reference/worktree-usage-and-tradeoffs.md](reference/worktree-usage-and-tradeoffs.md)
- [reference/subagent-model-override.md](reference/subagent-model-override.md) — subagent だけ別モデル（sonnet/haiku）で動かす方法
- [reference/customizing-bundled-skills.md](reference/customizing-bundled-skills.md) — `/review` `/simplify` 等の bundled skill をチームルール用にカスタマイズする方法

## 振り返り（Task まとめクイズ）

（採点済み: 2026-05-22 — ✅1 / ⚠️2 / ❓3。Q1,Q3,Q4,Q5,Q6 を復習キューへ）

回答は各問の `**回答**:` 行の下に記入してください。
全問記入後に `/learning-flow:grade --summary` を実行すると、Claude が採点して進捗を更新します。
（Task 2 は Topic クイズの多くをスキップしたため、まとめクイズで横断論点を厚めに確認します）

---

### Q1. 「コンテキスト × ツール」軸で 3 つの委譲手段を整理する

`/btw`・通常の subagent・plan mode 中の **Plan subagent** は、いずれも「メインの処理とは別枠で何かをやらせる」仕組みだが、**「メインの context が見えるか」「ツールが使えるか」**の 2 軸で性質が大きく異なる。3 つを 2 軸で表に整理し、**「今 Claude が読んだコードを踏まえた質問を、履歴を汚さず投げたい」ときにどれを使うべきか**、そして**それを subagent でやってはいけない理由**を説明せよ。

**参考**:
- [Interactive mode > Side questions with /btw](https://code.claude.com/docs/en/interactive-mode#side-questions-with-%2Fbtw)
- [Sub-agents > Built-in subagents](https://code.claude.com/docs/en/sub-agents#built-in-subagents)

**関連ノート**: [06-interactive-features.md](06-interactive-features.md), [reference/plan-mode-vs-plan-subagent.md](reference/plan-mode-vs-plan-subagent.md)

**回答**:
/btw (by the way)コマンドで聞けばいい。subagentはmain　agentのコンテキストを持っていないので聞けない

btw -> メインのcontext 見える tool 読み取りのみ
plan subagent -> メインのcontext見えない、 tool　読みとりのみ
subagent -> メインのcontext見えない、tool全部使える


---

### Q2. plan mode への 3 つの入り方と、Plan subagent との関係

plan mode に入る方法は教材中に少なくとも 3 つ出てきた（CLI フラグ / キーボード / スラッシュコマンド）。**3 つを挙げよ**。さらに、「plan mode」と「Plan subagent」は**同じ層のものではない**。両者の関係を「枠組み」と「ヘルパー」という言葉を使って説明し、**なぜ plan mode 中の調査をメイン agent が全部やるのではなく Plan subagent に委譲する設計になっているのか**（context の観点）を述べよ。

**参考**:
- [Permission modes > plan mode](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)
- [CLI reference > --permission-mode](https://code.claude.com/docs/en/cli-reference)

**関連ノート**: [01-cli-reference.md](01-cli-reference.md), [03-keyboard-shortcuts.md](03-keyboard-shortcuts.md), [reference/plan-mode-vs-plan-subagent.md](reference/plan-mode-vs-plan-subagent.md)

**回答**:
plan mode に入るには、/plan かShift + Tab, --permission-mode planなどで使用可能
plan modeはガードレール的なもので、claudeのpermissionをいじるもの。plan subagentは本体のcontext windowを削減
するためのplan用の動きをするagentなので、前者は決定論的で後者は非決定論的なものだし、レイヤが違う

---

### Q3. worktree が本当に効く場面 — branch 切替・`/sandbox` との違い

`claude -w feature-x` と `git checkout -b feature-x && claude` は、**単一セッションならほぼ等価**である。それでも worktree を使う価値が出る場面を**2 つ**挙げよ。また、worktree を「Claude を閉じ込めるセキュリティサンドボックス」だと考えるのは誤りである。**worktree が隔離するもの／しないもの**を述べ、本物の隔離を担う別機能の名前を答えよ。

**参考**:
- [Run parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees)
- [CLI reference > --worktree](https://code.claude.com/docs/en/cli-reference)

**関連ノート**: [01-cli-reference.md](01-cli-reference.md), [reference/worktree-usage-and-tradeoffs.md](reference/worktree-usage-and-tradeoffs.md)

**回答**:
わからない
---

### Q4. シェルコマンドを動かす 3 つの経路と、permission／context への影響

「ターミナルでコマンドを実行する」のに、Task 2 では実質 3 つの経路が出てきた:(A) shell mode（`! git diff`）、(B) Claude に「git diff して」と頼んで Bash ツールで実行させる、(C) background bash（`Ctrl+B`）。**(A) と (B) で permission の扱いがどう違うか**、**(A) を多用すると何が問題になるか（context の観点）**、そして **(C) が解決する問題は何か**を説明せよ。

**参考**:
- [Interactive mode > Shell mode with ! prefix](https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix)
- [Interactive mode > Background bash commands](https://code.claude.com/docs/en/interactive-mode#background-bash-commands)

**関連ノート**: [05-history-and-shell-mode.md](05-history-and-shell-mode.md)

**回答**:
A コマンドを直接打ち込むので普通のuser権限と変わらず
B claudeの権限範囲までのことしかできない
C ちょっとやってることが違って、処理が非同期になる。terraform applyとか待ってられないようなのを非同期処理してさっさと次のことをするため
---

### Q5. subagent のモデルを親と別にする — 優先順位とコストの罠

親セッションが Opus 4.7 のとき、subagent のモデルは **3 つのレイヤー**の優先順位で決まる。**3 つを優先順位順に挙げよ**。また「**意図しない高コスト**の最大の原因」はどのケースか、なぜそれが起きるかを説明せよ。さらに、`frontmatter` に `model: opus` と固定された agent は、親セッションを `/model` で sonnet に変えても **opus で動く** —— これは Q&A のどの観点（subagent と context window の本質）と整合しているか、一言添えよ。

**参考**:
- [Sub-agents > model field](https://code.claude.com/docs/en/sub-agents)

**関連ノート**: [reference/subagent-model-override.md](reference/subagent-model-override.md), [reference/claude-agents-cli-and-sources.md](reference/claude-agents-cli-and-sources.md)

**回答**:
覚えていない
---

### Q6. bundled skill をチームルールでカスタマイズする 4 つの道

`/review` 相当のチーム独自レビューを実装する手段は 4 つあった:(A) CLAUDE.md にルール追記 / (B) 別名 skill 自作 / (C) 同名で bundled を上書き / (D) plugin 配布。**(B) が「最も健全」とされる理由を、context 効率の観点から (A) と対比して**述べよ。また、**(C)（同名上書き）が推奨されない理由を 2 つ**挙げよ。

**参考**:
- [Skills - where skills live / precedence](https://code.claude.com/docs/en/skills)
- [Commands - [Skill] マーク](https://code.claude.com/docs/en/commands)

**関連ノート**: [02-slash-commands.md](02-slash-commands.md), [reference/customizing-bundled-skills.md](reference/customizing-bundled-skills.md)

**回答**:
わからん