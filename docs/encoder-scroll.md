# ロータリーエンコーダーでのスクロール設定

## 概要

左手のロータリーエンコーダー（MKB_L_MODULE_ENC）でページスクロールを実現するための設定と、動作しなかった原因の記録。

## 設定箇所

`boards/shields/MKB/MKB_behaviors.dtsi`

```dts
#define ZMK_POINTING_DEFAULT_SCRL_VAL 80

#include <dt-bindings/zmk/pointing.h>

/ {
    enc_scroll: enc_scroll {
        compatible = "zmk,behavior-sensor-rotate";
        #sensor-binding-cells = <0>;
        tap-ms = <20>;
        bindings = <&msc SCRL_UP>, <&msc SCRL_DOWN>;
    };
};
```

`config/MKB.keymap` の各レイヤーに以下を記述：

```
sensor-bindings = <&enc_scroll &inc_dec_kp PG_UP PG_DN>;
```

## 動作しなかった原因

### 原因1: `msc` に生の整数を渡していた

`msc 80` のように直接整数を渡していたが、`msc`（`zmk,behavior-input-two-axis`）の引数は `pointing.h` で定義されたエンコード済みマクロ（`SCRL_UP` / `SCRL_DOWN`）を使う必要がある。

- `SCRL_UP = MOVE_Y(ZMK_POINTING_DEFAULT_SCRL_VAL) = val & 0xFFFF`
- `SCRL_DOWN = MOVE_Y(-ZMK_POINTING_DEFAULT_SCRL_VAL)`

`#define ZMK_POINTING_DEFAULT_SCRL_VAL 80` を `#include <dt-bindings/zmk/pointing.h>` より前に定義することでスクロール量を設定する。

### 原因2（本質的な原因）: `tap-ms` と `trigger-period-ms` のタイミング競合

`msc` は `zmk,behavior-input-two-axis` として実装されており、「速度制御型」のビヘービア：

1. キー pressed → speed を加算 → `trigger-period-ms`（デフォルト **16ms**）後にスクロールイベントをスケジュール
2. キー released → speed を減算（0に戻る） → tick_work をキャンセル

一方、`zmk,behavior-sensor-rotate` の `tap-ms`（pressed から released までの間隔）のデフォルトは **5ms**。

5ms < 16ms なので、スクロールイベントが送信される前に released が実行されてキャンセルされていた。

**解決策：** `tap-ms = <20>` を設定し、`trigger-period-ms`（16ms）より長くすることで、1回の回転ごとに最低1回スクロールイベントが送信されるようになった。

## 関連ファイル

| ファイル | 役割 |
|---|---|
| `boards/shields/MKB/MKB_behaviors.dtsi` | `enc_scroll` ビヘービア定義 |
| `config/MKB.keymap` | 各レイヤーの `sensor-bindings` |
| `boards/shields/MKB/MKB_L_ENC.overlay` | エンコーダーのピン・ステップ設定 |

## 注意

- `MKB_behaviors.dtsi` は `MKB.dtsi` から include されるため L/R 全ビルドで有効
- `config/MKB.keymap` は keymap-editor で保存すると上書きされるため、`enc_scroll` の定義は `MKB_behaviors.dtsi` で管理する
- `C_AC_SCROLL_UP/DOWN`（Consumer Control）は macOS で認識されないため `msc` を使う
