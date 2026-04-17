# subagent ごとの memory と context 分離

出典: [sub-agents#enable-persistent-memory](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory)

**Phase 3 の本題。Task 1 で「agent ごとに異なる memory を注入できるか？」という疑問を追いかけた先取り調査。**

## できる: subagent は独自の persistent memory を持てる

subagent 定義の frontmatter に `memory:` フィールドを書くだけ。

```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---

You are a code reviewer. As you review code, update your agent memory with
patterns, conventions, and recurring issues you discover.
```

## 3 つの memory scope

| scope | 保存先 | 使い所 |
|---|---|---|
| `user` | `~/.claude/agent-memory/<agent-name>/` | 全プロジェクト横断の学習 |
| `project` | `.claude/agent-memory/<agent-name>/` | プロジェクト固有、**git で共有可（推奨デフォルト）** |
| `local` | `.claude/agent-memory-local/<agent-name>/` | プロジェクト固有だが git に載せない |

**subagent ごとに別ディレクトリ**（`<agent-name>/` で分離）。複数 subagent が同じ repo で独立して知識を蓄積。

## memory enable 時の挙動

- subagent の system prompt に **memory 読み書きの指示**が入る
- `MEMORY.md` の先頭 **200 行 / 25KB** が system prompt に注入（main session と同じ閾値）
- **Read / Write / Edit tool が自動で有効化**される（memory 管理のため）

## main session の auto memory との関係

- **完全に分離**。main は `~/.claude/projects/<project>/memory/`、subagent は `agent-memory/<name>/`
- subagent の探索結果は main context を汚さない
- main の memory は subagent に見えない（context 節約のため）

→ code-reviewer は「よく見るバグパターン」、security-reviewer は「過去の脆弱性事例」、test-writer は「テスト慣習」をそれぞれ独立に育てられる。

## memory 以外の subagent ごと context 注入手段

memory 以外にも「subagent ごとに違う context」を作る手段が複数ある:

| frontmatter | 役割 |
|---|---|
| `prompt` (= markdown 本文) | subagent の system prompt を**完全上書き**（main の system prompt は継承しない） |
| `initialPrompt` | 起動時の最初のメッセージ |
| `skills` | 指定 skill の**全文**を subagent context に注入（invoke 可能にするだけじゃなく） |
| `mcpServers` | subagent だけに有効な MCP 接続 |
| `tools` / `disallowedTools` | tool アクセスを制限 |
| `hooks` | subagent 専用ライフサイクル hook |
| `model` | subagent 専用モデル（軽い作業は Haiku など） |
| `effort` | reasoning effort レベル |
| `isolation: worktree` | 一時的な git worktree で実行（repo を独立コピー） |

つまり「この subagent だけ MCP Notion 使える」「この subagent には security skill 全文を持たせる」みたいな細かい context 設計ができる。

## 運用 Tips（docs から）

- **デフォルトは `project` 推奨**。チーム全員の subagent が同じ学びを蓄積できる
- subagent に **「作業前に memory を確認して」「作業後に学びを memory に保存して」** と明示的に依頼すると育つ
- subagent の markdown 本文に以下を書いておくと自律的に維持する:
  ```
  Update your agent memory as you discover codepaths, patterns, library
  locations, and key architectural decisions. This builds up institutional
  knowledge across conversations. Write concise notes about what you found
  and where.
  ```

## 実装例（frontmatter 全フィールド）

```yaml
---
name: security-reviewer
description: Reviews code for security vulnerabilities. Checks auth, injection, secrets.
memory: project
tools: Read, Grep, Glob, Bash
disallowedTools: Edit, Write
model: opus
effort: high
skills:
  - security-patterns
mcpServers: []
hooks: []
background: false
isolation: worktree
color: red
---

You are a senior security engineer. Review code for OWASP Top 10 vulnerabilities...
```

## 関連: `/agents` コマンド

- `/agents` で subagent 一覧・編集
- **Configure memory** ステップで User/Project/Local/None を選択
- 選ぶと自動で上記 `memory:` フィールドが入る

## 学習プロジェクトでの接続

- Phase 3 で **sub-agents.md の完全読解**と、実際に custom subagent 2〜3 個を作る
- memory 付き subagent を持つ / 持たないでの動作差を体験する
- main session との context 分離の効果を `/context` で可視化
