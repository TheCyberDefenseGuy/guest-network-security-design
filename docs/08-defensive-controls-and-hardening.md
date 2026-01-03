# Defensive Controls and Hardening

## Purpose
This document provides a **defense-first hardening baseline** for enterprise guest networks, with a primary objective of **protecting the Guest network itself** (availability, fairness, user safety), while also preventing Guest from becoming a pivot into internal environments.

**Key principle:** Guest is an **untrusted, attacker-accessible zone**. Security must be **explicit**, **layered**, and **validated**. (Zero Trust aligns with protecting resources over trusting network location.)

---

## 1) Threats this hardening must address
Guest networks are vulnerable to:

- **Guest-to-Guest abuse**: scanning, opportunistic sniffing attempts, ARP-based AiTM attempts  
- **Infrastructure abuse**: attacks on DHCP, DNS, WLC/AP/controller resources, firewall state tables  
- **DoS / resource exhaustion**: airtime abuse, association floods, DHCP starvation, DNS query floods  
- **Onboarding/portal attacks**: phishing lookalike portals, redirect manipulation, TLS trust degradation  
- **Recon**: learning topology via DNS leakage, service discovery via multicast, etc.

MITRE ATT&CK correlations commonly seen in Guest scenarios include:
- Network Service Discovery (T1046)
- Adversary-in-the-Middle (T1557)
- Network Denial of Service (T1498)

---

## 2) Layered defensive model (what “secure guest” actually means)

### Layer A — Wireless/Edge isolation (reduce Guest-to-Guest harm)
**Goal:** protect guests from each other and reduce local attack surface.

**Controls**
1. **Enable client isolation / peer-to-peer blocking on the Guest SSID**
   - Cisco Catalyst 9800: peer-to-peer blocking is **not enabled by default** and **does not apply to multicast traffic**.
   - Meraki: client isolation for Bridge mode SSIDs is **disabled by default**.
   - Aruba: “Deny Intra VLAN Traffic / Client Isolation” disables peer-to-peer communication and is configured explicitly.

2. **Explicit multicast/broadcast stance**
   - Do not assume client isolation handles multicast; for Cisco 9800 peer-to-peer blocking, multicast is explicitly not covered.
   - If multicast discovery is not needed, restrict it. If needed, scope it tightly and rate-limit where possible.

**Common failure modes**
- Isolation enabled on SSID but broken by upstream routing/VLAN pooling assumptions
- Multicast still allows service discovery patterns (use-case dependent)
- “Temporary exceptions” for printers/Chromecast/etc. silently reintroduce lateral surfaces

---

### Layer B — L3/L4 segmentation and policy enforcement (prevent pivot and reduce blast radius)
**Goal:** make Guest a **contained security domain** with minimal exposure.

**Controls**
1. **Dedicated Guest VRF/VLAN + strict boundary firewall policy**
   - Default deny Guest → RFC1918/internal
   - Allowlist only explicitly required services (e.g., portal VIP, Guest DNS resolvers)

2. **Enforce DNS egress**
   - Allow Guest clients to reach **only** approved Guest DNS resolvers on UDP/TCP 53
   - Block direct DNS to arbitrary resolvers unless your policy explicitly allows it

3. **State-table and connection protection**
   - Apply per-client connection rate limits to protect firewalls and gateways from state exhaustion (practical DoS resilience)
   - Prefer “abuse-tolerant” policy design for Guest (shorter timeouts, thresholds)

---

### Layer C — Dedicated Guest DNS design (stop recon, reduce outage coupling)
**Goal:** protect Guest users and infrastructure by treating DNS as a security control, not just plumbing.

**Controls**
1. **Dedicated Guest recursive resolvers**
   - Do **not** use the same internal DNS resolvers used by corporate endpoints.
   - This reduces internal naming disclosure and blast radius.

2. **Minimal split-horizon usage**
   - If you require split-horizon for portal FQDN reachability, scope it to the minimum set of names required.

**Standards-backed guidance**
- DNS core architecture is defined in RFC 1034 and RFC 1035.
- DNS terminology (to avoid design confusion) is consolidated in RFC 8499. 
- NIST provides enterprise DNS security deployment guidance in SP 800-81. 

---

### Layer D — Portal/certificate trust (protect users from phishing and trust downgrade)
**Goal:** preserve trust in onboarding and prevent portal mechanics from becoming an attack surface.

**Controls**
- Use a dedicated portal FQDN that guests can resolve without internal DNS access.
- Use a public CA-signed certificate that matches the portal FQDN (avoid “click through” culture).
- Inspect redirect chains to ensure they do not expose internal hostnames/IPs.

---

### Layer E — DoS resilience (keep Guest usable under attack)
**Goal:** keep the guest network stable and fair under hostile conditions.

**Controls**
1. **DHCP protections**
   - Rate limits, pool sizing, monitoring for starvation patterns
2. **DNS protections**
   - Resolver capacity planning, rate limiting, caching strategy
   - Avoid coupling Guest DNS to internal DNS availability
3. **Wireless protections**
   - Monitor association/authentication anomalies (flood patterns)
   - Enable management frame protection where feasible (e.g., PMF/802.11w)

MITRE mapping: Network Denial of Service (T1498).

---

## 3) Hardening checklist (design + configuration acceptance criteria)

### Wireless / SSID
- [ ] Client isolation / P2P blocking enabled on Guest SSID
- [ ] Multicast behavior explicitly defined (allowed/blocked/limited)
- [ ] VLAN pooling reviewed for bypass risk
- [ ] Guest SSID policy does not rely on “defaults” (vendor-specific defaults)

### Network boundary
- [ ] Dedicated Guest VLAN/VRF
- [ ] Guest → internal/RFC1918 denied by default
- [ ] Only allowlisted infra reachable from Guest (DNS, portal VIP, etc.)

### DNS
- [ ] Guest uses dedicated recursive resolvers
- [ ] DNS egress is enforced (block arbitrary resolvers)
- [ ] No internal zone leakage to Guest
- [ ] DNS hardening aligned with NIST guidance

### Portal and certificate
- [ ] Portal FQDN resolvable from Guest
- [ ] TLS certificate matches portal FQDN
- [ ] Redirect chain does not leak internal hostnames/IPs

### Monitoring and detection
- [ ] Alerts for DHCP anomalies (starvation, spikes)
- [ ] Alerts for DNS query rate spikes
- [ ] Alerts for WLC/AP association/auth spikes
- [ ] Guest firewall state usage monitored

---

## 4) Practical “secure guest” definition
A Guest network is “secure enough” when:
- Guest-to-guest abuse is materially reduced (isolation works)
- Infrastructure services are resilient to abuse (DNS/DHCP/WLC not fragile)
- The boundary policy is enforceable and auditable
- The onboarding flow is trustworthy
- All of the above is validated continuously (see validation chapter)
