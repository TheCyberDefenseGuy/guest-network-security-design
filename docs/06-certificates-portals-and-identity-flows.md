# Certificates, Portals, and Identity Flows
## Trust chains, captive portals, and how onboarding becomes an attack surface

## Why this matters
Guest onboarding (captive portal, web auth redirects, sponsor/self-registration, AUP) is the moment when:
- users are asked to trust *something* on an untrusted network
- the network deliberately manipulates connectivity (walled garden)
- exceptions are created (DNS, portal reachability, PKI validation)

This combination makes onboarding one of the **highest-risk surfaces** in a Guest design.

---

## 1) Captive portals and “captivity” (standards baseline)

### 1.1 Captive portals are not a standard security control
Captive portals are primarily a **policy/onboarding mechanism** (AUP, minimal accountability, access gating),
not strong authentication by themselves.

Modern standards acknowledge captive portals as a common network pattern and provide mechanisms for clients
to understand and interact with captivity more safely:

- **RFC 7710** defines a DHCP option / RA extension to signal that a client is behind a captive portal and provide the portal URI.
- **RFC 8908** defines a Captive Portal API so clients can discover captive portal status and portal information in a consistent way.

**Security implication:** captive portals change normal expectations of connectivity. If this is not designed carefully,
you create phishing-friendly conditions and misconfiguration-driven leaks.

---

## 2) Portal models and redirect flows (what happens on the wire)

Guest onboarding typically uses one of these patterns:

### 2.1 Layer-3 WebAuth / Central WebAuth (CWA-like)
- Client joins SSID, gets IP (DHCP)
- Network allows limited access (walled garden)
- HTTP/HTTPS traffic triggers a redirect to a portal
- User authenticates / accepts AUP / obtains credentials

Cisco ISE guest deployments frequently use web redirection and authorization profiles to steer guests to a portal,
and the prescriptive guide explicitly covers redirect ACLs and guest flow validation and troubleshooting.

### 2.2 “External” portal flows (ISE-hosted portal vs external web tier)
Portals can be hosted on:
- the NAC platform (e.g., ISE portal nodes)
- an external web tier (reverse proxy / DMZ)
- a VIP/load balancer

**Security trade-off:**  
Externalizing the portal can simplify “guest-safe” exposure and WAF protection, but introduces more moving parts
and more certificate/DNS dependencies.

---

## 3) Certificate trust: why public CA is the default for Guest

### 3.1 Unmanaged endpoints cannot be assumed to trust corporate PKI
Guest devices (and many BYOD endpoints) won’t trust:
- internal CA roots
- enterprise MDM-installed trust anchors
- custom certificate chains

So most robust guest portals use a **publicly trusted certificate** to avoid:
- browser warnings
- “click through” normalization
- portal phishing enablement

Cisco’s guest portal configuration and troubleshooting references highlight portal certificate best practices and
common issues around certificate behavior in guest flows.

### 3.2 Certificate and identity matching: don’t guess the rules
TLS server identity verification is standardized. The baseline logic for verifying that a hostname matches a certificate is described in:
- **RFC 6125** (historical, widely referenced)
- **RFC 9525** (newer, obsoletes RFC 6125)

**Design requirement:**
- Your portal FQDN must match certificate SAN entries (CN-only assumptions are not enough in modern clients).
- Avoid brittle wildcard usage unless you fully control subdomain behavior.

### 3.3 Certificate format and revocation plumbing
X.509 certificate profile and CRL semantics used on the Internet are defined in:
- **RFC 5280**

**Guest impact:** Some clients will attempt revocation checks (OCSP/CRL).
If your walled garden blocks OCSP/CRL endpoints, you may see:
- slow portal load
- TLS “soft fail” behavior
- inconsistent user experience

**Defensive recommendation:** explicitly decide whether Guest is allowed to reach:
- OCSP responders / CRL distribution points for the portal certificate chain  
and document this in your allowlist policy (do not “open the internet” broadly to fix PKI side effects).

---

## 4) Portal FQDN, DNS, and “topology leakage”

### 4.1 Portal reachability depends on correct DNS behavior
A Guest portal must be reachable via a name that:
- resolves from the Guest network
- matches the certificate identity
- does not require broad access to internal DNS

Cisco guest deployment references emphasize DNS issues as a frequent root cause and cover sponsor portal FQDN and portal setup patterns.

### 4.2 The dangerous pattern: public portal FQDN → internal IP
A recurring design mistake:
- `guest.example.com` resolves to an **internal IP**
- to make it work, teams punch holes in firewalls or expose internal DNS
- the “temporary exception” becomes permanent

**Why this is dangerous**
- leaks internal addressing and routing structure
- creates privileged paths from an untrusted zone
- increases blast radius during exploitation and DoS
- complicates troubleshooting and change management

**Preferred pattern**
- `guest.example.com` resolves (for Guest clients) to a **guest-safe endpoint**:
  - DMZ VIP / reverse proxy / anchor path
  - or a dedicated portal exposure tier
- internal mappings can differ (split-horizon), but guests should not gain the ability to query internal zones broadly

### 4.3 Captive portal discovery standards reduce “DNS lying”
RFC 7710’s option is designed in part to avoid relying on “fake DNS interception” patterns to steer users to portals.
If your environment supports modern captive portal signaling, it can reduce some redirect ambiguity and improve client UX.

---

## 5) Identity flows for Guest: what you are really enforcing

Guest “identity” is often about **policy** and **accountability** more than strong authentication.

Cisco ISE supports multiple guest portal modes (self-registered, sponsored, hotspot-style), with documentation describing guest creation and portal usage flows.

### 5.1 Common Guest identity models (and security posture)
1. **Hotspot / click-through**
   - lowest friction, weakest accountability
   - use when you primarily need AUP acceptance and timeboxing

2. **Self-registered Guest**
   - user creates account; may require sponsor approval
   - better audit trail; still weak proofing by default
   - can be targeted by automation/abuse

3. **Sponsored Guest**
   - a sponsor issues credentials
   - increases accountability and time scoping
   - operationally heavier

4. **Employee-backed / federated onboarding**
   - guest uses an external IdP or employee approval workflows
   - can improve assurance, but increases complexity and privacy considerations

**Open standard perspective:**  
Digital identity assurance is a risk-based choice. NIST SP 800-63 provides models for identity proofing, authentication, and federation decisions.
(You generally do not want “high assurance identity proofing” for a coffee-shop-like guest experience, but you do want the right balance for enterprise contexts.)

---

## 6) Portal security hardening (web security on a hostile network)

A captive portal is a web application exposed to untrusted clients. Treat it like one.

### 6.1 Minimum web hardening baseline
- HTTPS only (no HTTP login endpoints)
- No mixed content
- Minimal data collection (privacy)
- Rate-limit registration and login endpoints
- Protect against automation (captcha / throttling / abuse detection)
- Strong session management
- Least privilege: portal should not be a bridge into internal resources

### 6.2 HSTS: useful, but apply carefully
HTTP Strict Transport Security is standardized in **RFC 6797**.

**Guest nuance:**  
HSTS is generally beneficial to prevent downgrade attacks on hostile networks, but:
- You must ensure all relevant hostnames/subdomains truly support HTTPS if you use includeSubDomains.
- Avoid turning operational mistakes into hard failures.

---

## 7) Attack chronology: how portals/certs get abused (and how to defend)

### Phase A — User is forced into captive flow
**Attacker goal:** exploit confusion and expectations.
- Rogue portals / lookalike portal pages
- Social engineering during captive experience

**Defense**
- Public CA certificate
- Clean portal hostname (no internal naming)
- Minimize redirects and exposed parameters

### Phase B — Name resolution and trust establishment
**Attacker goal:** manipulate what the user sees as “the portal”.
- DNS abuse if DNS is not enforced
- Captive portal phishing if users are habituated to cert warnings

**Defense**
- Enforce DNS egress to your Guest resolvers
- Never require users to accept TLS warnings
- Validate FQDN ↔ cert SAN alignment using standardized rules (RFC 6125/9525).

### Phase C — Portal web exploitation or automation abuse
**Attacker goal:** abuse registration, brute force, or exploit portal web tier.
**Defense**
- WAF/reverse proxy if needed
- Rate limits and abuse detection
- Minimal privileges and strong logging

### Phase D — Exception sprawl
**Attacker goal:** exploit “allow portal access” exceptions to reach internal services.
**Defense**
- strict allowlist model:
  - portal VIP
  - guest DNS
  - optional OCSP/CRL endpoints for portal chain
- periodic review of all portal-related firewall rules (treat as sensitive)

---

## 8) Acceptance criteria (what “good” looks like)

### Portal trust
- Portal loads without warnings on unmanaged devices
- Certificate chain is valid and matches portal FQDN (SAN alignment)
- Redirect chain reveals no internal hostnames or RFC1918 addresses

### DNS + FQDN
- Guests can resolve portal FQDN via Guest resolvers
- Guests cannot resolve internal-only zones
- Portal FQDN does not map to internal IPs in the guest view

### Identity flow safety
- Registration/login endpoints are abuse-resistant (rate limits)
- Guest accounts are timeboxed and auditable
- Sponsor/self-registration policies align to required assurance (NIST risk-based thinking).

---

## 9) Where this connects in the repo
- DNS design and exposure controls: `docs/05-dns-design-split-horizon-and-exposure.md`
- Cisco ISE and wireless deployment considerations: `docs/07-cisco-ise-and-wireless-design-considerations.md`
- Hardening controls: `docs/08-defensive-controls-and-hardening.md`
- Validation tests to prove this works: `docs/09-validation-and-security-assurance.md`

---

## References (key)
- Cisco ISE Guest Access Prescriptive Deployment Guide
- Cisco ISE Self-Registered Guest Portal configuration/troubleshooting  
- Cisco ISE Admin Guide sample: Guest access overview 
- NIST SP 800-63 Digital Identity Guidelines
- RFC 7710 Captive-Portal DHCP/RA option 
- RFC 8908 Captive Portal API
- RFC 5280 X.509 PKI profile
- RFC 6125 / RFC 9525 TLS service identity verification 
- RFC 6797 HTTP Strict Transport Security (HSTS)
