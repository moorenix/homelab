# river MokerLink POE-2G080110GSM — 192.168.24.2

Managed PoE switch at `river` `192.168.24.0/23` — gateway `192.168.24.1` is **not** this device; this is the PoE/access switch for river hosts. Firmware `V1.9.1` `2024-06-14` `MAC 78:D8:00:31:97:B0`.

Web: `http://192.168.24.2` (`login.cgi` `hex_md5(username+password)` → `admin` cookie) — user `nicholas` (password lives only in Bitwarden/vault, **never in this repo**; change via `user.cgi` → `mname/mpass`). PoE control: `pse_port.cgi` `portid 0-7 → Port 1-8` `state 0 Disable / 1 Enable` `cmd=poe`. `pse_system.cgi` shows `Consumption 15.759W`.

## Port map (verified 2026-09-02, `pse_port.cgi` + `port.cgi` + `mac.cgi`)

| Port | PoE State | PoE On/Off | Class | Power/Current | Data State | Speed | Use | Host | IP | MAC |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Enable | Off | — | — | Enable | Auto → Link Down | spare | — | — | — |
| 2 | Enable | Off | — | — | Enable | Auto → Link Down | spare | — | — | — |
| 3 | Enable | Off | — | — | Enable | Auto → 2500Full | spare/uplink? | — | — | 38:05:25:38:6D:08 (port 3 MAC table) |
| 4 | Enable | Off | — | — | Enable | Auto → 1000Full | spare/uplink? | — | — | E8:70:72:D6:72:C8 etc. |
| **5** | **Enable** | **On** | **Class3** | **4.8W 51V 95mA** | **Enable** | **Auto → 100Full** | **Pi** | **192.168.24.10** | **B8:27:EB:2D:F3:D8** (`smsc95xx` via `r8152` hub, `ST2000LM007` HDD `sda` `1.8T` `btrfs` `72a988...` + `Tripp Lite INTERNET600U sn 3408AV4...` behind Pi hub) |
| 6 | Enable | Off | — | — | Enable | Auto → 2500Full | spare/uplink? | — | — | — |
| 7 | Enable | On | Class4 | 4.5-8.3W 51V 88-162mA | Enable | Auto → 2500Full | other PoE device (not Pi) | — | 28:73:F6:93:CB:52 etc. (port 7 MAC table) |
| 8 | Enable | Off | — | — | Enable | Auto → 2500Full | spare/uplink? | — | — | 94:83:C4:A0:5B:FF etc. (port 8) |
| 9 (SFP) | Enable | — | — | — | Enable | Auto → Link Down | spare | — | — | — |

- **Port 5 is Pi** — user confirmed, `4.488W→4.845W Class3 95mA 100Full` matches Pi `100M` `eth0` `192.168.24.10/24` `ip=...:rpi:eth0:off` (static). Power-cycled 2026-09-02 via `pse_port.cgi portid=4 Disable→Enable` (rebooted Pi, `up 6 min` `ping 2.6ms`).
- **Port 7 is other PoE** `7.9W Class4 155mA` — not Pi; left `Enable On` (was `8.262W` before reboot).
- Switch **does not support port description/alias** in `V1.9.1` web UI (`port.cgi` only State/Speed/Duplex/Flow Control) or `config_back.cgi` binary (no text alias). Telnet `23` and `ssh 22` timeout — no CLI. Labeling is **external docs only** (this file). Future firmware may add alias; backup via `config_back.cgi?cmd=conf_backup` → `2.6K` binary.

## Operations

```bash
# PoE power cycle Pi (Port 5 → portid 4). Password from vault — never in repo:
# admin_cookie=$(md5sum <<< "$(vault_password)" | cut -d' ' -f1)  # login.cgi hex_md5(username+password)
curl -s -b "admin=REDACTED" -X POST -d "portid=4&state=0&submit=Apply&cmd=poe" http://192.168.24.2/pse_port.cgi
sleep 5
curl -s -b "admin=..." -X POST -d "portid=4&state=1&submit=Apply&cmd=poe" http://192.168.24.2/pse_port.cgi
# verify
curl -s -b "admin=..." http://192.168.24.2/pse_port.cgi | grep -E "Port 5" -A2
ping -c2 192.168.24.10
```

Inventory: `river/README.md` stub + `pcola/proxmox-host-setup/vars/tailscale_vars.yml:17` `192.168.24.1` `enabled:false` (gateway) vs `192.168.24.2` switch here.

Update this table when you populate ports 1-4/6/8-9; also update `homelab/README.md` site table and `homelab/.gitmodules` if river gets `river/pod2` hosts.

Site-2 Pi USB troubleshooting (Tripp Lite `09ae:3024` 3.3 s flap on the Pi Zero 2 W + Waveshare PoE/ETH/USB HUB HAT): `pcola/homelab-monitoring/planned/site2-remote-b/USB_FLAP_RESEARCH.md`.
