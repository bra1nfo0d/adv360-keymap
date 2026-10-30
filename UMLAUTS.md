# German umlauts on the Advantage360 Pro (US layout)

## 1. Goal

Type **ä, ö, ü** and **ß** (and **Ä, Ö, Ü** with Shift) on a Kinesis Advantage360 Pro while the computer keeps a US-based keyboard layout. Hold the **Fn** key and press **A, O, U** or **S**. This uses only four bindings on the existing Fn layer, with no additional layers or behaviors (details in the [umlaut table](#4-umlaut-table)).

## 2. Prerequisites

The keyboard does not send the character "ä" itself. Each umlaut binding sends **Right Alt + a letter** (`RA(Q)`, `RA(P)`, `RA(Y)`, `RA(S)`). On layouts that have an AltGr key, Right Alt acts as AltGr, and the layout turns AltGr+Q/P/Y/S into ä/ö/ü/ß. That is why the computer's layout setting matters:

| OS | Layout to select | AltGr + Q / P / Y / S | Shift + AltGr + Q / P / Y / S |
| --- | --- | --- | --- |
| Windows | **United States-International** | ä / ö / ü / ß | Ä / Ö / Ü / § |
| Linux | **`us(altgr-intl)`**, shown as "English (intl., with AltGr dead keys)" | ä / ö / ü / ß | Ä / Ö / Ü / § |
| macOS | not supported without changes, see below | – | – |

- **Windows:** Settings → Time & language → Language & region → `…` next to your language → Language options → Keyboards → Add a keyboard → *United States-International*. Win+Space switches between installed layouts ([Microsoft](https://support.microsoft.com/en-gb/help/4496404)). The mappings in the table come from the Unicode CLDR data for this layout.
- **Linux:** select "English (intl., with AltGr dead keys)" in your desktop's keyboard settings, or on X11 run `setxkbmap -layout us -variant altgr-intl`. In xkeyboard-config's `symbols/us`, `altgr-intl` includes the `us(intl)` mappings shown above and `level3(ralt_switch)`, which makes Right Alt the AltGr key.
- **macOS:** on the standard *U.S.* layout, Option + Q / P / Y types œ / π / ¥. Only Option + S gives ß (Unicode CLDR data for macOS "U.S."). On that layout, macOS types umlauts with the dead key Option + U followed by the vowel. Using this setup on a Mac therefore needs different bindings, which it does not include. *Not verified:* whether the macOS layout "U.S. International – PC" maps Option + Q/P/Y/S the same way Windows does.

## 3. How it works

A ZMK keymap is a stack of layers. When you press a key, ZMK uses the binding of the highest active layer at that key position. A `&trans` (transparent) binding passes the press down to the next active layer below.

The Fn key is bound to `&mo 2` (momentary layer). Layer 2, the Fn layer, is enabled immediately when you press the key and disabled immediately when you release it. On the Fn layer, the positions of A, O, U and S now hold `&kp RA(...)` bindings, so while Fn is held these keys send AltGr + Q/P/Y/S. Modifiers you hold are added to the output: Shift + Fn + A arrives as Shift+AltGr+Q, which gives Ä. The Shift keys are `&trans` on the Fn layer, so they keep working while Fn is held.

Advantages:

- **No timing:** nothing has to be pressed within a time window.
- **No delay:** the letter keys are sent immediately, nothing is held back.
- **No false triggers:** an umlaut only appears while Fn is held.

Trade-off: before this change, the four positions were `&trans` on the Fn layer. Fn + A, O, U or S therefore typed the key of the layer below: a plain a, o, u or s on the Base layer. With the Keypad layer toggled on, Fn + U and Fn + O gave keypad 7 and keypad 9. These combinations now type umlauts instead. All other Fn-layer bindings (F1–F12 on the number row and everything else) are unchanged.

## 4. Umlaut table

| Output | With Shift | Keys | Binding |
| --- | --- | --- | --- |
| ä | Ä | Fn + A | `&kp RA(Q)` |
| ö | Ö | Fn + O | `&kp RA(P)` |
| ü | Ü | Fn + U | `&kp RA(Y)` |
| ß | § | Fn + S | `&kp RA(S)` |

The Fn key exists twice: positions 60 and 75, the bottom-left and bottom-right corner keys. The letters are at positions 29 (A), 25 (O), 23 (U) and 30 (S), see [assets/key-positions.md](assets/key-positions.md). A and S are on the left hand and O and U on the right, so holding the Fn key on the other hand usually works best.

## 5. The code

File: [`config/adv360.keymap`](config/adv360.keymap). Both halves use it, because `adv360_left.keymap` and `adv360_right.keymap` only `#include` it. Only two lines of the `fn` layer changed (rows 2 and 3 of its `bindings`). Each of them replaces two `&trans` bindings:

```dts
        &trans &trans &trans &trans &trans &trans  &none                                           &none &trans &kp RA(Y) /* ü */ &trans &kp RA(P) /* ö */  &trans  &trans
        &trans &kp RA(Q) /* ä */ &kp RA(S) /* ß */ &trans &trans &trans  &none        &trans &trans &trans &trans        &none &trans &trans &trans &trans  &trans  &trans
```

The `/* … */` comments are ordinary devicetree comments and do not change the binding count. The `fn` layer still has 76 entries, like every other layer.

## 6. Customizing

To put an umlaut on a different key of the Fn layer:

1. Find the position of the key you want. A position is the 0-based index in a layer's `bindings` list, counted left to right and top to bottom ([assets/key-positions.md](assets/key-positions.md) has a picture).
2. In the `fn` layer, replace the binding at that position with the umlaut binding, for example `&kp RA(Q)` for ä.
3. Set the old position back to `&trans`.
4. Only replace bindings, never add or remove one. Every layer must keep exactly 76 entries.
5. Prefer positions that are `&trans` on the Fn layer. Anything else (for example F1–F12 on the number row) would be replaced.

Then rebuild and flash as described below.

## 7. Build and flash

**GitHub Actions** ([`.github/workflows/build.yml`](.github/workflows/build.yml)) runs on every push and pull request, and manually via "Run workflow". Two jobs build the firmware in the `zmkfirmware/zmk-build-arm:stable` container:

| Job | Artifact | Contents |
| --- | --- | --- |
| Build (Legacy) | `firmware-no-clique` | `<YYYYMMDDHHMM>-<commit>-left.uf2` and `…-right.uf2` |
| Build (Clique) | `firmware-clique` | Same file names; the left half is built with ZMK Studio enabled (`CONFIG_ZMK_STUDIO=y`), for Kinesis Clique |

To download: open your fork on GitHub → **Actions** → **Build** → the run for your commit → **Artifacts** section → click the artifact. Then extract the `.uf2` files from the downloaded archive. GitHub keeps artifacts for 90 days by default ([GitHub Docs](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/download-workflow-artifacts)).

**Local build:** run `make` with Docker or Podman, as described in the README section [Building the Firmware in a local container](README.md#building-the-firmware-in-a-local-container). The local build produces the Clique variant (see `bin/build.sh`).

**Flashing:** follow the README section [Flashing firmware](README.md#flashing-firmware). It links to the official Kinesis [Quick Start Guide](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-QSG-v8-25-22.pdf) (page 8) and [User Manual](https://kinesis-ergo.com/wp-content/uploads/Advantage360-ZMK-KB360-PRO-Users-Manual-v3-10-23.pdf). The bootloader key combinations, quoted from the README:

> Press Mod+macro1 to put the left side into bootloader mode; it should attach to your computer as a USB drive.

> Press Mod+macro3 to put the right side into bootloader mode to attach it as a USB drive.

The README calls the files `left.uf2` and `right.uf2`. In the artifacts they have the timestamp and commit prefix shown above.

## 8. Troubleshooting

- **Wrong characters or no character at all:** the computer is not using one of the layouts from [Prerequisites](#2-prerequisites). Without an AltGr layout, Right Alt + letter acts like an Alt shortcut, and on a Mac you get œ, π or ¥. Check the active layout (Windows: Win+Space).
- **Quotes and apostrophes behave strangely on Windows:** United States-International turns `'` `"` `` ` `` `~` `^` into dead keys, so `"` followed by `a` gives ä. Type the symbol followed by Space to get the symbol on its own. This comes from the layout, not from the keyboard firmware. Linux `altgr-intl` puts these dead keys on AltGr only, so `'` and `"` type normally there.
- **Caps Lock does not give Ä/Ö/Ü on Windows:** with United States-International, AltGr+Q gives ä whether Caps Lock is on or not. Hold Shift instead.
- **The umlaut does not appear:**
  - Keep Fn held while you press the letter. Fn works only while it is held (`&mo 2`).
  - Check the computer's layout (see the first entry above).
  - Make sure the new firmware is on the keyboard. Mod+V types the build date, branch and commit hash (README, [Versioning](README.md#versioning)).
- **Uppercase does not work:** Shift has to reach the computer while Fn is held. On the Fn layer, the Shift positions (46 and 59) must stay `&trans`, so the Shift keys of the Base layer pass through. Hold Shift and Fn together, then press the letter. Shift + Fn + S gives §, because the layout has no uppercase ß.

## 9. Credits and links

- ZMK documentation: [Keymaps and layers](https://zmk.dev/docs/keymaps), [Layer behaviors (`&mo`)](https://zmk.dev/docs/keymaps/behaviors/layers), [Transparent (`&trans`)](https://zmk.dev/docs/keymaps/behaviors/misc), [Modifiers](https://zmk.dev/docs/keymaps/modifiers)
- Original config repository: [KinesisCorporation/Adv360-Pro-ZMK](https://github.com/KinesisCorporation/Adv360-Pro-ZMK)
- ZMK fork used by this repository: [ReFil/zmk, branch adv360-z3.5-2](https://github.com/ReFil/zmk/tree/adv360-z3.5-2)
- Kinesis support: [Advantage360 Pro support page](https://kinesis-ergo.com/support/kb360pro/)
- Layout data: Unicode CLDR keyboard data (release 40) for [Windows United States-International](https://github.com/unicode-org/cldr/blob/release-40/keyboards/windows/en-t-k0-windows-extended.xml) and [macOS U.S.](https://github.com/unicode-org/cldr/blob/release-40/keyboards/osx/en-t-k0-osx.xml) (recorded from macOS 10.9); xkeyboard-config [`symbols/us`](https://gitlab.freedesktop.org/xkeyboard-config/xkeyboard-config/-/blob/master/symbols/us)
