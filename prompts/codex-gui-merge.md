# Codex GUI 向け: work メモ統合依頼

次の手順で作業してください。

1. 現在の作業ツリーを確認する
2. 既存のユーザー変更は戻さない
3. `code/merge-YYYYMMDD-HHMMSS` 形式のブランチを作成する
4. `work/*.md` を読み、`note/note.md` へ H2 単位で統合する
5. 取り込み済み `work/*.md` を `work/merged/` へ移動する
6. 変更を stage する
7. 次のメッセージで commit する

コミットメッセージ:
`chore(note): merge work notes into note.md`
