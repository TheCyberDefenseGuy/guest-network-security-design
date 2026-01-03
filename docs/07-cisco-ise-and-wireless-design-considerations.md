# Cisco ISE and Wireless Guest Design Considerations

## Cisco ISE Guest is not just a portal

ISE Guest introduces:
- DNS dependencies
- Certificate dependencies
- Redirect flows
- Firewall exceptions

Each one is a security decision.

Cisco ISE Guest is often perceived as a simple **captive portal**, but in reality it introduces **multiple architectural dependencies** that directly affect the security and behavior of a guest network.

Treating ISE Guest as “just a portal” leads to fragile designs, DNS leakage, and unintended trust exposure.

**What Cisco ISE Guest Actually Introduces:**

- **DNS dependencies**  
  Guest clients must resolve specific FQDNs for portal redirection, authentication, and post-authentication flows. Improper DNS design can expose internal zones or enable reconnaissance.

- **Certificate dependencies**  
  ISE Guest relies on valid TLS certificates. Misconfigured or internally issued certificates can force guest access to internal PKI infrastructure or weaken trust boundaries.

- **Redirect flows**  
  HTTP/HTTPS traffic is intercepted and redirected to ISE nodes. These flows require precise handling to avoid bypass, loops, or unintended access paths.

- **Firewall exceptions**  
  Guest traffic must be explicitly permitted to reach ISE services, DNS resolvers, and sometimes external identity providers. Overly broad exceptions increase the attack surface.

**Security Implications in Guest Networks:**

Each of these dependencies represents a **potential reconnaissance and abuse vector** if not tightly controlled:

- DNS exposure enables **TA0043 – Reconnaissance**
- Redirect logic failures enable access bypass
- Certificate trust misuse leaks internal infrastructure
- Firewall misconfigurations expand blast radius

**Guest-Focused Design Principle:**

> Cisco ISE Guest should be treated as a **security-integrated service**, not a standalone portal.

A secure guest deployment requires:
- Dedicated guest DNS infrastructure
- Publicly trusted certificates
- Minimal and explicit firewall rules
- Strong client isolation
- Continuous validation of redirect behavior

**Key Takeaway:**

In guest networks, **every dependency introduced by ISE must be intentionally designed and constrained**. Failure to do so turns the guest network into a reconnaissance platform rather than a controlled access environment.

---

## Common ISE Guest risks

- Portal FQDN resolves to internal IP
- Guest uses internal DNS
- Broad ACLs for “portal reachability”
- No validation of client isolation

Misconfigurations in **Cisco ISE Guest** deployments frequently undermine the security of guest networks. These risks often stem from treating the guest portal as a simple access feature rather than a tightly controlled security service.

**High-Risk Patterns:**

- **Portal FQDN resolves to an internal IP**  
  When the guest portal FQDN resolves to internal (RFC1918) addresses, guest clients gain visibility into internal addressing schemes. This enables passive reconnaissance and weakens network segmentation.

- **Guest network uses internal DNS**  
  Allowing guest clients to query internal DNS infrastructure exposes internal zones, naming conventions, and service records, significantly increasing reconnaissance capabilities.

- **Overly broad ACLs for “portal reachability”**  
  Wide or poorly scoped firewall and ACL rules created to “make the portal work” often permit unnecessary access to internal services, expanding the attack surface.

- **No validation of client isolation**  
  Assuming client isolation is enabled without verification can allow guest devices to communicate with each other, enabling lateral movement and peer reconnaissance.

**Security Impact:**

These misconfigurations directly support adversary activity related to:
- **TA0043 – Reconnaissance**
- **TA0007 – Discovery**
- **TA0008 – Lateral Movement**

Attackers can leverage these weaknesses to map internal infrastructure, identify exposed services, and prepare follow-on attacks from an untrusted network.

**Guest Network Security Principle:**

> If a configuration exists only to “make the portal work,” it must be reviewed for security impact.

**Mitigation Guidance:**

- Ensure portal FQDNs resolve only to **guest-accessible IPs**
- Use **dedicated guest DNS resolvers**
- Apply **minimal, explicit ACLs**
- Actively test and validate **client isolation behavior**
- Monitor DNS and portal access logs for abuse patterns

**Key Takeaway:**

Cisco ISE Guest deployments fail most often due to **assumptions**, not limitations. A secure guest network requires continuous validation of DNS, routing, isolation, and access controls.

---

## Secure ISE Guest principles

- Public certificate
- Dedicated Guest DNS
- Minimal allowlists
- Explicit Guest policies
- Logging and visibility

A secure **Cisco ISE Guest** deployment relies on clear trust boundaries and intentional design. These principles help ensure guest access remains functional while minimizing reconnaissance and attack opportunities.

**Core Security Principles:**

- **Publicly trusted certificates**  
  Use certificates issued by a public Certificate Authority. This avoids exposing internal PKI infrastructure and prevents trust dependencies between guest devices and internal systems.

- **Dedicated Guest DNS**  
  Guest networks must use DNS resolvers that are fully separate from internal DNS. Guest DNS should resolve only what is required for internet access and portal functionality.

- **Minimal allowlists**  
  Firewall rules and ACLs must be explicitly scoped to required destinations and ports. Avoid broad rules created for convenience or troubleshooting.

- **Explicit Guest policies**  
  Guest traffic should be governed by policies written specifically for untrusted users. Do not reuse internal or “default” access policies.

- **Logging and visibility**  
  DNS queries, portal access, authentication events, and policy decisions must be logged and monitored to detect abuse, reconnaissance, and misconfiguration.

**Security Alignment:**

When applied consistently, these principles reduce exposure related to:
- **TA0043 – Reconnaissance**
- **TA0007 – Discovery**
- **TA0008 – Lateral Movement**

They also support **defense-in-depth** by combining network controls, identity enforcement, and telemetry.

**Guest Network Design Mindset:**

> Guest access is a controlled exception, not an extension of the internal network.

Designing ISE Guest with this mindset ensures usability without sacrificing security or visibility.

