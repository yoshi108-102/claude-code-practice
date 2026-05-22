# 03. Checkpointing — 編集と会話の巻き戻し

> 出典: [Checkpointing](https://code.claude.com/docs/en/checkpointing)（閲覧日 2026-05-22）
> このノートは公式ドキュメント「Checkpointing」の構造を起点に Claude が自動生成した教材です。

## 概要

Claude Code は作業中、**Claude のファイル編集を自動でスナップショット**し、いつでも前の状態へ巻き戻せる安全網を提供する。これにより「大胆で広範な変更」を、いつでも戻れる前提で安心して任せられる。

Task 1 の復習で出た通り、checkpoint は「会話の状態」だけでなく「**ファイル編集の自動スナップショット**」も守る。ただし**守れる範囲には明確な境界**がある（後述の Limitations が本章の山）。

## 公式docsに沿った解説

### How checkpoints work — Automatic tracking（自動追跡）

Claude のファイル編集ツールによる変更を全て追跡する。

- **ユーザーのプロンプトごとに新しい checkpoint が作られる**
- checkpoint は**セッションを跨いで永続**(`/resume` した会話でもアクセス可能)
- セッションと共に**30 日後に自動クリーンアップ**（設定変更可）

### Rewind and summarize（巻き戻しと要約）

`/rewind`、または**プロンプト入力が空のときに `Esc` を 2 回**押すと rewind メニューが開く。

> 注意: 入力にテキストがあると `Esc` 2 回は**テキストのクリア**になる（メニューは開かない）。クリアしたテキストは入力履歴に保存されるので、`Up` で呼び戻せる。

rewind メニューはセッション中に送った各プロンプトを一覧表示し、地点を選んでアクションを選ぶ:

| アクション | 効果 |
|---|---|
| **Restore code and conversation** | コードと会話の両方をその地点へ戻す |
| **Restore conversation** | 会話だけ戻す（**コードは現状維持**） |
| **Restore code** | ファイル変更だけ戻す（**会話は維持**） |
| **Summarize from here** | その地点**以降**を要約に圧縮（context 解放） |
| **Summarize up to here** | その地点**以前**を要約に圧縮（後半は維持） |
| **Never mind** | 何もせず一覧に戻る |

### Restore vs. summarize（復元と要約の違い）

これが混同しやすいポイント。

- **Restore 系**: 状態を**巻き戻す**。コード変更・会話履歴・その両方を「無かったこと」にする
- **Summarize 系**: ディスク上のファイルは**変えず**、会話の一部を AI 要約に**圧縮**するだけ
  - **Summarize from here**: 選んだメッセージより**前**はそのまま、選んだメッセージ以降を要約に置換 → 「脇道の議論を捨てて、序盤の文脈は詳細に残す」
  - **Summarize up to here**: 選んだメッセージより**前**を要約に置換、選んだメッセージ以降はそのまま（会話の末尾に留まる）→ 「序盤のセットアップ議論を圧縮し、最近の作業は詳細に残す」

どちらの場合も**元メッセージはセッション transcript に保存**されるので、Claude は必要なら細部を参照できる。要約のフォーカス指示も付けられる(`/compact` の地点指定版というイメージ)。

> 別の道: 要約は同じセッション内で context を圧縮する。元セッションを保ったまま別アプローチを試したいなら、要約ではなく **fork**（`claude --continue --fork-session`）を使う。

### Common use cases（典型的な使いどころ）

- **代替案の探索**: 出発点を失わずに別の実装を試す
- **ミスからの復旧**: バグを入れた変更を素早く取り消す
- **機能の反復**: 動く状態に戻れる前提でバリエーションを試す
- **context 空間の解放**: 冗長なデバッグ会話を途中から要約し、最初の指示は詳細に残す

### Limitations（限界 — 最重要）

#### Bash コマンドの変更は追跡されない

checkpoint が追跡するのは**Claude のファイル編集ツール経由の直接編集だけ**。bash コマンドによるファイル変更は**巻き戻せない**。

```bash
rm file.txt        # ← rewind で戻せない
mv old.txt new.txt # ← 同上
cp source.txt dest.txt
```

#### 外部の変更は追跡されない

現セッション内で編集されたファイルのみ対象。Claude Code の外で手で編集したり、別の同時セッションが行った編集は、（たまたま同じファイルを触らない限り）**捕捉されない**。

#### バージョン管理の代替ではない

checkpoint は**セッションレベルの素早い復旧**用。永続的な履歴・コラボには:

- コミット・ブランチ・長期履歴は引き続き **Git** を使う
- checkpoint は Git を補完するが**置き換えない**
- **「checkpoint = local undo」「Git = permanent history」** と捉える

## 重要ポイント

- checkpoint は**ユーザープロンプトごと**に自動作成、セッション跨ぎで永続、30 日で自動削除
- 入口は `/rewind` または **入力が空のときの `Esc` 2 回**（テキストがあるとクリアになる罠）
- **Restore（巻き戻し）と Summarize（圧縮）は別物**。Restore は状態を戻す、Summarize はファイルを変えず会話を要約
- Restore は「コードのみ / 会話のみ / 両方」を選べる（Task 1 の復習で押さえた点）
- **追跡できないもの 3 つ**: ①bash コマンドの副作用（`rm`/`mv`/`cp`）②外部・別セッションの編集 ③（そもそも）git の代替ではない
- 別アプローチを並行で試すなら summarize ではなく **fork**

## コード例 / 図

### rewind の入口と分岐

```text
入力欄が空の状態で Esc Esc（または /rewind）
  → rewind メニュー（過去プロンプト一覧）
    ├─ Restore code and conversation … 両方巻き戻し
    ├─ Restore conversation          … 会話だけ（コード維持）
    ├─ Restore code                  … コードだけ（会話維持）
    ├─ Summarize from here           … 選択地点以降を要約
    ├─ Summarize up to here          … 選択地点以前を要約
    └─ Never mind                    … 中止
```

### checkpoint が守る / 守らない

```text
守る:   Claude の Edit/Write ツールによるファイル編集、会話状態
守らない: bash の rm/mv/cp、外部エディタの編集、別セッションの編集、
         外部 DB / API へ送信済みのリクエスト
→ 永続履歴は Git。checkpoint は「session-local な undo」
```

## 関連

- 議論・Q&A: （`lesson` 中に発生したら `reference/` 配下にリンクが追加されます）
- 前の教材: [02-compaction-and-context-tools.md](02-compaction-and-context-tools.md) — compaction と context ツール
- 関連 docs: [Interactive mode](https://code.claude.com/docs/en/interactive-mode) / [Commands > /rewind](https://code.claude.com/docs/en/commands) / [Sessions > branch a session](https://code.claude.com/docs/en/sessions#branch-a-session)

---

_Auto-generated at 2026-05-22 via /learning-flow:material（公式docs駆動）_
