## 概要

Seeeduino XIAO BLE をコントローラーとするモジュラー分割キーボード **MeKaBu (MKB)** の ZMK ファームウェア設定リポジトリ。左右それぞれに交換可能な「Node」モジュール（トラックボール・ジョイスティック・エンコーダー・トラックパッド・追加キー）を装着できる。

## ビルド & CI

ファームウェアは **GitHub Actions のみ** でビルドされる。ローカルにビルドツールチェーンはない。`config/`・`boards/`・`build.yaml` を push するとビルドが自動トリガーされる。

- `.github/workflows/build.yml` — ZMK の `build-user-config.yml` を呼び出し、コンパイル済み `.uf2` を `firmware/<ブランチ名>/` にコミット
- `.github/workflows/draw-keymap.yml` — `config/` または `keymap-drawer/config.yaml` の変更時に `caksoylar/keymap-drawer` で `keymap-drawer/MKB.svg` を自動生成

手動でビルドをトリガーする場合: **Actions → Build ZMK firmware → Run workflow**

## リポジトリ構成

```
config/
  MKB.keymap       ← キーマップ定義（レイヤー・バインディング） ★主な編集対象
  MKB.json         ← keymap-drawer 可視化用の物理キー配置
  west.yml         ← ZMK west マニフェスト（外部モジュール・ドライバーの宣言）
boards/shields/MKB/
  MKB.dtsi         ← ハードウェア基本定義（マトリクス変換・物理レイアウト・LED・エンコーダー）
  MKB_pinctrl_L/R.dtsi  ← 左右それぞれのピン制御定義
  MKB_L_<MODULE>.overlay / .conf  ← 左側モジュール overlay
  MKB_R_<MODULE>.overlay / .conf  ← 右側モジュール overlay
snippets/Default/
  Default.overlay  ← トラックボール・磁気センサー・ジョイスティック・トラックパッドの入力プロセッサ設定
build.yaml         ← ビルドターゲット一覧（シールドの組み合わせ）
firmware/          ← CI がコンパイルした .uf2 をブランチ別に自動コミット
```

## モジュールシステム

左右それぞれ **Base シールド** + **Module シールド** を組み合わせる構成：

| モジュールサフィックス | ハードウェア |
|---|---|
| `TB` | トラックボール（PMW3610、SPI接続） |
| `TBv3` | トラックボール v3（右側のみ） |
| `JOY` | ジョイスティック（アナログ入力） |
| `ENC` | ロータリーエンコーダー |
| `RZT` | ロータリーエンコーダー別バリアント |
| `TPD` | トラックパッド（Cirque） |
| `KEY` | 追加キー |

左側モジュールは `rgbled_adapter`・`nice_oled`・`Default` snippet を標準で含む。右側は `Default` snippet を含まない。

## キーマップの編集

`config/MKB.keymap` がキーマップの主な編集対象。編集方法は2通りある：

- **keymap-editor**（推奨）: ブラウザ上の GUI ツールで編集し、GitHub リポジトリに直接コミットされる。`config/MKB.keymap` と `config/MKB.json` が更新される。
- **直接編集**: `config/MKB.keymap` を手動で編集して push する。

## 外部 ZMK モジュール（west.yml）

使用する ZMK fork は `te9no/zmk` の `v0.3-branch+dya`（公式 zmkfirmware ではない）。主な追加モジュール：

- `zmk-pmw3610-driver` (badjeff) — トラックボールドライバー
- `zmk-analog-input-driver` (badjeff) — ジョイスティック
- `zmk-nice-oled` (te9no) — OLED ディスプレイ
- `zmk-layout-shift` (kot149) — レイアウトシフト動作
- `zmk-input-processor-keybind` (te9no) — 入力プロセッサ
- `cirque-input-module` (petejohanson) — トラックパッド
- `zmk-driver-MLX90393` (te9no) — 磁気センサー
- `zmk-module-runtime-input-processor` (cormoran) — ランタイム入力処理

## 本家との同期

このリポジトリは `te9no/zmk-config-MKB2` の fork。本家の更新を取り込む：

```sh
/sync-upstream
```

または手動で：

```sh
git fetch upstream
git merge upstream/main
git push origin main
```

コンフリクトが発生しやすいのは `config/MKB.keymap`。

## ファームウェアの書き込み

CI ビルド後、`firmware/<ブランチ>/` に `.uf2` ファイルが生成される。XIAO BLE のリセットボタンをダブルクリックしてブートローダーモードに入り、該当の `.uf2` をドラッグ&ドロップして書き込む。Windows 向け PowerShell スクリプトは `scripts/` にある。
