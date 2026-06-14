# IR-2026-002 — CloutHaus Incident Investigation Report
**Credential Compromise, Spearphishing, and Data Exfiltration at CloutHaus**

> **Analyst:** Muhammad Essam · **Handle:** 0xM4R7YR · **Severity:** HIGH · **Status:** CONTAINED

---

## ⚠️ Before Reading

This is a **structured DFIR overview** of the CloutHaus investigation — designed as a portfolio reference.  
For the full Q&A-style walkthrough including KQL queries, screenshots, and step-by-step reasoning, check the full investigation writeup.

---

## Overview

This repository contains the incident investigation report for **IR-2026-002 (CloutHaus)** — a simulated two-phase cyber operation conducted against CloutHaus and its employee, rising influencer **Afomiya Storm**, within the **KC7 cybersecurity training platform**.

The scenario was built by former Microsoft DART analysts and mirrors real-world SOC investigation methodology. All company names, individuals, IP addresses, and artifacts are entirely simulated for training purposes.

---

## The Incident — TL;DR

A threat actor conducted a prolonged OSINT and social engineering campaign against Afomiya Storm — a CloutHaus Influencer Partner — ultimately harvesting her credentials via a spearphishing link, compromising her corporate email, and systematically exfiltrating sensitive personal and financial documents over several days.

**Zero malware. Zero exploits. One open inbox and a lot of oversharing.**

| Metric | Finding |
|---|---|
| Operation duration | April 3 – April 8, 2025 (6 days active) |
| OSINT pre-operation period | ~6 months (Oct 2024 – Apr 2025) |
| Initial access method | Credential harvest via spearphishing link |
| Accounts compromised | 2 — corporate email + Instagram |
| Phishing emails delivered | 1 (targeted spearphish) |
| Time-to-click (spearphish) | 39 minutes after delivery |
| Time-to-auth (post-click) | 60 minutes after click |
| Emails exfiltrated | 21 forwarded to attacker-controlled inbox |
| Data exfiltrated | Passport scan, payment details, tax documents, brand NDAs, personal contact info |
| Overall severity | **HIGH** |

---

## Attack Chain Summary

```
[Reconnaissance]
    └─ ~6 months of OSINT via Instagram (location, schedule, contacts, personal details)
    └─ 27 suspicious queries logged from 182.45.67.89 against clouthaus.com

[KBA Bypass Attempt — BLOCKED]
    └─ Security questions answered via Instagram Q&A session data
    └─ MFA on personal Gmail blocked password reset

[Spearphishing]
    └─ Fake Dior collaboration email → collabs@dior-partners.com
    └─ Link: https://super-brand-offer.com/login (credential harvesting page)
    └─ Afomiya clicked at 2025-04-03T11:20:00Z — credentials entered 2 seconds later

[Account Compromise]
    └─ Corporate email (afomiya_storm@clouthaus.com) — MFA disabled
    └─ Instagram — password reused from harvested credentials

[Post-Compromise Activity]
    └─ Attacker authenticated at 2025-04-03T12:20:00Z from 182.45.67.89
    └─ Mailbox searched and auto-forwarding rule established → noreply@influencer-deals.net
    └─ 21 emails exfiltrated April 3–8, 2025

[Instagram Account Abuse]
    └─ Followers targeted with financial scam DMs
    └─ Brand partners contacted to redirect package deliveries to Afomiya's apartment
    └─ Apartment location confirmed via reverse image search of Instagram posts
    └─ Physical mailbox key duplicated from Instagram photo
```

---

## Phase 1 — Reconnaissance

**Duration:** ~October 2024 – April 7, 2025

The threat actor conducted extended OSINT against Afomiya Storm using publicly available Instagram content and the CloutHaus public website.

- **Instagram OSINT** — Location (Washington D.C.), daily schedule, travel history, email for brand deals (`afomiya.storm@gmail.com`), apartment view, physical key photo, personal relationships
- **KBA harvesting** — Security question answers extracted from a public Instagram Q&A session: childhood pet (`Arsema`), first school (`Lalibela High`), mother's maiden name (`Kidus`), hometown (`Washington, DC`)
- **Website reconnaissance** — 27 HTTP requests logged from `182.45.67.89` against `clouthaus.com`, including queries targeting Afomiya's home address, Venmo history, agent contact, and apartment reverse-image search methodology
- **KBA bypass attempted — BLOCKED** — Password reset submitted against her personal Gmail using harvested KBA answers; blocked by OTP-based MFA on her personal account

**First attacker appearance:** `2025-04-03T00:00:00Z`  
**Last recon log:** `2025-04-07T12:36:01Z`

---

## Phase 2 — Spearphishing & Initial Access

**Date:** April 3, 2025

The attacker deployed a targeted spearphishing email impersonating the luxury brand **Dior**, crafted to exploit Afomiya's professional role as an Influencer Partner.

- **Sender:** `collabs@dior-partners.com` (PassiveDNS confirms domain resolves to attacker IP `182.45.67.89`)
- **Subject:** `[EXTERNAL] Exclusive Partnership Opportunity with Dior`
- **Payload link:** `https://super-brand-offer.com/login` (credential harvesting page; domain resolves to `198.51.100.12`)
- **Email verdict:** `CLEAN` — bypassed email gateway filtering
- **Click confirmed:** `2025-04-03T11:20:00Z` via OutboundNetworkEvents from `10.10.0.3`
- **Credentials entered:** `2025-04-03T11:20:02Z` (username: `afstorm`)

---

## Phase 3 — Account Compromise & Exfiltration

**Date:** April 3–8, 2025

With credentials harvested, the attacker authenticated to Afomiya's corporate email and Instagram within the hour.

**Corporate Email Compromise**
- Attacker authenticated at `2025-04-03T12:20:00Z` from `182.45.67.89`
- User-Agent: `Mozilla/5.0 (compatible; MSIE 5.0; Windows NT 5.2; Trident/4.1)` — anomalous ancient browser fingerprint
- Auto-forwarding rule established to `noreply@influencer-deals.net`
- 21 emails exfiltrated April 3–8, including:

| Forwarded Subject | Sensitivity |
|---|---|
| Afomiya's passport scan – confidential | Identity document |
| Afomiya's payment details – direct deposit form | Financial / banking |
| Re: Tax documents – year-end summary | Financial |
| Afomiya's personal phone number – private contact info | PII |
| Re: Brand campaign insights – confidential report | Corporate IP |
| Travel confirmation – flight and hotel details | Physical security |
| Influencer NDA agreement | Legal / contractual |
| Gmail login details | Credential |

**Instagram Compromise**
- Password reused from harvested CloutHaus credentials
- MFA not enabled on CloutHaus account
- Attacker impersonated Afomiya, sending financial scam DMs to followers
- Brand partners contacted to redirect physical deliveries to Afomiya's apartment
- Physical mailbox access facilitated by key duplication from Instagram photo

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Reconnaissance | Search Open Websites / Domains — Social Media | T1593.001 |
| Reconnaissance | Gather Victim Identity Info — Credentials | T1589.001 |
| Reconnaissance | Search Open Technical Databases — Passive DNS | T1596.001 |
| Resource Development | Acquire Infrastructure — Domain | T1583.001 |
| Resource Development | Compromise Accounts — Email | T1586.002 |
| Initial Access | Phishing — Spearphishing Link | T1566.002 |
| Credential Access | Steal Web Session Cookie / Web Credentials | T1555.003 |
| Credential Access | Modify Authentication Process — MFA / Security Questions | T1556 |
| Persistence | Valid Accounts | T1078 |
| Collection | Email Collection — Remote Email Collection | T1114.002 |
| Exfiltration | Exfiltration Over Web Service | T1567 |
| Impact | Financial Theft | T1657 |

---

## Indicators of Compromise (IOCs)

| Indicator | Type | Context |
|---|---|---|
| `182.45.67.89` | IP | Attacker exit node — recon, auth, phishing infra (Shandong, China) |
| `198.51.100.12` | IP | Secondary attacker infrastructure |
| `dior-partners.com` | Domain | Phishing sender domain — resolves to attacker IP |
| `super-brand-offer.com` | Domain | Credential harvesting page |
| `influencer-deals.net` | Domain | Exfiltration destination |
| `collabs@dior-partners.com` | Email | Phishing sender |
| `noreply@influencer-deals.net` | Email | Exfiltration forwarding destination |
| `https://super-brand-offer.com/login` | URL | Credential harvesting page |
| `afomiya_storm@clouthaus.com` | Email | Compromised corporate account |
| `10.10.0.3` | IP | Afomiya's internal IP — click confirmed |
| `afstorm` | Username | Compromised credential |
| `MSIE 5.0 / Windows NT 5.2 / Trident/4.1` | User-Agent | Anomalous browser fingerprint used during compromise |

---

## Key Investigative Decisions

**Why start with InboundNetworkEvents instead of AuthenticationEvents?**  
The investigation was intel-led, with a suspicious IP identified before authentication events occurred. Starting at AuthenticationEvents — the default T1 triage path — would have missed the full 6-month reconnaissance phase and the KBA bypass attempt entirely. Log source selection must match the current phase of the attack chain.

**Why did PassiveDNS unlock the infrastructure?**  
A single attacker IP (`182.45.67.89`) queried against PassiveDNS resolved to `dior-partners.com` and `influencer-deals.net`, linking the phishing sender domain directly to the exfiltration destination. One IP. Full infrastructure mapped.

**What made this attack succeed?**  
No malware. No exploit. The attacker walked through doors that were already open:
- MFA disabled on the corporate CloutHaus account
- Password reused across personal and corporate accounts
- Six months of OSINT exposure via unguarded Instagram content
- Email gateway returning `CLEAN` on a socially engineered spearphish
- No behavioral alerting on post-authentication mailbox forwarding rules

---

## Defensive Recommendations

**Immediate**
- Enforce MFA on all corporate accounts — non-negotiable for public-facing employees
- Block or alert on newly registered domains at the email gateway (≤30 days old would have blocked `dior-partners.com`)
- Audit and purge all mailbox forwarding rules immediately post-incident

**Behavioral Monitoring**
- Alert on new external forwarding rules created within authenticated sessions
- Alert on bulk email forwarding to domains not on an approved list
- Flag authentication events with anomalous User-Agent strings (e.g., legacy IE versions)
- Alert on impossible travel — same account, different countries, short time window

**Organizational**
- Public-facing employees (influencers, executives) require bespoke OPSEC briefings — standard security awareness does not account for their elevated OSINT exposure
- Password reuse is a policy failure, not a user failure — enforce unique credentials via SSO and a password manager
- Eliminate KBA as an authentication mechanism entirely; replace with FIDO2/passkeys

---

## KQL Query Reference

```kql
// Q1 — Identify attacker IP via recon activity
InboundNetworkEvents
| where src_ip == "182.45.67.89"
| order by timestamp asc

// Q2 — Confirm phishing email delivery
Email
| where recipient == "afomiya_storm@clouthaus.com"
| where subject contains "dior" or links contains "dior"

// Q3 — PassiveDNS pivot on attacker IP
PassiveDns
| where ip == "182.45.67.89"

// Q4 — PassiveDNS pivot on harvesting domain
PassiveDns
| where domain contains "super-brand-offer.com"

// Q5 — Confirm victim clicked phishing link
OutboundNetworkEvents
| where url contains "super-brand-offer"
| where src_ip == "10.10.0.3"

// Q6 — Confirm attacker authentication
AuthenticationEvents
| where username == "afstorm"
| where src_ip == "182.45.67.89"

// Q7 — Map full attacker infrastructure
PassiveDns
| where ip == "198.51.100.12"
| distinct domain

// Q8 — Identify exfiltrated emails
Email
| where sender contains "afomiya_storm@clouthaus.com"
| where recipient contains "dior-partners" or recipient contains "influencer-deals"
```

---

## About This Report

This investigation was completed on the **KC7 cybersecurity training platform** — a free, browser-based SOC simulation environment built by former Microsoft DART analysts. The scenario uses real-world log sources (KQL queries against InboundNetworkEvents, OutboundNetworkEvents, Email, PassiveDns, AuthenticationEvents, Employees) and mirrors production incident investigation methodology.

> Published as part of Muhammad Essam's public cybersecurity portfolio.

**Written by Muhammad Essam · 0xM4R7YR · IR-2026-002**
