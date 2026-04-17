# 04. 効果的なプロンプトと CLAUDE.md の書き方

出典: [best-practices.md](https://code.claude.com/docs/en/best-practices.md), [quickstart.md](https://code.claude.com/docs/en/quickstart.md)

## 黄金ルール: context を管理せよ

best-practices のほぼ全ては「context window が埋まると性能が落ちる」への対策。覚えるのはこれ 1 つ。

## 5 大プラクティス

### 1. Claude に「自分で検証する手段」を与える

- テストケースを渡す、スクリーンショットを渡す、期待出力を明示する
- UI は Claude in Chrome でスクショ比較できる
- **"仕様通り動いたか"を Claude 自身が判定できる**形にすると、人間の review 負担が激減する

### 2. 「探索 → 計画 → 実装 → コミット」の 4 フェーズ

小さな typo や 1 行変更は直接やらせる。複雑な変更は以下：

1. **Explore** (Plan mode): `src/auth/` を読んで session ハンドリングを理解
2. **Plan** (Plan mode): 計画を作る。`Ctrl+G` でエディタで直接編集可
3. **Implement** (Normal mode): 計画通りに実装 + テスト
4. **Commit**: 記述的メッセージで commit + PR

### 3. 具体的な context を入れる

| 悪い | 良い |
|---|---|
| "login のバグを直して" | "session timeout 後に login が失敗する。`src/auth/` の token refresh 周辺。失敗する test を書いてから直して" |
| "tests を書いて" | "`foo.py` のログアウト済みユーザーのエッジケースに絞って書いて。mock は使わず" |
| "dashboard を良くして" | "[スクショ] これを実装。実装後にスクショ撮って元と比較、差分を listup して直して" |

- `@path/to/file` で参照
- 画像は paste/drag で渡す
- `cat error.log \| claude` で pipe

### 4. subagent で context を汚さず調査

「auth system を調査して」と main で頼むと数十ファイル読んで context が埋まる。subagent は別 context で読んでサマリだけ返す。

### 5. 早めの course correct

Claude が変な方向に行ったら、`Esc` で止めて方向修正。2 回同じ修正をしても直らなければ `/clear` して、学んだ情報を組み込んだプロンプトで再スタートする方が速い。

## CLAUDE.md の書き方

- `/init` で雛形生成。その後手動で pruning
- **載せるべき**: Claude が推論できないもの（ビルドコマンド、独自スタイル、env vars、レポジトリの慣例）
- **載せるべきでない**: Claude が code を読めば分かること、標準慣習、頻繁に変わる情報
- 各行で自問: "これを消すと Claude がミスを起こすか？ No なら削る"
- 強調したいルールは "IMPORTANT" / "YOU MUST" を付ける
- 長すぎると無視される。**短く、レビュー、剪定**を継続

### 設置場所

| パス | スコープ |
|---|---|
| `~/.claude/CLAUDE.md` | 全セッション |
| `./CLAUDE.md` | プロジェクト共有（git にコミット） |
| `./CLAUDE.local.md` | 個人メモ（.gitignore に） |
| `parent/CLAUDE.md` + `parent/sub/CLAUDE.md` | monorepo で階層的に合成 |

`@path/to/file` で他ファイルを import できる。

## 初心者が最初に身につけたい Tips 3 選

1. **describe the symptom**: 「バグを直して」ではなく「症状 + 関連コード + 直った状態」を書く
2. **use `/clear`**: 別タスクに移るとき必ず。context を溜めない
3. **use plan mode (`Shift+Tab` x2)**: 少しでも複雑と思ったら plan → review → implement
