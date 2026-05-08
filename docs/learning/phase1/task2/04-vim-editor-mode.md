# 04. Vim エディタモード

> 出典: [Interactive mode > Vim editor mode](https://code.claude.com/docs/en/interactive-mode#vim-editor-mode)（閲覧日 2026-05-08）
> このノートは公式ドキュメント「Interactive mode」の Vim 節を起点に Claude が自動生成した教材です。

## 概要

入力欄に **vim 風の編集モード**を入れる機能。`/config` → Editor mode で有効化。

**注**: 旧 `/vim` コマンドは v2.1.92 で削除された。今は `/config` から切り替える。

ブロック単位の visual mode（`Ctrl+V`）は**非対応**。それ以外は概ね vim そのまま。

## 公式docsに沿った解説

### モード切替

| コマンド | 動作 | どのモードから |
|---|---|---|
| `Esc` | NORMAL モードへ | INSERT, VISUAL |
| `i` | カーソル位置で挿入 | NORMAL |
| `I` | 行頭で挿入 | NORMAL |
| `a` | カーソル後ろで挿入 | NORMAL |
| `A` | 行末で挿入 | NORMAL |
| `o` | 下に新規行を開いて挿入 | NORMAL |
| `O` | 上に新規行を開いて挿入 | NORMAL |
| `v` | 文字単位 visual 選択開始 | NORMAL |
| `V` | 行単位 visual 選択開始 | NORMAL |

### Navigation（NORMAL モード）

| コマンド | 動作 |
|---|---|
| `h`/`j`/`k`/`l` | 左/下/上/右 |
| `Space` | 右へ |
| `w` | 次の単語 |
| `e` | 単語末 |
| `b` | 前の単語 |
| `0` | 行頭 |
| `$` | 行末 |
| `^` | 最初の非空白文字 |
| `gg` | 入力先頭 |
| `G` | 入力末尾 |
| `f{char}` | 次の `{char}` へジャンプ |
| `F{char}` | 前の `{char}` へジャンプ |
| `t{char}` | 次の `{char}` の**直前**へジャンプ |
| `T{char}` | 前の `{char}` の**直後**へジャンプ |
| `;` | 直近の f/F/t/T を繰り返す |
| `,` | 直近の f/F/t/T を**逆方向**で繰り返す |

**重要**: NORMAL モードでカーソルが入力先頭または末尾にあって動けない時、`j`/`k` や矢印キーは**コマンド履歴ナビ**に化ける。

### Editing（NORMAL モード）

| コマンド | 動作 |
|---|---|
| `x` | 文字削除 |
| `dd` | 行削除 |
| `D` | 行末まで削除 |
| `dw` / `de` / `db` | 単語削除（前方 / 末尾 / 後方） |
| `cc` | 行を変更（行を消して INSERT） |
| `C` | 行末まで変更 |
| `cw` / `ce` / `cb` | 単語変更 |
| `yy` / `Y` | 行をヤンク（コピー） |
| `yw` / `ye` / `yb` | 単語ヤンク |
| `p` | カーソル後ろにペースト |
| `P` | カーソル前にペースト |
| `>>` / `<<` | インデント / デデント |
| `J` | 行を結合 |
| `u` | undo |
| `.` | 直前の変更を繰り返す |

### Text objects（NORMAL モード）

`d` / `c` / `y` などのオペレータと組み合わせて使う。

| コマンド | 動作 |
|---|---|
| `iw` / `aw` | 単語の内 / 周（aw は周辺空白も含む） |
| `iW` / `aW` | WORD（whitespace-delimited）の内 / 周 |
| `i"` / `a"` | ダブルクォートの内 / 周 |
| `i'` / `a'` | シングルクォートの内 / 周 |
| `i(` / `a(` | 丸括弧の内 / 周 |
| `i[` / `a[` | 角括弧の内 / 周 |
| `i{` / `a{` | 波括弧の内 / 周 |

例: `ci"` → ダブルクォート内を変更、`da(` → 丸括弧ごと削除。

### Visual mode

`v` で文字単位、`V` で行単位の選択を開始。motion で選択範囲を広げ、operator が直接効く。

| コマンド | 動作 |
|---|---|
| `d` / `x` | 選択削除 |
| `y` | 選択ヤンク |
| `c` / `s` | 選択を変更 |
| `p` | レジスタ内容で選択を置換 |
| `r{char}` | 選択中の各文字を `{char}` で置換 |
| `~` / `u` / `U` | 大小文字 toggle / 小文字化 / 大文字化 |
| `>` / `<` | インデント / デデント |
| `J` | 行結合 |
| `o` | カーソルとアンカーを入れ替え |
| `iw` / `aw` / `i"` / … | text object 選択 |
| `v` / `V` | 文字単位 / 行単位の切替、または抜ける |

**未対応**: `Ctrl+V` のブロック visual mode はサポートされない。

## 重要ポイント

- vim mode と keybindings は**独立に動く**
  - vim mode はテキスト入力レベル（カーソル / モード / motion）
  - keybindings はコンポーネントレベル（task list toggle / submit 等）
  - **vim mode の `Esc` は INSERT → NORMAL の切替**で、`chat:cancel` は走らない
  - 大半の `Ctrl+key` ショートカットは vim mode を貫通して keybinding に届く
  - vim NORMAL で `?` は help（vim の挙動）
- 端での `j`/`k` がコマンド履歴ナビになる挙動を覚えておくと便利
- ブロック visual mode（`Ctrl+V`）が無いので、矩形選択が必要なときは別アプローチ必要
- text objects は強力。`ci"` / `da(` のようなオペレーション粒度で考えると入力編集が速くなる
- `.` の repeat-last-change は INSERT セッション終了後の「最後の編集の繰り返し」になる典型 vim 挙動

## コード例 / 図

### 典型的な編集フロー

```text
[NORMAL] cursor at start of "fooBar"
press: ciw                # 単語まるごと変更
[INSERT] type: helloWorld
press: Esc                # NORMAL に戻る
press: .                  # 別の単語でも同じ操作を繰り返したい場合は移動して .
```

### クォート内テキストの差し替え

```text
入力: const name = "old value here"
カーソル位置: 適当 (クォート内ならどこでも)
press: ci"
[INSERT] type: new
press: Esc
結果: const name = "new"
```

### 入力履歴を遡る（j/k がフォールバックする例）

```text
[NORMAL] カーソルが入力末尾、これ以上 j で下に行けない
press: j   ← コマンド履歴の次のエントリへ
press: k   ← 戻る
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 次の教材: [05-history-and-shell-mode.md](05-history-and-shell-mode.md) — コマンド履歴とシェル統合
- 関連 docs: [Customize keybindings > Vim mode interaction](https://code.claude.com/docs/en/keybindings#vim-mode-interaction)

---

_Auto-generated at 2026-05-08 via /learning-flow:material（公式docs駆動）_
