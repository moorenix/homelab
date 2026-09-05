# river site DNS — `192.168.24.1` (GL-X3000, GL.iNet fw 4.7.4 / OpenWrt 21.02)

Resolver for the river LAN is **dnsmasq 2.85** on the gateway. Last verified 2026-09-05.

## Resolution paths

| Query | Answered by |
|---|---|
| `*.river.moorenix.com` (DHCP clients) | main dnsmasq from leasefile (`expand-hosts`, `domain=river.moorenix.com`, `local=/river.moorenix.com/`) |
| `*.river.moorenix.com` (static devices) | static `host` sections in `/etc/config/dhcp` (see table below) |
| `river.moorenix.com` (apex) | `/etc/hosts` → `192.168.24.1` |
| `*.pcola.moorenix.com` | forwarded to `192.168.16.1` (pcola gateway) **source-bound to tailnet IP** (see gotcha) |
| `*.ts.net` | `100.100.100.100` (tailscaled MagicDNS) |
| everything else | `1.1.1.1`, `1.0.0.1` (`noresolv=1`, carrier DNS ignored) |

Effective UCI (`/etc/config/dhcp`, section `dhcp.@dnsmasq[0]`):

```
list server '1.1.1.1'
list server '1.0.0.1'
list server '/pcola.moorenix.com/192.168.16.1@100.79.38.67'
option noresolv '1'
option filter_aaaa '1'
option rebind_protection '0'
option localservice '0'
```

Generated config: `/var/etc/dnsmasq.conf.cfg01411c` (restart via `/etc/init.d/dnsmasq restart`).

## Static hosts

| Hostname | IP | Notes |
|---|---|---|
| `monitoring-pi.river.moorenix.com` | `192.168.24.10` | DNS-only host entry (`option dns '1'`), no DHCP reservation |

DHCP: LAN pool `192.168.24.50–249`/24, lease 720m; DHCPv6/RA disabled (IPv4-only by design: `filter_aaaa=1`).

## GL.iNet gotcha: `process_mark_dns` bypasses VPN for dnsmasq

GL firmware marks all packets from the `dnsmasq` GID with fwmark `0x8000/0xc000`
(`iptables -t mangle -L OUTPUT`, rule `process_mark_dns`), forcing them off the
WireGuard tunnel (table 8000) onto table 52 (`192.168.16.0/21 dev tailscale0`).
Consequence: **a plain `server=/pcola.moorenix.com/192.168.16.1` forward egresses
tailscale0 with source `172.16.200.2` (non-tailnet IP) and gets no replies** —
verified via `tcpdump -i tailscale0` + conntrack `UNREPLIED`.

Fix: bind the forward to the router's tailnet IP (dnsmasq `@source` syntax):

```
server=/pcola.moorenix.com/192.168.16.1@100.79.38.67
```

Do NOT "fix" by changing dnsmasq's user/group or deleting the mark rule — that
breaks GL's DNS-leak prevention design (see GL docs: VPN Dashboard → Tunnel
Options → "Services from GL.iNet Use VPN"). The sanctioned heavier alternative
is that tunnel option, which moves *all* router-originated traffic into the VPN.

Related: the GL VPN-policy dnsmasq instance (`/etc/dnsmasq.conf.vpn`, port 1653,
runs as `root:explict_vpn`) forwards streaming domains in `/tmp/dnsmasq.d/via_domain`
to `192.168.16.1` over `wgclient` — unaffected, works.

## UCI options this firmware's init script ignores

- `list hostrecord` on `dhcp.@dnsmasq[0]` is stored but **not rendered** into the
  generated config — use `/etc/hosts` or `dhcp host` sections instead.

## GUI warning

These settings were made via UCI/SSH, not the web UI (NETWORK → DNS only knows
Automatic/Manual/Encrypted/Proxy modes; `gl-dns` mode is `auto`,
`override_vpn='0'`). **Re-check `/etc/config/dhcp` after any GUI DNS change** —
the GUI may rewrite the dnsmasq section. AdGuardHome and Stubby are present but
disabled (`enabled='0'`).

## Quick verification (from a river LAN client)

```
dig +short brix.pcola.moorenix.com @192.168.24.1        # → 192.168.16.21
dig +short monitoring-pi.river.moorenix.com @192.168.24.1  # → 192.168.24.10
```

Trailing "No answer" on BusyBox `nslookup` is the filtered AAAA query, not a failure.
