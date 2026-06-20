# IR-2026-002 — CloutHaus Incident Investigation Report
 
> **Spearphishing, Credential Harvesting, and Data Exfiltration at CloutHaus**
>
> Analyst: Muhammad Essam · Handle: 0xM4R7YR · Severity: HIGH · Status: CONTAINED
 
---
## Investigation Report Documentation
 
I did an Investigation Report for this specific case as part of my learning journey, you can find it on my GitHub in PDF format.
 
<img width="1760" height="990" alt="Copy of All rights reserved to 0xM4R7YR (1)" src="https://github.com/user-attachments/assets/fcfe14f8-7d67-45d7-b903-1a2f87c4cbeb" />

 
## ⚠️ Before Reading Please Read
 For a Q&A styled investigation walkthrough, please feel free to check my **Medium Writeup** where I solve the same investigation — Q&A Style.
 
---
 
## Overview
 
This repository contains the full incident investigation report that I shared as part of my learning journey for **CloutHaus** — a simulated social-media-enabled phishing operation targeting influencer Afomiya Storm, conducted within the KC7 cybersecurity training platform.
 
The scenario was built by former Microsoft DART analysts and mirrors real-world SOC investigation methodology. All company names, individuals, IP addresses, and artifacts are entirely simulated for training purposes.
 
---
 
## The Incident — TL;DR
 
A threat actor conducted a patient, multi-stage operation against Afomiya Storm — a rising influencer and new employee at CloutHaus — beginning with weeks of OSINT reconnaissance and ending with silent email exfiltration of her most sensitive personal and financial documents.
 
| Metric | Finding |
|---|---|
| Reconnaissance start | March 31, 2025 (infrastructure staged) |
| Active stalking period | April 3 – April 7, 2025 |
| Initial access method | Spearphishing link — fake Dior collaboration email |
| Time-to-click (spearphish) | 39 minutes after delivery |
| Attacker IP | 182.45.67.89 (Shandong, China) |
| Phishing infrastructure | 3 domains tied to 2 attacker IPs |
| Accounts compromised | Corporate email + Instagram |
| MFA status | Disabled on corporate account |
| Exfiltration destination | noreply@influencer-deals.net |
| Documents exfiltrated | 21 forwarded emails including passport scan, payment details, tax documents |
| Overall severity | **HIGH** |
 
---
 
## Attack Chain Summary
 
### Phase 1 — OSINT Reconnaissance
 
A threat actor at IP `182.45.67.89` spent weeks conducting targeted reconnaissance against Afomiya Storm before a single phishing email was sent:
 
- Queried Afomiya's home address, Venmo history, agent contact, and apartment view across 27 logged requests to the CloutHaus website
- Extracted her personal business email `afomiya.storm@gmail.com` from her public Instagram profile
- Socially engineered KBA security question answers through a fake Instagram Q&A session, harvesting her childhood pet name (`Arsema`), first school (`Lalibela High`), mother's maiden name (`Kidus`), and hometown (`Washington, DC`)
- Attempted Gmail account takeover via KBA bypass — blocked by MFA on her personal account
- Used reverse image search on her apartment view photos to pinpoint her physical address at **The Apartments at CityCenter, Washington D.C.**
- Identified her physical mailbox access via a key photo she shared publicly
### Phase 2 — Spearphishing and Credential Harvest
 
After Afomiya joined CloutHaus and MFA was not enabled on her corporate account:
 
- Attacker sent a fake Dior collaboration email from `collabs@dior-partners.com` to `afomiya_storm@clouthaus.com`
- Subject: `[EXTERNAL] Exclusive Partnership Opportunity with Dior` — email verdict: `CLEAN`
- Afomiya clicked the link `https://super-brand-offer.com/login` at `2025-04-03T11:20:00Z`
- Credential harvest confirmed — username `afstorm` and password submitted to attacker-controlled page
- Attacker authenticated from `182.45.67.89` at `2025-04-03T12:20:00Z` using user agent `Mozilla/5.0 (compatible; MSIE 5.0; Windows NT 5.2; Trident/4.1)`
- Stolen credentials reused to compromise her Instagram account — no MFA on that account either
- Attacker impersonated Afomiya in DMs to solicit money from followers and accepted brand deal packages to her known physical address
### Phase 3 — Post-Compromise Exfiltration
 
- Silent email forwarding rules established, routing Afomiya's inbox to `noreply@influencer-deals.net`
- 21 emails exfiltrated between `2025-04-08T15:51Z` and end of session
- Exfiltrated material included passport scan, direct deposit form, tax documents, brand campaign NDA, PR contact list, travel confirmations, and personal phone number
- Attacker objective assessed as: **identity theft + financial fraud + long-term espionage access**
---
 
## MITRE ATT&CK Mapping
 
| Technique | ID |
|---|---|
| Search Victim-Owned Websites | T1594 |
| Gather Victim Identity Info — Social Media | T1593.001 |
| Phishing for Information — Spearphishing via Social Engineering | T1598 |
| Phishing — Spearphishing Link | T1566.002 |
| Steal Web Session Credentials | T1539 |
| Valid Accounts | T1078 |
| Compromise Accounts — Email Accounts | T1586.002 |
| Email Forwarding Rule | T1114.003 |
| Exfiltration Over Web Service | T1567 |
| Financial Theft | T1657 |
| Search Open Technical Databases — Passive DNS | T1596.001 |
 
---
 
## Indicators of Compromise (IOCs)
 
| Indicator | Type | Context |
|---|---|---|
| `182.45.67.89` | IP | Attacker exit node — recon, auth, exfiltration |
| `198.51.100.12` | IP | Secondary attacker IP — phishing infrastructure |
| `dior-partners.com` | Domain | Phishing sender domain |
| `super-brand-offer.com` | Domain | Credential harvesting page |
| `influencer-deals.net` | Domain | Exfiltration destination |
| `collabs@dior-partners.com` | Email | Phishing sender |
| `noreply@influencer-deals.net` | Email | Exfiltration forwarding address |
| `afomiya_storm@clouthaus.com` | Email | Compromised corporate account |
| `afomiya.storm@gmail.com` | Email | Targeted personal account (MFA saved it) |
| `afstorm` | Username | Harvested from credential phishing page |
| `10.10.0.3` | IP | Afomiya's internal IP |
| `https://super-brand-offer.com/login` | URL | Credential harvesting page |
 
---
 
## Key Investigative Decisions
 
**Why start with `InboundNetworkEvents` before authentication logs?**
The earliest signal was a suspicious IP conducting reconnaissance on the CloutHaus website weeks before any account was touched. Starting from `AuthenticationEvents` would have missed the entire pre-attack OSINT phase. The correct starting point is always the table that matches where the attacker currently is in the kill chain, not where you expect the damage to have happened.
 
**Why did the PassiveDNS pivot matter so much?**
The phishing email's sender domain `dior-partners.com` appeared unconnected to the attacker IP at first glance. A PassiveDNS query linked both attacker IPs to all three malicious domains — `dior-partners.com`, `super-brand-offer.com`, and `influencer-deals.net` — collapsing what looked like separate infrastructure into a single coordinated operation. One pivot connected the reconnaissance, the phish, the credential page, and the exfiltration destination.
 
**What made this attack succeed?**
No malware. No zero-day. Every door was already open:
- MFA disabled on the corporate CloutHaus account
- Password reused across corporate email and Instagram
- Weeks of publicly available OSINT from Afomiya's own social media content
- Email security filter returning `CLEAN` on a targeted spearphish
- No behavioral monitoring on forwarding rule creation or bulk email export
---
 
## Defensive Recommendations
 
**Immediate**
- Enable MFA on all corporate accounts at onboarding — non-negotiable for public-facing employees
- Block newly registered domains at the email gateway (all three attacker domains had no legitimate history)
- Flag outbound connections to credential-harvesting page patterns (`/login` on non-corporate domains)
**Behavioral Monitoring**
- Alert on new email forwarding rules created to external domains — this is not normal user behavior
- Alert on bulk external email forwarding within a single session
- Monitor for login events from anomalous geolocations against known user baselines (impossible travel)
**Organizational**
- Public-facing employees carry elevated OSINT exposure by default — standard security awareness training does not account for this. Influencers, executives, and any staff with public social media presence require bespoke OPSEC briefings
- Password reuse is an organizational risk, not just an individual one. Enforce unique credentials via a managed password manager
---
 
## KQL Query Reference
 
```kql
// Q1 — Suspicious IP Reconnaissance Activity
InboundNetworkEvents
| where src_ip contains "182.45.67.89"
 
// Q2 — Phishing Email Identification
Email
| where recipient == "afomiya_storm@clouthaus.com"
| where subject contains "dior" or links contains "dior"
 
// Q3 — PassiveDNS Infrastructure Pivot
PassiveDns
| where domain contains "dior-partners.com"
 
// Q4 — Full Attacker Infrastructure via IP
PassiveDns
| where ip contains "198.51.100.12"
| distinct domain
 
// Q5 — Click Confirmation
OutboundNetworkEvents
| where url contains "super-brand-offer"
 
// Q6 — Attacker Authentication
AuthenticationEvents
| where username contains "afstorm"
 
// Q7 — Exfiltration via Email Forwarding
Email
| where sender contains "afomiya_storm@clouthaus.com"
| where recipient contains "dior-partners" or recipient contains "influencer-deals"
```
 
---
 
## About This Report
 
This investigation was completed on the [KC7 cybersecurity training platform](https://kc7cyber.com/) — a free, browser-based SOC simulation environment built by former Microsoft DART analysts. The scenario uses real-world log sources (KQL queries against `InboundNetworkEvents`, `OutboundNetworkEvents`, `Email`, `PassiveDns`, `AuthenticationEvents`, `Employees`) and mirrors production incident investigation methodology.
 
If you have any comments or suggestions please feel free to contact me.
 
Published as part of Muhammad Essam's public cybersecurity portfolio.
 
---
 
*Written by Muhammad Essam · 0xM4R7YR*
