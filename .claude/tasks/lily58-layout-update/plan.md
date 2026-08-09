# Plan: Lily58 Layout Update

> Implementation plan. This is the contract — sessions execute against this, not freeform.
> All changes in `config/lily58.keymap` unless noted. Exact layer diagrams live in the spec:
> `docs/superpowers/specs/2026-08-09-lily58-layout-update-design.md`

## Approach

Four small, independent firmware changes that fix the delete-word bug and complete the 3-row core, while leaving the base layer's left half untouched for gaming. No new modules, no host-config changes — core ZMK behaviors only (`mod-morph`, `conditional-layers`).

## Alternatives Considered

| Approach | Why not |
|----------|---------|
| DEL stays on LOWER; add separate `LA(BSPC)` word-delete key | Treats the symptom; mod-morph frees the slot and gives Shift+BSPC=DEL for free |
| Hyper key for AeroSpace (hold-tap `\`, LOWER+A, Karabiner left-ctrl) | Deployed and rolled back — see README "Settled Decisions"; alt thumb suffices |
| AeroSpace move mode (modal) | User rejects modal WM flows |
| urob zmk-leader-key module for dev-tool leaders | All leaders are software-side; module adds build complexity for zero gain |
| BT controls to a 4th "gaming/system" toggle layer | ADJUST via LOWER+RAISE is cheaper and needs no dedicated key |

## Steps

- [x] 1. Add `bspc_del` mod-morph behavior; bind base-layer Backspace thumb to `&bspc_del`; change LOWER's Backspace-thumb slot from `&kp DEL` to `&trans`
  - Files: `config/lily58.keymap`
  - Behavior: `bindings = <&kp BSPC>, <&kp DEL>; mods = <(MOD_LSFT|MOD_RSFT)>;`
  - This is the delete-word fix. Verify Opt+BSPC path: LOWER(hold) + S(=LALT) + BSPC.
- [x] 2. LOWER layer: `G` position `&trans` → `&kp LC(B)` (tmux/herdr prefix)
  - Files: `config/lily58.keymap`
- [x] 3. RAISE layer bottom-left row: replace `&bt BT_CLR &bt BT_SEL 0 &bt BT_SEL 1 &bt BT_SEL 2 &ext_power EP_TOG` with `` ` `` at X, `[` at C, `]` at V (rest `&trans`)
  - Files: `config/lily58.keymap`
  - Shift gives `~ { }`. Serves 3-row core + tuicr `[ ] { }` navigation.
- [x] 4. Add ADJUST layer (index 3) containing the BT row + EP_TOG in the same bottom-left positions; add `conditional_layers` node (`if-layers = <1 2>; then-layer = <3>;`)
  - Files: `config/lily58.keymap`
  - ADJUST must be defined after RAISE in the keymap node (higher index).
- [x] 5. Verify build via GitHub Actions — run 31342667192 succeeded for both halves (branch push; PR #1 build also triggered)
- [ ] 6. Flash + hands-on verification against README success criteria; append results to log.md

## Testing Strategy

- Firmware compiles for `lily58_left` and `lily58_right` (nice_nano_v2 + nice_view).
- On-device: each success criterion in README.md, plus regression checks — plain typing on all three layers, gaming keys (WASD/numbers/LCTRL/LSHFT), BT profile switching via LOWER+RAISE.

## Risks

- **Mod-morph masks Shift:** `Shift+BSPC` sends unshifted `DEL` — desired here, but don't "fix" it with `keep-mods`.
- **ZMK Studio is enabled** (`CONFIG_ZMK_STUDIO=y`): runtime edits made via Studio are overwritten on flash; this repo is the source of truth.
- [config/lily58.conf](../../../config/lily58.conf) sets press debounce to 1 ms while its comment says 3 ms — not part of this task, but bump to 3 ms if key chatter ever appears.

## Definition of Done

- [ ] All steps completed and checked off
- [ ] GitHub Actions build green for both halves
- [ ] Success criteria in README.md verified on hardware
- [ ] log.md updated with session entry
