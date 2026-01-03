# Threat Model and Attack Surface

## Primary attack surfaces

1. Wireless medium
2. Client-to-client communication
3. DNS infrastructure
4. DHCP services
5. Captive portal and redirect flows
6. Firewall and routing boundaries

---

## Typical threat actors

- Opportunistic attackers
- Insider threats using Guest
- Malicious visitors
- Compromised BYOD devices

---

## High-risk assumptions (common failures)

- “Client isolation is enabled by default”
- “Guest DNS can be the same as internal”
- “Portal exceptions are harmless”
- “No internal routing means no risk”
