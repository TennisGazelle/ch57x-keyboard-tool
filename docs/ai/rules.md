# Rules for agents — ch57x-keyboard-tool

1. Edit `your-config.yaml` to change the map. It is the only source of truth.
2. Run `validate` before `upload`. Always.
3. Use `sudo` for `upload` until the udev rule is installed.
4. Do not edit `discovery.yaml` to change the real map. It is a probe tool.
5. Do not change upstream files (`src/`, `Cargo.*`, `example-mapping.yaml`) for
   owner config tasks.
6. Ask the owner before you flash the device.
7. Update [macropad.md](macropad.md) in the same turn as a map change.
8. Keep docs thin. Write in short sentences, active voice, present tense.
