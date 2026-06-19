---
title: Code intelligence = LSP plugin = IDE の F12 ジャンプ等を Claude が使えるようにする仕組み
phase: 3
topic: 01-features-overview
created: 2026-06-12
---

# Code intelligence と LSP — IDE の F12 ジャンプ等を Claude に与える拡張機能

出典:
- [Extend Claude Code](https://code.claude.com/docs/en/features-overview) — 概要表 + Code intelligence の説明
- [Tools reference - LSP tool behavior](https://code.claude.com/docs/en/tools-reference#lsp-tool-behavior)
- [Discover plugins - code intelligence](https://code.claude.com/docs/en/discover-plugins#code-intelligence)

---

## 質問

> Code intelligence プラグインというのは何?

## 結論

**LSP (Language Server Protocol) 経由で Claude にシンボル単位の情報を提供する拡張**。
平たく言うと **「VS Code / JetBrains の F12 ジャンプや型ホバーが Claude にも使えるようになる」**。

## 仕組み

```
[Claude / IDE]  ←→ [LSP プロトコル] ←→ [TypeScript Server / pyright / rust-analyzer / ...]
                                          ↑
                                  言語ごとの解析エンジン
                                  （AST 解析・型推論・symbol index）
```

LSP は VS Code 等の IDE が言語ごとの解析サーバーと通信するための共通プロトコル。
**既に IDE 用に存在する言語サーバーを再利用**できる。

## Claude が LSP 経由で得られる 4 つの能力

| 能力 | 何ができる | IDE での相当機能 |
|---|---|---|
| **Go to definition** | symbol 定義場所を直接ジャンプ | F12 |
| **Find references** | symbol 使用箇所をリスト | Shift+F12 |
| **Type info** | 変数 / 関数の型・docstring 取得 | ホバー / Ctrl+K I |
| **Real-time diagnostics** | 編集直後に型エラー / lint warning | Problems パネル |

加えて Symbol Search（IDE の Ctrl+T）、（対応してれば）Rename、Code Actions も。

## デフォルトでは無効 — plugin 必須

Claude Code 自体は **LSP に話しかける口（LSP tool）だけ**を内蔵。
**実際の言語サーバー接続は plugin 経由で配布**:

- TypeScript / JavaScript 用 plugin
- Python 用 plugin (pyright 接続)
- Rust 用 plugin (rust-analyzer 接続)
- Go 用 plugin (gopls 接続)
- 等々…

→ **言語ごとに別 plugin を入れる**構造。
→ Plugin list: https://code.claude.com/docs/en/discover-plugins#code-intelligence

## Without vs With LSP の比較

### Without LSP（grep ベース）

```
[Claude] "getUser 関数の使用箇所を全部教えて"
   ↓
[Grep] grep -r "getUser" .
   ↓
1000 行のマッチ（コメント・文字列リテラル・別の getUser も含む）
   ↓
[Claude] 50 ファイル Read して文脈確認...
   ↓
context window が膨らむ
```

### With LSP（indexed answer）

```
[Claude] "getUser 関数の使用箇所を全部教えて"
   ↓
[LSP] Find references on getUser
   ↓
正確な 12 箇所（型情報込み）
   ↓
1 レスポンスで終わり
```

## コスト面の意外な事実 — context は逆に減る

直感に反するが公式 docs 明記:

> Symbol lookups often replace broad file reads, so **net context use can go down**.

Plugin 入れると context が**増える**のではなく、**減る**。
「全部読む」が「indexed answer」に置き換わるから。

## 使うべきタイミング

| 状況 | LSP 入れるべき? |
|---|---|
| typed 言語 (TypeScript / Rust / Java / Go / Swift / Kotlin) | ✅ 強く推奨 |
| 大規模リポ（数千〜数万ファイル） | ✅ grep が遅い・不正確 |
| Python（型注釈ある） | ✅ pyright で型エラー検出も |
| JavaScript（型なし）| 半々（symbol navigation は効くが型情報は薄い） |
| シェルスクリプト・設定ファイル中心 | ❌ 効果薄い |
| 単一ファイル・小規模 | ❌ 過剰 |

## Phase 1 / Task 1 との接続

Phase 1 / Task 1 の Q3「plugin が必要な tool カテゴリ」の正解は **Code intelligence**。
Web (WebFetch, WebSearch) は標準装備、**Code intelligence のみ plugin 経由**という設計思想がここで明確になる。

`tools-reference` には他の組み込みツール（Read, Edit, Bash, Glob, Grep, WebFetch 等）が並ぶが、LSP 系だけは「枠だけ用意、実体は plugin」という扱い。理由は「言語ごとに固有の依存（言語サーバーバイナリ）が必要 → コア CLI に bundle すると肥大化」。

## 関連メモ

- [features-overview index.md](../index.md)
- Phase 1 / Task 1 復習キュー Q3（plugin が必要な tool カテゴリ）
- [tools-reference](https://code.claude.com/docs/en/tools-reference) — Phase 3 / Topic 07 で扱う
