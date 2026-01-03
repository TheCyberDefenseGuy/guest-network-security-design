# Guest Network Security Design Approach

## Guest is a security project, not a checkbox

Guest access is often added late in projects, driven by:
- business convenience
- user experience
- compliance requirements

Security is usually added **after connectivity works**, creating fragile designs.

This document defines Guest as:
- A **security boundary**
- A **high-risk segment**
- A **permanent attack surface**

---

## Design objectives

A secure Guest network must:
1. Prevent lateral attacks between guests
2. Protect internal infrastructure and services
3. Limit reconnaissance opportunities
4. Contain blast radius during DoS events
5. Be auditable and testable

---

## Core architectural principles

- Least privilege networking
- Explicit allowlists
- Separation of trust domains
- Dedicated infrastructure where necessary
- Validation over assumptions
