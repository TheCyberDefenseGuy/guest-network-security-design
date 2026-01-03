# Guest Network Security Design
## Attack Surface, Threat Modeling, and Secure Deployment Patterns

### Why this repository exists

Enterprise Guest networks are often designed with a **false assumption**:

> “Guest is isolated, therefore it is safe.”

In reality, Guest networks are:
- Highly exposed
- Frequently misconfigured
- Poorly validated
- Rarely threat-modeled

They often become:
- A platform for lateral attacks between guests
- A reconnaissance surface for internal infrastructure
- A DoS vector against wireless, DNS, DHCP, and firewalls
- A weak point in Zero Trust architectures

This repository documents **how to design Guest networks as a security project**, not just a connectivity feature.

---

## Scope and approach

This is **not** a basic configuration guide.

This repository focuses on:
- **Security-driven design decisions**
- **Attack surface analysis**
- **Threat modeling**
- **Failure modes and real risks**
- **Mitigations mapped to each attack phase**
- **Validation and assurance**

Configuration examples are discussed **only when they support a security decision**.

---

## Design philosophy

The Guest network must be treated as:

- Untrusted by default
- Internet-adjacent
- Attacker-accessible
- High churn / low identity assurance

Security is achieved through:
- Explicit isolation (not implicit trust)
- Layered controls
- Minimal exposure
- Strong DNS and identity design
- Continuous validation

---

## Technology context

The design principles here are:
- **Vendor-agnostic**
- Based on **open standards**
- Aligned with **Zero Trust (NIST SP 800-207)**

Cisco ISE and Cisco Wireless are used as **reference implementations**, not as requirements.

---

## Key topics covered

- Guest attack surface and threat modeling
- Full attack chronology (Kill Chain + MITRE ATT&CK)
- Client Isolation: what it really does and where it fails
- DNS as a primary exposure vector
- Certificates, FQDNs, and portal trust chains
- Cisco ISE Guest design considerations
- Secure reference architecture
- Validation playbooks

---

## Intended audience

- Network Security Engineers
- Wireless Architects
- NAC / Zero Trust Engineers
- Blue Team practitioners
- Security Architects

---

## Disclaimer

All scenarios are for **defensive and educational purposes only**.
