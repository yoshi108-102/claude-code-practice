# 05. コマンド履歴とシェル統合

> 出典:
> - [Interactive mode > Command history](https://code.claude.com/docs/en/interactive-mode#command-history)
> - [Interactive mode > Background bash commands](https://code.claude.com/docs/en/interactive-mode#background-bash-commands)
> - [Interactive mode > Shell mode with `!` prefix](https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix)
> （閲覧日 2026-05-08）
>
> このノートは「Interactive mode」のコマンド履歴 / バックグラウンド bash / shell mode 節を起点に Claude が自動生成した教材です。

## 概要

Claude Code は対話セッション内で:

1. **入力履歴**を保持し、矢印 / `Ctrl+R` で過去のプロンプトに戻れる
2. bash コマンドを**バックグラウンド実行**でき、Claude は実行中も次の指示を受けられる
3. `!` プレフィックスで **shell mode** に入り、Claude を介さずに直接シェルコマンドを叩ける

これらは「対話の流れを止めずに作業する」ための仕組み。

## 公式docsに沿った解説

### Command history（コマンド履歴）

- **作業ディレクトリごと**に履歴を保持
- `/clear` で**新会話を開始した時点でリセット**（前会話自体は `/resume` で復活可能）
- `↑` / `↓` で履歴ナビ（前項のキーボードショートカット参照）
- **history expansion (`!`) はデフォルト OFF**（bash のような `!!` は効かない）

### Reverse search（`Ctrl+R`）

`Ctrl+R` で対話的な reverse history search に入る。

| 操作 | 動作 |
|---|---|
| `Ctrl+R` | reverse history search 起動 |
| 文字入力 | クエリ。マッチ部分はハイライト |
| `Ctrl+R`（再度） | より古いマッチへ |
| `Ctrl+S` | スコープを循環: **all projects（既定）→ this session → this project** |
| `Tab` / `Esc` | 現在のマッチを採用してそのまま編集 |
| `Enter` | 採用 + 即実行 |
| `Ctrl+C` | キャンセルして元の入力に戻す |
| 空クエリで `Backspace` | キャンセル |

スコープ循環のおかげで「この session に閉じた検索」「プロジェクト単位の検索」「全プロジェクト横断の検索」を切替できる。

### Background bash commands（バックグラウンド bash）

長時間 bash を**非同期で**走らせ、即時に background task ID を返して会話を続けられる仕組み。

#### 仕組み

- 出力はファイルに書かれ、Claude は `Read` tool で取り出せる
- background task は**ユニーク ID**で管理
- セッション終了時に**自動クリーンアップ**
- 出力が **5GB を超えると自動終了**（stderr に理由が書かれる）

#### バックグラウンド化の方法

1. プロンプトで「バックグラウンドで走らせて」と Claude に指示
2. 通常の `Bash` 実行中に **`Ctrl+B`** を押して background へ送る（tmux ユーザーは tmux のプレフィックスとぶつかるので 2 回押し）

#### 全停止
- `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` で background 機能を完全に切れる

#### 典型的な利用例

| 用途 | 例 |
|---|---|
| ビルドツール | webpack, vite, make |
| パッケージマネージャ | npm, yarn, pnpm install |
| テストランナー | jest, pytest |
| 開発サーバー | `npm run dev` |
| 長時間プロセス | docker, terraform |

### Shell mode（`!` プレフィックス）

入力を `!` で始めると Claude を**介さずに**シェルコマンドが直接実行される。

```bash
! npm test
! git status
! ls -la
```

#### Shell mode の特徴

- コマンドと出力を**会話コンテキストに追加**（後で Claude が参照できる）
- リアルタイム進捗 / 出力を表示
- 同じく `Ctrl+B` でバックグラウンド化可能
- **Claude の解釈や承認は経由しない**（permission 不要）
- **history-based autocomplete**: 部分入力 + `Tab` で**現プロジェクトの過去 `!` コマンド**から補完
- **抜け方**: 空入力で `Escape` / `Backspace` / `Ctrl+U`
- `!` で始まるテキストを空プロンプトにペーストすると自動的に shell mode に入る

#### 使いどころ

- ファイル一覧 / 簡単な確認 (`ls`, `cat`) を素早く
- Claude にコマンドを指示するより自分で叩いた方が速いケース
- 出力をコンテキストに残して、その後 Claude に解釈させたい時（例: `! git diff` → 「これをレビューして」）

## 重要ポイント

- **`/clear` は履歴を消す**。`/resume` で前会話自体は戻せるが、「現在の入力履歴」はリセットされる
- `Ctrl+R` のスコープ切替（`Ctrl+S`）は地味に強力。「このプロジェクトで前にどう書いたっけ」が刺さる
- **bash バックグラウンド化は `Ctrl+B` の 1 押し**。tmux 使ってるなら 2 押し（tmux のプレフィックスとぶつかる）
- shell mode (`!`) は **permission を経由しない**ので「絶対に自分の意図で叩いている」前提。スクリプト等で発火させると危険
- shell mode の出力もコンテキストを食う。気軽に `! ls` を連発するとコンテキストが膨らむ
- 5GB 出力で background task が自動終了する → 巨大ログを延々吐くプロセスは要注意

## コード例 / 図

### 長時間ビルドを背景化して、終わるまで会話を続ける

```text
ユーザー: npm run build を走らせて、ビルド中はログ調査やってて
Claude: [npm run build を background task として実行開始]
       [task ID = 1234, 出力は ~/.claude/.../build-1234.log]
       並行してログ調査を開始します...
ユーザー: そのバックグラウンド task の最新ログ見せて
Claude: [Read で build-1234.log を tail]
```

### 自分で叩いて、結果を Claude にレビューさせる

```text
! git diff
[diff が表示され、コンテキストに入る]

これ rebase 漏れてないかチェックして
[Claude が直前の diff を見てレビュー]
```

### `Ctrl+R` でスコープ別検索

```text
Ctrl+R         → 検索開始 (all projects)
"deploy"       → "deploy ..." のマッチ
Ctrl+S         → スコープを this session に
Ctrl+S         → さらに this project に
Tab            → 現マッチを採用してプロンプトに戻る
```

### 入力履歴のリセットタイミング

```text
[セッション開始]
プロンプト 1
プロンプト 2
プロンプト 3
/clear         ← 履歴リセット、新会話
プロンプト 4   ← 履歴は 4 だけ
↑              ← プロンプト 4 のみ
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 次の教材: [06-interactive-features.md](06-interactive-features.md) — /btw・タスクリスト・リキャップ等
- 関連 docs: [Environment variables](https://code.claude.com/docs/en/env-vars) / [Bash tool](https://code.claude.com/docs/en/tool-reference)

---

_Auto-generated at 2026-05-08 via /learning-flow:material（公式docs駆動）_
