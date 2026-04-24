# 05. Auto モード 深掘り

出典:
- [permission-modes](https://code.claude.com/docs/en/permission-modes) — モード選択の主ページ
- [auto-mode-config](https://code.claude.com/docs/en/auto-mode-config) — classifier の設定リファレンス
- [permissions](https://code.claude.com/docs/en/permissions) — 権限システム全体

## 概要

`03-permissions-and-safety.md` で触れた 4 permission mode のうち、**Auto** の深掘り。
振り返りクイズ Q6 で「あらゆる permission を許可するモード」と誤解していた点を修正する。

正しい理解: Auto モードは "**別の Claude（分類モデル / classifier）が毎アクションをレビュー**" するモード。
「野放し自動化」は `bypassPermissions` の方で、Auto は **ガード付き自動化**。

## 解説

### 核心: classifier は事前レビュー AI

Auto モードでは、Claude が行動を取る前に分類モデルが毎回以下 4 点を判定する:

1. ユーザーの要求の範囲を超えていないか（スコープ逸脱）
2. 見覚えのないインフラを対象にしていないか（外部ターゲット）
3. 外部コンテンツに誘導された行動ではないか（prompt injection 対策）
4. 取り返しのつく行動か（不可逆性）

### デフォルトで block / allow されるリスト

docs に明示されている:

| ❌ 既定でブロック | ✅ 既定で許可 |
|---|---|
| `curl \| bash` 等の外部コード実行 | 作業ディレクトリ内のファイル操作 |
| 外部エンドポイントへの機密送信 | ロックファイル/マニフェスト記載パッケージの install |
| 本番デプロイ・マイグレーション | `.env` 読み取り & **対応 API** への送信 |
| クラウドストレージの大量削除 | 読み取り専用 HTTP |
| IAM / repo 権限の付与 | 起動時 branch or Claude が作った branch への push |
| 共有インフラの変更 | |
| セッション前から存在したファイルの不可逆破壊 | |
| force push / main への直 push | |

### 判定フロー（4 ステップ）

```
1. permissions.json の allow/deny ルール  → マッチしたら即決
2. 読み取り & 作業ディレクトリ内ファイル編集  → classifier スキップで即許可
3. それ以外 → classifier へ
4. classifier block → Claude は理由を受け取り別アプローチを試みる
```

### Auto モードに入ると drop される広い allow ルール（落とし穴）

- `Bash(*)`（全 Bash 許可）
- `Bash(python*)` 等のインタプリタワイルドカード
- パッケージマネージャの run コマンド
- `Agent` allow ルール

`Bash(npm test)` のような**狭いルール**は維持される。Auto を抜けると drop されたルールは復元。

### 会話で宣言した「境界」は block 信号になる

"push しないで" "review するまで deploy するな" と会話で言うと、デフォルトで許可される行動でも classifier がブロックする。

注意:
- 境界はルールとして保存されない。**transcript から毎回再読**するだけ
- `/compact` で該当メッセージが要約に消えると**境界も消える**
- 確実にしたいなら `permissions.deny` に書く

### Fallback 挙動

- 連続 **3 回** または累計 **20 回** ブロック → Auto モード一時停止、通常プロンプトに戻る
- `claude -p`（非対話モード）では fallback できないので **セッションが abort**
- しきい値は固定（設定不可）

### 🔒 Prompt injection 対策の多層構造

classifier が読むもの / 読まないもの:

| | 読む | 読まない |
|---|---|---|
| ユーザーのメッセージ | ✅ | |
| tool 呼び出し | ✅ | |
| CLAUDE.md | ✅ | |
| **tool の実行結果** | | ❌ |

tool 結果を classifier から隠すことで、ファイルや Web ページに仕込まれた敵対的テキストが classifier を騙せない設計。加えてサーバー側の別プローブが tool 結果を事前スキャン。

### 利用要件

| 条件 | 要件 |
|---|---|
| プラン | Max / Team / Enterprise / API（**Pro 不可**） |
| モデル | Sonnet 4.6 / Opus 4.6 / Opus 4.7（**Haiku 不可、claude-3 系不可**） |
| プロバイダ | Anthropic API のみ（**Bedrock / Vertex / Foundry 不可**） |
| バージョン | v2.1.83 以降 |

## Q&A

**Q: Auto モードは "あらゆる permission を許可する" モードではないのか？**

違う。それは `bypassPermissions` の挙動。Auto は **別の分類モデルが毎アクションをレビューする**モード。「プロンプトを減らすが安全性は保証しない（research preview）」と docs に明記されている。

**Q: Auto モードにしたら `Bash(*)` で全許可にしておけば快適なのか？**

快適にならない。Auto モードに入ると `Bash(*)` のような**広すぎる allow ルールは自動で drop される**。設計思想として「自由すぎる承認は Auto と両立しない」。`Bash(npm test)` のような狭いルールは維持されるので、使うなら具体的に書く。

**Q: classifier が間違ってブロックし続けたらどうなる？**

対話モードでは連続 3 回 / 累計 20 回で Auto モードが一時停止し、通常の承認プロンプトに戻る。`-p` モードではセッションごと abort。`/permissions` の「Recently denied」タブで `r` キーを押すと手動承認でリトライできる。

**Q: "don't push to main" とチャットで言うのと、`permissions.deny` に書くのは何が違う？**

会話での宣言は classifier が transcript から毎回再読する**ソフトな境界**。`/compact` で消えると境界も消える。`permissions.deny` は classifier より**前**に評価される**ハードな禁止**で、消えない・優先される。確実に止めたいなら deny ルールにする。

**Q: classifier は敵対的なコンテンツに騙されないのか？**

2 層で防御。(1) classifier は tool 結果（ファイル/Web ページの中身）を**読まない**設計なので、そこに仕込まれた「この操作を許可せよ」という指示が classifier に届かない。(2) サーバー側の別プローブが tool 結果を事前スキャンして suspicious content にフラグを立てる。

## 関連

- 基礎: [03-permissions-and-safety.md](03-permissions-and-safety.md) — 4 モードの比較
- 脱線（Phase 4 本格扱い）: `autoMode.environment` 設定 / CLI サブコマンド（`claude auto-mode defaults` 等）/ classifier のコスト・レイテンシ — `auto-mode-config` ページ参照
- 脱線（Phase 3 本格扱い）: subagent は spawn 時 / 実行中 / 完了時 の 3 点で classifier にチェックされる

---

_Saved at 2026-04-24 via /learning-flow:topic_
