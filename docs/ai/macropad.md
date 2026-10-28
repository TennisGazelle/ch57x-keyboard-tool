# Macropad spoke — device, config, and flash

This spoke is the project memory. Read it before you change the map.

## Device facts

- Model: `ch57x-1`. USB id: `1189:8840`. 12 keys, 2 knobs, 3 layers.
- The firmware key order is not left-to-right.
- The tool cannot read the current map off the device.
- You flash the map only over the USB cable. The 2.4GHz dongle and Bluetooth
  run an already-flashed map. They do not program the device.

## Files

| File | Role |
|------|------|
| `your-config.yaml` | Owner layout. This is the live map on the device. Edit this. |
| `discovery.yaml` | Probe map. One unique key per cell to find the physical→cell grid. |
| `example-mapping.yaml` | Upstream example. Shows every action type. Reference only. |
| `doc/actions.md` | Full list of key, mouse, and media actions. |
| `ch57x-keyboard-tool` | Local build only. Not tracked by git. Build it per machine (see below). |
| `Cargo.toml` / `Cargo.lock` | Rust build inputs. |
| `50-ch57x-macropad.rules` | udev rule to flash without sudo. Installed on the owner machine (per-machine step). |

## Current owner layout (`your-config.yaml`)

- Top knob: volume. ccw = down, press = mute, cw = up. Same on all layers.
- Bottom knob: scroll. ccw = wheeldown, press = click, cw = wheelup.
- Layer 1: editing shortcuts (copy, paste, cut, undo / select-all, save, find,
  redo / new-tab, close-tab, alt-tab, reopen-tab).
- Layer 2: F1 to F12.
- Layer 3: numpad (7 8 9 / 4 5 6 / 1 2 3) with play, prev, next on the right column.

## Build (per machine)

The binary is not committed. It is a dynamically-linked ELF for one CPU
architecture and glibc. Build it on each machine:

```shell
cargo build --release
# output: target/release/ch57x-keyboard-tool
```

`target/` is gitignored. Run the commands below with `target/release/` prefix,
or copy the binary to the repo root.

## Commands

Run these from the repo root.

```shell
./ch57x-keyboard-tool show-keys              # list valid action names
./ch57x-keyboard-tool validate your-config.yaml
sudo ./ch57x-keyboard-tool upload your-config.yaml
./ch57x-keyboard-tool led <layer> <mode>     # control the top LEDs
```

## Flash procedure

1. Connect the device with the USB cable.
2. Run `validate` to check the YAML.
3. Run `upload`. Exit code 0 means success. The tool prints no message on success.

Without the udev rule, `upload` fails with `Access denied (insufficient
permissions)`. In that case run it again with `sudo`.

The udev rule removes the sudo need. It is per-machine. Install it on each new
machine:

```shell
sudo cp 50-ch57x-macropad.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

## Action rules (from doc/actions.md)

- Combine modifiers with a hyphen: `ctrl-c`, `ctrl-shift-z`.
- Make a sequence with commas: `h,e,l,l,o`. Max 5 steps.
- Media keys (`volumeup`, `play`, `mute`) cannot mix with normal keys in one action.
- Knobs take `ccw`, `press`, and `cw`. Knobs are listed top to bottom.

## Advanced: key code to host action

The pad sends only key codes. The pad cannot start a program or call an API.
To run a custom action, use two mappings that meet at one key code:

1. On the pad: bind a key or knob to an uncommon code.
2. On the host: a hotkey daemon listens for that code and runs the command.

### Use F13 to F24

Map pad controls to F13 through F24. These codes are real but almost no app
uses them, so they do not collide with normal shortcuts. Use the HID code form
if the tool has no name, for example `<104>` for F13. Check `doc/actions.md`
and the USB HID table for codes.

### Host daemon per OS

The action handler must exist on each machine that fronts the KVM. Each OS
needs its own daemon.

| Host OS | Bind code to command with |
|---------|---------------------------|
| Linux (X11) | `xbindkeys`, or window manager binds (i3/sway) |
| Linux (Wayland) | compositor binds (sway/Hyprland), or `keyd` |
| Windows | AutoHotkey |
| macOS | Hammerspoon, Karabiner-Elements, or BetterTouchTool |

The command can be anything: an app launch, a shell script, or a `curl` call
to a local microservice.

### Test-first workflow

1. Bind the target code to the command in the host daemon (example: F13).
2. Test the command. Press the real F13 chord on a keyboard, or trigger the
   daemon directly. Confirm the action runs.
3. Only then map the pad key or knob to that same code and `upload`.
4. Now the pad control triggers the host action.

### Intended actions

The owner wants host actions like "open Slack" and "dim WiZ lights" later.
These are not built and are not mapped. See [vision.md](vision.md) for intent.
