# fzf-note アプリ説明

## 概要

`fzf-note` は、`Bash + fzf + Markdown` で動くシンプルなノート管理 CLI アプリです。

日々の未整理メモを `work/YYYY-MM-DD.md` に書きため、`./bin/fzf-note merge` で `note/note.md` の末尾に単純追記します。必要なときだけ `./bin/fzf-note format-ai` で `note/note.md` 全体を AI 整形できます。

`work/` と `note/` はデフォルトでこのリポジトリ配下を使いますが、環境変数で別リポジトリへ切り出せます（後述）。

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
- `prompts/format.md`
- `prompts/organize-fast.md`

### `add`

今日の日付の作業メモファイルをエディタで開きます（デフォルト: `vim`）。

```bash
./bin/fzf-note add
```

### `search`

`note/note.md` の各行を `fzf` で検索し、選択行を `エディタ +行番号 note/note.md` で開きます。

```bash
./bin/fzf-note search
```

### `merge`

`work/*.md` を `note/note.md` の末尾へ単純に追記し、追記済みファイルを `work/merged/` に移動します。

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
- `FZF_NOTE_WORK_DIR`: `work` ディレクトリの配置先（デフォルト: `<repo>/work`）
- `FZF_NOTE_NOTE_DIR`: `note` ディレクトリの配置先（デフォルト: `<repo>/note`）
- `FZF_NOTE_NOTE_FILE`: 統合ノートファイルのパス（デフォルト: `$FZF_NOTE_NOTE_DIR/note.md`）
- `FZF_NOTE_BACKUP_DIR`: `backup` コマンドの保存先ディレクトリ
- `AI_FORMAT_CMD`: `format-ai` で使う外部 AI CLI コマンド

例:

```bash
export FZF_NOTE_EDITOR=nvim
export FZF_NOTE_WORK_DIR="$HOME/repos/fzf-note-work/work"
export FZF_NOTE_NOTE_DIR="$HOME/repos/fzf-note-note/note"
./bin/fzf-note add
```

## `work/` と `note/` を別 Git リポジトリで管理する構成例

`fzf-note` 本体（このリポジトリ）とは別に、データ用リポジトリを 2 つ用意します。

```text
~/repos/
  fzf-note/        # アプリ本体
  fzf-note-work/   # 作業メモ (work/)
  fzf-note-note/   # 統合ノート (note/)
```

セットアップ例:

```bash
mkdir -p ~/repos
cd ~/repos
git clone <app-repo> fzf-note
git clone <work-repo> fzf-note-work
git clone <note-repo> fzf-note-note

mkdir -p ~/repos/fzf-note-work/work
mkdir -p ~/repos/fzf-note-note/note
touch ~/repos/fzf-note-note/note/note.md

cat >> ~/.bashrc <<'EOF'
export FZF_NOTE_EDITOR=nvim
export FZF_NOTE_WORK_DIR="$HOME/repos/fzf-note-work/work"
export FZF_NOTE_NOTE_DIR="$HOME/repos/fzf-note-note/note"
EOF
```

この構成にすると:

- `add` は `fzf-note-work` 側に日次メモを作成
- `merge` は `fzf-note-note` 側の `note.md` に追記
- `work` と `note` を用途別に独立してコミット／バックアップ可能

### `format-ai`

`note/note.md` 全体を AI で整形します（手動実行）。

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
