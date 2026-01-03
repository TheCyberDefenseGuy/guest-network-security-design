# Secure Guest Reference Architecture

## Core components

- Guest SSID with client isolation
- Dedicated Guest VLAN / VRF
- Dedicated Guest DNS resolvers
- Public CA-signed portal certificate
- Firewall default deny to internal
- Rate-limiting and DoS protection

---

## Design summary

| Area | Decision |
|----|----|
Wireless | Isolation enabled, multicast controlled |
DNS | Dedicated Guest resolvers |
Identity | Public cert, minimal trust |
Routing | Guest isolated from RFC1918 |
Validation | Mandatory |

---

## Final statement

A secure Guest network is:
- Designed intentionally
- Validated continuously
- Treated as hostile by default
