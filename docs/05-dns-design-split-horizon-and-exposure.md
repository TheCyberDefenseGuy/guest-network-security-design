# DNS Design – The Most Common Guest Failure

## DNS is reconnaissance

### DNS leaks:
- Internal naming
- Service structure
- Identity systems (SRV records)

In **guest networks**, DNS is one of the **primary reconnaissance mechanisms** available to attackers. Because guest environments are designed for open access and minimal trust, DNS often becomes the most reliable source of intelligence about the network and its surroundings.

### Guest clients typically have:
- No authentication or weak identity enforcement
- Limited network access controls
- Broad internet connectivity
- Reduced inspection compared to corporate networks

This makes DNS an ideal, low-friction channel for reconnaissance.

### Why DNS Is Especially Valuable in Guest Networks:

- **Allowed by design**  
  DNS must function for guests to access the internet, making it difficult to block without breaking usability.

- **Low visibility and low noise**  
  DNS queries blend into normal traffic and rarely trigger security alerts in guest environments.

- **Information-rich responses**  
  DNS reveals domain structures, cloud services, email platforms, CDNs, and third-party integrations.

- **Recon without direct interaction**  
  Attackers can enumerate assets without scanning IP ranges or touching internal systems.

### Common Guest-Focused DNS Reconnaissance Activities:

- Subdomain brute-forcing of public domains
- Discovery of externally exposed applications
- Identification of cloud providers and SaaS services
- Enumeration of email and authentication infrastructure (MX, TXT, SPF, DKIM)
- Detection of misconfigurations such as split-horizon DNS leakage

### Relation to MITRE ATT&CK:

DNS-based reconnaissance in guest networks maps primarily to:
- **TA0043 – Reconnaissance**
- **TA0007 – Discovery** (when internal visibility exists due to misconfiguration)

This intelligence is often used to support:
- **TA0001 – Initial Access**
- **TA0008 – Lateral Movement** (when isolation controls fail)

Security Perspective:

In guest networks, DNS should be treated as a **high-risk but unavoidable service**. Without proper controls, it enables attackers to:
- Map the organization’s digital footprint
- Identify attack surfaces without detection
- Pivot toward phishing, credential abuse, or exposed services

Effective guest network security requires **governing DNS usage**, not simply allowing it by default.

### Defensive Considerations:

In **guest networks**, DNS reconnaissance is especially relevant because clients are typically **untrusted** and may include compromised or malicious devices. Defensive controls should prioritize **visibility, containment, and abuse prevention** rather than trust-based access.

### Key defensive considerations for guest environments include:

- **Restrict DNS resolution paths**  
  Force guest clients to use **approved DNS resolvers only** (e.g., internal recursive resolvers or secure public DNS). Block direct DNS queries to arbitrary external resolvers.

- **Monitor for enumeration behavior**  
  Detect high-volume or patterned DNS queries that indicate subdomain brute-forcing, service discovery, or automated scanning originating from guest clients.

- **Prevent DNS zone transfers**  
  Ensure authoritative DNS servers do not allow AXFR requests from guest or external networks.

- **Apply Client Isolation**  
  Enforce client-to-client isolation to prevent guest devices from using DNS responses to identify and laterally probe other connected clients.

- **Limit DNS response exposure**  
  Avoid exposing internal naming conventions, split-horizon misconfigurations, or excessive DNS record details to guest networks.

- **Rate-limit and log DNS queries**  
  Apply query rate limits per client and maintain DNS logs as a primary telemetry source for detecting reconnaissance activity.

- **Integrate DNS security controls**  
  Use DNS filtering, reputation-based blocking, and threat intelligence to prevent known reconnaissance and command-and-control domains from being resolved.

Security Perspective:
In guest networks, DNS should be treated as a **controlled service, not an open utility**. Strong DNS governance reduces reconnaissance opportunities and supports mitigation of:
- **TA0043 – Reconnaissance**
- **TA0007 – Discovery**
- **TA0008 – Lateral Movement**

DNS visibility combined with **Client Isolation** and **egress controls** forms a strong defensive baseline for untrusted wireless environments.

---

## Relevant standards

- RFC 1034 / 1035 – DNS architecture
- RFC 8499 – DNS terminology
- NIST SP 800-81 – Secure DNS Deployment

---

## Anti-pattern: shared internal DNS

Using the same DNS for Guest and internal:
- Breaks trust separation
- Increases blast radius
- Enables enumeration attacks

Using the **same internal DNS infrastructure for both Guest and internal networks** is a common but high-risk anti-pattern. While it may simplify operations, it fundamentally breaks security boundaries and increases exposure.

### Why This Is a Problem:

- **Breaks trust separation**  
  Guest networks are inherently untrusted. Allowing guest clients to query internal DNS violates the trust model and exposes internal services to unknown devices.

- **Increases blast radius**  
  Any abuse, misconfiguration, or compromise originating from the guest network directly impacts internal infrastructure and DNS availability.

- **Enables enumeration attacks**  
  Internal DNS often contains rich information such as hostnames, service records, and naming conventions. Guest access enables attackers to enumerate internal systems without scanning the network.

### Security Impact:

This anti-pattern significantly strengthens adversary capabilities related to:
- **TA0043 – Reconnaissance**
- **TA0007 – Discovery**
- **TA0008 – Lateral Movement** (when combined with weak isolation)

Attackers can use DNS responses to:
- Identify internal systems and roles
- Discover legacy or poorly secured services
- Plan targeted phishing or credential abuse
- Prepare follow-on attacks from outside the network

### Recommended Approach:

- Use **dedicated DNS resolvers** for Guest networks
- Restrict DNS responses to **internet-only resolution**
- Prevent exposure of internal zones and records
- Apply strict logging and rate limiting
- Combine with **Client Isolation and egress controls**

### Key Takeaway:

DNS is not just a utility service. In guest networks, it is a **powerful reconnaissance channel**. Sharing internal DNS with untrusted clients creates unnecessary risk and undermines defense-in-depth principles.

---

## Split-horizon DNS

Split-horizon is valid **only when used minimally**.

Correct use:
- Portal FQDN only
- No internal zone exposure
- Guest resolvers separate from internal DNS

In **guest networks**, split-horizon DNS is acceptable **only as a minimal exception** to support required services such as captive portals. Any broader use introduces unnecessary risk and directly enables reconnaissance.

### When Split-Horizon DNS Is Acceptable:

Split-horizon DNS should be used **only** when strictly required for guest access functionality:

- **Portal FQDN only**  
  Expose a single DNS record required for captive portal redirection or authentication.

- **No internal zone visibility**  
  Internal domains, subdomains, hostnames, or service records must never be resolvable from the guest network.

- **Dedicated guest DNS resolvers**  
  Guest networks must use DNS resolvers that are fully separated from internal DNS infrastructure.

### What to Avoid:

In guest environments, split-horizon DNS must **not** be used to:
- Provide access to internal applications
- Reuse internal DNS zones
- Expose naming conventions or service metadata
- Simplify operations at the cost of trust boundaries

### Security Impact:

Misconfigured split-horizon DNS in guest networks enables:
- Passive DNS reconnaissance
- Enumeration of internal services
- Targeting of exposed applications
- Support for follow-on attacks

These risks directly align with:
- **TA0043 – Reconnaissance**
- **TA0007 – Discovery**

### Key Rule for Guest Networks:

> Guest DNS should answer only what is absolutely required, nothing more.

Treat split-horizon DNS as a **temporary, tightly scoped exception**, not a standard design pattern, in untrusted wireless environments.

