# Remote Desktop Protocol v1

The media plane is WebRTC H.264 video. Reliable keyboard, button, and scroll
events use the ordered `input` data channel. High-frequency mouse movement uses
the unordered, short-lived `input-move` channel.

Signaling is intentionally transport-neutral. The schemas define payloads that
may be carried over HTTP, WebSocket, ACP during migration, or another gateway.
