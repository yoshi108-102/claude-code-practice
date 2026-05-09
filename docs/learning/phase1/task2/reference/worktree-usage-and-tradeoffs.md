# Worktree の実用 — 必須条件 / merge 方法 / branch 切替との比較

出典:
- [Run parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees)
- [CLI reference > `--worktree` / `--tmux`](https://code.claude.com/docs/en/cli-reference)
- [Sub-agents > frontmatter `isolation: worktree`](https://code.claude.com/docs/en/sub-agents)
- [Hooks > `WorktreeCreate` / `WorktreeRemove`](https://code.claude.com/docs/en/hooks)

**Phase 1 / Task 2 の lesson 中、CLI flags 章で `--worktree` を解説した直後の Q&A から派生。**

## 議論のきっかけ

教材で `claude -w feature-auth` の表面的な使い方は書いたが、実用上の Q が 4 つ続いた:

1. git が無いと使えない？
2. 「Claude のサンドボックスに別環境を作る」って認識で合ってる？
3. どうやって merge するの？
4. branch 切ってから claude 起動するのと変わらなくない？

→ いずれも「**worktree を使う意義が腑に落ちるか**」を問う良い質問。

## Q&A

### Q1. `--worktree` は git 必須？

**A. 既定では yes、hook で逃げ道あり。**

公式 docs:
> Everything below assumes a git repository.

`--worktree` は内部で `git worktree add` を呼ぶ。`.git` が無いと失敗。

**逃げ道**: `WorktreeCreate` / `WorktreeRemove` hook を仕込むと、`git worktree` ロジックを丸ごと差し替えられる。SVN / Perforce / Mercurial 等にも対応可（Phase 3 hooks territory）。

| 状況 | `--worktree` |
|---|---|
| `.git` あり | ✅ そのまま使える |
| 非 git VCS + hook 仕込み済 | ✅ 使える |
| バージョン管理なし、hook なし | ❌ 使えない |

### Q2. 「Claude のサンドボックスに別環境を作る感じ」で合ってる？

**A. "別の作業環境" は ✅、"サンドボックス" は ⚠️ 誤解しやすい。**

worktree は**ファイル並びの隔離**であって、**セキュリティサンドボックスではない**。

| 観点 | git worktree | セキュリティサンドボックス |
|---|---|---|
| ファイル単位の独立 | ✅ あり | ✅ あり |
| `.git` 履歴 / remote | **同じ `.git` を共有** | 通常は分離 |
| ファイルシステム的閉じ込め | **無い**（他 worktree のファイルも読める） | あり |
| プロセス権限の制限 | 無い | あり |
| ネットワーク制限 | 無い | あり |

→ worktree A で動く Claude は worktree B のファイルも普通に読めるし、`rm -rf ~/Documents` も普通に効く。**ホスト保護目的では使えない**。

紛らわしいことに Claude Code には `/sandbox` という**別の機能**があり、これがファイルシステム / ネットワークの本物の隔離を扱う（Phase 2 の sandboxing で本格扱い）。

正しいメンタルモデル:

```
リポジトリ（.git は1つ）
├── メイン checkout (main ブランチ)
│   └── ファイル
├── worktree feature-a (別ブランチ)
│   └── ファイル ← Claude セッション A が触る
└── worktree feature-b (別ブランチ)
    └── ファイル ← Claude セッション B が触る
```

「ファイル並びは別、git 履歴は共有、セキュリティ的閉じ込めは無い」。

### Q3. どうやって merge するの？

**A. 普通の git merge / PR ワークフロー。worktree 専用の merge は無い。**

`.git` を共有しているので、**全 worktree のブランチが同じ history グラフ上にいる**。任意の checkout から他のブランチを merge できる。

**ブランチ命名の罠**: `claude --worktree feature-auth` で作ると、ブランチ名は `worktree-feature-auth`（**`worktree-` プレフィックス付き**）。`feature-auth` ではない。

#### 推奨ワークフロー（PR 経由）

```bash
# === worktree 内 ===
cd .claude/worktrees/feature-auth/
git add .
git commit -m "Add OAuth flow"
git push -u origin worktree-feature-auth

# === GitHub で PR 作って merge ===
gh pr create --fill
gh pr merge --squash

# === メインに戻る ===
cd /path/to/main-repo
git checkout main && git pull

# === worktree 片付け ===
git worktree remove .claude/worktrees/feature-auth
# あるいは Claude Code session 内で exit すれば「残す? 消す?」プロンプト
```

#### 直 merge（remote 経由しない）

```bash
cd /path/to/main-repo
git checkout main
git merge worktree-feature-auth
git push
git worktree remove .claude/worktrees/feature-auth
```

#### 制約: 同じブランチを 2 箇所で checkout できない

worktree のブランチをメインで checkout したい時は、**先に worktree を削除**して branch を解放する必要あり:

```bash
git worktree remove .claude/worktrees/feature-auth
git checkout worktree-feature-auth   # これで OK
```

#### 典型やらかし

| やらかし | 対処 |
|---|---|
| commit せず exit で worktree 削除 → 変更ロスト | exit 前に必ず commit |
| ブランチ名思い出せない | `git worktree list` |
| 「already checked out」エラー | 先に `git worktree remove` |
| `.gitignore` に `.claude/worktrees/` 入れ忘れ | メインで untracked file 地獄 |
| `node_modules` の再インストール忘れ | worktree ごとに `npm install` 必要 |

### Q4. branch 切ってから claude 起動するのと変わらなくない？

**A. 単一セッションだけならほぼ一緒。worktree が真価を出すのは 4 シナリオ。**

#### 比較

```bash
# パターン A: branch 切替
git checkout -b feature-auth
claude

# パターン B: worktree
claude -w feature-auth
```

機能的には**ほぼ同じ**。1 セッションで完結する作業なら A の方が:
- `node_modules` 再インストール不要
- ディレクトリ移動なし
- ビルド artifact 共有

で**簡単**。

#### worktree が勝つ 4 シナリオ

##### 1. 複数 Claude セッションの並行実行
git は「**同じブランチを 2 箇所で checkout できない**」制約。

```
[ターミナル A] feature-auth で作業
[ターミナル B] 別 feature やりたい
   → 同じ checkout で git checkout すると A が壊れる
   → 別 clone は重い + .env 移動だるい
   → stash は流れが分断
```

worktree なら 3 セッション同時に動かせて、ファイル編集が混ざらない。**これが最大の存在意義**。

##### 2. subagent の並列 edit
custom subagent の frontmatter に `isolation: worktree` を入れると subagent ごとに一時 worktree が作られる。3 つの subagent に同じファイル群を並列で改修させても衝突しない。同じ checkout だと `Edit` の競合で破綻する。

##### 3. メインを汚さず実験
- Claude が変な変更を入れた → worktree ごと削除で終了
- メインの作業状態には一切影響しない
- branch 方式だと untracked file が travel する / `node_modules` がガチャ替わり / ビルド artifact が混ざる

##### 4. PR の隔離レビュー / 修正
```bash
# branch 方式: 自分の作業中変更を stash / commit してから checkout
# worktree 方式
claude -w '#1234'   # メインに触れず PR 状態でレビュー → exit で捨てる
```

「今の作業を中断せず別 PR を見たい」が頻繁にあるなら worktree が圧倒的に楽。

#### 判断フローチャート

```
複数の Claude セッションを同時に走らせたい？
├─ YES → worktree 一択
└─ NO
   ├─ subagent に並列 edit させたい？
   │  ├─ YES → worktree 必須（isolation: worktree）
   │  └─ NO
   │     ├─ メインの作業状態を汚さず実験したい？
   │     │  ├─ YES → worktree が楽
   │     │  └─ NO
   │     │     └─ PR を頻繁に隔離レビューする？
   │     │        ├─ YES → worktree が楽
   │     │        └─ NO → branch 切替で十分
```

## 結論 / 押さえるポイント

- `--worktree` は内部で **`git worktree add` を呼ぶ薄いラッパー**
- 「並列性」「隔離性」「使い捨て性」のいずれかを欲しい時の道具
- セキュリティサンドボックスではない（**ファイル並びの隔離だけ**）
- merge は普通の git ワークフロー（PR 経由 / 直 merge どちらも可）
- ブランチ命名は **`worktree-<name>`** プレフィックス付き
- `node_modules` 等 untracked / gitignored は再インストールが必要（`.worktreeinclude` で `.env` 等は自動コピー可）
- 単一 Claude セッションしか動かさないなら、`git checkout -b` で十分

## 関連

- 教材: [01-cli-reference.md](../01-cli-reference.md)（`--worktree` / `--tmux` フラグ）
- 関連 reference: [plan-mode-vs-plan-subagent.md](plan-mode-vs-plan-subagent.md), [claude-agents-cli-and-sources.md](claude-agents-cli-and-sources.md)
- 後続関連: Phase 2 の sandboxing（worktree とは別物）, Phase 3 の subagent + `isolation: worktree`, Phase 3 の `WorktreeCreate` hook
