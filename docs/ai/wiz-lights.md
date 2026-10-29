# WiZ lights — local control and discovery fix

This doc holds verified findings for the owner's WiZ bulbs. It supports the
macropad "dim lights" goal in [vision.md](vision.md). Local control works. The
pad mapping is not built yet.

## Device inventory

Source: WiZ app export at `~/dev/fun/wiz_home_setup.json` (home "Cathedral
Building", room "Bedroom"). IPs confirmed live on 2026-10-27.

| Name | IP | MAC | Firmware | Traits |
|------|-----|-----|----------|--------|
| Lower Corner Light | `10.0.0.30` | `44:4f:8e:9d:50:8e` | 1.38.0 | dimmable, tunable white 2200-6500K, RGB |
| Upper Corner Light | `10.0.0.129` | `44:4f:8e:99:e3:dc` | 1.38.0 | dimmable, tunable white 2200-6500K, RGB |

Notes:
- The bulbs answer the plain local UDP API on port 38899. The
  `udp_signing_key` in the export is not needed for pywizlight.
- IPs come from DHCP. They can change. The MACs are stable. Set a DHCP
  reservation on the router (`10.0.0.1`) to keep the IPs fixed.

## Install the control tool (per machine)

The CLI is `pywizlight`. Install it with pip for a Python 3.10+ interpreter:

```shell
python3.10 -m pip install --user pywizlight
```

This gives two ways to run it. Use either:

```shell
python3.10 -m pywizlight.cli <command>   # module form (used in the scripts)
wizlight <command>                        # entry point in ~/.local/bin
```

On a new machine, confirm the interpreter and path:

```shell
python3.10 -m pywizlight.cli --help   # must print the command list
command -v wizlight                   # shows the entry-point path
```

The host scripts call the module form with the absolute interpreter path
`/usr/bin/python3.10`. Change that path if your Python 3.10 is elsewhere.

## Control tool

The CLI is `pywizlight`, installed for `python3.10`. Entry point: `wizlight`.

```shell
wizlight status 10.0.0.30               # read state
wizlight on 10.0.0.30 --brightness 40   # on, brightness 0-255
wizlight brightness 10.0.0.30 128       # set brightness 0-255
wizlight off 10.0.0.129                 # off
wizlight discover --broadcast 10.0.0.255
```

## Discovery fix (firewall + broadcast)

Problem: `wizlight discover --broadcast 10.0.0.255` finds no bulbs, but the
bulbs answer direct queries and the phone app controls them.

Root cause: the host firewall (UFW) blocks the reply packets.

1. Discovery sends a UDP broadcast to `10.0.0.255:38899`.
2. Each bulb replies with a unicast from its own IP, source port 38899.
3. Connection tracking has an entry only for the broadcast destination.
4. The unicast reply does not match that entry. The default deny-incoming rule
   drops it.
5. Direct per-IP queries work because the unicast request and reply match.

### Fix (per machine on this LAN)

Allow inbound UDP 38899 from the local subnet:

```shell
sudo ufw allow from 10.0.0.0/24 to any port 38899 proto udp comment 'WiZ bulb local discovery/control'
sudo ufw status | rg 38899
wizlight discover --broadcast 10.0.0.255   # now lists both bulbs
```

Scope the rule to the LAN subnet. Do not open the port to all sources.

### Checklist for a new computer with the same problem

Do these checks in order. Stop when discovery works.

1. Confirm the broadcast address. Run `ip addr`. Use the `brd` value of the
   active interface (example `10.0.0.255`).
2. Confirm the bulbs are reachable. Run `ip neigh | rg -i 44:4f:8e`. The MACs
   `44:4f:8e:...` are WiZ bulbs. No entry can mean AP/client isolation on the
   Wi-Fi. Fix that on the router.
3. Test direct control. Run `wizlight status <bulb-ip>`. A reply proves local
   control is on and only discovery is blocked.
4. Open the firewall. Add the UFW rule above. Re-run discovery.
5. If discovery still fails, check for AP/client isolation and for a second
   firewall (firewalld, nftables).

## F13-F24 symbol map — host `daniell-ubuntu-dellpromax` (work machine)

The pad can send `f13`-`f24`. Desktop Linux renames these codes to `XF86`
symbols. Many are real functions. Only `f14`-`f18` are inert and safe as
triggers. Probed on 2026-10-28.

| Pad code | Reported symbol / behavior | Safe trigger? |
|----------|----------------------------|---------------|
| `f13` | `XF86Tools` → opens GNOME Settings (control center) | ❌ taken |
| `f14` | `XF86Launch5` | ✅ inert |
| `f15` | `XF86Launch6` | ✅ inert |
| `f16` | `XF86Launch7` | ✅ inert |
| `f17` | `XF86Launch8` | ✅ inert |
| `f18` | `XF86Launch9` | ✅ inert |
| `f19` | breaks modal/dialog focus | ❌ avoid |
| `f20` | mic mute (`XF86AudioMicMute`) | ❌ real function |
| `f21` | touchpad toggle | ❌ real function |
| `f22` | touchpad on | ❌ real function |
| `f23` | touchpad off | ❌ real function |
| `f24` | touchpad off (same as `f23`) | ❌ real function |

Bind your host daemon to the reported symbol (example `XF86Launch5`), not to
`F14`. A bind to `F14` does not fire. This map can differ on another desktop
environment. Re-probe with `wev` or `xev` on each new machine.

`f13` is not free: `XF86Tools` opens GNOME Settings. Only `f14`-`f18`
(`XF86Launch5`-`9`) are absent from the GNOME registry and safe.

### Check if a key is free (GNOME level)

There is no single OS registry of all shortcuts. Application-internal shortcuts
(what an app does with `F9` while focused) are not listed anywhere. The
desktop-environment bindings ARE queryable:

```shell
for s in org.gnome.desktop.wm.keybindings org.gnome.mutter.keybindings \
         org.gnome.shell.keybindings org.gnome.settings-daemon.plugins.media-keys; do
  gsettings list-recursively "$s"
done | rg -i 'F9'   # replace with the key/symbol to check
```

Empty output means the key is free at the desktop level. Standard function
keys (`F1`-`F12`) can still be grabbed inside apps, so test them in context.
`f9` is free at the GNOME level on this machine, but apps may use it.

## Registration table — Layer 2 WiZ controls

Flashed to the pad on Layer 2. Register each symbol in the host daemon
(xbindkeys / sway bind) to the command shown. `<IP>` is `10.0.0.30` or
`10.0.0.129`.

| Physical control (Layer 2) | Pad code | Bind this symbol | Intended command |
|----------------------------|----------|------------------|------------------|
| Far-right, top | `f14` | `XF86Launch5` | brighter (step up brightness) |
| Far-right, middle | `f15` | `XF86Launch6` | toggle on/off |
| Far-right, bottom | `f16` | `XF86Launch7` | softer (step down brightness) |
| Far-left, top | `f17` | `XF86Launch8` | always on (full brightness) |
| Far-left, bottom | `f18` | `XF86Launch9` | always off |

`f13` cells are unassigned placeholders. Do not rely on them.

## Host wiring on GNOME (host `daniell-ubuntu-dellpromax`)

GNOME runs the key-to-command binding. The pad sends an `XF86Launch` symbol.
A GNOME custom shortcut catches the symbol and runs a script.

### Gotcha: GNOME runs executables, not shell functions

GNOME custom shortcuts do NOT source `.zshrc` or `.bashrc`. A command like
`lights_on` defined as a zsh function fails silently. The command must be a
real executable file with an absolute path. This is true for any GUI hotkey
runner (GNOME, xbindkeys, sway).

### Scripts

Standalone executables in `~/.local/bin`. Each loops over both bulb IPs.

| Script | Action |
|--------|--------|
| `lights_on` | both bulbs on at full brightness |
| `lights_off` | both bulbs off |
| `lights_toggle` | read power of bulb 1, flip both to match |
| `lights_brighter` | step the tracked level up by 32 (max 255) |
| `lights_softer` | step the tracked level down by 32 (min 16) |

`lights_brighter` and `lights_softer` track the level in
`$XDG_STATE_HOME/wiz/brightness` (default `~/.local/state/wiz/brightness`).
WiZ has no relative brightness command, so the script sets an absolute value.

### GNOME custom shortcuts

Set with `gsettings`. Each binds an `XF86Launch` symbol to a script path.

| Shortcut | Pad code | Binding | Command |
|----------|----------|---------|---------|
| custom0 | `f17` | `Launch8` | `~/.local/bin/lights_on` |
| custom1 | `f18` | `Launch9` | `~/.local/bin/lights_off` |
| custom2 | `f14` | `Launch5` | `~/.local/bin/lights_brighter` |
| custom3 | `f15` | `Launch6` | `~/.local/bin/lights_toggle` |
| custom4 | `f16` | `Launch7` | `~/.local/bin/lights_softer` |

To recreate on a new GNOME machine:

```shell
BASE=/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings
S=org.gnome.settings-daemon.plugins.media-keys.custom-keybinding
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings \
  "['$BASE/custom0/','$BASE/custom1/','$BASE/custom2/','$BASE/custom3/','$BASE/custom4/']"
# then for each customN set: name, command (absolute script path), binding (LaunchN)
gsettings set "$S:$BASE/custom0/" binding 'Launch8'
gsettings set "$S:$BASE/custom0/" command '/home/USER/.local/bin/lights_on'
# ... repeat for custom1..custom4
```

Note: the binding uses `Launch8`, not `XF86Launch8`. GNOME stores the short
form.

### Shell wrappers in `.zshrc`

The `.zshrc` light functions call the same scripts. This keeps one source of
truth for the terminal and the macropad. The functions pass `"$@"` through.

| Function | Calls |
|----------|-------|
| `lights_on` | `~/.local/bin/lights_on` |
| `lights_off` | `~/.local/bin/lights_off` |
| `lights_toggle` | `~/.local/bin/lights_toggle` |
| `lights_brighter` | `~/.local/bin/lights_brighter` |
| `lights_softer` | `~/.local/bin/lights_softer` |

`.zshrc` also defines scene presets (`lights_blue_white`, `lights_cozy`) that
call `pywizlight` directly. These are not scripts.

`.zshrc` sets the bulb IPs and the `bulb_ips` array:

```shell
lower_corner_light_ip="10.0.0.30"
upper_corner_light_ip="10.0.0.129"
```

### `lights_status` (terminal only)

`lights_status` replaced the old `lights_list`. It prints live status for each
bulb and the tracked dimmer level:

```shell
lights_status
# === <ip> ===  Power / Brightness / Color Temp  (per bulb)
# === tracked dimmer level ===  <level>  (<state file path>)
```

It is a zsh function, not a script. Use it to check drift between the tracked
level and the real bulb brightness.

Note: `pywizlight status` shows `Brightness: None` while a bulb is in a scene
(example "Night light"). After the scripts set an explicit brightness, the
numeric value reports correctly.

## Prerequisites on the bulb side

- Keep the bulbs on the same subnet as the computer.
- Keep local communication on in the WiZ app. The owner's bulbs already allow it.
