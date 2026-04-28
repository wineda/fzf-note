# fzf-note アプリ説明

## 概要

`fzf-note` は、`Bash + fzf + Markdown` で動くシンプルなノート管理 CLI アプリです。

日々の未整理メモを `work/YYYY-MM-DD.md` に書きため、`./bin/fzf-note merge` で `note/note.md` の末尾に単純追記します。必要なときだけ `./bin/fzf-note format-ai` で `note/note.md` 全体を AI 整形できます。

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

今日の日付の作業メモファイルを `vim` で開きます。

```bash
./bin/fzf-note add
```

### `search`

`note/note.md` の各行を `fzf` で検索し、選択行を `vim +行番号 note/note.md` で開きます。

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
- vim

## 任意要件

- `bat`: 検索プレビューを見やすくする
- `tmux`: tmux 内で検索 UI を popup 表示する
- 外部 AI CLI: `format-ai` を使う場合に必要
