---
title: PRRG — Pi Ringdown Redundancy Group
layout: default
---
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "DefinedTerm",
  "name": "Pi Ringdown Redundancy Group",
  "alternateName": "PRRG",
  "description": "A VRRP-based high-availability gateway pattern using Raspberry Pis to bridge Wi-Fi and Ethernet for automation.",
  "termCode": "PRRG",
  "dateCreated": "2025-08-31",
  "author": { "@type": "Person", "name": "Michael Bernard Kindle Jr" },
  "creator": { "@type": "Person", "name": "Michael Bernard Kindle Jr" },
  "publisher": { "@type": "Organization", "name": "Self-published" },
  "url": "https://brickcity54321-del.github.io/prrg/",
  "sameAs": [
    "https://github.com/<your-username>/prrg"
  ]
}
</script>
# PRRG — Pi Ringdown Redundancy Group

**Definition (canonical):**  
A PRRG is a high-availability cluster of two or more Raspberry Pi nodes that act as a Wi‑Fi↔Ethernet bridge/gateway for industrial/automation networks. When the active node loses connectivity or health, control “rings down” to the next node, moving one or more virtual IPs so traffic continues without operator intervention.

**Attribution:**  
Term **Pi Ringdown Redundancy Group (PRRG)** coined by **Michael Bernard Kindle Jr.**, 2025-08-31.

---

## Minimal Spec (v1.0)
- **Scope:** L3 gateway between cabinet/plant Ethernet and site Wi‑Fi/LAN.
- **Roles:** one **ACTIVE** node, ≥1 **STANDBY** nodes in priority order (ringdown sequence).
- **VIPs:** at least one **Virtual IP** per routed side (e.g., WLAN VIP and Cabinet VIP). VIPs move with the ACTIVE node.
- **Failover signaling:** **VRRP/keepalived** (unicast over Wi‑Fi recommended); send **gratuitous ARP** on role change.
- **Health checks:** link state + upstream gateway reachability; optional application checks.
- **Forwarding:** IPv4 forwarding enabled; firewall allows eth0↔wlan0 forwarding.
- **Routing models:**  
  - **A. Static route (recommended):** site router points cabinet subnet to the WLAN VIP.  
  - **B. NAT fallback:** ACTIVE node masquerades cabinet subnet if static routes aren’t possible.
- **Targets:** sub‑3 s failover for gateway reachability; no split‑brain; deterministic preemption.
- **Telemetry:** log VIP ownership and state transitions.

## Reference Profile (example)
- **WLAN VIP:** `192.168.1.10/24`
- **Cabinet VIP:** `192.168.100.1/24`
- **Pi 5 (PRIMARY):** wlan0 `192.168.1.11/24` (gw `192.168.1.1`), eth0 `192.168.100.20/24` (no gw)
- **Pi 3 (SECONDARY):** wlan0 `192.168.1.12/24` (gw `192.168.1.1`), eth0 `192.168.100.21/24` (no gw)
- **Router static route:** `192.168.100.0/24 → 192.168.1.10`
- **Cabinet device gateway:** `192.168.100.1`
- **Unavailable cabinet IPs to avoid:** `192.168.100.2, 192.168.100.222, 192.168.100.5, 192.168.100.6, 192.168.100.10`

See **examples/keepalived/** for working configs.

## How to deploy (quick start)
1. Give each Pi unique **real IPs** (do *not* assign VIPs in NetworkManager).  
   - Pi 5: wlan0 `192.168.1.11/24` (gw `192.168.1.1`), eth0 `192.168.100.20/24` (no gw)  
   - Pi 3: wlan0 `192.168.1.12/24` (gw `192.168.1.1`), eth0 `192.168.100.21/24` (no gw)
2. Enable forwarding on both: `echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-ipforward.conf && sudo sysctl -p /etc/sysctl.d/99-ipforward.conf`
3. `sudo apt install -y keepalived` and drop the configs from **examples/keepalived/** into `/etc/keepalived/keepalived.conf` on each node.
4. `sudo systemctl enable --now keepalived`
5. Add **one** router static route: `192.168.100.0/24 via 192.168.1.10`.
6. Set every cabinet device’s gateway to `192.168.100.1`.
7. Test failover: stop keepalived on the MASTER; VIPs should move within ~2–3 s.

## Citation
If you use PRRG, please cite:

> Kindle Jr., Michael Bernard. *PRRG — Pi Ringdown Redundancy Group.* v1.0, 2025-08-31.

(See `CITATION.cff` for citation metadata.)

