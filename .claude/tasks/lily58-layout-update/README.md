# Task: Lily58 Layout Update

## Status

| Field       | Value            |
|-------------|------------------|
| Status      | `planning`       |
| Phase       | `plan` (spec written; awaiting explicit human go for implement) |
| Owner       | kenneth.yeh      |
| Branch      | `claude/zmk-lily58-config-950d23` |
| Created     | 2026-08-09       |
| Updated     | 2026-08-09       |

## Problem

The Lily58 keymap ([config/lily58.keymap](../../../config/lily58.keymap)) has one functional bug and several ergonomic gaps:

1. **Delete-word is broken on Mac.** LOWER's home-row mods (S=Alt, D=Cmd, F=Shift) are revealed by the same layer hold that remaps the Backspace thumb to Delete — so `LOWER + S + BSPC` sends Opt+ForwardDelete instead of Opt+Backspace.
2. Non-gaming typing should anchor to a Corne-style **3-row core** (main 3 rows + thumbs); `` ` ~ [ ] { } `` currently require the physical number row or the center keys.
3. tmux and herdr both use a `Ctrl+B` prefix; it deserves a single-press key.
4. Bluetooth controls occupy prime RAISE real estate needed for brackets.

Full design: [docs/superpowers/specs/2026-08-09-lily58-layout-update-design.md](../../../docs/superpowers/specs/2026-08-09-lily58-layout-update-design.md)

## Design Constraints (hard-won — read before proposing changes)

These came from an extended design session on 2026-08-09. Violating any of them re-litigates a settled decision.

1. **Gaming is sacred:** base layer left half (number row, WASD, LCTRL/LSHFT column) must not change. Windows PC gaming (Valorant/CS/Apex) runs on the default layer, no layer switching.
2. **No dual-role keys on typed keys:** no hold-taps / layer-taps on keys that produce characters (tapping-term misfires are unacceptable). A `\`-thumb hold-tap for hyper was designed and rejected on this rule.
3. **No modal WM flows:** an AeroSpace "move mode" was built and rejected. Window moves must be direct chords (`modifier + shift + key` pattern).
4. **Everything must work away from the Lily58** on the MacBook's built-in keyboard with no exotic setup.
5. **One-handed right-hand operation is required ONLY for arrows and media** — both already satisfied (RAISE thumb + HJKL arrows; RAISE bottom-right media). It is NOT a requirement for window management.
6. **Thumb ergonomics:** inner tucked thumb keys = taps (Space/Backspace), natural resting thumb keys = layer holds (LOWER/RAISE).
7. **Mac-first modifiers;** Windows matters only for gaming + `PSCRN` (keep it on RAISE outer right thumb — Windows has no physical PrintScreen here). Mac screenshot chords (Cmd+Shift+4) are typed normally; do NOT add a macro.
8. **Laptop caps lock = Control** via macOS System Settings. Karabiner rules that match `left_control` will also catch remapped Caps (macOS remap applies first) — this silently removed ALL Control from the laptop once. Don't mix the two remap systems.

## Settled Decisions (do not re-open without new evidence)

- **AeroSpace = stock `alt-*` bindings, driven by the base-layer Alt thumb.** A Hyper key was designed, partially deployed (aerospace.toml dual bindings, Karabiner left-ctrl rule, LOWER+A home-row hyper), and **fully rolled back on 2026-08-09**. Reasons: (a) letter workspaces break under LOWER — the layer remaps most letters, and workspace A's letter *is* the hyper key; (b) dual alt+hyper paths made an incoherent story; (c) the Karabiner rule chained with the caps→ctrl macOS remap and removed Control from the laptop; (d) decisive: alt/Option app-shortcut conflicts never actually bothered the user. Alt thumb + hjkl/digit/letter (+Shift pinky to move) covers 100% of usage on both keyboards.
- **No firmware leader-key module** (urob's zmk-leader-key considered, rejected): every leader in the toolchain is software-side — tuicr's `;`, nvim's Space, tmux/herdr behind one `C-b` key.
- **Handoff doc from the pre-2026-08-09 session is stale.** The keymap on disk is the source of truth: right thumbs are `BSPC / RAISE / RET / BSLH`, the Cmd+Shift+4 macro never existed, RAISE has arrows on home row + media on bottom row (commit `092bf37`).

## Scope

In scope: `config/lily58.keymap` changes per the spec (4 items, see plan.md). Out of scope: host configs (aerospace.toml and karabiner.json are back at stock/pre-session state — leave them), hyper anything, leader-key modules, base-layer left-half changes.

## Success Criteria

- `LOWER+S+BSPC` = delete word backward (Opt+BSPC); `LOWER+D+BSPC` = delete to line start; `Shift+BSPC` = forward delete; plain `BSPC` unchanged.
- `` ` ~ [ ] { } `` typable within the 3-row core (RAISE bottom-left + existing center keys).
- `LOWER+G` sends `Ctrl+B` once (tmux/herdr prefix).
- `LOWER+RAISE` held together exposes BT_CLR / BT_SEL 0-2 / EP_TOG (same physical spots they occupy on RAISE today).
- Base layer left half byte-identical; GitHub Actions build passes for both halves.

## Links

- Spec: `docs/superpowers/specs/2026-08-09-lily58-layout-update-design.md`
- Tools referenced: [tuicr](https://github.com/agavra/tuicr) (uses `[ ] { }` for hunk/file nav), [herdr](https://github.com/herdrdev/herdr) (tmux-style `C-b` prefix)
