# Vision — intended future use

Status: NOT IMPLEMENTED. This doc holds intent only. It describes goals the
owner wants later. Do not treat any item here as done. Do not add these
mappings to `your-config.yaml` until the owner confirms each one works.

For what is implemented now, read [macropad.md](macropad.md).

## Goal

Make some pad controls trigger host-side actions, not just key codes. Use the
key-code-to-host-action pattern in [macropad.md](macropad.md) (F13 to F24 plus
a host daemon).

## Intended actions (not built)

| Control | Intended action | Blocker / status |
|---------|-----------------|------------------|
| A pad key | Open Slack | Needs a host daemon bind and a launch command. Not built. |
| Second knob (bottom) | Dim WiZ lights | ccw = dim down, cw = dim up, press = toggle. Needs WiZ control (script or microservice). Repo not confirmed yet. |

## Order of work (per action)

1. Build and test the host command first (example: a WiZ control script or a
   local microservice endpoint).
2. Bind a spare code (example F14/F15/F16) to that command in the host daemon.
3. Test the command with the raw chord. Confirm the action runs.
4. Only then map the pad key or knob to that code in `your-config.yaml` and
   upload.
5. Move the item from this vision doc to the live layout in [macropad.md](macropad.md).

## Notes

- The WiZ knob reuses the bottom knob. The bottom knob is scroll today. Put the
  WiZ binds on a non-default layer so scroll stays on Layer 1.
- The host daemon must exist on every machine that fronts the KVM.
