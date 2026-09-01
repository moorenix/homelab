# river site — `192.168.24.0/23`

Future/second site — currently **stubbed**. `pcola` (`192.168.16.0/21`, gateway `192.168.16.1`) is primary.

## Planned

* **Gateway** `192.168.24.1` — OpenWrt + `node_exporter :9100` (tailscale0 + wwan0 if LTE)
* **CIDR** `192.168.24.0/23` (covers `192.168.24.0 - 192.168.25.255`)
* Prometheus scrape `192.168.24.1:9100` (see `pcola/pod1/monitoring/prometheus/tailscale.yml.example`)
* Grafana dashboard `tailscale-adapter` will auto-discover `192.168.24.1:9100`

## Inventory stubs

* `pcola/proxmox-host-setup/vars/tailscale_vars.yml:17` `ip: 192.168.24.1` `enabled: false`
* `pcola/proxmox-host-setup/vars/wireguard_vars.yml:18` `ip: 192.168.24.1` `enabled: false` (legacy)
* `pcola/pod1/README.md: Adding 192.168.24.1 later (Tailscale — river site)`
* `pcola/pod1/monitoring/grafana/dashboards/Tailscale/tailscale.json` — `Instance` variable `label_values(node_network_receive_bytes_total{device="tailscale0"}, instance)` will include `192.168.24.1:9100` after scrape

## When river device ships

1. Flash OpenWrt, configure `lan` `192.168.24.1/23`, `node_exporter` `:9100`, `tailscale0`
2. In `pcola/proxmox-host-setup/vars/tailscale_vars.yml` set `192.168.24.1` `enabled: true` (add `site: river, cidr: 192.168.24.0/23`)
3. Copy `pcola/pod1/monitoring/prometheus/tailscale.yml.example` -> `~/monitoring/prometheus/prometheus.yml` `openwrt-site2` job and `kill -HUP 1` prometheus on `pod1`
4. Verify: `curl http://192.168.24.1:9100/metrics | grep tailscale0` and `curl "http://192.168.16.146:9090/api/v1/query?query=node_network_receive_bytes_total{device=\"tailscale0\",instance=\"192.168.24.1:9100\"}"`
5. Add river hosts under this `river/` directory (e.g., `river/pod2/`, `river/proxmox-...`) mirroring `pcola/` layout

No secrets committed — APN/PIN for any future LTE at river goes in vault (`vars/proxmox_vars.yml` or `river/vars/`), not here.

## TODO

- [ ] **Gateway** `192.168.24.1/23` `OpenWrt` `node_exporter :9100` `tailscale0` (`wwan0` if LTE) — flash, set `lan` `192.168.24.1/23`, verify `curl http://192.168.24.1:9100/metrics | grep tailscale0` — see `pcola/proxmox-host-setup/vars/tailscale_vars.yml:17` `river` `192.168.24.0/23`
- [ ] **Prometheus scrape** — uncomment `pcola/pod1/monitoring/prometheus/tailscale.yml.example` `openwrt-site2` `192.168.24.1:9100` into `pod1:~/monitoring/prometheus/prometheus.yml`, `podman exec prometheus kill -HUP 1`
- [ ] **Grafana** — `tailscale-adapter` `label_values(node_network_receive_bytes_total{device="tailscale0"}, instance)` should auto-show `192.168.24.1:9100` after scrape; verify `curl "http://192.168.16.146:9090/api/v1/query?query=node_network_receive_bytes_total{device=\"tailscale0\",instance=\"192.168.24.1:9100\"}"`
- [ ] **Hosts** — add `river/pod2/` (mirror `pcola/pod1` layout) + `river/proxmox-...` docs here, update `homelab/README.md` site table + `homelab/.gitmodules` when provisioning
- [ ] **LTE if needed** — reuse `pcola/proxmox-host-setup/docs/pcola-homelab.md` QMI/`mwan3` pattern for river `wwan0` backup; document APN in vault `river/vars/` or `pcola/proxmox-host-setup/vars/`
