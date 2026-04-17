# claude-code-practice — 学習コンテキスト

## プロジェクトの目的

Claude Code の学習が主目的。公式 docs/reference（全114ページ）を5 Phase に分けて網羅的に読みつつ、各 Phase で実際の成果物（CLAUDE.md / skill / hook / subagent / plugin / MCP server / Agent SDK agent 等）を構築する。コード完成が目的ではなく、**公式 docs を根拠に Claude Code を理解すること**が最優先。

## Phase構成（全5 Phase）

各 Phase で新規概念を段階的に導入し、公式 docs を根拠にじっくり学ぶ。

| Phase | テーマ | 新規概念（主要） | 担当 docs ページ数 | 状態 |
|---|---|---|---|---|
| 1 | 基礎 — Claude Code の世界観を掴む | agentic loop / CLI基本 / 各Surface (CLI/VS Code/Desktop/Web/JetBrains/Chrome) / context window / checkpointing | 32 | 🔜 準備中 |
| 2 | カスタマイズ — 記憶・設定・権限 | CLAUDE.md / auto memory / settings.json / permissions / output styles / statusline / sandboxing | 12 | ⏳ 未着手 |
| 3 | 拡張機能 — Skills / Hooks / Subagents | skills / hooks (Pre/Post/Stop/SessionStart) / subagents / agent teams / tools reference | 7 | ⏳ 未着手 |
| 4 | プラグイン化・配布・チーム連携・運用 | plugins / marketplace / GitHub Actions / GitLab CI / channels / routines / enterprise (Bedrock/Vertex/Foundry) / monitoring | 31 | ⏳ 未着手 |
| 5 | 高度な統合 — MCP / Agent SDK | MCP接続 / 自作MCPサーバー / Agent SDK (Python/TS) / custom tools / sessions / structured outputs / ultraplan / ultrareview | 32 | ⏳ 未着手 |

**合計: 114ページ（公式 docs llms.txt の全エントリ）**

設計書: `plans/claude-code-learning-phases-design.md`

## 現在の進捗

**Phase 1** の学習準備段階。

## タスクごとの学習フロー

各タスクで以下のサイクルを回す:

1. **公式 docs を読む**: 該当 Phase の担当ページから、今回の Task に関連するページを読む
2. **解説**: 何をするか、なぜそうするか、docs の内容を要約しつつ設計思想やベストプラクティスとの関連を示す
3. **確認**: ユーザーが理解したか確認、質問があれば回答
4. **実装**: ユーザーの「進めて」を受けてから実行
5. **振り返り**: 生成されたコードの解説、docs の該当箇所とのリンク、疑問の議論
6. **docs チェック**: 読んだページを Phase 設計書の URL チェックリストでマーク
7. **次へ**: ユーザーの「次に進む」を受けてから次のタスクへ

**重要: 勝手に進めない。** 各ステップでユーザーの確認を取る。

## ドキュメント構成ルール

### 学習ノート

```
docs/learning/phase{N}/task{M}/
├── main.md                     ← 目次 + 振り返り + 読んだ docs URL
├── 01-{トピック名}.md           ← トピック別の解説・Q&A
├── 02-{トピック名}.md
└── reference/
    ├── {テーマ}-{詳細}.md       ← 公式 docs + Web検索の詳細リファレンス
    └── ...
```

### ルール

- **大枠ごとに md を分ける**（1つの md にすべてを入れない）
- **公式 docs の URL を必ず引用**: 各トピック md の冒頭に参照したページを列挙
- **reference は深掘り用**: docs からの引用、詳細な比較表、具体例、ソース付き
- **トピック md は Q&A 中心**: 解説 + 質疑応答の記録
- **main.md は目次**: トピック md と reference へのリンク + 振り返り + 今回読んだ docs URL
- **コードにもコメント**: 該当コードの上に1行の説明 + docs へのリンク

## 関連ドキュメントの場所

| ドキュメント | パス |
|---|---|
| Phase 設計書（全 114 URL チェックリスト付き） | `plans/claude-code-learning-phases-design.md` |
| 全ページインデックス（公式） | https://code.claude.com/docs/llms.txt |

---

_Generated at 2026-04-17 by learning-flow plugin_
