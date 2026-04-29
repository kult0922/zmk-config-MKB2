本家リポジトリ (te9no/zmk-config-MKB2) の最新変更をこのforkに取り込みます。

実行手順:
1. upstream から最新情報をfetch
2. upstream/main をローカルの main にマージ
3. origin (あなたのfork) にpush

以下のコマンドを順番に実行してください:

```
git fetch upstream
git merge upstream/main
git push origin main
```

コンフリクトが発生した場合は、ユーザーに内容を報告して、解決方法を確認してから進めてください。
特に `config/MKB.keymap` はカスタマイズが衝突しやすいファイルです。
