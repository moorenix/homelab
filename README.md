# homelab

Multi-site homelab — `pcola` is primary, `river` is stubbed future site.

```
homelab/
├── pcola/                         # site pcola: 192.168.16.0/21 (gateway 192.168.16.1)
│   ├── pod1/                      # Fedora CoreOS 44 — 192.168.16.146 — podman quadlets
│   ├── proxmox-host-setup/        # Proxmox VE host brix + LXC, ansible playbooks
│   │   ├── vars/openwrt_vars.yml  # 192.168.16.1 GL-AP1300 + Quectel EC25-AF LTE (backup)
│   │   ├── vars/tailscale_vars.yml # tailscale0 192.168.16.1 (pcola), 192.168.24.1 (river stub)
│   │   └── docs/pcola-homelab.md  # gateway inventory + LTE failover design
│   ├── turnstone/                 # Turnstone AI orchestration quadlets (deploy on pod1)
│   └── homelab-monitoring/        # UPS + Grafana/Prometheus/Telegraf (NUT vpn 192.168.16.4, pod1 stack)
└── river/                         # site river: 192.168.24.0/23 (stub, future device)
    └── README.md                  # placeholder — add 192.168.24.1 openwrt + hosts here
```

## Sites

### pcola `192.168.16.0/21`

* **Gateway** `192.168.16.1` `OpenWrt.pcola.moorenix.com` GL.iNet GL-AP1300 (IPQ40xx) OpenWrt 24.10.0 + Quectel EC25-AF `2c7c:0125` QMI (`cdc-wdm0`/`wwan0`/`ttyUSB0..3`) — not PCI, USB. LTE is **backup** for `wan` (mwan3/qmi, metric 10 vs 20).
* **pod1** `192.168.16.146` Fedora CoreOS — edge-caddy, monitoring (prometheus `openwrt:9100`), turnstone pod.
* **brix** `brix.pcola.moorenix.com` Z790 Proxmox VE 9.2.0 — no LTE, all LTE on gateway.

See `pcola/proxmox-host-setup/docs/pcola-homelab.md` and `pcola/proxmox-host-setup/vars/openwrt_vars.yml`.

### river `192.168.24.0/23`

Stubbed — `192.168.24.1` disabled in `pcola/proxmox-host-setup/vars/tailscale_vars.yml` + `vars/wireguard_vars.yml`. Prometheus target `192.168.24.1:9100` not yet scraped. When river device ships:

1. Deploy OpenWrt at `192.168.24.1` (node_exporter `:9100` + tailscale0)
2. Enable `tailscale_vars.yml: ip 192.168.24.1 enabled:true`
3. Add `pcola/pod1/monitoring/prometheus/tailscale.yml.example` job
4. Grafana `tailscale-adapter` instance variable auto-discovers `192.168.24.1:9100`

## Inventory sources

* **pcola gateway + LTE:** `pcola/proxmox-host-setup/vars/openwrt_vars.yml` (structured), `pcola/proxmox-host-setup/docs/pcola-homelab.md` (human + failover plan)
* **Tailscale endpoints:** `pcola/proxmox-host-setup/vars/tailscale_vars.yml`
* **WireGuard (legacy):** `pcola/proxmox-host-setup/vars/wireguard_vars.yml` (kept for history)
* **Pod1 services:** `pcola/pod1/README.md` `pcola/pod1/UPGRADE.md`
* **Turnstone:** `pcola/turnstone/quadlets/README.md`
* **Monitoring (UPS/metrics):** `pcola/homelab-monitoring/` — NUT + telegraf/prometheus/grafana (owned, linked from pod1)

## Deploy

```bash
# pcola site — example: validate ansible
uv run ansible-playbook -i pcola/proxmox-host-setup/inventory pcola/proxmox-host-setup/main.yml --syntax-check --vault-password-file=~/.ansible-vault/ansible_key.key
uv run ansible-playbook -i pcola/proxmox-host-setup/inventory pcola/proxmox-host-setup/main.yml --check --diff --vault-password-file=~/.ansible-vault/ansible_key.key

# pod1
scp -r pcola/pod1/edge/quadlets/* nicholas@192.168.16.146:~/.config/containers/systemd/
scp -r pcola/turnstone/quadlets/* nicholas@192.168.16.146:~/.config/containers/systemd/system/
```

Old paths `proxmox-host-setup/`, `pod1/`, `turnstone/` at homelab root are now `pcola/...` — update scripts accordingly.
