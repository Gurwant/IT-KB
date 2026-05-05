# DMARC 
#### Message Authentication Reporting & Conformance

It serves two primary functions: it tells receiving email servers how to handle messages that fail authentication, and it provides domain owners with reports on who is sending mail on their behalf.


## 1. DMARC Configuration

DMARC is configured by publishing a DNS TXT record under the `_dmarc` subdomain (e.g., `_dmarc.example.com`). The record consists of several `tag=value` pairs:

### Required Settings
*   **`v` (Version):** Must be exactly `v=DMARC1`, no other version for now.
*   **`p` (Policy):** Dictates the action to take on failing emails:
    *   `none`: Do nothing (used for monitoring).
    *   `quarantine`: Quarantine failing emails, most of the time sent to the spam folder.
    *   `reject`: Block failing emails entirely.

### Optional Settings
*   **`sp` (Subdomain Policy):** Sets a distinct policy for subdomains.
*   **`rua` & `ruf` (Reporting):** Specifies email addresses (e.g., `mailto:dmarc@example.com`) to receive daily aggregate reports (`rua`) or real-time forensic failure reports (`ruf`).
*   **`adkim` & `aspf` (Alignment strictness):** Defines if the DKIM (`adkim`) and SPF (`aspf`) domains must match the `From` domain exactly (`s` for strict) or if subdomains are allowed (`r` for relaxed).
*   **`pct` (Percentage):** Specifies the percentage of failing emails (0-100) the policy should apply to, allowing for a phased rollout.
*   **`fo` (Failure Options):** Determines which exact failures trigger a forensic report (e.g., if *all* mechanisms fail, or if *any* mechanism fails).

## 2. How DMARC Validates an Email

DMARC does not authenticate emails alone; it acts as a unifier for **SPF** (Sender Policy Framework) and **DKIM** (DomainKeys Identified Mail) by enforcing **Identifier Alignment**. This ensures the technical domains used by SPF and DKIM actually match the visible `From` domain the user sees.

When a server receives an email, DMARC validation works like this:

1.  **Extract the Domain:** The receiver looks at the visible `From` header.
2.  **Check DKIM:** The receiver verifies the cryptographic signature. For DMARC to pass here, the domain in the DKIM signature (`d=` tag) must align with the visible `From` domain.
3.  **Check SPF:** The receiver verifies if the sending IP is authorized. For DMARC to pass here, the hidden Return-Path domain (used by SPF) must align with the visible `From` domain.
4.  **The Verdict:** A message **passes DMARC** if it achieves an aligned pass for **either** SPF or DKIM. 
5.  **Enforcement:** If **both** fail or are misaligned, the message fails DMARC, and the receiver applies the domain owner's policy (`none`, `quarantine`, or `reject`).
---
**Example**:
Here is the DMARC setup for the *linkedin.com* domain :
```
~$ dig +short TXT _dmarc.linkedin.com
"v=DMARC1; p=reject; rua=mailto:d@rua.agari.com,mailto:yfy3q-9359@rua.dmarc.emailanalyst.com; ruf=mailto:d@ruf.agari.com"
```

1. Here is our `From` domain in the email we are trying to validate : <img width="382" height="24" alt="image" src="https://github.com/user-attachments/assets/23d7b9fe-0e3f-4b94-8965-c4c641905b50" />
2. We already did the DKIM validation, but we can confirm that the domain used in the DKIM-signature does match : <img width="702" height="70" alt="image" src="https://github.com/user-attachments/assets/88e917c5-18b9-47e7-9385-627f7ae25d84" />
3. The return path also does match as a subdomain because *relaxed* is the default mode in the DMARC config : <img width="715" height="38" alt="image" src="https://github.com/user-attachments/assets/eedfe01c-b943-4692-9909-7c829a6d989e" />

This email does pass the DMARC check and was sent to the recipient.

---
**Sources**:
- [RFC7489](https://datatracker.ietf.org/doc/html/rfc7489)
- [Fortinet](https://www.fortinet.com/fr/resources/cyberglossary/dmarc)
