# AI docs hub — ch57x-keyboard-tool

This repo is a cloned upstream tool that flashes a key map to a CH57x macropad.
This hub holds project memory for AI agents. Read the hub first. Then read one
spoke. Do not read the whole repo.

## What this repo is

- Upstream tool: `kriomant/ch57x-keyboard-tool` (Rust).
- Owner use: configure one personal CH57x macropad (USB `1189:8840`).
- The binary is not committed. Build it per machine with `cargo build --release`.

## Task router

| Task | Read this spoke | Then edit |
|------|-----------------|-----------|
| Change the key map | [macropad.md](macropad.md) | `your-config.yaml` |
| Flash the device | [macropad.md](macropad.md) | run `upload` (needs sudo) |
| Find which cell a key is | [macropad.md](macropad.md) | `discovery.yaml` |
| Future intent (not built) | [vision.md](vision.md) | — |
| Control WiZ lights / fix discovery | [wiz-lights.md](wiz-lights.md) | — |
| Rules for agents | [rules.md](rules.md) | — |

## Quick facts

- The device cannot report its current map. The YAML is the only source of truth.
- Flash only over the USB cable. Wireless modes run an already-flashed map.
- Owner config file: `your-config.yaml` (repo root).
