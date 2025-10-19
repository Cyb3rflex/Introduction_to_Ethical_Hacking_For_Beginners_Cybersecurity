# Introduction_to_Ethical_Hacking_For_Beginners_Cybersecurity

# Ethical Hacking Workshop — Week Plan (Oct 28–31)

**Audience:** Students beginning ethical hacking / penetration testing (labs in isolated VMs).

**Prepared for:** Cyberflex

**Overview**
This repository contains the workshop plan, notes, lab guides, and templates for a 4-day ethical hacking workshop. Each day includes learning objectives, key concepts, tool commands, lab instructions, documentation templates, and assessment criteria.

---

## Table of Contents

* [Learning objectives](#learning-objectives)
* [Schedule (Oct 28–31)](#schedule-oct-28-31)
* [Prerequisites & safe environment setup](#prerequisites--safe-environment-setup)
* [Rules of Engagement & legal/ethical constraints](#rules-of-engagement--legalethical-constraints)
* [Reconnaissance: concepts (passive vs active)](#reconnaissance-concepts-passive-vs-active)
* [Day-by-day notes and tasks](#day-by-day-notes-and-tasks)

  * [Tuesday — Introduction & Recon Basics](#tuesday--introduction--recon-basics)
  * [Wednesday — Recon Tools Deep-Dive](#wednesday--recon-tools-deep-dive)
  * [Thursday — Hands-on Lab: Recon & Scanning](#thursday--hands-on-lab-recon--scanning)
  * [Friday — Guided Exploitation & Reporting](#friday--guided-exploitation--reporting)
* [Documentation templates & reporting format](#documentation-templates--reporting-format)
* [Recommended commands & quick reference](#recommended-commands--quick-reference)
* [Assessment rubric & deliverables](#assessment-rubric--deliverables)
* [Safety notes, remediation suggestions, and next steps](#safety-notes-remediation-suggestions-and-next-steps)

---

## Learning objectives

By the end of the workshop participants should be able to:

* Explain scope, rules of engagement, and legal/ethical constraints for penetration testing.
* Differentiate passive and active reconnaissance and choose the correct approach for a given situation.
* Use WHOIS, DNS enumeration, Google dorking, basic fingerprinting and scanning tools to build a target profile.
* Document findings clearly and produce a short technical report with remediation recommendations.
* Perform safe, controlled exploitation on lab VMs and explain the methodology used.

---

## Schedule (Oct 28–31)

* **Tuesday (Oct 28):** Introduction to Ethical Hacking, scope, rules of engagement, legal/ethical constraints. Reconnaissance basics and scanning tools (passive vs active recon).
* **Wednesday (Oct 29):** Recon tools deep-dive: WHOIS, DNS enumeration, Google dorking, basic fingerprinting — documenting findings and building target profiles.
* **Thursday (Oct 30):** Hands-on lab: reconnaissance and scanning on demo VM (students perform passive + active scans, capture notes/screenshots).
* **Friday (Oct 31):** Guided hacking exercise: safe exploitation demo on lab VM; students perform assigned tasks and submit mini reports summarizing methodology, findings, and remediation suggestions.

---

## Prerequisites & safe environment setup

**Hardware / Software:**

* Laptop with virtualization support (VT-x/AMD-V).
* VirtualBox, VMware Player, or equivalent.
* Kali Linux (or Parrot) VM for attacker tools.
* Demo target VM(s) — intentionally vulnerable images (e.g., Metasploitable, OWASP Juice Shop) *isolated from any production network*.

**Network isolation:**

* Use host-only network or an internal virtual network to ensure no traffic leaks to the internet or university network.
* Confirm NAT/bridged interfaces are disabled for target VMs.

**Accounts & access:**

* Each student/team should have a sandboxed VM snapshot checkpoint to revert changes.
* Provide SSH/RDP credentials for the lab VM if required.

**Safety checklist before scanning/exploitation:**

* Confirm explicit written permission to test the provided lab VMs.
* Ensure target VMs are on an isolated virtual network.
* Notify instructors of the start/end times.
* Take snapshots before exploitation.

---

## Rules of Engagement & legal/ethical constraints

**High-level rules** (must be followed by all participants):

1. **Scope defined:** Only test systems explicitly listed in the lab instructions. Do not test external devices or services.
2. **Authorization required:** No scanning or exploitation of systems outside the lab environment. Unauthorized testing is illegal and will be treated seriously.
3. **Non-disclosure:** Do not publicly publish exploit details or screenshots that contain credentials or sensitive information.
4. **Data handling:** Do not exfiltrate any real data. Use only the lab-supplied data.
5. **Report responsibly:** If you accidentally discover an unrelated vulnerability on the infrastructure used to run labs, report it immediately to instructors — do not exploit it.
6. **Safety first:** Stop immediately if an action risks damaging infrastructure outside the lab or other students' work.

**Legal / ethical constraints — short list:**

* Laws vary by jurisdiction. In general, computer misuse, unauthorized access, and data theft are criminal offenses.
* Even discovery without exploitation can be problematic if done outside an authorized scope.

**Consequences:** Breaking the rules may result in removal from the workshop and reporting to relevant authorities or institutions.

---

## Reconnaissance: concepts (passive vs active)

**Passive reconnaissance** — collecting information without interacting directly with the target’s systems. Examples:

* Searching public records and archives.
* WHOIS lookups, DNS registry data, public certificate transparency logs.
* Google dorking and social media research.

**Pros:** low risk of detection, legal-safety when using public data.
**Cons:** limited depth; may not uncover network-level details.

**Active reconnaissance** — direct interaction with the target to discover live services and open ports. Examples:

* Ping sweeps, port scans, banner grabbing, version detection using `nmap`.

**Pros:** more detail and actionable data.
**Cons:** higher chance of detection, potential to disrupt services if done irresponsibly.

**Best practice:** Start passively, escalate to active only with explicit permission and a clear plan. Document each step taken.

---

# Day-by-day notes and tasks

### Tuesday — Introduction & Recon Basics (Oct 28)

**Objectives:**

* Define scope, targets, and rules of engagement.
* Understand legal/ethical limits.
* Learn difference between passive vs active recon.
* Overview of common recon/scanning tools.

**Instructor talking points:**

* What is ethical hacking vs malicious hacking.
* Why rules of engagement exist — professional and legal reasons.
* Overview of recon workflow: intelligence gathering → enumeration → vulnerability discovery → exploitation (in lab-only).

**Quick demo topics:**

* WHOIS lookup example.
* Simple `dig`/`nslookup` to view DNS records.
* High-level `nmap` scan demo (show SAFE flags/options).

**Homework / prep for Wednesday:**

* Read the WHOIS and DNS sections below.
* Make sure your Kali VM is updated and the lab VM snapshot exists.

---

### Wednesday — Recon Tools Deep-Dive (Oct 29)

**Objectives:**

* Use WHOIS, DNS enumeration, Google dorking, and basic fingerprinting tools.
* Document findings and build a target profile.

**Tools covered & examples**

* **WHOIS** — whois registration records.

  ```bash
  # Query WHOIS for example.com
  whois example.com
  ```

  **What to extract:** registrar, registrant email (if public), creation/expiry dates, name servers.

* **DNS enumeration** — `dig`, `nslookup`, `host`, `dnsrecon`, `dnsenum`.

  ```bash
  # Get A record
  dig +short example.com A

  # Fetch MX records
  dig +short example.com MX

  # Zone transfer attempt (educational only)
  dig axfr @ns1.example.com example.com
  ```

  **Goal:** identify subdomains, mail servers, name servers, and potential zone misconfigurations.

* **Google dorking** — using targeted queries to find sensitive files or endpoints indexed by search engines.

  Examples (do *not* run these against real/unauthorized domains):

  ```text
  site:example.com "index of" password
  site:example.com ext:sql | ext:log | ext:env
  site:example.com "admin" intitle:login
  ```

  **Documentation tip:** capture the exact search string and the URL results.

* **Basic fingerprinting** — banner grabbing and service detection.

  ```bash
  # Simple banner grab with netcat
  nc -v example.com 80
  # Nmap service/version scan
  nmap -sV -p 1-1000 example.com
  ```

  **What to record:** service name, port, version string, HTTP headers, and any default pages.

**Target profile checklist** (what to include in your notes):

* Domain and subdomains discovered
* IP address ranges and ASNs (if public)
* Public-facing services and versions
* Exposed admin panels, mail servers, and SSO endpoints
* Any leakage found via Google dorking

---

### Thursday — Hands-on Lab: Recon & Scanning (Oct 30)

**Objectives:**

* Perform passive and active recon on the provided lab VM.
* Capture screenshots and notes for every step.
* Produce a short lab log for submission.

**Lab setup**

* Restore the lab VM snapshot provided at the start of the class.
* Ensure attacker VM (Kali) and target VM are on an **isolated internal network**.

**Task list (suggested order):**

1. Passive open-source intelligence (OSINT):

   * WHOIS and DNS queries against the lab domain (if applicable).
   * Google dorking on lab-hosted web pages (only within provided domain).
2. Passive non-intrusive checks:

   * Check public HTTP headers and robots.txt via `curl`.
   * Look for publicly accessible directories (e.g., `/uploads`, `/backup`).
3. Active enumeration:

   * Ping sweep (only inside lab network).
   * `nmap` TCP SYN scan (e.g., `nmap -sS`) on allowed ports — **limit the scope** to the lab host.
   * Service version detection with `nmap -sV`.
4. Lightweight vulnerability identification:

   * Check known CVEs for discovered versions (demonstrate how to search vulnerabilty DBs — offline copies if needed).

**Evidence capture**

* Save screenshots for: WHOIS output, `dig` output, `nmap` scans, browser views of pages, `curl` outputs.
* Save raw `nmap` XML output for instructor review: `nmap -sV -oX output.xml target_ip`

**Sample lab command sequence**

```bash
# WHOIS
whois lab-target.test > whois.txt

# DNS
dig lab-target.test ANY +noall +answer > dig.txt

# Simple curl header check
curl -I http://lab-target.test/ > curl_headers.txt

# Nmap scan (TCP SYN) and save XML
nmap -sS -p 1-65535 -T4 -oA nmap_full lab-target.test
```

**Deliverable for Thursday:**

* Lab log (markdown) with timestamps, commands used, notes, and screenshots attached.
* `nmap` XML/grepable output files.

---

### Friday — Guided Exploitation & Reporting (Oct 31)

**Objectives:**

* Instructor demonstrates a safe exploitation on the lab VM (pre-approved, low-impact exploit).
* Students perform assigned tasks and document methodology.
* Students submit mini-reports summarizing methodology, findings, and remediation suggestions.

**Instructor demo topics (example):**

* Exploiting a known vulnerable service with Metasploit (on the lab VM only).
* Demonstrate post-exploitation safety: only read designated test files, do not pivot to other VMs.

**Student tasks (example set):**

* Task A: Find and exploit a simple misconfigured web app file upload vulnerability to upload and view a file.
* Task B: Exploit a known CMS vulnerability to read a low-privilege file (demo-only payloads).
* Task C: Demonstrate credential discovery from an exposed `config.php` or `.env` file and propose remediation.

**Reporting requirements (each student/team):**

* Short methodology: step-by-step commands used and why.
* Evidence: screenshots, `nmap` output, relevant tool outputs.
* Findings: what was vulnerable and why it matters.
* Remediation: prioritized, actionable fixes.
* Risk rating: Low / Medium / High with justification.

**Submission format:**

* `mini-report.md` in the repository under `/submissions/<team-name>/mini-report.md` with attachments in the same folder.

---

## Documentation templates & reporting format

**mini-report.md template**

```markdown
# Mini Report — <Team / Student Name>

## Summary
Short summary of what you tested and the main findings.

## Scope
List target IPs/hostnames and confirmation that testing was limited to lab environment.

## Methodology (step-by-step)
- Command / action
- Output (attach screenshot or paste output)

## Findings
1. Vulnerability title
   - Description
   - Evidence (screenshots, logs)
   - CVSS (if known) / Risk (Low/Med/High)

## Remediation Recommendations
- Actionable steps to fix the issue (e.g., update package X to version Y, remove default credentials)

## Appendix
- Raw tool output files (nmap.xml, whois.txt)
```

**Logging best practices**

* Timestamp every command/result.
* Use short descriptive filenames for screenshots: `teamname_step_n_command.png`.
* Keep raw outputs in `/submissions/<team>/raw/` and processed report in `/submissions/<team>/report/`.

---

## Recommended commands & quick reference

> Always run commands against the lab systems only. Use `--help` to learn about flags.

**WHOIS**

```bash
whois example.com
```

**DNS**

```bash
dig example.com ANY
dig @ns1.example.com example.com AXFR   # educational only
```

**Nmap**

```bash
# Fast port scan (top 1000 ports)
nmap -sS -T4 -p- --open -oA quick_scan lab-target

# Service/version detection and scripts
nmap -sV -sC -oA svc_scan lab-target

# Save XML
nmap -sV -oX svc_scan.xml lab-target
```

**Netcat / Banner grabbing**

```bash
nc -v target_ip 80
```

**Curl**

```bash
curl -I http://lab-target/
curl -L http://lab-target/robots.txt
```

**Google dorking guidance**

* Save the exact search query and the result URLs.
* Do not run broad dorking against live external sites — keep it limited to the lab domain.

---

## Assessment rubric & deliverables

**Deliverables**

* `mini-report.md` with evidence in `/submissions/<team>/`.
* Raw outputs (nmap.xml, whois.txt, screenshots).

**Grading rubric (example)**

* Completeness of recon: 30%
* Quality of evidence & documentation (timestamps, screenshots): 30%
* Correctness of methodology and safety practices: 20%
* Quality and prioritization of remediation suggestions: 20%

**Late policy:**

* Submissions accepted up to 24 hours after session end with a 10% penalty. After 48 hours, submissions receive zero credit (adjust as you see fit).

---

## Safety notes, remediation suggestions, and next steps

**Safety recap:**

* Never test systems outside the authorized lab scope.
* Document and report accidental findings to instructors immediately.

**Common remediation suggestions (examples for reports):**

* **Update & patching:** Keep services updated; apply vendor patches.
* **Remove default credentials:** Replace default accounts and disable unused default services.
* **Harden configs:** Disable outdated protocols, restrict access by IP, implement rate-limiting.
* **Input validation & output encoding:** Fix file upload validation, sanitize user input, validate file types.
* **Least privilege:** Run services using dedicated low-privilege accounts.

**Suggested next steps for students:**

* Study vulnerability databases (NVD, CVE) and practice mapping version strings to CVEs.
* Learn web application security (OWASP Top 10) and network exploitation basics.

---

## Acknowledgements & references

* Workshop prepared by Cyberflex.
* Tools referenced: `whois`, `dig`, `nmap`, `curl`, `nc`, `theharvester`, `dnsrecon`.
