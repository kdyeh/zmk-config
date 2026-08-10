# Lily58 Layout Update — Design

**Date:** 2026-08-09
**Status:** Awaiting approval
**Hardware:** Typeractive Lily58, nice!nano v2, nice!view displays. Hosts: Mac (dev, primary), Windows (gaming).

## Goals

1. Fix delete-word: `LOWER + S(Alt) + Backspace` must send Opt+Backspace (Mac delete-word-backward).
2. Anchor non-gaming typing to a Corne-style 3-row core (main 3 rows + thumbs); the physical number row remains for gaming only.
3. AeroSpace driven by the existing Alt thumb (`alt-*` stock bindings) — identical on the laptop's Option key. Hyper plan shelved 2026-08-09: alt conflicts never materialized in practice.
4. Single-press tmux/herdr prefix (`Ctrl+B`).
5. Comfortable `[ ] { } ~ \`` access (also serves tuicr's `[ ] { }` hunk/file navigation).
6. Preserve gaming: base layer left half untouched (number row, WASD, Ctrl/Shift column).
7. Keep one-handed right-hand arrows (already works: RAISE thumb + HJKL).
8. Keep `PSCRN` for Windows screenshots; Mac screenshot chords typed normally.

## Decisions

| Topic | Decision |
|---|---|
| Delete-word | `bspc_del` mod-morph on base backspace thumb; LOWER backspace slot → `&trans` |
| AeroSpace | Alt thumb (base layer) does everything: `alt-hjkl` focus, `alt-shift-hjkl` move, `alt-<digit/letter>` workspaces, `alt-shift-<digit/letter>` send-to-workspace. Zero firmware or host config changes. Hyper experiment (LOWER+A, Karabiner left-ctrl, dual aerospace bindings) rolled back 2026-08-09 — alt conflicts never bit; LOWER A position stays free for future use |
| tmux/herdr prefix | `&kp LC(B)` on LOWER at the G position |
| Brackets/grave | RAISE bottom-left: `` ` `` on X, `[` on C, `]` on V (Shift → `~ { }`) |
| Bluetooth/system | Moved from RAISE to new ADJUST layer (LOWER+RAISE held), same physical positions |
| Screenshot | `PSCRN` stays on RAISE outer right thumb (Windows); no Mac macro |
| Leader key module | Dropped — all leaders live in software (tuicr `;`, nvim Space, tmux/herdr behind `C-b`) |

## New behaviors

```dts
bspc_del: backspace_delete {
    compatible = "zmk,behavior-mod-morph";
    #binding-cells = <0>;
    bindings = <&kp BSPC>, <&kp DEL>;
    mods = <(MOD_LSFT|MOD_RSFT)>;
};

```

Plus:

```dts
conditional_layers {
    compatible = "zmk,conditional-layers";
    adjust {
        if-layers = <1 2>;   /* LOWER + RAISE */
        then-layer = <3>;    /* ADJUST */
    };
};
```

## Layer maps (after)

### Layer 0 — Base (Mac / Gaming)

One change, right side:
- Backspace thumb: `&kp BSPC` → `&bspc_del` (Shift+BSPC = forward delete)

Left half unchanged for gaming. `\` thumb stays a plain `&kp BSLH`.

### Layer 1 — LOWER

```
F1    F2   F3   F4   F5    F6                     F7    F8    F9   F10   F11  F12
ESC   1    2    3    4     5                      6     7     8    9     0    ___
___   ___  ⌥    ⌘    ⇧     C-b                    ←     ↓     ↑    →     ___  ___
___   ___  ___  ___  ___   ___   {        }      ___   ___   ___  ___   ___  ___
           ___   ___   ▓(held)  ___    | ‹trans›  ___   ___   ___   ___
```

Changes: `C-b` added at G position; backspace-thumb slot `DEL` → `&trans`.

### Layer 2 — RAISE

```
___   ___  ___  ___  ___   ___                    ___   ___   ___  ___   ___  PG_UP
___   !    @    #    $     %                      ^     &     *    (     )    PG_DN
___   +    =    _    -     |                      ←     ↓     ↑    →     \    ___
___   ___  `    [    ]     ___   {        }      PREV  VOL-  VOL+ NEXT  ___  ___
           ___   ___   ___      ⏯      | ___      ▓(held) ___  PSCRN
```

Changes: bottom-left BT row replaced with `` ` [ ] ``; everything else as today.

### Layer 3 — ADJUST (new; hold LOWER + RAISE)

All `&trans` except bottom-left row (identical positions to old RAISE placement):

```
BT_CLR  BT0  BT1  BT2  EP_TOG  ___
```

## Companion changes (outside this repo)

None. The hyper additions to aerospace.toml and the Karabiner left-ctrl rule were rolled back on 2026-08-09 — aerospace.toml is back to stock `alt-*` bindings, which the Lily58 drives via the base-layer Alt thumb and the laptop drives via plain Option.

## Success criteria

- `LOWER + S + BSPC` deletes word backward on Mac; `LOWER + D + BSPC` deletes to line start; `Shift + BSPC` forward-deletes.
- `` ` ~ [ ] { } `` all typable within the 3-row core.
- AeroSpace unchanged and fully drivable from the base layer: Alt thumb + hjkl/digit/letter (+ Shift pinky for moves).
- `LOWER + G` sends `Ctrl+B` (tmux and herdr prefix).
- `LOWER + RAISE` exposes BT profile switching; profiles still pair/switch.
- Base layer left half byte-identical to current (gaming unaffected).
- GitHub Actions firmware build passes for both halves.

## Risks / tuning

- **Mod-morph masks Shift:** `Shift+BSPC` sends plain `DEL` (Shift masked) — desired behavior here.
- **ADJUST ordering:** ADJUST must be a higher layer index than LOWER/RAISE (it is: 3).
- **ZMK Studio stored overrides:** Studio edits persist in the settings partition across UF2 flashes and shadow the compiled keymap per-key at boot. If a flashed keymap change doesn't take effect, check zmk.studio for a stale override on that key first (this bit the `bspc_del` rollout). Full reset requires `settings_reset` firmware on both halves (also wipes BT bonds).
