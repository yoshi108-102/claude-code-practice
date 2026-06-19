# 02. Auto memory（MEMORY.md）の仕組み

> 出典: [How Claude remembers your project > Auto memory](https://code.claude.com/docs/en/memory#auto-memory)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「memory」の auto memory 節を起点に Claude が自動生成した教材です。

## 概要

auto memory は、**あなたが何も書かなくても Claude が自分でセッションを跨いだ知識を蓄積する**仕組み。ビルドコマンド・デバッグの知見・設計メモ・好み・ワークフローの癖などを、Claude が「将来役立つか」で判断して保存する（毎回は保存しない）。

CLAUDE.md が「**あなたが書く指示**」なのに対し、auto memory は「**Claude が書く学び**」。両方とも毎セッション冒頭にロードされる。

> このセッションでまさに使われている。「学習は実践多めで」をさっき記憶したのが auto memory への書き込み。実物で仕組みを確認できる絶好の題材。

## 公式docsに沿った解説

### CLAUDE.md vs auto memory

| | CLAUDE.md | Auto memory |
|---|---|---|
| 誰が書く | **あなた** | **Claude** |
| 中身 | 指示・ルール | 学び・パターン |
| スコープ | project / user / org | **リポジトリ単位**（worktree 間で共有） |
| ロード | 毎セッション（フル） | 毎セッション（**先頭200行 or 25KB**） |
| 用途 | コーディング規約・ワークフロー・設計 | ビルドコマンド・デバッグ知見・発見した好み |

要件: **Claude Code v2.1.59 以降**（`claude --version`）。subagent も自前の auto memory を持てる。

### 有効化 / 無効化

- 既定で **ON**
- `/memory` のトグル、または project settings の `autoMemoryEnabled: false`
- 環境変数 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`

### 保管場所（復習キュー Task1-Q1 の核心）

```text
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 簡潔なインデックス。毎セッションにロード
├── debugging.md       # 詳細ノート（必要時のみ読まれる）
├── api-conventions.md
└── ...
```

- `<project>` パスは **git リポジトリから導出** → 同一リポの全 worktree / サブディレクトリで**1つの memory ディレクトリを共有**。git 外なら project root を使う
- **マシンローカル**。マシン間・クラウド環境では共有されない
- 保存先変更: user settings(`~/.claude/settings.json`)の `autoMemoryDirectory`（絶対パス or `~/` 始まり）。**project/local settings からは受け付けない**（clone したリポが書込先を機密領域へ向ける攻撃を防ぐため）

> 復習キュー Task1-Q1 で間違えた「`.claude` 配下で全プロジェクト共通」は誤りで、正しくは **`~/.claude/projects/<project>/memory/` でプロジェクト（リポジトリ）単位**。ここで実物を開いて確認する。

### How it works（ロードの仕組み）

- **`MEMORY.md` の先頭200行 or 25KB（先に達した方）**が毎セッション冒頭にロード。それを超える分は起動時には載らない
- この上限は **MEMORY.md だけ**。CLAUDE.md は長さに関係なくフルロード（ただし短い方が遵守率は良い）
- `debugging.md` 等の**トピックファイルは起動時ロードされない** → Claude が必要時に通常のファイルツールで読む
- 「Writing memory」「Recalled memory」表示時に、Claude が memory ディレクトリを更新/参照している

### 監査・編集 / `/memory`

- memory はただの markdown。いつでも編集・削除可
- `/memory` は「ロード済みの CLAUDE.md / CLAUDE.local.md / rules 一覧」「auto memory トグル」「memory フォルダへのリンク」を提供
- 「pnpm を使って（npm でなく）」のように頼むと auto memory へ。「CLAUDE.md に追加して」と言えば CLAUDE.md へ

## 重要ポイント

- auto memory = **Claude が自分で書く学び**、CLAUDE.md = **あなたが書く指示**。両方毎回ロード
- 保管は **`~/.claude/projects/<project>/memory/`、リポジトリ単位、マシンローカル**（Task1-Q1 の正解）
- ロードは **MEMORY.md の先頭200行/25KB だけ**。トピックファイルは遅延読み込み → MEMORY.md はインデックスに徹する
- 保存先変更は **user/policy settings のみ**（project/local 不可。セキュリティ設計）
- どちらも「context として扱われ、強制ではない」。**具体的・簡潔ほど遵守される**
- 中身はいつでも `/memory` で監査・編集できる（プレーンな md）

## コード例 / 図

### CLAUDE.md と auto memory の棲み分け

```text
「2-space indent を使って」          → どっちでも可。恒久ルールなら CLAUDE.md
「このリポの API テストは Redis 必須」 → 発見した事実 → auto memory（Claude が自動保存）
「コミット前に make lint」            → 必ず実行 → hook（memory ではない）
「src/billing 配下は plan mode で」    → 局所ルール → .claude/rules/ に paths: 付き
```

### 自分の memory を覗く

```text
/memory            # ロード済みファイル一覧 + auto memory フォルダへのリンク
# 実体:
~/.claude/projects/<project>/memory/MEMORY.md
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 前の教材: [01-claude-md-hierarchy-and-rules.md](01-claude-md-hierarchy-and-rules.md)
- 次の教材: [03-claude-directory-map.md](03-claude-directory-map.md) — `.claude` ディレクトリの全体地図
- 関連 docs: [Subagent memory](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory) / [Context window](https://code.claude.com/docs/en/context-window#what-survives-compaction)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_

## 振り返りクイズ

回答は各問の `**回答**:` 行の下に記入してください。
全問記入後に `/learning-flow:grade 02-auto-memory` で採点します。

---

### Q1. auto memory の保管と特性

auto memory（MEMORY.md）の保管場所をフルパスの形で答えよ。また「リポジトリ単位」「マシンローカル」がそれぞれ実務で何を意味するか（チーム共有されるか? worktree 間で共有されるか?）を説明せよ。

**参考**:
- [Memory > Auto memory > Storage location](https://code.claude.com/docs/en/memory#storage-location)

**回答**:
~/.claude/projects/<id>/memory/MEMORY.md
リポジトリ単位というのはチーム共有される(git)
マシンローカルというのは自分だけということ
---

### Q2. MEMORY.md を「インデックスに徹させる」理由

auto memory のディレクトリには複数の md を置けるのに、`MEMORY.md` は短いインデックスに保ち、詳細は別ファイル（`debugging.md` 等）に逃がすのが推奨される。なぜか。ロードの仕組み（先頭200行/25KB、トピックファイルの遅延読み込み）に触れて説明せよ。

**参考**:
- [Memory > Auto memory > How it works](https://code.claude.com/docs/en/memory#how-it-works)

**回答**:
コンテキスト戦略。memoryはプロジェクトでセッション開始時に常に読み込まれるコンテキストである一方で、詳細ファイルは必要に応じてclaudeが読みにいくファイルであるため。先頭200行云々は先述した話が本質だと思うが、memory.mdはセッション読み込み時に先頭200行/25kBしか読み込まないので長く書いても間あんまり意味がない。
