# 07. キーバインドのカスタマイズ

> 出典: [Customize keyboard shortcuts](https://code.claude.com/docs/en/keybindings)（閲覧日 2026-05-08）
> このノートは公式ドキュメント「Customize keyboard shortcuts」の構造を起点に Claude が自動生成した教材です。

## 概要

Claude Code v2.1.18 以降、`~/.claude/keybindings.json` でキーバインドを**ユーザー設定で上書き**できる。`/keybindings` でファイルを開く / 作成。**変更は自動検知され再起動不要**。

バインディングは「**コンテキスト** + **キーストローク** → **アクション**」の対応関係。コンテキスト（Chat / Autocomplete / Confirmation 等）を絞れるので、同じキーをコンテキストごとに別意味にできる。

## 公式docsに沿った解説

### 設定ファイル全体構造

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null
      }
    }
  ]
}
```

| フィールド | 役割 |
|---|---|
| `$schema` | エディタ補完用の JSON Schema URL |
| `$docs` | ドキュメント URL（メモ用） |
| `bindings` | コンテキストごとのバインディングブロックの配列 |

各ブロックは `context` と `bindings`（キー → アクション名）。**`null` を指定するとアンバインド**。

### Contexts（コンテキスト一覧）

| Context | 説明 |
|---|---|
| `Global` | アプリ全体 |
| `Chat` | 主入力欄 |
| `Autocomplete` | autocomplete メニューが開いている時 |
| `Settings` | settings メニュー |
| `Confirmation` | permission / confirmation dialog |
| `Tabs` | タブナビゲーション |
| `Help` | ヘルプメニュー |
| `Transcript` | transcript viewer |
| `HistorySearch` | `Ctrl+R` の履歴検索 |
| `Task` | バックグラウンドタスク実行中 |
| `ThemePicker` | theme picker |
| `Attachments` | select dialog 内の画像添付ナビ |
| `Footer` | フッターのインジケータ（task / team / diff 等） |
| `MessageSelector` | rewind / summarize 用のメッセージ選択 |
| `DiffDialog` | diff viewer |
| `ModelPicker` | model picker の effort level |
| `Select` | 一般 select / list コンポーネント |
| `Plugin` | plugin dialog |
| `Scroll` | fullscreen 時の会話スクロール / テキスト選択 |
| `Doctor` | `/doctor` 診断画面 |

### Available actions（主要アクション抜粋）

`namespace:action` 形式（`chat:submit`, `app:toggleTodos` 等）。

#### App（Global）

| Action | 既定 | 役割 |
|---|---|---|
| `app:interrupt` | Ctrl+C | キャンセル |
| `app:exit` | Ctrl+D | 終了 |
| `app:redraw` | (unbound) | 画面再描画 |
| `app:toggleTodos` | Ctrl+T | task list 表示 toggle |
| `app:toggleTranscript` | Ctrl+O | transcript viewer toggle |

#### History

| Action | 既定 | 役割 |
|---|---|---|
| `history:search` | Ctrl+R | reverse search |
| `history:previous` | ↑ | 前のエントリ |
| `history:next` | ↓ | 次のエントリ |

#### Chat（主要のみ抜粋）

| Action | 既定 | 役割 |
|---|---|---|
| `chat:cancel` | Esc | 入力キャンセル |
| `chat:clearInput` | Ctrl+L | 入力残して画面再描画。fullscreen で 2 秒以内 2 度押すと `/clear` |
| `chat:clearScreen` | Cmd+K | fullscreen 専用の clear |
| `chat:killAgents` | Ctrl+X Ctrl+K | bg agent を全 kill |
| `chat:cycleMode` | Shift+Tab\* | permission mode 循環 |
| `chat:modelPicker` | Meta+P | モデル選択 |
| `chat:fastMode` | Meta+O | fast mode toggle |
| `chat:thinkingToggle` | Meta+T | extended thinking toggle |
| `chat:submit` | Enter | 送信 |
| `chat:newline` | Ctrl+J | 改行（送信せず） |
| `chat:undo` | Ctrl+_, Ctrl+Shift+- | 直前操作の取消 |
| `chat:externalEditor` | Ctrl+G, Ctrl+X Ctrl+E | 外部エディタで編集 |
| `chat:stash` | Ctrl+S | プロンプトを stash |
| `chat:imagePaste` | Ctrl+V (Win は Alt+V) | 画像ペースト |

\* Windows で VT モードが無い場合（Node <24.2.0 / <22.17.0、Bun <1.2.23）は `Meta+M` がデフォルト。

#### その他のコンテキスト（網羅）

- **Autocomplete**: `accept` (Tab) / `dismiss` (Esc) / `previous` (↑) / `next` (↓)
- **Confirmation**: `yes` (Y, Enter) / `no` (N, Esc) / `nextField` (Tab) / `toggle` (Space) / `cycleMode` (Shift+Tab) / `toggleExplanation` (Ctrl+E)
- **Permission**: `toggleDebug` (Ctrl+D)
- **Transcript**: `toggleShowAll` (Ctrl+E) / `exit` (q, Ctrl+C, Esc)
- **HistorySearch**: `next` (Ctrl+R) / `accept` (Esc, Tab) / `cancel` (Ctrl+C) / `execute` (Enter) / `cycleScope` (Ctrl+S)
- **Task**: `background` (Ctrl+B)
- **Theme**: `toggleSyntaxHighlighting` (Ctrl+T)
- **Help**: `dismiss` (Esc)
- **Tabs**: `next` (Tab, →) / `previous` (Shift+Tab, ←)
- **Attachments**: `next` (→) / `previous` (←) / `remove` (Backspace, Delete) / `exit` (↓, Esc)
- **Footer**: `next` (→) / `previous` (←) / `up` (↑) / `down` (↓) / `openSelected` (Enter) / `clearSelection` (Esc)
- **MessageSelector**: vi 風の `j`/`k` / `Ctrl+P`/`Ctrl+N` / `Shift+J`/`Shift+K` 等
- **Diff**: `dismiss` (Esc) / `previousSource` (←) / `nextSource` (→) / `previousFile` (↑) / `nextFile` (↓) / `viewDetails` (Enter) / `back`
- **ModelPicker**: `decreaseEffort` (←) / `increaseEffort` (→)
- **Select**: `next` (↓, J, Ctrl+N) / `previous` (↑, K, Ctrl+P) / `accept` (Enter) / `cancel` (Esc)
- **Plugin**: `toggle` (Space) / `install` (I) / `favorite` (F)
- **Settings**: `search` (/) / `retry` (R) / `close` (Enter)
- **Doctor**: `fix` (F)
- **Voice**（Chat 内）: `voice:pushToTalk` (Space)
- **Scroll**（fullscreen 内）: `scroll:lineUp/Down` / `scroll:pageUp/Down` / `scroll:top` (Ctrl+Home) / `scroll:bottom` (Ctrl+End) / `selection:copy` (Ctrl+Shift+C / Cmd+C) / `selection:extendLeft/Right/Up/Down` (Shift+矢印) / `selection:extendLineStart/End` (Shift+Home/End)

### Keystroke syntax（キー記法）

#### モディファイア

| 記法 | キー |
|---|---|
| `ctrl` / `control` | Control |
| `shift` | Shift |
| `alt` / `opt` / `option` / `meta` | Alt（Win/Linux）/ Option（macOS） |
| `cmd` / `command` / `super` / `win` | macOS Command / Win キー / Linux Super |

`cmd` グループは **Super 修飾を report する terminal でのみ**検知される（Kitty キーボードプロトコルや xterm の `modifyOtherKeys` モード対応など）。一般的な terminal は送らないので、**全環境で動かしたいなら `ctrl` か `meta`** を使う。

例:
```
ctrl+k          Ctrl + K
shift+tab       Shift + Tab
meta+p          macOS は Option+P, それ以外は Alt+P
ctrl+shift+c    複数モディファイア
```

#### 大文字の扱い

- 単独の大文字 `K` は `shift+k` と等価（vim 風バインドで便利）
- モディファイア付きの大文字（`ctrl+K`）は**スタイル上の差**だけで Shift は含意しない（`ctrl+k` と等価）

#### Chord（連打）

スペース区切りで連続キーストローク:

```
ctrl+k ctrl+s   Ctrl+K を押して離してから Ctrl+S
```

#### Special keys

- `escape` / `esc`
- `enter` / `return`
- `tab`
- `space`
- `up` / `down` / `left` / `right`
- `backspace` / `delete`

### Unbind default shortcuts

`null` を割り当てるとアンバインド:

```json
{
  "bindings": [
    { "context": "Chat", "bindings": { "ctrl+s": null } }
  ]
}
```

chord も同様に `null` で外せる。**同じプレフィックスの chord を全部外せばそのプレフィックスを単独キーにバインド可能**:

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+x ctrl+k": null,
        "ctrl+x ctrl+e": null,
        "ctrl+x": "chat:newline"
      }
    }
  ]
}
```

逆に「一部の chord だけ外し、他は残す」と、プレフィックスを押した時点で chord-wait モードに入る挙動は維持される。

### Reserved shortcuts（再バインド不可）

| ショートカット | 理由 |
|---|---|
| `Ctrl+C` | ハードコードされた interrupt / cancel |
| `Ctrl+D` | ハードコードされた exit |
| `Ctrl+M` | terminal 上で Enter と同じ（CR） |
| Caps Lock | terminal アプリには届かない |

### Terminal conflicts（既知の衝突）

| ショートカット | 衝突 |
|---|---|
| `Ctrl+B` | tmux のプレフィックス（**2 回押し**で透過） |
| `Ctrl+A` | GNU screen のプレフィックス |
| `Ctrl+Z` | UNIX プロセス suspend (SIGTSTP) |

### Vim mode との相互作用

`/config` → Editor mode で vim mode を有効にしている時:

- **Vim mode はテキスト入力レベル**（カーソル / モード / motion）を扱う
- **Keybindings はコンポーネントレベル**（task list toggle / submit 等）を扱う
- 両者は**独立**に動く
- vim mode の `Esc` は INSERT → NORMAL の切替で、`chat:cancel` は走らない
- `Ctrl+key` 系は vim mode を**貫通**して keybindings に届く
- vim NORMAL での `?` は help（vim 挙動）

### Validation

`/doctor` でキーバインド警告を確認できる。検出される問題:

- パースエラー（不正な JSON）
- 不正なコンテキスト名
- Reserved shortcut との衝突
- Terminal multiplexer との衝突
- 同コンテキスト内の重複

## 重要ポイント

- **コンテキストごとにバインドを分けられる**ので、Chat の `Ctrl+S` と Settings の `Ctrl+S` を別アクションにできる
- **`null` でアンバインド**できることが重要。デフォルトで邪魔なバインドを外せる
- `cmd` 系修飾は terminal 依存。**ポータブルにしたいなら `ctrl` か `meta`**
- 大文字記法は単独 (`K` = `Shift+K`) とモディファイア付き (`ctrl+K` = `ctrl+k`) で**意味が違う**
- Chord はプレフィックスを `null` 化することで「単独キーバインド」に転用できる（高度な技）
- Reserved shortcut（`Ctrl+C`, `Ctrl+D`, `Ctrl+M`, Caps Lock）は**絶対に再バインド不可**
- `Ctrl+B` を tmux で使う時の「2 回押し」は仕様。回避は keybindings ではなく tmux 設定側で対応する
- 設定ファイルは**ホットリロード**。再起動なしで反映される

## コード例 / 図

### よくあるカスタマイズ例

#### `Ctrl+E` を Chat の外部エディタ起動に
```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor"
      }
    }
  ]
}
```

#### vim 派が `J` / `K` を MessageSelector で使う（既定で対応済み）
```json
{
  "bindings": [
    {
      "context": "MessageSelector",
      "bindings": {
        "j": "messageSelector:down",
        "k": "messageSelector:up"
      }
    }
  ]
}
```

#### `Ctrl+X` プレフィックスを潰して単独キーとして使う
```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+x ctrl+k": null,
        "ctrl+x ctrl+e": null,
        "ctrl+x": "chat:newline"
      }
    }
  ]
}
```

### 確認手順

```text
1. /keybindings で設定ファイルを開く
2. 編集して保存
3. 自動検知で即反映
4. /doctor で警告チェック
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 関連 docs: [Interactive mode](https://code.claude.com/docs/en/interactive-mode) / [Voice dictation](https://code.claude.com/docs/en/voice-dictation) / [Fullscreen rendering](https://code.claude.com/docs/en/fullscreen) / [Settings](https://code.claude.com/docs/en/settings)

---

_Auto-generated at 2026-05-08 via /learning-flow:material（公式docs駆動）_
