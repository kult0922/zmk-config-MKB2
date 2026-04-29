MKB キーボードのファームウェア書き込みをサポートします。

## 書き込みの流れ

### 左側

左側は常に `MKB_L_MODULE_ENC.uf2`（ロータリーエンコーダー）を使う。モジュールをユーザーに確認しない。

1. ユーザーに XIAO BLE のリセットボタンをダブルクリックするよう案内する
2. `ls /Volumes/` で `XIAO-SENSE` がマウントされているか確認する
3. マウントされていたら即座に `cp` でコピーする：
   ```
   cp firmware/main/firmware/MKB_L_MODULE_ENC.uf2 /Volumes/XIAO-SENSE/
   ```
4. ユーザーに完了を確認する

### 右側

右側はモジュールをユーザーに確認してからコピーする。

1. 右側のモジュールをユーザーに確認する
2. ユーザーに XIAO BLE のリセットボタンをダブルクリックするよう案内する
3. `ls /Volumes/` で `XIAO-SENSE` がマウントされているか確認する
4. マウントされていたら即座に `cp` でコピーする：
   ```
   cp firmware/main/firmware/<ファイル名>.uf2 /Volumes/XIAO-SENSE/
   ```

## 注意事項

- ブランチはデフォルトで `main` を使用する（別ブランチを使う場合はユーザーに確認）
- `cp` 実行時に "fcopyfile failed: Input/output error" が出ても、直後に `XIAO-SENSE` がアンマウントされていれば書き込み成功
- 左側を書き込んだあと、右側のリセットをユーザーに促してから次に進む

## 右側ファイル名一覧

| モジュール | ファイル名 |
|---|---|
| トラックボール v4 | MKB_R_MODULE_TBv4.uf2 |
| トラックボール v3 | MKB_R_MODULE_TBv3.uf2 |
| エンコーダー | MKB_R_MODULE_ENC.uf2 |
| ジョイスティック | MKB_R_MODULE_JOY.uf2 |
| トラックパッド | MKB_R_MODULE_TPD.uf2 |

利用可能なファイルは `find firmware/main/firmware -name "*.uf2"` で確認できる。
