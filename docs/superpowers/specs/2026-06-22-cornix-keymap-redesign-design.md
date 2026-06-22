# Cornix キーマップ再設計（v7 ベース）設計書

作成日: 2026-06-22
ブランチ: `cornix-keymap-redesign`（`cleanup-dead-homerow-mods` = phase1 の上）

## 目的

takasek が fork した wataruw/zmk-keyboard-cornix に対し、takasek 独自のキーマップ
（"v7" = スプレッドシート準拠の Dvorak 6 層構成）を、意味のあるクリーンな差分として
作り直す。staged にあった v7 草案は `.orig` 混入・重複 conf・不整合 west テンプレ・
naginata 空層化などの問題を抱えていたため、設計を中間レベルで見直して再実装する。

スコープ = **中間**: v7 のレイアウト/ビヘイビア設計は維持し、特定の不満点のみ設計変更。

## ベース設計（v7 維持）

### レイヤー構成
- L0 `BASE_DV` Dvorak（常時 active な土台）
- L1 `QWERTY`
- L2 `SYM` 記号・数字
- L3 `FUNC` F1〜F15・矢印・メディア
- L4 `TENKEY` テンキー
- L5 `NAGINATA` 薙刀式（後述・当面休眠）

### カスタム behavior
- `lt_tk`（hold-tap）: hold=MO(TENKEY) / tap=LANG2(英数)
- `lt_func`（hold-tap）: hold=MO(FUNC) / tap=LANG1(かな)
- `td_shift`（tap-dance）: 1tap=LSHFT / 2tap=CapsWord
- `td_paren`（tap-dance）: 1tap=`(` / 2tap=`wrap_par`
- `td_angle`（tap-dance）: 1tap=`<` / 2tap=`wrap_ang`
- `mm_9`（mod-morph）: 素=`9` / Shift時=`td_paren`
- `mm_comma`（mod-morph）: 素=`,` / Shift時=`td_angle`
- `wrap_par`（macro）: `( ) ←`（囲んで内側へ）
- `wrap_ang`（macro）: `< > ←`
- `gui_qw` / `ctl_qw`（macro）: GUI/Ctrl 押下しつつ QWERTY 層を一時併用

### combo
- `ht_esc`: key-positions=<19 20>（=H+T）@ L0 → ESC、timeout 40ms
- `jk_esc`: key-positions=<19 20>（=J+K）@ L1 → ESC、timeout 40ms

### matrix（Cornix 50 キー、position 番号）
```
 0  1  2  3  4  5        6  7  8  9 10 11
12 13 14 15 16 17       18 19 20 21 22 23
24 25 26 27 28 29 30 31 32 33 34 35 36 37
38 39 40 41 42 43       44 45 46 47 48 49
```
- pos30 = 左ノブ押込、pos31 = 右ノブ押込（標準 48 + 中央 2 = 50。pos30 は
  wataruw 原版が `&kp C_MUTE` をバインド済 = 左ノブ押込である実績）
- pos38-43 = 左親指、pos44-49 = 右親指

## 設計変更 4 点

### ① エンコーダ押込（pos30 / pos31）をレイヤー別にバインド
v7 草案はここを `&none &none` にして押込機能を喪失していた。復活させる。

| 層 | pos30(左ノブ) | pos31(右ノブ) |
|----|----|----|
| BASE_DV / QWERTY | `&kp C_MUTE` | `&kp F24`（音声入力） |
| SYM | `&kp C_MUTE` | `&kp F24` |
| FUNC | `&kp C_PP`（再生/停止） | `&mkp LCLK`（左クリック） |
| TENKEY / NAGINATA | `&kp C_MUTE` | `&trans` |

注: 左ノブ=pos30 は wataruw 実績あり。右ノブ=pos31 は原版が `&none` のため
**未検証** → 実機で 1 回押して反応位置を確認すること（pos31 でなければ要修正）。

### ② BSPC 不在の解消
v7 草案は base に backspace が無く DEL のみ。右親指列に BSPC を復活。
- base 右親指: pos45 = `&kp BSPC`（v7 でここは DEL だった）
- `DEL` は FUNC 層の同位置 pos45 に置く（thumb=BSPC、FUNC層=DEL）
- 配置の最終確定は実機の好みで調整可。

### ③ GLOBE キー（pos39、各層左親指）
用途 = macOS 入力ソース/Fn（🌐）。`&kp GLOBE` を使用。
- リスク: pinned ZMK（v0.3.0）の keys.h に `GLOBE` 定義が在るか未検証。
- 対応: CI/実機ビルドで確認。未定義でビルド失敗する場合は CONFIG 調整または
  代替 keycode へフォールバック（用途は維持）。

### ④ 囲み機能（mod-morph + tap-dance + macro、設計 A 維持）
Shift+9 を 2 連打 → `()`、Shift+, を 2 連打 → `<>`（カーソル内側）。
`mm_9` → Shift時 `td_paren` → 2tap `wrap_par` の 3 段ネストをそのまま採用。
- リスク: 3 段ネストの実機挙動は未検証 → フラッシュ後に動作確認。

## 薙刀式（NAGINATA）の設計

### 方針: 仕組みは作り込むが当面休眠（後で開放）
学習負荷を抑えるため当面は英字 Dvorak のみで運用。薙刀の機構は設計に織り込むが
入口（かなキー）は未配線にしておき、慣れたら配線するだけで開放できる状態にする。

### `&ng` とは
wataruw/eswai の `zmk-naginata` モジュールが提供する behavior。日本語の同時押し
入力（同時に押されたキー群 → 仮名）を実現するため、薙刀層では全キーを `&ng X`
でラップしてエンジンに通す。`#include <behaviors/naginata.dtsi>` が必要。
モジュールは config/west.yml に既に取り込み済み（wataruw remote、revision main）。

### 「来た面に戻る」を `&tog` で実現（単一層）
要件: QWERTY 面・Dvorak 面のどちらからでも「かな」で薙刀へ入り、「英数」で
**元の面**に戻る。

`&to`（行き先固定）では原点を覚えられない。代わりに層スタックとトグル状態を使う:

- **QWERTY を `&tog QWERTY` に変更**（v7 の `&to` 往復から変更）。これにより
  QWERTY は L0 の上に積まれ、on/off 状態が残る。
- 薙刀入口/出口は `&tog NAGINATA`。
  - Dvorak中(active={0}) → かな → {0,5} → 英数 → {0} = Dvorak に戻る
  - QWERTY中(active={0,1}) → かな → {0,1,5} → 英数 → {0,1} = QWERTY に戻る
- 原点は QWERTY トグルの on/off が記憶。薙刀層は **1 枚**で済む。

NAGINATA(L5) は最上位（>QWERTY=1）なので入った時は必ず勝つ。`&mo SYM/FUNC` とも
番号上競合しない。

### IME 同期マクロ（定義はするが入口は休眠）
`&tog` 単体では macOS の入力モードが切り替わらないため、薄いマクロで包む:
```
ng_on:  &macro_tap &kp LANG1 &kp INTL4    +  &tog NAGINATA   // かな + 層
ng_off: &macro_tap &kp LANG2 &kp INTL5    +  &tog NAGINATA   // 英数 + 層
```
（LANG1=かな、LANG2=英数。INTL4/5 は wataruw 原版 ng_on/ng_off 準拠）

休眠の作り込み:
- NAGINATA 層（`&ng` 配列）と ng_on / ng_off マクロは **定義する**
- **かなキーに ng_on を割り当てない**（かなは当面 `lt_func` tap=LANG1 のまま、層へ飛ばさない）
- `&tog QWERTY` 化は今実施（薙刀と無関係に無害、将来の前提）
- 開放時 = かなキーの binding を ng_on に差し替えるだけ
- 出口 ng_off は NAGINATA 層内に置いておく（入っても出られるように）。ただし入口が
  無いので通常到達しない。

薙刀層の `&ng` 配列は wataruw 原版の実装を保全（QWERTY 土台前提のマッピング）。
開放時に Dvorak 土台での物理位置→仮名マップを実機で検証すること。

## 同時に行う掃除（中間方針）

- `.orig` バックアップファイルを一切含めない
- `config/cornix.conf` を**作らない** — POINTING / EC11 / STUDIO は全て
  `cornix_left_defconfig` に既出。重複不要。
- keymap は `config/cornix.keymap` 単一を正とする。board の stock keymap
  (`boards/arm/cornix/cornix.keymap`) は board 既定として触らない（二重管理しない）
- 全層に `sensor-bindings` を付与（v7 草案で tenkey/naginata が欠落していた）
- west の english / naginata テンプレートは**作らない**（config/west.yml が既に
  v0.3.0 + naginata モジュール込み）

## 成果物
- `config/cornix.keymap`（再設計後の全層）
- 設計書（本ファイル）

## 検証の限界（正直に明記）
- ローカルに west ビルド環境が無く、fork は GitHub Actions 無効のため
  **ローカルビルド検証は不可**。ビルド可否は実機フラッシュ / 別途 CI 有効化時に確認。
- 以下は実機フラッシュ後に確認が必要:
  - `&kp GLOBE` がビルド・動作するか（③）
  - 囲み機能 3 段ネストの実挙動（④）
  - 右ノブ押込が pos31 か（①）
  - （薙刀開放時）Dvorak 土台での `&ng` マッピング

## 非目標（YAGNI）
- 薙刀入口の配線（当面休眠）
- 設計判断そのものの白紙見直し（Dvorak 採用・6 層構成は維持）
- board 定義（upstream 由来）への変更
- 不要な west テンプレート / 重複 conf
