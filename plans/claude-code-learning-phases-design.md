# claude-code-practice - Claude Code 学習 Phase 設計書

## 概要

Claude Code を公式 docs/reference（全114ページ）に沿って段階的に学ぶ。成果物は Phase ごとに異なる（CLAUDE.md → skill → hook → subagent → plugin → MCP server → Agent SDK agent）。各 Phase で新規概念を段階的に導入し、**Phase 完了時に担当 docs ページを全て読み終わっている**状態を目指す。

## 設計方針

- **公式 docs が根拠**: 全 Task は公式ページを読むことから始める
- **全114ページ網羅**: Phase 終了時に学習終了時点ですべての公式リファレンスに触れている状態
- **個人 → チーム → 外部統合** の自然な難易度上昇
- **手を動かす**: 各 Phase で実際の成果物を作る

## 全体像

| Phase | テーマ | 新規概念（主要） | 担当 docs ページ数 | 推奨成果物 |
|---|---|---|---|---|
| 1 | 基礎 — Claude Code の世界観を掴む | agentic loop / CLI基本 / 各Surface / context window / checkpointing | 32 | 自分用 CLAUDE.md + 各 Surface で小タスクを実行 |
| 2 | カスタマイズ — 記憶・設定・権限 | CLAUDE.md / auto memory / settings.json / permissions / output styles | 12 | 詳細 CLAUDE.md + settings.json + output style + statusline |
| 3 | 拡張機能 — Skills/Hooks/Subagents | skills / hooks / subagents / agent teams | 7 | skill 3〜5個 + hook 2個 + subagent 2〜3個 |
| 4 | プラグイン化・配布・運用 | plugins / marketplace / CI/CD / channels / enterprise | 31 | チーム用 plugin + GitHub Actions + marketplace 登録 |
| 5 | 高度な統合 — MCP / Agent SDK | MCP / Agent SDK / custom tools / sessions | 32 | 自作 MCP server + Agent SDK エージェント |

**合計: 114 ページ**

---

## Phase 1: 基礎 — Claude Code の世界観を掴む（32ページ）

### ゴール

Claude Code の「何ができるのか」「どう動くのか」を理解し、CLI/IDE/Desktop/Web の各 Surface で基本タスクを実行できるようになる。

### 新規概念

- agentic loop の仕組み
- 4つのコアツール（Read/Edit/Bash/Glob/Grep）
- context window と checkpointing
- 各 Surface の特徴と使い分け
- slash commands の基本 / interactive-mode / keybindings
- troubleshooting と error reference

### 推奨成果物

- [ ] 自分用の最小 CLAUDE.md を作成
- [ ] CLI と VS Code の両方で同じタスクを実行して比較
- [ ] Desktop/Web も触って Surface の違いを体感
- [ ] checkpointing で変更を巻き戻す体験
- [ ] troubleshooting の FAQ を一度ざっと眺める

### 担当 docs URL チェックリスト

#### 導入・概要

- [ ] https://code.claude.com/docs/en/overview — Claude Code overview
- [ ] https://code.claude.com/docs/en/quickstart — Quickstart
- [ ] https://code.claude.com/docs/en/how-claude-code-works — How Claude Code works
- [ ] https://code.claude.com/docs/en/best-practices — Best Practices for Claude Code
- [ ] https://code.claude.com/docs/en/common-workflows — Common workflows
- [ ] https://code.claude.com/docs/en/platforms — Platforms and integrations

#### CLI の基本操作

- [ ] https://code.claude.com/docs/en/cli-reference — CLI reference
- [ ] https://code.claude.com/docs/en/commands — Commands
- [ ] https://code.claude.com/docs/en/interactive-mode — Interactive mode
- [ ] https://code.claude.com/docs/en/keybindings — Customize keyboard shortcuts
- [ ] https://code.claude.com/docs/en/voice-dictation — Voice dictation
- [ ] https://code.claude.com/docs/en/fast-mode — Speed up responses with fast mode
- [ ] https://code.claude.com/docs/en/fullscreen — Fullscreen rendering

#### セットアップと実行基盤

- [ ] https://code.claude.com/docs/en/setup — Advanced setup
- [ ] https://code.claude.com/docs/en/authentication — Authentication
- [ ] https://code.claude.com/docs/en/terminal-config — Optimize your terminal setup
- [ ] https://code.claude.com/docs/en/context-window — Explore the context window
- [ ] https://code.claude.com/docs/en/checkpointing — Checkpointing

#### IDE / Surface

- [ ] https://code.claude.com/docs/en/vs-code — Use Claude Code in VS Code
- [ ] https://code.claude.com/docs/en/jetbrains — JetBrains IDEs
- [ ] https://code.claude.com/docs/en/desktop — Use Claude Code Desktop
- [ ] https://code.claude.com/docs/en/desktop-quickstart — Get started with the desktop app
- [ ] https://code.claude.com/docs/en/claude-code-on-the-web — Use Claude Code on the web
- [ ] https://code.claude.com/docs/en/web-quickstart — Get started with Claude Code on the web
- [ ] https://code.claude.com/docs/en/chrome — Use Claude Code with Chrome (beta)

#### トラブルシュート・情報源

- [ ] https://code.claude.com/docs/en/troubleshooting — Troubleshooting
- [ ] https://code.claude.com/docs/en/errors — Error reference
- [ ] https://code.claude.com/docs/en/changelog — Changelog
- [ ] https://code.claude.com/docs/en/whats-new/index — What's new
- [ ] https://code.claude.com/docs/en/whats-new/2026-w13 — Week 13 · March 23–27, 2026
- [ ] https://code.claude.com/docs/en/whats-new/2026-w14 — Week 14 · March 30 – April 3, 2026
- [ ] https://code.claude.com/docs/en/whats-new/2026-w15 — Week 15 · April 6–10, 2026

---

## Phase 2: カスタマイズ — 記憶・設定・権限（12ページ）

### ゴール

CLAUDE.md/auto memory/settings/permissions を使いこなし、プロジェクト/チーム固有の指示を長期保守できるようになる。

### 新規概念

- CLAUDE.md の階層（project / user / local）
- auto memory の仕組み（MEMORY.md）
- `.claude/` ディレクトリの構造
- settings.json（permissions, env vars, model config）
- permission modes（Plan / Auto-accept / Default）
- output styles, statusline のカスタマイズ
- sandboxing, server-managed settings（チーム展開の前準備）

### 推奨成果物

- [ ] 詳細な CLAUDE.md + CLAUDE.local.md
- [ ] settings.json で権限を pre-approve
- [ ] auto memory をオンにして使い込む
- [ ] カスタム statusline を1つ
- [ ] 自分用 output style を1つ

### 担当 docs URL チェックリスト

- [ ] https://code.claude.com/docs/en/memory — How Claude remembers your project
- [ ] https://code.claude.com/docs/en/claude-directory — Explore the .claude directory
- [ ] https://code.claude.com/docs/en/settings — Claude Code settings
- [ ] https://code.claude.com/docs/en/permissions — Configure permissions
- [ ] https://code.claude.com/docs/en/permission-modes — Choose a permission mode
- [ ] https://code.claude.com/docs/en/env-vars — Environment variables
- [ ] https://code.claude.com/docs/en/model-config — Model configuration
- [ ] https://code.claude.com/docs/en/output-styles — Output styles
- [ ] https://code.claude.com/docs/en/statusline — Customize your status line
- [ ] https://code.claude.com/docs/en/sandboxing — Sandboxing
- [ ] https://code.claude.com/docs/en/server-managed-settings — Configure server-managed settings
- [ ] https://code.claude.com/docs/en/data-usage — Data usage

---

## Phase 3: 拡張機能 — Skills / Hooks / Subagents（7ページ）

### ゴール

再利用可能なワークフローを Skills / Hooks / Subagents の3形態で実装し、使い分けを身につける。Claude Code の真骨頂。

### 新規概念

- **Skills**: slash command 化した再利用可能ワークフロー（frontmatter, trigger patterns）
- **Hooks**: PreToolUse/PostToolUse/Stop/SessionStart による自動化
- **Subagents**: 専門化エージェント、タスク委譲、コンテキスト分離
- **Agent teams**: 複数エージェントの並列実行
- **Tools reference**: Claude が使える全ツールの仕様

### 推奨成果物

- [ ] 自分用の skill 3〜5個（例: `/pr-review`, `/test-gen`, `/refactor-plan`）
- [ ] PreToolUse hook 1個（例: 危険コマンド検知）+ PostToolUse hook 1個（例: auto-format）
- [ ] カスタム subagent 2〜3個（例: explore/plan/test 専門）
- [ ] Agent teams で複数タスクを並列実行

### 担当 docs URL チェックリスト

- [ ] https://code.claude.com/docs/en/skills — Extend Claude with skills
- [ ] https://code.claude.com/docs/en/hooks-guide — Automate workflows with hooks
- [ ] https://code.claude.com/docs/en/hooks — Hooks reference
- [ ] https://code.claude.com/docs/en/sub-agents — Create custom subagents
- [ ] https://code.claude.com/docs/en/agent-teams — Orchestrate teams of Claude Code sessions
- [ ] https://code.claude.com/docs/en/features-overview — Extend Claude Code
- [ ] https://code.claude.com/docs/en/tools-reference — Tools reference

---

## Phase 4: プラグイン化・配布・チーム連携・運用（31ページ）

### ゴール

Phase 3 で作った個人用ツールを plugin として整理・配布し、CI/CD や各種 Surface、エンタープライズ環境で Claude Code を運用できるようになる。

### 新規概念

- **Plugin**: plugin.json、components 整理、marketplace、dependency 管理
- **CI/CD**: GitHub Actions / GitLab CI/CD / GitHub Enterprise Server / Code Review
- **Collaboration Surface**: Slack / Channels / Remote Control
- **Scheduling & Automation**: Routines / desktop-scheduled-tasks / scheduled-tasks / headless / computer-use
- **運用**: analytics / monitoring / costs / security / legal / ZDR / devcontainer
- **エンタープライズ**: Bedrock / Vertex / Foundry / LLM gateway / network config

### 推奨成果物

- [ ] 個人用 skill/hook/subagent を束ねた plugin 1つを作成
- [ ] plugin を local marketplace に登録
- [ ] GitHub Actions で自動 PR レビューを設定
- [ ] Slack 連携 or Channels で外部イベント受信
- [ ] routines で定期タスクを1つ設定
- [ ] analytics/monitoring で利用状況を確認

### 担当 docs URL チェックリスト

#### プラグイン

- [ ] https://code.claude.com/docs/en/plugins — Create plugins
- [ ] https://code.claude.com/docs/en/plugins-reference — Plugins reference
- [ ] https://code.claude.com/docs/en/plugin-marketplaces — Create and distribute a plugin marketplace
- [ ] https://code.claude.com/docs/en/discover-plugins — Discover and install prebuilt plugins
- [ ] https://code.claude.com/docs/en/plugin-dependencies — Constrain plugin dependency versions

#### CI/CD

- [ ] https://code.claude.com/docs/en/github-actions — Claude Code GitHub Actions
- [ ] https://code.claude.com/docs/en/gitlab-ci-cd — Claude Code GitLab CI/CD
- [ ] https://code.claude.com/docs/en/github-enterprise-server — Claude Code with GitHub Enterprise Server
- [ ] https://code.claude.com/docs/en/code-review — Code Review

#### Collaboration / Surface

- [ ] https://code.claude.com/docs/en/slack — Claude Code in Slack
- [ ] https://code.claude.com/docs/en/channels — Push events into a running session with channels
- [ ] https://code.claude.com/docs/en/channels-reference — Channels reference
- [ ] https://code.claude.com/docs/en/remote-control — Continue local sessions from any device with Remote Control

#### Scheduling & Automation

- [ ] https://code.claude.com/docs/en/routines — Automate work with routines
- [ ] https://code.claude.com/docs/en/desktop-scheduled-tasks — Schedule recurring tasks in Desktop
- [ ] https://code.claude.com/docs/en/scheduled-tasks — Run prompts on a schedule
- [ ] https://code.claude.com/docs/en/headless — Run Claude Code programmatically
- [ ] https://code.claude.com/docs/en/computer-use — Let Claude use your computer from the CLI

#### 運用・セキュリティ

- [ ] https://code.claude.com/docs/en/analytics — Track team usage with analytics
- [ ] https://code.claude.com/docs/en/monitoring-usage — Monitoring
- [ ] https://code.claude.com/docs/en/costs — Manage costs effectively
- [ ] https://code.claude.com/docs/en/security — Security
- [ ] https://code.claude.com/docs/en/legal-and-compliance — Legal and compliance
- [ ] https://code.claude.com/docs/en/zero-data-retention — Zero data retention
- [ ] https://code.claude.com/docs/en/devcontainer — Development containers

#### エンタープライズ配備

- [ ] https://code.claude.com/docs/en/third-party-integrations — Enterprise deployment overview
- [ ] https://code.claude.com/docs/en/amazon-bedrock — Claude Code on Amazon Bedrock
- [ ] https://code.claude.com/docs/en/google-vertex-ai — Claude Code on Google Vertex AI
- [ ] https://code.claude.com/docs/en/microsoft-foundry — Claude Code on Microsoft Foundry
- [ ] https://code.claude.com/docs/en/llm-gateway — LLM gateway configuration
- [ ] https://code.claude.com/docs/en/network-config — Enterprise network configuration

---

## Phase 5: 高度な統合 — MCP / Agent SDK（32ページ）

### ゴール

MCP で外部システムと連携し、Agent SDK で完全に独立したカスタムエージェントを構築する。Claude Code の先にある世界を体験する。

### 新規概念

- **MCP (Model Context Protocol)**: 外部ツール連携、stdio/HTTP/SSE
- **自作 MCP サーバー**: 自社システムとの統合
- **Agent SDK**: Python / TypeScript による独立エージェント構築
- **Custom tools**: Agent SDK でのツール定義
- **Sessions / streaming / structured outputs / todo tracking**: 高度な制御
- **Observability / hosting / secure deployment**: 本番運用
- **ultraplan / ultrareview**: クラウド連携の高度機能

### 推奨成果物

- [ ] MCP サーバーを1〜2個 Claude Code に接続（例: GitHub, Notion）
- [ ] 自作 MCP サーバーを1つ実装（stdio or HTTP）
- [ ] Agent SDK（Python or TS）でカスタムエージェントを1つ作成
- [ ] custom tools / MCP / sessions を組み合わせた自律ワークフロー
- [ ] ultraplan / ultrareview を試す

### 担当 docs URL チェックリスト

#### MCP

- [ ] https://code.claude.com/docs/en/mcp — Connect Claude Code to tools via MCP

#### クラウド高度機能

- [ ] https://code.claude.com/docs/en/ultraplan — Plan in the cloud with ultraplan
- [ ] https://code.claude.com/docs/en/ultrareview — Find bugs with ultrareview

#### Agent SDK — 概要・導入

- [ ] https://code.claude.com/docs/en/agent-sdk/overview — Agent SDK overview
- [ ] https://code.claude.com/docs/en/agent-sdk/quickstart — Quickstart
- [ ] https://code.claude.com/docs/en/agent-sdk/migration-guide — Migrate to Claude Agent SDK
- [ ] https://code.claude.com/docs/en/agent-sdk/agent-loop — How the agent loop works
- [ ] https://code.claude.com/docs/en/agent-sdk/claude-code-features — Use Claude Code features in the SDK

#### Agent SDK — リファレンス

- [ ] https://code.claude.com/docs/en/agent-sdk/python — Agent SDK reference - Python
- [ ] https://code.claude.com/docs/en/agent-sdk/typescript — Agent SDK reference - TypeScript
- [ ] https://code.claude.com/docs/en/agent-sdk/typescript-v2-preview — TypeScript SDK V2 interface (preview)

#### Agent SDK — ツール・拡張

- [ ] https://code.claude.com/docs/en/agent-sdk/custom-tools — Give Claude custom tools
- [ ] https://code.claude.com/docs/en/agent-sdk/tool-search — Scale to many tools with tool search
- [ ] https://code.claude.com/docs/en/agent-sdk/mcp — Connect to external tools with MCP
- [ ] https://code.claude.com/docs/en/agent-sdk/plugins — Plugins in the SDK
- [ ] https://code.claude.com/docs/en/agent-sdk/skills — Agent Skills in the SDK
- [ ] https://code.claude.com/docs/en/agent-sdk/slash-commands — Slash Commands in the SDK
- [ ] https://code.claude.com/docs/en/agent-sdk/subagents — Subagents in the SDK
- [ ] https://code.claude.com/docs/en/agent-sdk/hooks — Intercept and control agent behavior with hooks

#### Agent SDK — 実行制御

- [ ] https://code.claude.com/docs/en/agent-sdk/sessions — Work with sessions
- [ ] https://code.claude.com/docs/en/agent-sdk/streaming-output — Stream responses in real-time
- [ ] https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode — Streaming Input
- [ ] https://code.claude.com/docs/en/agent-sdk/structured-outputs — Get structured output from agents
- [ ] https://code.claude.com/docs/en/agent-sdk/todo-tracking — Todo Lists
- [ ] https://code.claude.com/docs/en/agent-sdk/user-input — Handle approvals and user input
- [ ] https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts — Modifying system prompts
- [ ] https://code.claude.com/docs/en/agent-sdk/file-checkpointing — Rewind file changes with checkpointing
- [ ] https://code.claude.com/docs/en/agent-sdk/permissions — Configure permissions

#### Agent SDK — 運用

- [ ] https://code.claude.com/docs/en/agent-sdk/hosting — Hosting the Agent SDK
- [ ] https://code.claude.com/docs/en/agent-sdk/secure-deployment — Securely deploying AI agents
- [ ] https://code.claude.com/docs/en/agent-sdk/observability — Observability with OpenTelemetry
- [ ] https://code.claude.com/docs/en/agent-sdk/cost-tracking — Track cost and usage

---

## Phase 間の依存関係

```
Phase 1 (基礎)
    ↓
Phase 2 (設定・記憶)           ← Phase 1 の CLI/CLAUDE.md 基礎を前提
    ↓
Phase 3 (拡張: Skills/Hooks)   ← Phase 2 の .claude/ 構造を前提
    ↓
Phase 4 (プラグイン化・配布)   ← Phase 3 の個別コンポーネントを束ねる
    ↓
Phase 5 (MCP / Agent SDK)     ← Claude Code の外の世界、独立エージェント
```

## 最終到達点

Phase 5 完了時点で到達する状態:

- 公式 docs 全114ページを通読し、どこに何が書いてあるか把握している
- 個人ワークフローを plugin にまとめ、チームで共有できる
- 自作 MCP サーバーで外部システムと統合できる
- Agent SDK で Claude Code から独立したカスタムエージェントを作れる

---

_Generated at 2026-04-17 by learning-flow plugin_
