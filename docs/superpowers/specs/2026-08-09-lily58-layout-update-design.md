# Lily58 Layout Update — Design

**Date:** 2026-08-09
**Status:** Awaiting approval
**Hardware:** Typeractive Lily58, nice!nano v2, nice!view displays. Hosts: Mac (dev, primary), Windows (gaming).

## Goals

1. Fix delete-word: `LOWER + S(Alt) + Backspace` must send Opt+Backspace (Mac delete-word-backward).
2. Anchor non-gaming typing to a Corne-style 3-row core (main 3 rows + thumbs); the physical number row remains for gaming only.
3. Better AeroSpace support via a Hyper key, one-handed from the right hand, without breaking laptop-keyboard usage.
4. Single-press tmux/herdr prefix (`Ctrl+B`).
5. Comfortable `[ ] { } ~ \`` access (also serves tuicr's `[ ] { }` hunk/file navigation).
6. Preserve gaming: base layer left half untouched (number row, WASD, Ctrl/Shift column).
7. Keep one-handed right-hand arrows (already works: RAISE thumb + HJKL).
8. Keep `PSCRN` for Windows screenshots; Mac screenshot chords typed normally.

## Decisions

| Topic | Decision |
|---|---|
| Delete-word | `bspc_del` mod-morph on base backspace thumb; LOWER backspace slot → `&trans` |
| AeroSpace | Hyper hold-tap on outer right thumb (`\`); dual-bind aerospace.toml (keep `alt-*`, add `ctrl-alt-shift-cmd-*`) |
| Laptop fallback | Existing `alt-*` bindings stay; optional Karabiner Caps→Hyper later |
| Window move | `alt-shift-hjkl` still works; hyper path uses AeroSpace "move" binding mode (`hyper-quote` → h/j/k/l nudge, workspace key sends+exits) |
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

hyp: hyper_hold_tap {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    bindings = <&kp>, <&kp>;
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

Only two changes, both right side:
- Backspace thumb: `&kp BSPC` → `&bspc_del` (Shift+BSPC = forward delete)
- Outer right thumb: `&kp BSLH` → `&hyp LS(LC(LA(LGUI))) BSLH` (tap `\`, hold Hyper)

Left half unchanged for gaming.

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

**aerospace.toml** — add alongside existing `alt-*` bindings:

```toml
ctrl-alt-shift-cmd-h = 'focus left'
ctrl-alt-shift-cmd-j = 'focus down'
ctrl-alt-shift-cmd-k = 'focus up'
ctrl-alt-shift-cmd-l = 'focus right'
ctrl-alt-shift-cmd-1 = 'workspace 1'
# ... through workspace 9
```

Laptop-only usage is unchanged (`alt-*`). Optional later: Karabiner-Elements Caps Lock → Hyper rule for the laptop keyboard.

## Success criteria

- `LOWER + S + BSPC` deletes word backward on Mac; `LOWER + D + BSPC` deletes to line start; `Shift + BSPC` forward-deletes.
- `` ` ~ [ ] { } `` all typable within the 3-row core.
- Hold `\` thumb + HJKL switches AeroSpace focus one-handed; tapping `\` still types backslash.
- `LOWER + G` sends `Ctrl+B` (tmux and herdr prefix).
- `LOWER + RAISE` exposes BT profile switching; profiles still pair/switch.
- Base layer left half byte-identical to current (gaming unaffected).
- GitHub Actions firmware build passes for both halves.

## Risks / tuning

- **Layer-tap on `\`:** 200 ms tapping term may misfire as Hyper during fast `\` typing (paths, regex). Tune `tapping-term-ms` or revert that key to plain `&kp BSLH` if it annoys.
- **Mod-morph masks Shift:** `Shift+BSPC` sends plain `DEL` (Shift masked) — desired behavior here.
- **ADJUST ordering:** ADJUST must be a higher layer index than LOWER/RAISE (it is: 3).
- Existing `alt-*` AeroSpace usage keeps working throughout; hyper bindings are additive.
