# 01. CLAUDE.md の階層と `.claude/rules/`

> 出典: [How Claude remembers your project](https://code.claude.com/docs/en/memory)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「memory」の CLAUDE.md / rules 節を起点に Claude が自動生成した教材です。

## 概要

CLAUDE.md は「**毎セッション再説明せずに済ませたいこと**を書き留める場所」。毎回の起動時にフルでロードされ、context を消費する。だからこそ「どこに置くか」「何を書き、何を別ファイルに逃がすか」が実務の肝になる。

判断の核:**毎セッション必要な事実だけを CLAUDE.md に。手順やコードベースの一部にしか効かないものは skill か path-scoped rule へ。**

## 公式docsに沿った解説

### CLAUDE.md に「書くべきもの / 逃がすもの」

書くタイミング(docsの「When to add」):

- Claude が**同じミスを2回**した
- code review が「これは知っておくべきだった」を指摘した
- 前セッションと**同じ訂正**をまたチャットに打った
- 新メンバーが生産的になるのに必要な前提

逆に逃がすもの:

- **多段の手順** → skill（invoke 時のみロード）
- **コードベースの一部にしか効かないルール** → path-scoped rule（マッチ時のみロード）
- **特定タイミングで必ず実行**（コミット前など） → hook

### 配置場所（ロード順 = 広い → 狭い）

下にいくほど後にロードされ、後勝ちに近い扱い（より具体的な指示が後に来る）。

| スコープ | 場所 | 共有範囲 |
|---|---|---|
| **Managed policy** | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md` / Linux `/etc/claude-code/CLAUDE.md` / Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | 組織全員（個人設定で除外不可） |
| **User** | `~/.claude/CLAUDE.md` | 自分の全プロジェクト |
| **Project** | `./CLAUDE.md` または `./.claude/CLAUDE.md` | チーム（git 共有） |
| **Local** | `./CLAUDE.local.md` | 自分・このプロジェクトのみ（`.gitignore` 推奨） |

> これは復習キュー **Task1-Q1**（CLAUDE.md vs MEMORY.md の保管場所）の CLAUDE.md 側そのもの。「project はリポジトリ直下で git 共有、user は `~/.claude/`、local は gitignore」を実物で確認するのが今回のハンズオン。

### How CLAUDE.md files load（読み込みの解決順）

- cwd から**ディレクトリツリーを上に遡って**各階層の `CLAUDE.md` / `CLAUDE.local.md` を探す
- 見つかった全ファイルは**上書きではなく連結**。ルート → cwd の順（launch 地点に近いものが後＝最後に読まれる）
- 同一階層では `CLAUDE.local.md` が `CLAUDE.md` の後に追記される
- **サブディレクトリ**の CLAUDE.md は起動時ではなく「そのディレクトリのファイルを読んだとき」にロード（→ compaction で消えやすい、Phase 1 Task 3 と直結）
- **block-level HTML コメント `<!-- ... -->` は context 注入前に除去**される → 人間向けメモを置いても token を食わない（コードブロック内のコメントは保持）

### Imports（`@path` 構文）

CLAUDE.md は `@path/to/file` で他ファイルを取り込める。

```text
See @README for project overview and @package.json for npm commands.
- git workflow @docs/git-instructions.md
- 個人設定 @~/.claude/my-project-instructions.md
```

- 相対パスは「import を含むファイル基準」（cwd 基準ではない）
- 再帰 import 可、**最大5ホップ**
- **注意**: import してもロードは起動時・フルロードなので **context 節約にはならない**（整理目的）
- 初回は外部 import を列挙する承認ダイアログが出る。拒否すると以後無効

> **AGENTS.md**: Claude Code は `AGENTS.md` を読まない。既存があれば `CLAUDE.md` で `@AGENTS.md` と import するか symlink する。`/init` は既存 AGENTS.md / `.cursorrules` / `.windsurfrules` も取り込む。

### `.claude/rules/` — トピック別に分割

大きくなったら `.claude/rules/*.md` に分ける。1ファイル1トピック（`testing.md`, `api-design.md`）。再帰探索されるので `frontend/` 等のサブディレクトリも可。

**path-scoped rule**(YAML frontmatter の `paths`):

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API Development Rules
- 全エンドポイントに入力バリデーション必須
```

- `paths` なし rule → **無条件ロード**（`.claude/CLAUDE.md` と同優先）
- `paths` あり rule → **マッチするファイルを読んだときだけ**ロード（context 節約）
- glob: `**/*.ts`, `src/**/*`, `src/**/*.{ts,tsx}`（brace 展開可）
- `~/.claude/rules/` はユーザーレベル（全プロジェクト、project rule より低優先）
- symlink 対応（共有ルールを各プロジェクトへリンク）

### 大規模チーム運用

- **組織共通 CLAUDE.md**: managed policy location に配置 → 個人設定で除外不可。`managed-settings.json` の `claudeMd` キーで直接埋め込みも可
- **`claudeMdExcludes`**(`.claude/settings.local.json` 等): monorepo で無関係な祖先 CLAUDE.md を glob で除外（managed policy は除外不可）
- **使い分け**: 技術的「強制」は settings（`permissions.deny`, `sandbox.enabled`）、振る舞いの「ガイド」は CLAUDE.md。CLAUDE.md は強制レイヤーではない

## 重要ポイント

- CLAUDE.md = **毎セッション要る事実だけ**。手順は skill、局所ルールは path-scoped rule、確実な実行は hook
- 配置は4階層（managed > user > project > local）、**連結ロード・後勝ち寄り**
- **200行以内**を目標（長いと context 圧迫＋遵守率低下）
- `@import` は整理にはなるが **context 節約にはならない**（フルロード）
- `<!-- コメント -->` は context に入らない（人間メモに最適）
- path-scoped rule は「マッチ時のみロード」で context を節約できるが、**compaction で消える**（Phase 1 Task 3）
- CLAUDE.md は強制ではなくガイド。**強制したいことは settings / hook へ**

## コード例 / 図

### 何をどこに置くかの判断フロー

```text
その指示は…
├─ 毎セッション必要な短い事実？        → CLAUDE.md（200行以内）
├─ コードの一部にだけ効く？            → .claude/rules/ に paths: 付きで
├─ 多段の手順／たまにしか使わない？      → skill（invoke 時ロード）
├─ 特定タイミングで必ず実行？           → hook
└─ 組織全体で強制したい？              → managed policy CLAUDE.md / managed settings
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 次の教材: [02-auto-memory.md](02-auto-memory.md) — auto memory（MEMORY.md）の仕組み
- 関連 docs: [Skills](https://code.claude.com/docs/en/skills) / [Context window](https://code.claude.com/docs/en/context-window) / [Hooks guide](https://code.claude.com/docs/en/hooks-guide)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_

## 振り返りクイズ

回答は各問の `**回答**:` 行の下に記入してください。
全問記入後に `/learning-flow:grade 01-claude-md-hierarchy-and-rules` で採点します。

---

### Q1. path-scoped rule の「意味」

「rule ファイルはディスクに残るんだから、path で絞っても次の会話で消えるわけじゃない。意味あるの?」という主張に反論せよ。「消える/残る」と「context に載る/載らない」を区別し、path-scoped にして**何が嬉しいのか**（context が膨らむと起きる損失の観点）を説明せよ。

**参考**:
- [Memory > path-specific rules](https://code.claude.com/docs/en/memory#path-specific-rules)
- [Context window](https://code.claude.com/docs/en/context-window)

**回答**:
パススコープを設定すると、該当するfileが読み込まれるまでコンテキストに追加されない。ディスクには残るが
そのため、コンテキスト節約という点でメリットがある他、管理コストという意味でも嬉しい。
---

### Q2. CLAUDE.md の4階層と「強制 vs ガイド」

CLAUDE.md を置ける4階層を「共有範囲が広い順」に挙げよ。また、「全社で**絶対に**特定コマンドを禁止したい」場合、CLAUDE.md に書くのは適切か。適切でないなら何を使うべきか、その理由（CLAUDE.md の性質）と共に答えよ。

**参考**:
- [Memory > Choose where to put CLAUDE.md files](https://code.claude.com/docs/en/memory#choose-where-to-put-claude-md-files)
- [Memory > Manage CLAUDE.md for large teams](https://code.claude.com/docs/en/memory)

**回答**:
managed policy > user instruction > project instruction > local instruction
の順で共有範囲が広い。特に一番上のmanaged policyはユーザレベルで除外することができない仕組みなので、全社禁止コマンドとかあったらそこに書くべき。
とはいえコマンドを禁止するんだったらルールベースでできるようにしないといけないのでpermissionとかで解決した方が良いと思う。（非決定論的なので）