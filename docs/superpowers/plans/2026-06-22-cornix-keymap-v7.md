# Cornix キーマップ v7 実装プラン

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** `config/cornix.keymap` を v7 設計（Dvorak 6 層 + エンコーダ押込 + BSPC + 薙刀休眠）で全書き換えする。

**Architecture:** phase1 でクリーン化した `config/cornix.keymap`（wataruw fork ベース）を takasek v7 設計で丸ごと置き換える。変更対象は 1 ファイルのみ。ビルド検証はローカル不可なので構文チェックのみ実施、実機フラッシュで確認。

**Tech Stack:** ZMK firmware v0.3.0、DTS/devicetree keymap 形式、wataruw/zmk-naginata モジュール

## Global Constraints

- ZMK revision: v0.3.0（config/west.yml・root west.yml ともに固定済み）
- 書込先: `config/cornix.keymap` のみ（`boards/arm/cornix/cornix.keymap` は触らない）
- `config/cornix.conf` は作成しない（cornix_left_defconfig で設定済み）
- `.orig` ファイルは stash 内にあるが commit しない・作成しない
- 50 キー matrix: pos0-11 (row0) / pos12-23 (row1) / pos24-37 (row2, pos30=左ノブ押込, pos31=右ノブ押込) / pos38-49 (row3 thumb)
- 全層に sensor-bindings が必要（エンコーダ 2 個分）

---

## File Map

| ファイル | 操作 |
|----------|------|
| `config/cornix.keymap` | 全書き換え（唯一の変更対象） |

---

## Task 1: `config/cornix.keymap` 全書き換え

**Files:**
- Modify: `config/cornix.keymap`

**Verification (ZMK はローカルビルド不可のため構文チェックのみ):**
- `{` と `}` の数が一致
- 各層のバインディング数が 50（後述の期待値と一致）
- 未定義シンボルが無い（grep で確認）

---

### 完成形コード（そのまま書き込む）

- [ ] **Step 1: ファイルを書き込む**

```c
/*
 * Cornix LP — ZMK keymap v7+
 * L0 BASE_DV / L1 QWERTY / L2 SYM / L3 FUNC / L4 TENKEY / L5 NAGINATA
 *
 * 配列方針: OS=US固定 / Dvorak変換はZMK / IME=LANG+かわせみ4
 *
 * エンコーダ押込 (pos30=左ノブ, pos31=右ノブ):
 *   BASE/QW/SYM: C_MUTE / F24(音声入力)
 *   FUNC: C_PP(再生停止) / LCLK(左クリック)
 *   TENKEY/NAG: C_MUTE / trans
 *
 * 薙刀式: ng_on/ng_off マクロ定義済み。入口(かなキー)は未配線=休眠。
 * 開放時: lt_func tap を ng_on に差し替えるだけ。
 *
 * 要実機確認: GLOBE keycode / 右ノブ pos31 / 囲み3段挙動 / ng マッピング
 */

#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/pointing.h>
#include <behaviors/naginata.dtsi>

#define BASE_DV  0
#define QWERTY   1
#define SYM      2
#define FUNC     3
#define TENKEY   4
#define NAGINATA 5

/ {
    behaviors {
        lt_tk: lt_tenkey {
            compatible = "zmk,behavior-hold-tap";
            #binding-cells = <2>;
            tapping-term-ms = <200>;
            flavor = "hold-preferred";
            bindings = <&mo>, <&kp>;
        };
        lt_func: lt_func {
            compatible = "zmk,behavior-hold-tap";
            #binding-cells = <2>;
            tapping-term-ms = <200>;
            flavor = "hold-preferred";
            bindings = <&mo>, <&kp>;
        };
        td_shift: td_shift {
            compatible = "zmk,behavior-tap-dance";
            #binding-cells = <0>;
            tapping-term-ms = <200>;
            bindings = <&kp LSHFT>, <&caps_word>;
        };
        td_paren: td_paren {
            compatible = "zmk,behavior-tap-dance";
            #binding-cells = <0>;
            tapping-term-ms = <150>;
            bindings = <&kp LPAR>, <&wrap_par>;
        };
        td_angle: td_angle {
            compatible = "zmk,behavior-tap-dance";
            #binding-cells = <0>;
            tapping-term-ms = <150>;
            bindings = <&kp LT>, <&wrap_ang>;
        };
        mm_9: mm_9 {
            compatible = "zmk,behavior-mod-morph";
            #binding-cells = <0>;
            bindings = <&kp N9>, <&td_paren>;
            mods = <(MOD_LSFT|MOD_RSFT)>;
        };
        mm_comma: mm_comma {
            compatible = "zmk,behavior-mod-morph";
            #binding-cells = <0>;
            bindings = <&kp COMMA>, <&td_angle>;
            mods = <(MOD_LSFT|MOD_RSFT)>;
        };
    };

    macros {
        wrap_par: wrap_par {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            wait-ms = <0>;
            tap-ms = <0>;
            bindings = <&kp LPAR &kp RPAR &kp LEFT>;
        };
        wrap_ang: wrap_ang {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            wait-ms = <0>;
            tap-ms = <0>;
            bindings = <&kp LT &kp GT &kp LEFT>;
        };
        gui_qw: gui_qw {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            wait-ms = <0>;
            tap-ms = <0>;
            bindings
                = <&macro_press &kp LGUI &mo QWERTY>
                , <&macro_pause_for_release>
                , <&macro_release &kp LGUI &mo QWERTY>
                ;
        };
        ctl_qw: ctl_qw {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            wait-ms = <0>;
            tap-ms = <0>;
            bindings
                = <&macro_press &kp LCTRL &mo QWERTY>
                , <&macro_pause_for_release>
                , <&macro_release &kp LCTRL &mo QWERTY>
                ;
        };
        // 薙刀ON: IME→かな + 層トグル（入口未配線=休眠）
        ng_on: ng_on {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&macro_tap &kp LANGUAGE_1 &kp INTERNATIONAL_4 &tog NAGINATA>;
        };
        // 薙刀OFF: IME→英数 + 層トグル（NAGINATA層pos41に配置）
        ng_off: ng_off {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&macro_tap &kp LANGUAGE_2 &kp INTERNATIONAL_5 &tog NAGINATA>;
        };
    };

    combos {
        compatible = "zmk,combos";
        ht_esc {
            bindings = <&kp ESC>;
            key-positions = <19 20>;
            timeout-ms = <40>;
            layers = <BASE_DV>;
        };
        jk_esc {
            bindings = <&kp ESC>;
            key-positions = <19 20>;
            timeout-ms = <40>;
            layers = <QWERTY>;
        };
    };

    keymap {
        compatible = "zmk,keymap";

        // L0: Dvorak ベース
        // pos11=&tog QWERTY (v7の&to QWERTYから変更: 層スタック保持で薙刀from-face復帰を実現)
        // pos30=C_MUTE(左ノブ押込) pos31=F24(右ノブ押込=未実機確認)
        // pos45=BSPC (v7のDELから変更)
        base_dvorak {
            display-name = "Dvorak";
            bindings = <
&kp TAB    &kp SQT    &kp COMMA  &kp DOT   &kp P               &kp Y                              &kp F      &kp G    &kp C      &kp R     &kp L     &tog QWERTY
&ctl_qw    &kp A      &kp O      &kp E     &kp U               &kp I                              &kp D      &kp H    &kp T      &kp N     &kp S     &kp MINUS
&td_shift  &kp SEMI   &kp Q      &kp J     &kp K               &kp X      &kp C_MUTE  &kp F24    &kp B      &kp M    &kp W      &kp V     &kp UP    &kp Z
&gui_qw    &kp GLOBE  &kp LALT             &lt_tk TENKEY LANG2  &mo SYM    &kp SPACE   &kp ENTER  &kp BSPC   &lt_func FUNC LANG1             &kp LEFT  &kp DOWN   &kp RIGHT
            >;
            sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp PG_DN PG_UP>;
        };

        // L1: QWERTY
        // pos11=&tog QWERTY (トグルOFFで元の面に戻る)
        // v7の&to BASE_DVから変更
        qwerty {
            display-name = "QWERTY";
            bindings = <
&kp TAB    &kp Q      &kp W      &kp E     &kp R               &kp T                              &kp Y      &kp U    &kp I      &kp O     &kp P     &tog QWERTY
&ctl_qw    &kp A      &kp S      &kp D     &kp F               &kp G                              &kp H      &kp J    &kp K      &kp L     &trans    &kp BSLH
&td_shift  &kp Z      &kp X      &kp C     &kp V               &kp B      &kp C_MUTE  &kp F24    &kp N      &kp M    &kp COMMA  &kp DOT   &kp UP    &kp FSLH
&gui_qw    &kp GLOBE  &kp LALT             &lt_tk TENKEY LANG2  &mo SYM    &kp SPACE   &kp ENTER  &kp BSPC   &lt_func FUNC LANG1             &kp LEFT  &kp DOWN   &kp RIGHT
            >;
            sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp PG_DN PG_UP>;
        };

        // L2: SYM 記号・数字
        // pos45=&trans (BASE/QWERTYのBSPCに透過)
        sym {
            display-name = "Sym";
            bindings = <
&kp ESC    &kp N1     &kp N2     &kp N3    &kp N4              &kp N5                             &kp N6     &kp N7   &kp N8     &mm_9     &kp N0    &kp GRAVE
&ctl_qw    &trans     &trans     &trans    &kp SEMI            &kp MINUS                          &kp EQUAL  &kp SQT  &trans     &kp LBKT  &kp RBKT  &kp BSLH
&td_shift  &trans     &trans     &trans    &trans              &trans     &kp C_MUTE  &kp F24    &trans     &trans   &mm_comma  &kp DOT   &kp UP    &kp FSLH
&gui_qw    &kp GLOBE  &kp LALT             &trans              &trans     &kp SPACE   &kp ENTER  &trans     &trans              &kp LEFT  &kp DOWN   &kp RIGHT
            >;
            sensor-bindings = <&inc_dec_kp C_BRI_UP C_BRI_DN &inc_dec_kp PG_DN PG_UP>;
        };

        // L3: FUNC F1-F15・矢印・メディア
        // pos30=C_PP(再生停止) pos31=LCLK(左クリック)
        // pos45=DEL (BSPCの代替配置)
        func {
            display-name = "Func";
            bindings = <
&kp TAB    &kp F1     &kp F2     &kp F3    &kp F4              &kp F5                             &kp F6     &kp F7   &kp F8     &kp F9    &kp F10   &trans
&ctl_qw    &kp F11    &kp F12    &kp F13   &kp F14             &kp F15                            &kp LEFT   &kp DOWN &kp UP     &kp RIGHT &kp K_APP &trans
&td_shift  &trans     &trans     &trans    &trans              &trans     &kp C_PP    &mkp LCLK  &trans     &trans   &trans     &trans    &kp HOME  &trans
&gui_qw    &kp F23    &kp LALT             &trans              &trans     &trans      &trans     &kp DEL    &trans              &kp K_BACK &kp END  &kp K_FORWARD
            >;
            sensor-bindings = <&inc_dec_kp C_NEXT C_PREV &inc_dec_kp PG_DN PG_UP>;
        };

        // L4: TENKEY テンキー
        // v7から追加: sensor-bindings（欠落していた）
        tenkey {
            display-name = "10Key";
            bindings = <
&kp TAB    &trans     &trans     &trans    &trans              &trans                             &trans     &kp N7   &kp N8     &kp N9    &kp N0    &trans
&ctl_qw    &trans     &trans     &trans    &trans              &kp MINUS                          &kp EQUAL  &kp N4   &kp N5     &kp N6    &trans    &kp BSLH
&td_shift  &trans     &trans     &trans    &trans              &trans     &kp C_MUTE  &trans     &trans     &kp N1   &kp N2     &kp N3    &trans    &kp KP_DIVIDE
&gui_qw    &kp GLOBE  &kp LALT             &trans              &trans     &kp SPACE   &kp ENTER  &trans     &kp N0              &kp DOT   &trans    &trans
            >;
            sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp PG_DN PG_UP>;
        };

        // L5: NAGINATA 薙刀式（現在休眠: 入口への導線なし）
        // pos30=C_MUTE pos31=trans
        // pos41=&ng_off（万一入ったら出られるよう出口を置く）
        // &ng は wataruw/eswai zmk-naginata モジュールの behavior
        // 開放時要確認: Dvorak土台での物理位置→仮名マップ
        naginata {
            display-name = "Naginata";
            bindings = <
&kp TAB    &ng Q      &ng W      &ng E     &ng R               &ng T                              &ng Y      &ng U    &ng I      &ng O     &ng P     &trans
&ctl_qw    &ng A      &ng S      &ng D     &ng F               &ng G                              &ng H      &ng J    &ng K      &ng L     &ng MINUS &trans
&td_shift  &ng Z      &ng X      &ng C     &ng V               &ng B      &kp C_MUTE  &trans     &ng N      &ng M    &ng COMMA  &ng DOT   &trans    &ng FSLH
&gui_qw    &kp GLOBE  &kp LALT             &ng_off             &trans     &ng SPACE   &ng ENTER  &kp DEL    &trans              &trans    &trans    &trans
            >;
            sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp PG_DN PG_UP>;
        };
    };
};
```

- [ ] **Step 2: ブレース (`{`/`}`) の数を確認**

```bash
python3 -c "
c = open('config/cornix.keymap').read()
o, cl = c.count('{'), c.count('}')
print(f'open={o} close={cl}', 'OK' if o==cl else 'MISMATCH')
"
```

期待値: `open=N close=N OK`（数値は一致していれば何でもよい）

- [ ] **Step 3: 各層のバインディング数を確認（50個/層）**

```bash
python3 - <<'EOF'
import re, sys
text = open('config/cornix.keymap').read()
# keymap ブロックを抽出してから各 layer を解析
layers = re.findall(r'display-name\s*=\s*"([^"]+)"[^>]*bindings\s*=\s*<([^>]+)>', text, re.DOTALL)
ok = True
for name, body in layers:
    # & で始まるトークンを数える（&kp X は1バインディング, &lt_tk A B は1バインディング）
    # ZMK の DTS では < > 内の各バインディングは & で始まる
    bindings = re.findall(r'&\w+', body)
    # ただし &macro_press 等の macro control は除外
    bindings = [b for b in bindings if b not in ('&macro_press','&macro_release','&macro_pause_for_release','&macro_tap')]
    print(f"{name}: {len(bindings)} bindings {'OK' if len(bindings)==50 else 'EXPECTED 50'}")
    if len(bindings) != 50:
        ok = False
sys.exit(0 if ok else 1)
EOF
```

期待値: 全 6 層が `50 bindings OK`

- [ ] **Step 4: 未定義参照を grep チェック**

```bash
# behaviors/macros で使うシンボルが定義されているか確認
grep -E '&(lt_tk|lt_func|td_shift|td_paren|td_angle|mm_9|mm_comma|wrap_par|wrap_ang|gui_qw|ctl_qw|ng_on|ng_off|caps_word|ng)\b' config/cornix.keymap | wc -l
# → 0 より大きければ参照あり（定義側も同じファイル内にある）

# GLOBE が keys.h に存在するかは実機/CI でしか確認できない
# C_PP, C_NEXT, C_PREV, mkp LCLK も同様
echo "GLOBE/C_PP/mkp は実機確認必要（ローカルビルド不可）"
```

- [ ] **Step 5: コミット**

```bash
git add config/cornix.keymap
# .orig・cornix.conf は staging しない
git status  # staged に cornix.keymap のみ、.orig 等が含まれないことを確認
git commit -m "$(cat <<'EOF'
feat(keymap): Dvorak 6層キーマップ v7 を実装

- L0 BASE_DV / L1 QWERTY / L2 SYM / L3 FUNC / L4 TENKEY / L5 NAGINATA
- エンコーダ押込: BASE/QW/SYM=C_MUTE+F24, FUNC=C_PP+LCLK, TENKEY/NAG=C_MUTE+trans
- BSPC を pos45 (BASE/QWERTY) に復活; DEL は FUNC pos45 へ
- GLOBE(macOS Fn) 維持: pos39 各層
- 囲み機能: mm_9→td_paren→wrap_par の3段ネスト維持
- QWERTY を &tog 化（層スタック保持→薙刀から元の面に戻れる）
- 薙刀式: ng_on/ng_off 定義済み、入口未配線で休眠
- 全層に sensor-bindings 付与 (TENKEY は v7 で欠落していた)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

---

## チェックリスト（実機フラッシュ後）

以下はビルド環境がないため計画書に記録しておく。実機確認事項:

- [ ] `&kp GLOBE` がビルドエラーなく動作する（v0.3.0 の keys.h に定義ありかを確認）
- [ ] 右ノブ押込が pos31 に対応している（違えば cornix-layouts.dtsi の RC 番号を調べて修正）
- [ ] 囲み機能（Shift+9 を 2 連打 → `()` + カーソル内側）が想定どおり動く
- [ ] 薙刀開放時（かなキーを ng_on に変更後）: Dvorak 土台での `&ng` マッピングを確認

---

## 設計変更サマリー（v7 stash との差分）

| 変更点 | v7 stash | このプラン |
|--------|----------|-----------|
| ファイル位置 | `boards/arm/...` + `config/...` 両方 | `config/cornix.keymap` のみ |
| cornix.conf | 作成 | 作成しない |
| .orig | boards/ に追加 | 作成しない |
| pos30/31 (全層) | `&none &none` | 層別ノブ押込バインディング |
| pos45 BASE/QW | `&kp DEL` | `&kp BSPC` |
| pos45 FUNC | `&trans` | `&kp DEL` |
| QWERTY 往復 | `&to QWERTY` / `&to BASE_DV` | `&tog QWERTY` / `&tog QWERTY` |
| NAGINATA 出口 | `&to BASE_DV` | `&ng_off` |
| ng_on/ng_off | なし | 定義済み（入口未配線） |
| TENKEY sensor-bindings | 欠落 | 追加 |
