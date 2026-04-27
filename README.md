# fzf-note

`fzf + Bash + Git` で動くノート管理CLIのひな形です。

## できること

- `add` : 日付ごとの `work/YYYY-MM-DD.md` にメモを追記
- `search` : `note/` 配下のノートを `fzf` で高速検索
- `merge-ai` : `work/` から `note/` へ AI マージ（外部AIコマンド連携）

## 前提

- Bash 4+
- git
- fzf

## セットアップ

```bash
chmod +x bin/fzf-note
./bin/fzf-note init
```

必要に応じて `~/.bashrc` などにエイリアスを追加:

```bash
alias fnote='/path/to/fzf-note/bin/fzf-note'
```

## 使い方

### 1) メモを追加

```bash
./bin/fzf-note add "APIの仕様を確認"
./bin/fzf-note add
# 引数なしなら対話入力
```

- 追記先: `work/YYYY-MM-DD.md`

### 2) ノートを検索

```bash
./bin/fzf-note search
```

- `note/` 配下の `.md` を対象に `fzf` + `bat`(あれば)でプレビュー
- Enter で選択したファイルを表示

### 3) AIで work -> note をマージ

```bash
export AI_MERGE_CMD='your_ai_cli --stdin --stdout'
./bin/fzf-note merge-ai
```

`AI_MERGE_CMD` は「標準入力にプロンプトを渡すと、標準出力にMarkdownを返す」コマンドを想定しています。

`merge-ai` の流れ:

1. `work/*.md` を収集
2. AIに統合Markdownの生成を依頼
3. 出力を `note/YYYY-MM-DD.md` に保存
4. 取り込み済み `work/*.md` を `work/merged/` へ移動
5. 変更を `git add/commit`（オプション）

## ディレクトリ構成

```text
.
├─ bin/
│  └─ fzf-note
├─ work/
│  └─ .gitkeep
└─ note/
   └─ .gitkeep
```

## 今後の拡張例

- タグ管理
- noteの分割（トピック別）
- embedding + ベクトル検索
- PRベースでの自動整理
