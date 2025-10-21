# RECON TOOLS — Deep Dive

> Ready-to-commit reference for reconnaissance tooling, safe pipelines, and evidence handling.

---

## 1. Purpose & Workflow

Reconnaissance is the disciplined process of discovering and enumerating an engagement’s attack surface. Typical workflow:

1. Define scope & Rules of Engagement (RoE).
2. Passive discovery (OSINT, certs, public records).
3. Active discovery (DNS, port scans, service detection).
4. Service & application enumeration (web routes, APIs).
5. Correlation & enrichment (combine results, map relationships).
6. Prioritization and evidence collection.

Always start passive → then active for prioritized targets (with permission).

---

## 2. Tool Categories

* DNS & domain intelligence (`whois`, `dig`)
* Subdomain discovery (`amass`, `subfinder`, `assetfinder`)
* Certificate transparency (`crt.sh`, `certstream`)
* Host discovery & port scanning (`nmap`, `masscan`, `naabu`)
* Service/version fingerprinting (`nmap -sV`, `httpx`, `whatweb`)
* Web discovery / directories (`ffuf`, `gobuster`, `dirsearch`)
* URL aggregation & screenshotting (`gau`, `waybackurls`, `aquatone`)
* OSINT frameworks (`theHarvester`, `Recon-ng`, `SpiderFoot`)
* Internet search engines (`Shodan`, `Censys`)
* Packet capture (`tcpdump`, `Wireshark`)
* Automation / orchestration (bash/python pipelines, recon frameworks)

---

## 3. DNS & Domain Intelligence

```bash
# WHOIS
whois example.com

# DNS records
dig example.com A +short
dig example.com MX +short
```

Capture registrar, name servers, and contact emails. WHOIS data may be redacted.

---

## 4. Subdomain Discovery

Use multiple tools and sources to reduce false negatives.

```bash
# Passive amass
amass enum -passive -d example.com -o amass_passive.txt

# Active amass (brute/enum)
amass enum -d example.com -o amass_all.txt

# Simple pipeline: subfinder -> httprobe
subfinder -d example.com -silent > subs.txt
cat subs.txt | httprobe -c 50 > live.txt
```

Notes: passive is safe; active/brute-force is noisy — check RoE.

---

## 5. Certificate Transparency

Check `crt.sh` or CT logs to discover issued certificates and subdomains.

---

## 6. Shodan & Censys

Query for exposed services and banners (admin panels, outdated stacks).

Operational note: queries are external and may be logged.

---

## 7. Host Discovery & Port Scanning

```bash
# masscan (lab only — very noisy)
masscan -p1-65535 --rate=1000 203.0.113.0/24 -oG masscan.out

# nmap (safer / richer)
nmap -sS -sV -T4 -p- -oA nmap_full 203.0.113.15

# naabu (fast TCP scanner)
naabu -host target.example.com -o naabu_out.txt
```

Tuning: lower `--rate`/`-T` for stealth; `-sS` for SYN scan; UDP `-sU` is slow.

---

## 8. Service & Version Fingerprinting

```bash
nmap -sS -sV --script=banner 203.0.113.15 -oN svc_detect.txt
cat live.txt | httpx -title -status -tech-detect -o httpx_results.txt
whatweb https://sub.example.com
```

Correlate banners with CVEs (do not exploit in production).

---

## 9. Web Discovery / Content Enumeration

```bash
# directory fuzzing with ffuf
ffuf -u https://target.example.com/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -c -o ffuf.json

# gobuster
gobuster dir -u https://target.example.com -w /path/wordlist.txt -t 50 -o gobuster.txt
```

Start with smaller wordlists and escalate. Respect RoE and concurrency limits.

---

## 10. URL Aggregation & Screenshotting

```bash
# historical URLs
echo example.com | gau > gau_urls.txt

# probe live hosts
cat subdomains.txt | httprobe -c 50 > live_http.txt

# screenshots with aquatone
cat live_http.txt | aquatone -out aquatone_output
```

Useful for quick triage and visualization.

---

## 11. OSINT Frameworks

```bash
theHarvester -d example.com -b all -l 500 -f theHarvester_out.html
```

Use frameworks to aggregate emails, hosts, and names from many sources.

---

## 12. Packet Capture

```bash
# capture on interface eth0
sudo tcpdump -i eth0 -w capture.pcap

# tshark read
tshark -r capture.pcap -Y 'http.request' -T fields -e http.host -e http.request.uri
```

Capture only on networks you control or have authorization to monitor.

---

## 13. Automation & Pipelines

Example pipeline (subdomain -> probe -> fingerprint):

```bash
subfinder -d example.com -silent > subs.txt
cat subs.txt | httprobe -c 50 > live_http.txt
cat live_http.txt | httpx -status -title -tech-detect -o httpx_out.txt
# selective dir fuzzing
ffuf -u https://admin.example.com/FUZZ -w /path/wordlist.txt -t 30 -o admin_fuzz.json
```

Use `jq`, `grep`, `sort -u` to normalize outputs. Timestamp files for evidence.

---

## 14. Evidence Collection & Management

* Timestamp outputs: `date -u +"%Y-%m-%dT%H:%M:%SZ"`.
* Store raw outputs securely (nmap XML, ffuf JSON).
* Redact PII before sharing.

Example structure:

```
/evidence/
  2025-10-21_target-example_com/
    nmap_full_2025-10-21T10-05-00.xml
    subfinder_2025-10-21.txt
    httpx_2025-10-21.json
```

---

## 15. OPSEC & Operational Considerations

* Use dedicated testing infrastructure and avoid personal IP exposure.
* Rotate API keys used for OSINT tools.
* Be explicit in RoE about noisy tools (masscan, fuzzing).
* Notify SOC/ops if agreed.

---

## 16. Prioritization & Enrichment

Map findings to: `asset`, `exposure`, `confidence`, `impact`. Enrich with CVE data (without exploiting).

---

## 17. CI/CD & Integrations

* Schedule passive discovery (amass/subfinder) in CI to detect new assets.
* Run non-intrusive scans on staging only.
* Alert on new certs, subdomains, or exposed services.

---

## 18. Lab-safe Examples (Non-destructive)

```bash
amass enum -passive -d example.com -o amass_passive.txt
nmap -sS -sV -p22,80,443 -oN nmap_minimal.txt 203.0.113.15
cat subs.txt | httprobe -c 20 | httpx -title -status -o httpx_safe.txt
```

---

## 19. Common Pitfalls

* False positives from UDP or fuzzing.
* Stale OSINT data.
* Over-reliance on a single tool.
* Scanning without RoE — never do it.

---

## 20. Next Steps

* Master one tool per category (amass, nmap, httpx, ffuf, tcpdump).
* Build modular, repeatable pipelines.
* Practice on DVWA, Juice Shop, and VulnHub VMs.
* Create a recon playbook with commands, wordlists, and safe defaults
