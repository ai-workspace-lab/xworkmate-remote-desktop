# Extraction plan

## Baseline

The initial copy preserves working server and client code. No source code is
removed from `xworkmate-bridge` or `xworkmate-app` in this phase.

## Ownership boundary

This repository will own screen capture, encoding, WebRTC media and data
channels, input injection, signaling contracts, and reusable clients.
`xworkmate-bridge` will retain authentication and product routing during the
migration, then become independent of X11. `xworkmate-app` will retain product
UI and consume a standalone Flutter client adapter.

## Migration phases

1. Preserve and test the copied X11 implementation.
2. Introduce transport-neutral protocol types and a standalone signaling API.
3. Extract the Flutter client behind a signaling interface.
4. Package the server, systemd unit, and Ansible role.
5. Switch source projects to the standalone server/client.
6. Remove the duplicated implementations only after production validation.
7. Add PipeWire/portal capture and libei input backends for Wayland.
