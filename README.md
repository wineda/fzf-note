# fzf-note アプリ説明

## 概要

`fzf-note` は、`Bash + fzf + Markdown` で動くシンプルなノート管理 CLI アプリです。

日々の未整理メモを `document/work/YYYY-MM-DD.md` に書きため、`./bin/fzf-note merge` で `document/note/note.md` の末尾に単純追記します。追記済みファイルは `document/merge/` へ移動します。必要なときだけ `./bin/fzf-note format-ai` で `document/note/note.md` 全体を AI 整形できます。

`document/` はデフォルトでこのリポジトリ配下を使いますが、環境変数で任意の Git リポジトリへ切り出せます（後述）。

## コマンド

### `init`

必要なディレクトリとファイルを作成します。

```bash
./bin/fzf-note init
```

作成対象:

- `document/`
- `document/work/`
- `document/merge/`
- `document/note/`
- `document/note/note.md`
- `prompts/format.md`
- `prompts/organize-fast.md`

### `add`

今日の日付の作業メモファイルをエディタで開きます（デフォルト: `vim`）。

```bash
./bin/fzf-note add
```

### `search`

`document/note/note.md` の各行を `fzf` で検索し、選択行を `エディタ +行番号 document/note/note.md` で開きます。

```bash
./bin/fzf-note search
```

### `merge`

`document/work/*.md` を `document/note/note.md` の末尾へ単純に追記し、追記済みファイルを `document/merge/` に移動します。  
`document/` が Git リポジトリなら、`git add -A && git commit` を自動実行します。

コミットメッセージ例:

- 1 ファイル: `note.mdに2026-04-28.mdをマージ`
- 複数ファイル: `note.mdに2026-04-28.mdほか2件をマージ`

```bash
./bin/fzf-note merge
```

### `backup`

カレントディレクトリを `yyyymmdd-hhmmss.tar.gz` 形式でバックアップします。

```bash
export FZF_NOTE_BACKUP_DIR=/path/to/backups
./bin/fzf-note backup
```

保存先は環境変数 `FZF_NOTE_BACKUP_DIR` で指定します。

## 環境変数

- `FZF_NOTE_EDITOR`: `add` / `search` で使うエディタコマンド（デフォルト: `vim`）
- `FZF_NOTE_DOC_DIR`: `document` ディレクトリの配置先（デフォルト: `<repo>/document`）
- `FZF_NOTE_WORK_DIR`: `work` ディレクトリの配置先（デフォルト: `$FZF_NOTE_DOC_DIR/work`）
- `FZF_NOTE_MERGE_DIR`: `merge` ディレクトリの配置先（デフォルト: `$FZF_NOTE_DOC_DIR/merge`）
- `FZF_NOTE_NOTE_DIR`: `note` ディレクトリの配置先（デフォルト: `$FZF_NOTE_DOC_DIR/note`）
- `FZF_NOTE_NOTE_FILE`: 統合ノートファイルのパス（デフォルト: `$FZF_NOTE_NOTE_DIR/note.md`）
- `FZF_NOTE_BACKUP_DIR`: `backup` コマンドの保存先ディレクトリ
- `AI_FORMAT_CMD`: `format-ai` で使う外部 AI CLI コマンド

例:

```bash
export FZF_NOTE_EDITOR=nvim
export FZF_NOTE_DOC_DIR="$HOME/repos/fzf-note-doc/document"
./bin/fzf-note add
```

## `document/` を別 Git リポジトリで管理する構成例

`fzf-note` 本体（このリポジトリ）とは別に、データ用リポジトリを 1 つ用意します。

```text
~/repos/
  fzf-note/       # アプリ本体
  fzf-note-doc/   # ドキュメントデータ (document/)
```

セットアップ例:

```bash
mkdir -p ~/repos
cd ~/repos
git clone <app-repo> fzf-note
git clone <doc-repo> fzf-note-doc

mkdir -p ~/repos/fzf-note-doc/document/work
mkdir -p ~/repos/fzf-note-doc/document/merge
mkdir -p ~/repos/fzf-note-doc/document/note
touch ~/repos/fzf-note-doc/document/note/note.md

cat >> ~/.bashrc <<'EOF'
export FZF_NOTE_EDITOR=nvim
export FZF_NOTE_DOC_DIR="$HOME/repos/fzf-note-doc/document"
EOF
```

この構成にすると:

- `add` は `fzf-note-doc` 側の `document/work` に日次メモを作成
- `merge` は `document/note/note.md` に追記し、元ファイルを `document/merge` に移動
- `document` 全体をデータ専用リポジトリとしてコミット／バックアップ可能

### `format-ai`

`document/note/note.md` 全体を AI で整形します（手動実行）。

```bash
export AI_FORMAT_CMD='your_ai_cli --stdin --stdout'
./bin/fzf-note format-ai
```

`AI_FORMAT_CMD` は、標準入力でプロンプトを受け取り、標準出力で整形後 Markdown 全文を返すコマンドです。

`prompts/organize-fast.md` は、`note.md` 全文をプロンプトに与える際に、内容整理を高速化するための専用プロンプトです。

## 必須要件

- Bash 4 以上
- fzf
- `FZF_NOTE_EDITOR` で指定したエディタ（未指定時は `vim`）

## 任意要件

- `bat`: 検索プレビューを見やすくする
- `tmux`: tmux 内で検索 UI を popup 表示する
- 外部 AI CLI: `format-ai` を使う場合に必要
