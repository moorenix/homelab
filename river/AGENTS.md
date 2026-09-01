# AGENTS — river (`192.168.24.0/23`) stub

This directory is **not a separate Git repo** — it is part of `moorenix/homelab` (`homelab/river/`). `pcola/proxmox-host-setup/vars/tailscale_vars.yml:17` `192.168.24.1` `enabled:false` is the source-of-truth stub.

## TODO

Source of truth: `README.md#TODO` (gateway `192.168.24.1/23` `node_exporter :9100` `tailscale0`, prometheus `openwrt-site2`, grafana `tailscale-adapter`, `river/pod2` hosts, LTE QMI if needed).

**Agent rule:** at intake read `README.md#TODO` and `../AGENTS.md#TODO`. If you implement the gateway, flip the `tailscale_vars.yml` stub, add `pod1:~/monitoring/prometheus/prometheus.yml` scrape, verify `curl http://192.168.24.1:9100/metrics`, and update checkboxes here + `../README.md#TODO`. Propose next river TODO when idle.

## Layout when live

```
river/
├── AGENTS.md          # this file
├── README.md          # bring-up steps
├── pod2/              # mirror pcola/pod1 if needed
└── vars/              # river-specific ansible vars if split from pcola/proxmox-host-setup
```

Do not commit secrets (`APN`, `PIN`) — use `~/.ansible-vault/ansible_key.key` + Bitwarden, same as `pcola`.
