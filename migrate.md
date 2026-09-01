# homelab migration — old flat layout → pcola/river + submodules

This doc is for an agent (or human) with a pre-`d4fa225` checkout that still has the old layout:

* `homelab/` was **not** a git repo (or had stale `pcola/proxmox-host-setup` etc. without `.gitmodules`)
* `pod1`, `proxmox-host-setup`, `turnstone` lived at `homelab/pod1` (not `homelab/pcola/pod1`)
* `turnstone` pointed at `nicholasvmoore/turnstone` (archived) not `moorenix/turnstone`
* `ups-monitoring` / `homelab-monitoring` pointed at `nicholasvmoore/*` not `moorenix/homelab-monitoring`
* No `river/` stub, no `AGENTS.md`, no `README.md#TODO`, no `vars/openwrt_vars.yml` LTE backup

Current structure (see `README.md:5` + `AGENTS.md:1`):

```
homelab/  moorenix/homelab  main  d4fa225+  public
├── AGENTS.md                          # umbrella runbook + TODO/secrets contract
├── README.md                          # pcola 192.168.16.0/21 + river 192.168.24.0/23 map
├── migrate.md                         # this file
├── .gitmodules                        # pcola/* → moorenix/*
├── pcola/
│   ├── pod1/                  moorenix/pod1  main  ab497cf
│   ├── proxmox-host-setup/    moorenix/proxmox-host-setup  1-jellyfin-plex-lxc-install e87fd78
│   ├── turnstone/             moorenix/turnstone  main  2dd96e5  (unarchived)
│   └── homelab-monitoring/    moorenix/homelab-monitoring  master 631ed60
└── river/                             # stub, not a repo — part of homelab
    ├── README.md  river 192.168.24.0/23 gateway 192.168.24.1 stub
    └── AGENTS.md
```

## Option A — fresh clone (recommended)

Use when your local `homelab/` is stale, has local edits, or `homelab/.git` is missing/broken.

```bash
# 1. Backup stale checkout
mv ~/git/homelab ~/git/homelab.old.$(date +%Y%m%d)
# or: cp -a ~/git/homelab ~/git/homelab.old.$(date +%Y%m%d)  # keep until verified

# 2. Clone umbrella with submodules
git clone --recurse-submodules git@github.com:moorenix/homelab.git ~/git/homelab
cd ~/git/homelab

# 3. Verify
git log --oneline -2          # d4fa225, 3da86d1 etc.
git submodule status          # ab497cf pcola/pod1 (heads/main)
                              # e87fd78 pcola/proxmox-host-setup (heads/1-jellyfin-plex-lxc-install)
                              # 2dd96e5 pcola/turnstone (heads/main)
                              # 631ed60 pcola/homelab-monitoring (heads/master)
cat .gitmodules               # pcola/* → moorenix/*
ls -R | head -n 30
# Inventory moved: pcola/proxmox-host-setup/vars/openwrt_vars.yml, pcola/proxmox-host-setup/vars/tailscale_vars.yml:10

# 4. Verify per-repo remotes point at moorenix
git -C pcola/pod1 remote -v                    # moorenix/pod1
git -C pcola/proxmox-host-setup remote -v      # moorenix/proxmox-host-setup
git -C pcola/turnstone remote -v               # moorenix/turnstone (was nicholasvmoore, archived→unarchived)
git -C pcola/homelab-monitoring remote -v      # moorenix/homelab-monitoring (was nicholasvmoore/ups-monitoring)
```

## Option B — in-place sync (keep existing homelab/)

Use when you have local unpushed work or want to avoid re-cloning.

```bash
cd ~/git/homelab

# 1. If homelab/ was not a repo, init and attach to moorenix/homelab
if [ ! -d .git ]; then
  git init -b main
  git remote add origin git@github.com:moorenix/homelab.git
fi
git fetch origin
git checkout -B main origin/main
git submodule sync
git submodule update --init --recursive

# 2. Clean up old flat paths (if they still exist at root)
# Old: homelab/pod1, homelab/proxmox-host-setup, homelab/turnstone
# New: homelab/pcola/pod1 etc. — old dirs should be gone after checkout
ls -la                          # should show pcola/, river/, .gitmodules
ls -la pcola/                   # pod1, proxmox-host-setup, turnstone, homelab-monitoring
rmdir pcola 2>/dev/null || true # only succeeds if empty — indicates stale

# 3. Fix per-repo remotes if they still point at nicholasvmoore
git -C pcola/turnstone remote get-url origin
# if shows nicholasvmoore/turnstone:
git -C pcola/turnstone remote set-url origin git@github.com:moorenix/turnstone.git
git -C pcola/turnstone fetch && git -C pcola/turnstone checkout main && git -C pcola/turnstone pull --ff-only

# homelab-monitoring was nicholasvmoore/homelab-monitoring (and before that ups-monitoring)
git -C pcola/homelab-monitoring remote get-url origin # should be moorenix/homelab-monitoring
# if not:
git -C pcola/homelab-monitoring remote set-url origin git@github.com:moorenix/homelab-monitoring.git
git -C pcola/homelab-monitoring fetch && git -C pcola/homelab-monitoring checkout master && git -C pcola/homelab-monitoring pull --ff-only

# pod1 / proxmox-host-setup should already be moorenix — verify
git -C pcola/pod1 remote -v               # moorenix/pod1 main ab497cf
git -C pcola/proxmox-host-setup remote -v # moorenix/proxmox-host-setup 1-jellyfin-plex-lxc-install e87fd78
git -C pcola/proxmox-host-setup branch --show-current # 1-jellyfin-plex-lxc-install (tracked in .gitmodules:8)

# 4. Pick up river stub + new docs
git status --short
cat README.md | head -n 20                 # pcola 192.168.16.0/21 + river 192.168.24.0/23
cat pcola/proxmox-host-setup/vars/openwrt_vars.yml | head -n 20 # GL-AP1300 Quectel EC25-AF LTE backup
```

## After either option — catch up to new contracts

```bash
# 1. Read new agent contracts + TODOs (required at intake)
cat AGENTS.md                             # homelab umbrella runbook + TODO + secrets
cat river/AGENTS.md
cat pcola/pod1/AGENTS.md                  # pcola/pod1 TODO: river 192.168.24.1:9100, wwan0, stable pins
cat pcola/proxmox-host-setup/AGENTS.md    # TODO + docs/pcola-homelab.md#TODO + openwrt_vars river stub
cat pcola/turnstone/AGENTS.md             # TODO river LLM betelgeuse.river, stable 1.8
cat pcola/turnstone/quadlets/AGENTS.md
cat pcola/homelab-monitoring/AGENTS.md    # TODO + Proposed P2/P3/P5 wwan0

# 2. Read site TODOs
grep -n "TODO" README.md river/README.md pcola/pod1/README.md pcola/proxmox-host-setup/README.md pcola/turnstone/README.md pcola/homelab-monitoring/README.md

# 3. Secrets contract — repos are public open source, no secrets ever
# Scan before any commit (in homelab/ and each pcola/*):
git grep -IEni "password|passwd|secret|token|BEGIN.*PRIVATE KEY" $(git rev-list --all) | head
git diff --cached | head
# Real secrets live only on hosts (/var/lib/turnstone/turnstone.env chmod 600, ~/.ansible-vault/ansible_key.key, nut/upsd.users CHANGE_ME, grafana.db Telegram token, pve.yml token) or Bitwarden.

# 4. Verify clean
git status --short                # homelab: nothing to commit when clean; pcola/* show m only when bumping intentionally
git submodule status              # heads match origin/main (1-jellyfin-plex-lxc-install for proxmox)
git -C pcola/pod1 rev-list --left-right --count origin/main...HEAD           # 0 0
git -C pcola/proxmox-host-setup rev-list --left-right --count origin/1-jellyfin-plex-lxc-install...HEAD # 0 0
git -C pcola/turnstone rev-list --left-right --count origin/main...HEAD      # 0 0
git -C pcola/homelab-monitoring rev-list --left-right --count origin/master...HEAD # 0 0

# 5. Ansible sanity (if touching proxmox)
uv run ansible-playbook -i pcola/proxmox-host-setup/inventory pcola/proxmox-host-setup/main.yml --syntax-check --vault-password-file=~/.ansible-vault/ansible_key.key
```

## Future submodule bumps

After pushing to a sub-repo (e.g. `pcola/pod1`), bump the umbrella:

```bash
git submodule update --remote pcola/pod1 pcola/proxmox-host-setup pcola/turnstone pcola/homelab-monitoring
git add pcola/pod1 pcola/proxmox-host-setup pcola/turnstone pcola/homelab-monitoring
git commit -m "homelab: bump submodules" && git push
# verify
git clone --recurse-submodules git@github.com:moorenix/homelab.git /tmp/homelab-test && ls -R /tmp/homelab-test | head -n 20
```

Old paths `homelab/pod1`, `homelab/proxmox-host-setup`, `homelab/turnstone` at root are now `homelab/pcola/*` — update any local scripts, `scp`/`ssh` helpers, and CI that referenced them. Inventory moved: `vars/*` → `pcola/proxmox-host-setup/vars/*` (e.g. `vars/openwrt_vars.yml` → `pcola/proxmox-host-setup/vars/openwrt_vars.yml`).
