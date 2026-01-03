# Validation and Security Assurance

## Purpose
This document provides a **repeatable validation methodology** to prove the Guest network controls are effective.

**Rule:** configuration without validation is assumption.

---

## 1) Validation model (what we are proving)
We validate four core claims:

1. **Guest-to-Guest isolation works** (reduce lateral abuse)
2. **Boundary controls hold** (Guest cannot pivot where it shouldn’t)
3. **DNS is not leaking internal knowledge** (no internal recon surface)
4. **Guest remains usable under abuse** (basic DoS resilience)

This aligns with Zero Trust thinking: do not trust network location; validate control effectiveness.

---

## 2) Test prerequisites
- Two or more guest endpoints (laptop + phone, or two laptops)
- One endpoint with packet capture tooling (e.g., Wireshark)
- Ability to observe firewall logs/flows and DNS logs (ideal)
- A clear list of intended allowlist destinations (DNS resolvers, portal VIP/FQDN)

---

## 3) Test suite A — Client isolation (Guest protection against Guest)

### A1 — Peer reachability (L3)
**Procedure**
- Connect Client A and Client B to Guest SSID
- Attempt:
  - ICMP ping A↔B
  - TCP connect attempts to common ports (80/443/445/3389/22) from A to B

**Pass condition**
- Peer traffic blocked (unless explicitly intended)

**Why**
- Prevents direct scans and opportunistic lateral exploitation (MITRE: T1046).

---

### A2 — Peer discovery via ARP/ND observation
**Procedure**
- Run packet capture on Client A
- Observe ARP (IPv4) or ND (IPv6) activity
- Attempt to infer if peers are discoverable and reachable

**Pass condition**
- You may see some broadcast/ARP; however, you should not be able to build stable peer sessions.
- No successful peer connectivity.

**Note**
If your platform explicitly does not isolate multicast (e.g., Cisco 9800 P2P blocking), document the residual exposure and treat it with separate controls.

---

### A3 — Multicast / service discovery exposure
**Procedure**
- Observe mDNS/SSDP traffic patterns
- Determine whether clients can discover other services (printers, casting, etc.)

**Pass condition**
- For strict Guest: discovery should be blocked or minimized.
- If discovery is allowed for business reasons, document scope and apply rate limiting + monitoring.

---

## 4) Test suite B — Boundary enforcement (containment)

### B1 — RFC1918 reachability tests
**Procedure**
- Attempt traceroute/ping to representative RFC1918 ranges (10/8, 172.16/12, 192.168/16)
- Attempt TCP connections to known internal IP ranges (should fail)

**Pass condition**
- Guest cannot route to internal networks
- Explicitly allowlisted exceptions are the only reachable internal-adjacent endpoints

---

### B2 — Allowlist correctness (negative and positive testing)
**Procedure**
- Confirm:
  - Portal VIP/FQDN reachable as intended
  - Guest DNS resolvers reachable
- Confirm:
  - Non-allowlisted internal services are blocked

**Pass condition**
- Only the allowlist works; everything else fails.

---

## 5) Test suite C — DNS security assurance (prevent recon, reduce blast radius)

### C1 — Internal zone resolution attempts
**Procedure**
- Attempt to resolve known internal-only domains (e.g., `corp.local`, internal subdomains)
- Attempt to resolve common internal SRV records (if relevant to your environment)

**Pass condition**
- Guest cannot resolve internal zones

**Why**
DNS architecture is defined by RFC 1034/1035, and correct terminology matters for accurate validation.

---

### C2 — Resolver enforcement (egress control)
**Procedure**
- Attempt to query public resolvers directly (e.g., 1.1.1.1, 8.8.8.8) from Guest
- Compare behavior to policy:
  - If policy is “force guest DNS,” direct queries should be blocked.
  - If policy allows direct DNS, document the rationale and monitoring.

**Pass condition**
- Matches your intended security policy; no accidental open DNS paths.

---

### C3 — DNS abuse tolerance (lightweight, non-disruptive)
**Procedure**
- Generate controlled bursts of DNS queries from a test client (within safe bounds)
- Validate:
  - resolver logs capture spikes
  - alerts exist for sustained abnormal rates

**Standards-based guidance**
NIST SP 800-81 provides DNS deployment security guidance that supports hardening and resilience principles.

---

## 6) Test suite D — Portal and certificate trust validation

### D1 — TLS trust and SAN alignment
**Procedure**
- Access portal FQDN from Guest
- Validate certificate trust:
  - no warnings
  - correct SAN/CN alignment

**Pass condition**
- Portal is trusted by default on unmanaged clients (reduces phishing enablement)

---

### D2 — Redirect chain hygiene
**Procedure**
- Inspect captive portal redirect chain
- Ensure it does not expose:
  - internal hostnames
  - internal IPs
  - sensitive query parameters

**Pass condition**
- Redirect chain uses public/guest-safe names only.

---

## 7) Test suite E — DoS resilience (basic assurances)
Do not run disruptive tests in production without approvals. The goal is **sanity checks**.

### E1 — DHCP lease stress (safe, limited)
**Procedure**
- Perform a limited number of reconnects/renewals from test clients
- Watch DHCP pool utilization and logs

**Pass condition**
- DHCP remains stable; pool not easily exhausted by trivial behavior

### E2 — Association/auth spikes (monitoring proof)
**Procedure**
- Connect/disconnect in controlled bursts
- Confirm monitoring/alerts detect unusual behavior

**MITRE correlation**
Network Denial of Service (T1498).

---

## 8) Evidence collection (what to store as proof)
For each validation run, capture:
- timestamp, location, SSID, VLAN/VRF, policy version
- test results table (pass/fail)
- packet capture excerpts (where safe)
- firewall logs/denies summary
- DNS query logs summary
- screenshots for portal TLS proof

This becomes your “security acceptance package” for Guest.

---

## 9) Continuous assurance (make it sustainable)
A secure Guest design decays over time due to:
- new exceptions
- vendor upgrades
- topology changes
- portal/cert renewals
- DNS changes

**Recommendation**
- Run this validation:
  - after any change
  - after upgrades
  - periodically (e.g., quarterly)
- Treat failures as a security defect.

---

## Appendix — Quick mapping to MITRE ATT&CK
- Discovery: Network Service Discovery (T1046)
- AiTM: Adversary-in-the-Middle (T1557)
- Impact: Network Denial of Service (T1498) 

For “kill chain” methodology references, see Lockheed Martin Cyber Kill Chain resources.
