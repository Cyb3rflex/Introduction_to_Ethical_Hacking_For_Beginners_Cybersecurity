# Introduction to Ethical Hacking

> Comprehensive, workshop-ready README covering scope, rules of engagement, legal/ethical constraints, reconnaissance basics, and scanning tools (passive vs active).

---

## Table of Contents

1. [Overview](#overview)
2. [Scope](#scope)
3. [Rules of Engagement (RoE)](#rules-of-engagement-roe)
4. [Legal & Ethical Constraints](#legal--ethical-constraints)
5. [Reconnaissance — Basics](#reconnaissance---basics)
6. [Passive Reconnaissance](#passive-reconnaissance)
7. [Active Reconnaissance](#active-reconnaissance)
8. [Common Scanning & Recon Tools](#common-scanning--recon-tools)
9. [Passive vs Active — Comparison](#passive-vs-active--comparison)
10. [Interpreting Scan Results — Triage & Prioritization](#interpreting-scan-results---triage--prioritization)
11. [Reporting & Deliverables](#reporting--deliverables)
12. [Safe Testing Checklist (Pre-test)](#safe-testing-checklist-pre-test)
13. [Incident / Outage Escalation (Example)](#incident--outage-escalation-example)
14. [Responsible Disclosure & Post-test](#responsible-disclosure--post-test)
15. [Learning Resources & Next Steps](#learning-resources--next-steps)
16. [Quick Reference Commands (Lab Use Only)](#quick-reference-commands-lab-use-only)
17. [Teaching Tips](#teaching-tips)
18. [Example Rules of Engagement Snippet](#example-rules-of-engagement-snippet)
19. [Final Notes](#final-notes)

---

## Overview

**Ethical hacking** (aka penetration testing) is the authorized practice of simulating attacks against systems, networks, or applications to discover vulnerabilities before malicious actors do. The objective is to improve an organization’s security posture by identifying weaknesses, demonstrating risk, and recommending remediation.

Key principles:

* Obtain **explicit authorization** before testing.
* Define and respect the **scope** and **Rules of Engagement (RoE)**.
* Prioritize **confidentiality**, **integrity**, and **availability**.
* Provide clear, actionable documentation and remediation guidance.

---

## Scope

Scope defines *what* is tested, *when*, *how*, and *who* is responsible. A robust scope should include:

* **Assets in-scope**: IP ranges, domains, subdomains, applications, APIs, cloud services.
* **Assets out-of-scope**: critical production systems (e.g., payment processing, databases), employee endpoints, third-party services without explicit permission.
* **Allowed test types**: passive recon, active scans, authenticated tests, social engineering (explicit if allowed).
* **Test windows**: dates/times and maintenance windows.
* **Data sensitivity rules**: handling of PII, credentials, and logs.
* **POC & escalation**: emergency contacts and escalation steps.
* **Deliverables & timelines**: interim updates, final report dates.

**Example minimal scope**

* In-scope: `203.0.113.0/24`, `app.example.com`, `api.example.com/auth`.
* Out-of-scope: production DB clusters with hostnames `db-*`.
* Allowed: network and web app scans, manual testing.
* Prohibited: social engineering, physical intrusion.

---

## Rules of Engagement (RoE)

RoE formalize how testing will be executed. Include:

* **Authorization**: signed scope & legal agreement (NDA + RoE).
* **POC**: 24/7 emergency contact (phone + email).
* **Safety rules**: no destructive actions unless explicitly authorized.
* **Timing**: start/end, maintenance windows.
* **Communication cadence**: immediate contact for critical findings.
* **Environment**: prefer staging for intrusive tests.
* **Data retention**: evidence storage, encryption, retention period, deletion procedures.
* **Legal compliance**: applicable local and international laws.

**Immediate escalation template**: stop testing and notify POC immediately if the test causes service outage.

---

## Legal & Ethical Constraints

### Legal

* **Authorization is mandatory**. Unauthorised scanning or attacks are illegal in most jurisdictions.
* Comply with local laws and international regulations (e.g., Computer Misuse Acts, GDPR).
* Be mindful of cross-jurisdictional tests — data leaving the country or touching third-party infrastructure may require additional approvals.

### Ethical

* **Do no harm**: avoid destructive techniques unless explicitly authorized and controlled.
* **Minimize exposure**: redact or avoid storing unnecessary PII.
* **Responsible disclosure**: coordinate fixes with stakeholders and vendors before public disclosure.
* **Transparency**: clearly document methods and evidence.

If unsure about legality/ethics, **pause testing** and contact legal or the POC.

---

## Reconnaissance — Basics

Reconnaissance (recon) is the information-gathering phase used to build an inventory of assets and identify the initial attack surface. Recon splits into **passive** (low/no footprint) and **active** (direct interaction) methods.

Goals:

* Inventory public-facing assets.
* Identify technologies and exposures.
* Prioritize targets for deeper testing.

---

## Passive Reconnaissance

**Definition:** Gathering information from public sources without directly touching the target systems.

**Benefits:** Low risk, unlikely to be detected.

**Techniques & sources**:

* **WHOIS**: domain registration and name servers.
* **DNS records**: A, AAAA, MX, TXT, SPF, SRV, CNAME, and subdomains.
* **Certificate Transparency**: `crt.sh` to find issued certs and subdomains.
* **Search engines & Google dorking**: find indexed sensitive files.
* **OSINT**: LinkedIn, GitHub (leaked keys), paste sites, social media.
* **Shodan / Censys**: searches for exposed services and banners.
* **Passive DNS services**: historical DNS mappings.

**Best practice**: Log the source and timestamp for each finding (e.g., `crt.sh result — 2025-10-19`).

---

## Active Reconnaissance

**Definition:** Direct interaction with targets to enumerate ports, services, versions, and behavior.

**Risks:** Detectable, may impact services. Requires explicit authorization.

**Common techniques**:

* **Port scanning** (TCP/UDP) to discover open ports.
* **Service/version detection** to identify software and versions.
* **OS detection** (fingerprinting).
* **Directory/file enumeration** on web servers.
* **Banner grabbing**.
* **Vulnerability scanning** (use cautiously in production).

**nmap examples** (only against authorized targets):

```bash
# Quick TCP top-ports scan + version detection
nmap -sS -sV --top-ports 1000 -oA scans/target_quick target.example.com

# Full TCP port scan with OS detection (slower/more intrusive)
nmap -p- -A -T4 -oA scans/target_full target.example.com
```

> Note: `-sS` is a SYN scan (stealthier); `-A` is aggressive and more intrusive. Confirm allowed flags in your RoE.

**UDP scanning example**:

```bash
nmap -sU -p 53,67,68,123,161 -oA scans/udp_common target.example.com
```

UDP scans are slower and often produce false positives.

---

## Common Scanning & Recon Tools

* `whois`, `dig`, `nslookup` — domain & DNS queries
* `theHarvester`, `Sublist3r`, `Amass` — subdomain discovery (passive + active)
* `crt.sh`, Certificate Transparency logs — find certs & subdomains
* `Shodan`, `Censys` — search exposed devices
* `nmap` — port & service discovery
* `masscan` — very fast scanning (extremely noisy)
* `Nikto`, `OWASP ZAP`, `Burp Suite` — web application scanning & proxying
* `ffuf`, `Dirb`, `Dirbuster` — directory/file discovery
* `Recon-ng`, `SpiderFoot` — OSINT aggregation frameworks
* `tcpdump`, `Wireshark` — packet capture and analysis (use on networks you control)
* `Nessus`, `OpenVAS` — vulnerability scanners (use with authorization)

---

## Passive vs Active — Comparison

* **Visibility**: Passive = low; Active = high (noisy).
* **Risk**: Passive = safe for production; Active = can disrupt services.
* **Data richness**: Passive = context & discovery; Active = detailed version and behavior data.
* **Recommended workflow**: Passive first, then active for prioritized targets (with permission).

---

## Interpreting Scan Results — Triage & Prioritization

1. **Confirm accuracy**: rescan and cross-check with other tools.
2. **Map attack surface**: exposed management interfaces, default credentials.
3. **Map to CVEs**: search vulnerability databases (NVD) for reported issues.
4. **Assess impact**: data sensitivity and potential for lateral movement.
5. **Prioritize**: critical (RCE, auth bypass), high (admin interfaces), medium (info disclosure), low (limited impact).
6. **Validate**: non-destructive PoC only if authorized.

---

## Reporting & Deliverables

A practical pentest report should include:

* **Executive summary** for non-technical stakeholders.
* **Scope & methodology** with dates and tools used.
* **Findings**: title, description, evidence, severity, reproducible steps (if allowed).
* **Risk & impact assessment** (CVSS mapping or internal scale).
* **Remediation & mitigation guidance** (actionable steps).
* **Appendix**: raw outputs, logs, and commands (redact PII).

**Example finding structure**:

* Title: Outdated Apache HTTP Server (CVE-XXXX-XXXX)
* Evidence: `nmap` banner and HTTP response headers (timestamped)
* Impact: Possible remote code execution; sensitive data exposure
* Recommendation: Upgrade to patched version X.Y.Z; harden configurations; WAF rules.

---

## Safe Testing Checklist (Pre-test)

* Obtained signed authorization & scope approval.
* POC contacts tested and reachable.
* Backup & rollback plan exists (if testing can affect production).
* Test schedule confirmed (outside peak if requested).
* Logging/monitoring teams notified (if agreed).
* Tools & environment prepared (VPN, jump hosts).
* Evidence collection and secure storage plan defined.

---

## Incident / Outage Escalation (Example)

1. Stop testing immediately.
2. Notify POC: include incident description, time, actions performed, and mitigation attempts.
3. Assist with rollback if authorized.
4. Document timeline, root cause (if known), and remediation steps taken.

---

## Responsible Disclosure & Post-test

* Share the full report securely with stakeholders.
* Agree on remediation timelines before public disclosure.
* Coordinate vendor/CERT disclosure if needed (follow coordinated disclosure timelines).
* Retest after fixes and update the report.

---

## Learning Resources & Next Steps

* Practice in safe labs: local VMs, intentionally vulnerable VMs, or CTF platforms.
* Tools to learn: `nmap`, `tcpdump`, `Wireshark`, `Burp Suite`, `Amass`, `ffuf`.
* Methodology references: OWASP Testing Guide, PTES.
* Build templates: RoE template, evidence collection checklist, standard report format.

---

## Quick Reference Commands (Lab Use Only)

```bash
# WHOIS
whois example.com

# Basic DNS
dig +short example.com

# Quick nmap (authorized targets only)
nmap -sS -sV --top-ports 1000 -oN quick_scan.txt target.example.com

# Full TCP scan (slower/more intrusive)
nmap -p- -A -T4 -oA full_scan target.example.com

# HTTP directory brute force with ffuf
ffuf -u https://target.example.com/FUZZ -w /path/to/wordlist.txt -o ffuf_results.json
```

---

## Teaching Tips

* Start with laws & authorization; set expectations.
* Demonstrate passive recon first (show public exposure).
* Use a lab VM for active demos (never run intrusive tests on production).
* Emphasize evidence collection and remediation.
* Provide templates (RoE, report) for students to reuse.

---

## Example Rules-of-Engagement Snippet

```
RULES OF ENGAGEMENT
- Authorized testers: Alice (alice@example.com), Bob (bob@example.com)
- In-scope: 203.0.113.10-20, app.staging.example.com
- Out-of-scope: production DBs, employee endpoints
- Test hours: 2025-10-20 09:00 — 2025-10-20 17:00 UTC
- Allowed tests: passive recon, authenticated scans, web app testing (no social engineering)
- Emergency POC: ops-team@example.com, +234-XXXXXXXX
- Escalation: Stop immediately if any outage > 5 minutes and notify POC.
```

---

## Final Notes

* Ethical hacking is a blend of technical skill, clear communication, and risk management.
* Preserve evidence and be precise in your reports.
* Prioritize remediation and responsible disclosure over "hacking for the win."
* When in doubt, stop and consult legal/POC.

---

*Prepared for workshop use — feel free to adapt the scope, RoE, and command examples to your environment.*
