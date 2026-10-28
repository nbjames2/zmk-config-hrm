# Customization guide

Instructions for working on this ZMK config — written for coding agents, but equally usable by
humans adapting this repo as a starting point for their own keymap. For background on the keymap
and the build pipeline, see the [README](README.md).

## Ground rules

- **Building needs no local setup.** Pushing to a fork builds every target in `build.yaml` through
  GitHub Actions (`.github/workflows/build-nix.yml`, triggered by pushes and PRs touching
  `config/**` or `build.yaml`); the firmware is downloaded from the Actions tab. A fresh fork has
  Actions disabled — enable them once from the fork's Actions tab.
- **Building locally** requires the nix-based dev environment (nix + direnv, see the
  [README](README.md#local-build-environment)) plus a one-time `just init`, which turns the
  repository into a west workspace by pulling in ZMK, Zephyr and the modules. Then verify keymap
  changes with `just build <target>` (`just list` shows valid targets, `just build all` builds
  everything); compiled firmware lands in `firmware/`. If `just` or `west` are missing, the
  environment isn't active — run commands through `nix develop --command <cmd>` or activate with
  `direnv allow`.
- `config/west.yml` is maintained by [pin-west](https://github.com/urob/pin-west): never hand-edit
  pinned revisions, run `pin-west bump` instead. Adding or removing a module is fine — edit the
  entry itself and re-run `pin-west pin` to (re)pin.
- `dts-format` (packaged in the dev environment) formats devicetree files. It does not reformat
  C-preprocessor macros, so on the keymaps it is close to a no-op — the hand-aligned grids inside
  `ZMK_LAYER` are safe and need no guarding. It earns its keep on plain devicetree, i.e. when
  developing modules or board definitions. Without arguments it recurses over the working
  directory; pass files explicitly to narrow it.
- After keymap changes that affect the layout, regenerate the diagrams with `just draw` (renders
  `draw/base.svg` and `draw/overview.svg` from `config/lily58.keymap`).
- `just test` is a snapshot-test harness for developing the ZMK **modules** checked out under
  `modules/zmk/`. It does not test this repo's keymap; the keymap is validated by building.

## Keymap structure

`config/lily58.keymap` is the single entry keymap for a Lily58 Pro (nice!nano v2, shields
`lily58_left`/`lily58_right`). It defines every layer as a full 58-key `ZMK_LAYER`, using the
standardized key labels from [zmk-helpers](https://github.com/urob/zmk-helpers)
(`zmk-helpers/key-labels/lily58.h` with `LILY58_STANDARD_LABELS` defined): `LN0`–`LN5` number row,
`LT0`–`LT5`, `LM0`–`LM5`, `LB0`–`LB5` for the letter rows, `LEC` for the inner bottom key and
`LH0`–`LH3` for the thumbs, mirrored with `R` on the right (`0` is innermost). HRM trigger
positions, combos (`config/combos.dtsi`), the leader key and the mouse layer are all written
against these labels.

`LILY58_STANDARD_LABELS` must stay defined: without it the header labels the number row as `LT*`,
which shifts every combo and HRM trigger up one row.

## Where to change what

| Change                            | File                                                              |
| --------------------------------- | ----------------------------------------------------------------- |
| Layers, HRMs, thumb keys          | `config/lily58.keymap`                                            |
| HRM timing                        | `config/lily58.keymap` (`tapping-term-ms`, `require-prior-idle-ms`; see README troubleshooting) |
| Combos                            | `config/combos.dtsi` (position diagram at the top)                |
| Leader sequences                  | `config/leader.dtsi`                                              |
| Mouse layer                       | `config/mouse.dtsi`                                               |
| Board settings (BT, sleep, mouse) | `config/lily58.conf`                                              |
| OLED widgets and animations       | `config/lily58.conf` (`CONFIG_NICE_OLED_*`), `nice_oled` shield in `build.yaml` |
| Build targets                     | `build.yaml`                                                      |
| ZMK/module versions               | `config/west.yml` (via `pin-west bump` only)                      |
| Dev environment                   | `flake.nix`, `nix/` (test with `nix develop`)                     |

Removing a feature: delete its `#include` from `lily58.keymap` (and its bindings/combo references),
and if it was the only consumer of a module, drop the module's entry from `config/west.yml`. The
modules used per feature are commented at the top of `lily58.keymap`.
