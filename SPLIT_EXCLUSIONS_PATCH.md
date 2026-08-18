# IPv4 split-exclusions patch

This fork keeps the upstream split-tunneling UI and fixes IPv4-only AWG/WireGuard imports so that exclusion mode works when the peer has `0.0.0.0/0` without `::/0`.

Patched files:
- `client/vpnconnection.cpp`
- `client/platforms/ios/PacketTunnelProvider+WireGuard.swift`

Build helper:
- `.github/workflows/ios-ipa-manual.yml`
- `BUILD_WITHOUT_MAC_RU.md`
