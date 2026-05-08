# 03. 対話モードのキーボードショートカット

> 出典: [Interactive mode > Keyboard shortcuts](https://code.claude.com/docs/en/interactive-mode#keyboard-shortcuts)（閲覧日 2026-05-08）
> このノートは公式ドキュメント「Interactive mode」のキーボードショートカット節を起点に Claude が自動生成した教材です。

## 概要

対話セッション内で使う**キー操作の一覧**。プラットフォーム / ターミナルでショートカットが微妙に違うので、実機では `?` で利用可能なショートカットを確認できる。

**macOS の前提**: `Alt+B`, `Alt+F`, `Alt+Y`, `Alt+M`, `Alt+P` などの Option/Alt 系は、ターミナルで **Option を Meta キーとして扱う設定**にしないと動かない。

| ターミナル | 設定方法 |
|---|---|
| **iTerm2** | Settings → Profiles → Keys → General → Left/Right Option key を **"Esc+"** |
| **Apple Terminal** | Settings → Profiles → Keyboard → **"Use Option as Meta Key"** にチェック |
| **VS Code** | settings に `"terminal.integrated.macOptionIsMeta": true` を追加 |

## 公式docsに沿った解説

### General controls（汎用操作）

| ショートカット | 役割 |
|---|---|
| `Ctrl+C` | 入力 / 生成のキャンセル |
| `Ctrl+X Ctrl+K` | **全 background agent を kill**（3 秒以内に 2 回押しで確定） |
| `Ctrl+D` | セッション終了（EOF） |
| `Ctrl+G` または `Ctrl+X Ctrl+E` | 既定の外部エディタでプロンプトを編集（`Ctrl+X Ctrl+E` は readline ネイティブ） |
| `Ctrl+L` | 画面再描画（履歴・入力は保持） |
| `Ctrl+O` | **transcript viewer の toggle**（tool 呼び出し詳細、MCP 呼び出しの展開） |
| `Ctrl+R` | コマンド履歴の reverse search |
| `Ctrl+V` / `Cmd+V`（iTerm2）/ `Alt+V`（Windows） | 画像をクリップボードからペースト → `[Image #N]` chip 挿入 |
| `Ctrl+B` | 実行中の bash / agent を**バックグラウンド化**（tmux 利用時は 2 回押し） |
| `Ctrl+T` | task list の表示 toggle |
| `←` / `→` | permission dialog や menu のタブ切替 |
| `↑` / `↓`（または `Ctrl+P`/`Ctrl+N`） | 複数行入力中はカーソル移動。端に達したら**コマンド履歴ナビ**へ切替 |
| `Esc` + `Esc` | **rewind / summarize**（コードか会話、または両方を巻き戻す） |
| `Shift+Tab`（または `Alt+M` 構成によっては） | **permission mode を循環**（default → acceptEdits → plan → 有効な auto / bypassPermissions） |
| `Option+P` (mac) / `Alt+P` (Win/Linux) | **モデル切替**（プロンプトを消さずに） |
| `Option+T` / `Alt+T` | extended thinking の toggle（v2.1.132 以降は macOS で Option=Meta 設定不要） |
| `Option+O` / `Alt+O` | **fast mode** の toggle |

### Text editing（テキスト編集）

readline 風の編集キー。

| ショートカット | 役割 |
|---|---|
| `Ctrl+A` | 行頭へ |
| `Ctrl+E` | 行末へ |
| `Ctrl+K` | カーソルから行末まで削除（kill ring に保持） |
| `Ctrl+U` | カーソルから行頭まで削除（macOS の iTerm2 / Terminal.app は `Cmd+Backspace` も同じ動作） |
| `Ctrl+W` | 直前の単語を削除（Win では `Ctrl+Backspace` も同じ） |
| `Ctrl+Y` | `Ctrl+K` / `Ctrl+U` / `Ctrl+W` で削除した内容をペースト |
| `Alt+Y`（`Ctrl+Y` 直後） | ペースト履歴を循環（macOS は Option=Meta 必要） |
| `Alt+B` | 1 単語戻る |
| `Alt+F` | 1 単語進む |

### Theme and display

| ショートカット | 役割 |
|---|---|
| `Ctrl+T` | `/theme` picker メニュー内でのみ、**コードのシンタックスハイライト toggle** |

### Multiline input（複数行入力）

| 方法 | キー | 備考 |
|---|---|---|
| 確実に効く | `\` + `Enter` | どんなターミナルでも動く |
| Option キー | `Option+Enter` | macOS で Option=Meta 設定が必要 |
| `Shift+Enter` | `Shift+Enter` | iTerm2 / WezTerm / Ghostty / Kitty / Warp / Apple Terminal / Windows Terminal は**標準で動く** |
| Ctrl sequence | `Ctrl+J` | 設定不要でどこでも |
| Paste mode | コードブロックをそのまま貼る | コード / ログ向け |

VS Code / Cursor / Windsurf / Alacritty / Zed では `/terminal-setup` を 1 回流すと `Shift+Enter` のバインディングが入る。

### Quick commands（先頭文字での起動）

| ショートカット | 役割 |
|---|---|
| `/` 先頭 | コマンド / skill 起動 |
| `!` 先頭 | **shell mode** で直接コマンド実行（出力もセッションに入る） |
| `@` | ファイルパス mention の autocomplete を起動 |

### Transcript viewer（`Ctrl+O` で開く）

transcript viewer 内で使うキー。`Ctrl+E` は keybindings の `transcript:toggleShowAll` で再バインド可能。

| ショートカット | 役割 |
|---|---|
| `Ctrl+E` | 全コンテンツ表示 toggle |
| `[` | 会話全体をターミナルのスクロールバックに書き出し（`Cmd+F` / tmux copy mode で検索可能、fullscreen が必要） |
| `v` | `$VISUAL` / `$EDITOR` で会話を一時ファイルとして開く（fullscreen が必要） |
| `q`, `Ctrl+C`, `Esc` | viewer を抜ける（`transcript:exit` で再バインド可能） |

### Voice input（音声入力）

| ショートカット | 役割 |
|---|---|
| `Space` を hold / tap | voice dictation。**Hold モード**は押している間だけ録音、`/voice tap` で**タップトグル**に切り替え（rebindable） |

`/voice` で有効化が必要。

## 重要ポイント

- **`Esc+Esc`** が rewind の入口。checkpoint と組み合わせて、編集前の状態に戻れる（Phase 1 / Task 1 の checkpoint と直結）
- **`Shift+Tab`** で permission mode を循環するのが、安全運用の基本動線。`default` → `plan` でプランニング、結果見て `acceptEdits` や `auto` に上げる、という流れ
- **`Ctrl+B` でバックグラウンド化**できるので、長時間 bash 実行で session が固まらない
- `Ctrl+O` の transcript viewer は MCP 呼び出しなど詳細を見たいときの最重要ショートカット
- `!` (shell mode) と `@` (file mention) は強力。**Phase 2 以降の skill 作成でも頻出**
- macOS でショートカット動かない問題のほとんどは「Option=Meta 設定忘れ」
- `Ctrl+X Ctrl+K` の **2 回押し**で background agent kill。誤爆対策の確認ステップ

## コード例 / 図

### 安全運用のキー操作フロー

```text
ユーザー操作            permission mode    意図
-------------------------------------------------
[Shift+Tab]            default → plan    まず計画を立てさせる
[計画を見て OK]
[Shift+Tab]            plan → acceptEdits ファイル編集まで自動許可
[実装中、`Esc+Esc`]                        ←気になったら巻き戻す
[実装後、`Ctrl+O`]                          ←transcript viewer で詳細確認
[Ctrl+B でテスト走らせる]                  ←長時間処理は背景化
```

### 複数行入力のクロス環境ベストプラクティス

```text
# どこでも動く順
1. Ctrl+J          ← 設定不要、確実
2. \ + Enter       ← 設定不要、確実
3. Shift+Enter     ← 主要ターミナルは標準対応
4. Option+Enter    ← macOS で Option=Meta 設定後
```

### キャンセルと終了の使い分け

```text
Ctrl+C   ← 「今走ってる処理だけ」止める（プロンプトに戻る）
Ctrl+D   ← セッション自体を終了（EOF）
/exit    ← 同上、明示コマンド
Esc      ← 各種ダイアログを閉じる、agentic loop に介入
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 次の教材: [04-vim-editor-mode.md](04-vim-editor-mode.md) — Vim エディタモード
- 関連 docs: [Customize keybindings](https://code.claude.com/docs/en/keybindings) / [Terminal config](https://code.claude.com/docs/en/terminal-config) / [Permission modes](https://code.claude.com/docs/en/permission-modes)

---

_Auto-generated at 2026-05-08 via /learning-flow:material（公式docs駆動）_
