MKB キーボードのファームウェア書き込みをサポートします。

## 書き込みの流れ

1. ユーザーに書き込む左右それぞれのファイル名を確認する（未指定の場合）
2. 指定ファイルが `firmware/main/firmware/` に存在するか確認する
3. 左側から書き込む：
   - XIAO BLE のリセットボタンをダブルクリックするよう案内
   - `/Volumes/XIAO-SENSE` がマウントされるまで待つ（`ls /Volumes/` で確認）
   - マウントを確認したら `cp` でファイルをコピー
   - `XIAO-SENSE` がアンマウントされたことで書き込み成功を確認
4. 右側も同様に繰り返す

## 注意事項

- ブランチはデフォルトで `main` を使用する（別ブランチを使う場合はユーザーに確認）
- `cp` 実行時に "fcopyfile failed: Input/output error" が出ても、直後に `XIAO-SENSE` がアンマウントされていれば書き込み成功
- コピーコマンド: `cp firmware/main/firmware/<ファイル名>.uf2 /Volumes/XIAO-SENSE/`
- 左側を書き込んだあと、右側のリセットをユーザーに促してから次に進む

## ファイル名の例

| 側 | モジュール | ファイル名 |
|---|---|---|
| 左 | エンコーダー | MKB_L_MODULE_ENC.uf2 |
| 左 | トラックボール | MKB_L_MODULE_TB.uf2 |
| 右 | トラックボール v4 | MKB_R_MODULE_TBv4.uf2 |
| 右 | トラックボール v3 | MKB_R_MODULE_TBv3.uf2 |

利用可能なファイルは `find firmware/main/firmware -name "*.uf2"` で確認できる。
