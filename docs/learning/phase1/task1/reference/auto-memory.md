# auto memory — Claude が自分で書き貯めるメモ帳

出典: [memory#auto-memory](https://code.claude.com/docs/en/memory#auto-memory)

**Phase 2 の本題。Task 1 の overview / best-practices で「auto memory」が頻出したため先取り調査。**

## 本質

**Claude 自身が書いて貯めていくメモ帳**。ユーザーの修正、好み、発見、プロジェクトの gotcha を、Claude が「将来役立つ」と判断した時だけ自動保存する。

## CLAUDE.md との違い

|  | CLAUDE.md | auto memory |
|---|---|---|
| 書く人 | あなた | Claude |
| 中身 | ルール・指示 | 学習・パターン・発見 |
| スコープ | project / user / org | **git repo 単位**（worktree 横断で共有） |
| ロード | 毎セッション全部 | `MEMORY.md` の先頭 200行 or 25KB のみ、他は on-demand |
| 用途 | コード規約・ビルド手順・アーキテクチャ | 気づいたビルドコマンド・デバッグ知見・好み |

→ **"再説明したくないもの"** は CLAUDE.md、**"Claude が学んだ事実"** は auto memory。

## 保存構造

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 目次。毎セッション先頭 200行/25KB 読む
├── debugging.md       # トピック別、on-demand
├── api-conventions.md
└── ...
```

- **machine-local**（マシン間で共有されない）
- git repo 単位（同一 repo の worktree 全てで共有）
- 全部ただの markdown。いつでも直接編集可

## 書き込みタイミング

Claude が「将来の会話で役立つ」と判断した時だけ。毎セッション書くわけではない。

例:
- 「このリポのテストは `pnpm test:unit`」と教えた → memo
- 「always use X, not Y」 → memo
- 「API テストには Redis が必要」 → memo
- 単なるやりとり → memo しない

## 操作

| 操作 | 方法 |
|---|---|
| 中身を見る | `/memory` で一覧＋開く |
| ON/OFF | `/memory` のトグル / settings で `autoMemoryEnabled: false` / env で `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` |
| 編集・削除 | `/memory` から選んでエディタ、or 直接ファイル編集 |
| 保存先変更 | `autoMemoryDirectory` 設定（project settings では不可、policy/local/user のみ） |
| バージョン要件 | Claude Code **v2.1.59 以降** |

## 明示的に「記憶して/CLAUDE.md に書いて」の振り分け

| 言い方 | 保存先 |
|---|---|
| 「覚えて」「remember that...」 | **auto memory** に入る |
| 「CLAUDE.md に追加して」 | **CLAUDE.md** に書く |
| 「always use pnpm, not npm」系のルール | auto memory（後で CLAUDE.md に昇格可） |

## 仕様の細かい話

- MEMORY.md が 200 行超でも、**200 行/25KB 以降はセッション開始時ロードされない**。Claude は簡潔に保って詳細を topic ファイルに移す運用
- この 200/25KB 制限は `MEMORY.md` のみ。CLAUDE.md は全文ロード（ただし短い方が adherence 高い）
- topic ファイル（`debugging.md` 等）は起動時ロードされず、必要時に Claude が Read tool で読む
- `/compact` 後、CLAUDE.md は再注入されるが nested CLAUDE.md や memory topic ファイルは on-demand

## プライバシー / 監査

- 全部 plain markdown → いつでも人間がレビュー・編集・削除できる
- "Writing memory" / "Recalled memory" の UI 表示で更新タイミングが分かる
- マシン間で共有されないので、機密情報が意図せず他環境に飛ぶことはない

## 学習プロジェクトでの接続

- Phase 2 で **memory.md の完全読解**と、実際に `/memory` を使いこなす演習
- Phase 3 の subagent でも独自 memory を持てる（→ `subagent-memory.md` 参照）
