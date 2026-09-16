# XWorkmate Remote Desktop

Standalone remote-desktop transport extracted from `xworkmate-bridge` and
`xworkmate-app` without removing the original implementations yet.

## Layout

- `server/desktop`: X11 capture, H.264 RTP pipeline, Pion WebRTC peer, input
  injection, session lifecycle, and their original tests.
- `protocol/schema`: versioned signaling and input-event contracts.
- `client/flutter/legacy`: copied Flutter/WebRTC client adapter and tests. This
  first snapshot still imports XWorkmate App types and is not yet a standalone
  Flutter package.
- `docs`: extraction plan and copied operational runbooks.
- `docs/test-cases.md`: runnable baseline, protocol, integration, and future
  cross-client verification matrix.

## Current status

This repository is an extraction baseline, not a production replacement. The
source projects remain authoritative while the standalone signaling service,
client package boundaries, authentication, packaging, and deployment are
completed.

The server currently uses X11 (`ximagesrc`/`x11grab`) and `xdotool`. Long term,
platform backends will move behind capture and input interfaces so the server
can add PipeWire/portal capture and libei input while `xworkmate-bridge` becomes
fully independent of X11.
