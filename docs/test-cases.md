# Test cases and verification

This document separates checks that are runnable in the current extraction
baseline from checks that require a Linux desktop and a signaling entrypoint.
The copied code is intentionally X11-based today; Wayland validation is a
future backend milestone.

## Local baseline checks

Run from the repository root:

```bash
go test ./...
go vet ./...
for schema in protocol/schema/*.json; do jq empty "$schema"; done
git diff --check
```

Expected result: all Go tests pass, `go vet` exits successfully, every schema
parses, and `git diff --check` prints nothing.

## Server unit test cases

| ID | Area | Case | Verification |
|---|---|---|---|
| S-01 | Pipeline | Normalize empty/odd dimensions, FPS, bitrate, and RTP port | `go test ./server/desktop -run Pipeline` |
| S-02 | Pipeline | Generate browser-compatible H.264 GStreamer pipeline | Assert `I420`, `baseline`, `zerolatency`, `rtph264pay`, and `config-interval=1` in unit tests |
| S-03 | Pipeline | Generate FFmpeg fallback pipeline | Assert `x11grab`, `libx264`, `yuv420p`, baseline profile, and RTP output |
| S-04 | Input | Normalize display geometry and map mouse buttons/keys | `go test ./server/desktop -run Input` |
| S-05 | Input | Throttle/coalesce mouse movement without losing clicks or keys | Input injector unit tests |
| S-06 | Session | Replace a session with the same ID and release resources | `go test ./server/desktop -run Session` |
| S-07 | Session | Stop sessions sharing an RTP port while preserving other ports | Service unit tests |
| S-08 | WebRTC | Create a peer, attach H.264 track, accept offer, and return answer | WebRTC unit tests with a local Pion peer |
| S-09 | WebRTC | Accept only `input` and `input-move` data channels | Data-channel label tests |
| S-10 | WebRTC | Forward RTP packets to the local WebRTC track and report counters | RTP receiver test with a local UDP socket |

## Protocol checks

| ID | Case | Verification |
|---|---|---|
| P-01 | Offer requires protocol version, session ID, SDP, display, and video settings | Validate against `session-offer.schema.json` |
| P-02 | Answer binds to the same session and protocol version | Validate against `session-answer.schema.json` |
| P-03 | ICE candidate accepts nullable `sdpMid` and `sdpMLineIndex` | Validate against `ice-candidate.schema.json` |
| P-04 | Input events carry session ID and monotonic sequence | Validate against `input-event.schema.json` |
| P-05 | Coordinates remain normalized to 0..1 and event types are allowlisted | Negative schema tests |

## Linux integration checks

These require a Debian-family host with an X11 desktop, `DISPLAY`, Xauthority,
`xdotool`, and either GStreamer or FFmpeg:

```bash
command -v xdotool
command -v gst-launch-1.0 || command -v ffmpeg
test -n "$DISPLAY"
xdpyinfo >/dev/null
```

| ID | Case | Expected result |
|---|---|---|
| I-01 | Capture the active X11 display | RTP packets increase on localhost |
| I-02 | Decode the RTP stream with a local receiver | H.264 baseline/yuv420p frames decode without errors |
| I-03 | Connect one WebRTC client through the signaling adapter | ICE completes and first video frame renders |
| I-04 | Move pointer, click, type, and scroll | Events reach the intended X11 window |
| I-05 | Disconnect and reconnect with a new session ID | Old pipeline closes and the new session renders |
| I-06 | Start two clients against one node | Policy either rejects the second client or isolates sessions explicitly |

## Cross-client checks

The future stable clients must pass the same cases:

- Web browser: Chrome/Chromium desktop and mobile viewport.
- Flutter desktop: macOS/Linux/Windows adapter.
- Flutter mobile: touch gestures, virtual keyboard, rotation, reconnect.

Each client must verify video first-frame readiness, ICE failure reporting,
input-channel backpressure, reconnect, and clean session deletion.

## Current limitations

- There is no standalone signaling server command yet, so I-03 through I-06
  cannot be automated end-to-end in this repository today.
- The copied Flutter client remains a legacy snapshot and still imports
  XWorkmate App types; its tests belong to `xworkmate-app` until the package
  boundary is completed.
- The App extraction PR was merged with administrator override because its
  documentation-only branch inherited 23 unrelated baseline test failures.
  Those failures must be repaired and rerun before claiming the App repository
  is green.
- XRDP is a transport fallback for AI Desktop; it is not part of this WebRTC
  protocol test suite.
