# AGENTS — homelab (umbrella)

This is the **umbrella** `moorenix/homelab` repo (`pcola 192.168.16.0/21` + `river 192.168.24.0/23`). It holds **no live code** — only `README.md` site map, `river/` stub, and `pcola/*` submodules (`moorenix/pod1`, `moorenix/proxmox-host-setup`, `moorenix/turnstone`, `moorenix/homelab-monitoring`).

## Sites

* `pcola` `192.168.16.0/21` — gateway `192.168.16.1` `GL-AP1300` `Quectel EC25-AF 2c7c:0125` `wwan0` backup, `pod1 192.168.16.146`, `brix` Proxmox.
* `river` `192.168.24.0/23` — stub `192.168.24.1:9100` not yet deployed.

## Submodule contract

* Each `pcola/*` is a **real GitHub repo** under `moorenix` (see `README.md:1` table + `.gitmodules:1`). Do **not** edit files inside `pcola/*` directly unless you intend to commit there — they are separate remotes.
* Workflow after pushing a sub-repo: bump the pointer here:
  ```bash
  git submodule update --remote pcola/pod1 pcola/proxmox-host-setup pcola/turnstone pcola/homelab-monitoring
  git add pcola/pod1 pcola/proxmox-host-setup pcola/turnstone pcola/homelab-monitoring
  git commit -m "homelab: bump submodules" && git push
  ```
* Verify: `git clone --recurse-submodules git@github.com:moorenix/homelab.git`

## TODO

Canonical TODOs live **in each sub-project** plus this umbrella. All agents **must check TODO at intake and periodically**:

* **This repo:** `README.md#TODO` + `river/README.md#TODO`
* **`pcola/pod1`:** `pcola/pod1/README.md#TODO` (river `192.168.24.1:9100`, `wwan0` panels, stable pins)
* **`pcola/proxmox-host-setup`:** `pcola/proxmox-host-setup/README.md#TODO` + `pcola/proxmox-host-setup/docs/pcola-homelab.md#TODO before enabling` + `vars/openwrt_vars.yml:openwrt_lte_backup` + `vars/tailscale_vars.yml:17` river stub
* **`pcola/turnstone`:** `pcola/turnstone/README.md#TODO` + `pcola/turnstone/quadlets/AGENTS.md#TODO` (river LLM `betelgeuse.river.moorenix.com`, stable `1.8`, postgres `18`, ssh-proxmox MCP)
* **`pcola/homelab-monitoring`:** `pcola/homelab-monitoring/README.md#TODO` + `pcola/homelab-monitoring/AGENTS.md#Proposed` (site2 `P2/P3`, `pancakesmp` `P5`, `wwan0` LTE panels)
* **`river`:** `river/README.md#TODO` (gateway `192.168.24.1/23`)

**Agent rule:** at session start or before closing a task, `read` the relevant `README.md#TODO` (and `AGENTS.md#TODO` if present). If you complete a TODO item, **update the checkbox** (`- [ ]` → `- [x]` with date/commit) in that project's README and, for umbrella, bump the submodule. If idle and TODOs remain, **propose the next TODO** to the user instead of exiting.

## Runbook

1. **Intake:** confirm target site (`pcola` vs `river`) and sub-repo(s). Read that sub-repo's `AGENTS.md` + `README.md#TODO`.
2. **Plan:** break into tasks, delegate to specialist sub-agents if needed.
3. **Execute:** change the owning sub-repo first, push, then bump umbrella if needed. Keep `pcola/homelab-monitoring` `master` vs others `main` branch hygiene in mind (see `README.md#TODO`).
4. **Validate:** `git status --short` shows `m pcola/*` only when pointer bump is intended; `git submodule status` matches `origin/main` (`1-jellyfin-plex-lxc-install` for proxmox).
5. **Report:** summarize which TODOs were touched, which remain.
