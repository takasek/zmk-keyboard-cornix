# CLAUDE.md — Cornix キーマップ (takasek fork)

このリポジトリは ZMK ファームのキーボード定義 + 個人キーマップ設定。作業前にこの fork 状況を把握すること。

## fork チェーン

- `hitsmaxft/zmk-keyboard-cornix` — 大元
- → `wataruw/zmk-keyboard-cornix` — 薙刀式対応・日本語キーマップを足した親（このリポジトリの upstream）
- → `takasek/zmk-keyboard-cornix` — このリポジトリ

`git log` の作者が `wataruw <saruchichi@gmail.com>` のコミット（`5137e75 README.mdの日本語化` まで）が upstream 由来。それ以降が takasek の独自カスタマイズ。

upstream remote は未設定。upstream へ還元する場合は `gh pr create --repo wataruw/zmk-keyboard-cornix --head takasek:<branch> --base main`。

## ビルド構成

- 実キーマップは `config/cornix.keymap`。`boards/arm/cornix/cornix.keymap` は ZMK 既定サンプルで `config` 側が上書きする死蔵ファイル。キーマップ編集は必ず `config/cornix.keymap` を触る。
- feature 系 CONFIG は `boards/arm/cornix/cornix_left_defconfig` で有効化済み。`CONFIG_EC11`（エンコーダ）・`CONFIG_ZMK_POINTING`（マウス）・`CONFIG_ZMK_STUDIO`。よって `&inc_dec_kp` `&mkp` `&mmv` `&studio_unlock` はビルド可能で、keymap 側に重複の conf は不要。
- CI は `.github/workflows/build.yml`（`on: push`、paths は `boards/*` `config/*`）。**pull_request トリガは無い**。GitHub の fork はデフォルトで Actions 無効なので、takasek の fork では push しても自動ビルドが走らないことがある。
- ビルド対象マトリクスは `build.yaml`。`cornix_left` は studio スニペット + `-DCONFIG_ZMK_STUDIO=y` 付き。

## west.yml が2つある

- `config/west.yml` — ZMK user-config ビルド用（zmkfirmware の build-user-config がこれを使う）。zmk を `v0.3.0` に固定（Zephyr Main 起因の不具合回避＝「Zephyr Mainをやめた」）。
- ルートの `west.yml` — board をモジュールとして扱う開発用マニフェスト。zmk pin はルートも `v0.3.0` に揃える（両者がずれると board 単体ビルドで Zephyr 4.1 起因の不具合を踏む）。
- `zmk-naginata` は wataruw の fork を使用（remote=wataruw）。

## 薙刀式

- naginata 層は `&ng` behavior（`wataruw/zmk-naginata`、`#include <behaviors/naginata.dtsi>`）。
- 入口（かなキー配線）を外せば層は休眠する。upstream の旧構成では `combo_ng_on`（H+J）で `ng_on` マクロを叩いて入っていた。
- 開放するときは入口マクロ（`ng_on`/`ng_off`、`LANG1+INTL4` / `LANG2+INTL5` でIME切替）を任意キーに配線する。

## 注意

- `&ng` の物理位置→仮名マップは QWERTY 土台前提。Dvorak 土台に移植する場合は実機でマッピングを確認する。
- 手元に west ビルド環境が無い場合、キーマップ変更の実ビルド検証はできない。静的チェック（記号の残参照・brace 収支）止まりであることを明示すること。
