# fzf-note アプリ説明

## 概要

`fzf-note` は、`Bash + fzf + Git + Markdown` で動くシンプルなノート管理 CLI アプリです。

日々の未整理メモを `work/YYYY-MM-DD.md` に書きため、あとから `note/note.md` という単一の Markdown ノートへ統合します。検索には `fzf` を使い、AI コマンドと連携して未整理メモの自動マージもできます。

## 目的

- 日々のメモを素早く追記する
- ひとつの Markdown ノートに情報を集約する
- `fzf` でノートを高速検索する
- AI を使って `work/` の未整理メモを `note/note.md` に整理統合する
- Git ブランチとコミットでマージ作業の履歴を残す

## 技術スタック

- Bash
- Git
- fzf
- vim
- Markdown
- 任意の外部 AI CLI
- 任意で `bat`
- 任意で `tmux`

## ディレクトリ構成

```text
.
├── bin/
│   └── fzf-note
├── note/
│   └── note.md
├── work/
│   ├── YYYY-MM-DD.md
│   └── merged/
├── prompts/
│   ├── merge.md
│   └── codex-gui-merge.md
└── README.md
```

## コマンド

### `init`

必要なディレクトリとファイルを作成します。

```bash
./bin/fzf-note init
```

作成対象:

- `work/`
- `work/merged/`
- `note/`
- `note/note.md`

### `add`

今日の日付の作業メモファイルを `vim` で開きます。

```bash
./bin/fzf-note add
```

例:

```text
work/2026-04-28.md
```

`add` はコマンド引数でメモ本文を受け取りません。メモは `vim` 内で直接入力します。

### `search`

`note/note.md` の各行を `fzf` で検索します。

```bash
./bin/fzf-note search
```

挙動:

- `note/note.md` を行番号付きで `fzf` に渡す
- プレビューは下ペインに表示する
- `bat` があれば行番号・色付き・選択行ハイライトで表示する
- `bat` がなければ `awk` で周辺行を表示する
- Enter で選択した行を `vim +行番号 note/note.md` で開く
- キャンセル時は何もしない

tmux 内では `tmux display-popup` で `fzf` を開き、選択後に親ペイン側で `vim` を起動します。

### `merge-ai`

`work/*.md` の未整理メモを AI に渡し、出力された Markdown を `note/note.md` として保存します。

```bash
export AI_MERGE_CMD='your_ai_cli --stdin --stdout'
./bin/fzf-note merge-ai
```

`AI_MERGE_CMD` は、標準入力でプロンプトを受け取り、標準出力で更新後の Markdown を返すコマンドです。

Codex CLI を使う例:

```bash
export AI_MERGE_CMD='codex exec -'
```

### `merge-ai --commit`

AI マージ後に Git ブランチを作成し、変更をコミットします。

```bash
./bin/fzf-note merge-ai --commit
```

デフォルトのブランチ名:

```text
codex/merge-notes-YYYYMMDD-HHMMSS
```

コミットメッセージ:

```text
chore(note): merge work notes into note.md
```

ブランチ名を指定する場合:

```bash
./bin/fzf-note merge-ai --commit --branch codex/merge-notes
```

## ノート構造

`note/note.md` は単一の Markdown ファイルです。

H2 見出し `##` から次の `##` の直前までを、1つのノート項目として扱います。

## work メモ構造

`work/*.md` に日々の未整理メモを書きます。

こちらも H2 見出し `##` 単位で1つのメモとして扱います。

## AI マージ仕様

AI マージでは、次の情報をまとめて AI に渡します。

- `prompts/merge.md`
- 既存の `note/note.md`
- `work/*.md` の内容

AI に求める出力は、更新後の `note/note.md` 全体だけです。

マージルール:

- 既存ノートに同じ話題・近い話題の H2 セクションがあれば追記・統合する
- 該当セクションがなければ新しい H2 セクションとして追加する
- 重複内容はまとめる
- 既存ノートの情報は消さない
- 曖昧な内容や未確定事項は `要確認` として残す
- 説明文やコードフェンスではなく、Markdown 本文だけを出力する

## マージ後のファイル移動

AI マージまたは Codex GUI マージが完了したら、取り込み済みの `work/*.md` を以下へ移動します。

```text
work/merged/
```

## Codex GUI 用マージ

`prompts/codex-gui-merge.md` は、Codex GUI にマージ作業を依頼するためのプロンプトです。

このプロンプトでは以下を依頼します。

- 現在の作業ツリー確認
- 既存ユーザー変更を戻さない
- `code/merge-YYYYMMDD-HHMMSS` 形式のブランチ作成
- `work/*.md` を `note/note.md` に H2 単位で統合
- マージ済みメモを `work/merged/` に移動
- 変更を stage
- 指定メッセージで commit

コミットメッセージ:

```text
chore(note): merge work notes into note.md
```

## 必須要件

- Bash 4 以上
- Git
- fzf
- vim

## 任意要件

- `bat`: 検索プレビューを見やすくする
- `tmux`: tmux 内で検索 UI を popup 表示する
- 外部 AI CLI: `merge-ai` を使う場合に必要

## 主要なユーザー体験

1. ユーザーが `fzf-note add` を実行する
2. 今日の日付の `work/YYYY-MM-DD.md` が `vim` で開く
3. ユーザーが未整理メモを書く
4. 必要に応じて `fzf-note search` で `note/note.md` を検索する
5. ある程度メモが溜まったら `merge-ai` または Codex GUI で `work/*.md` を `note/note.md` に統合する
6. 統合済みの `work/*.md` は `work/merged/` に移動される
7. Git コミットで整理作業を履歴化する

## 再実装時の注意点

- `note/note.md` は複数ファイルに分割せず、単一ファイルとして扱う
- H2 見出し `##` をノート項目の境界にする
- ユーザーの既存ノート情報を削除しない
- マージ結果は読みやすい Markdown に整える
- `work/merged/` に移動するのはマージ済みの `work/*.md` のみ
- Git 操作時は、既存のユーザー変更を勝手に戻さない
- `search` は選択行を `vim +行番号 note/note.md` で開けるようにする
- tmux 内では popup を使えると望ましい
