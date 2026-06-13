# discord-proxy

C++ rewrite, plus fixed voice bypass via fake QUIC Initial packets

## Usage

1. Download `proxy` from the release page
2. Close Discord (or kill it in installer)
3. Enter your proxy:
    - **URL** field `http://127.0.0.1:10808` or `socks5://user:pass@host:1080`
    - **or** Fill manually and pick scheme/host/port/user/pass
4. Pick UDP packet preset if your provider needs more than the built in QUIC fake (try **QUIC google.com** first for Russian DPI)
5. Click **Install** and restart Discord

To uninstall: run the installer again and press **Uninstall**

## Features

- HTTP / HTTPS / SOCKS5 proxies (with `user:pass@` auth)
- All three Discord: Stable, PTB, Canary
- UDP voice bypass with fake QUIC initial payload instead of the original drover `0x00 0x01`
- Bundled packet presets (QUIC google.com, QUIC dbankcloud, STUN)
- Single file installer, zero runtime deps

## Manual install

1. Build (or download) `version.dll`
2. Copy `version.dll` and `discord-proxy.ini` into `%LocalAppData%\Discord\app-x.y.z\` (folder with `Discord.exe`).
3. Configure `discord-proxy.ini`

## Configuration for `discord-proxy.ini`
```ini
[proxy]
; if empty direct mode, no proxy, only UDP DPI bypass
url =

[udp]
; legacy - send 0x00 then 0x01 before real packet, get from https://github.com/hdrover/discord-drover (doesnt work for me)
; quic - send fake QUIC Initial (TLS ClientHello to www.google.com)
; file - send content of packet_file before the real packet
; auto - file if discord-proxy-packet.bin exists else quic
mode = auto

; optional override packet place near config
packet_file = discord-proxy-packet.bin

; resend fake every N seconds during voice (if 0 only at start)
resend_interval_sec = 0

[log]
; off, error, info, debug
level = error
; log file. Empty = no file
file =
```

### Optional `discord-proxy-packet.bin`

Place custom UDP fake packet next to `version.dll`. It will reread on new voice connection, no Discord restart needed

## Build from source

Requirements: Windows, Visual Studio 2022 (Desktop C++ workload), CMake 3.20+.
```cmd
cmake -B build -G "Visual Studio 17 2022" -A x64
cmake --build cmake-build-release --config Release
```

Outputs:
- `build\proxy\Release\version.dll`
- `build\installer\Release\discord-proxy-installer.exe`

MinHook is fetched automatically

## Troubleshooting

**Voice connects then drops to 5000 ms ping**
Try another packet

**0 dirs touched on install**
Discord wasnt found in `%LocalAppData%\Discord` / `%LocalAppData%\DiscordPTB` / `%LocalAppData%\DiscordCanary`
