# IR-2026-002 — CloutHaus Incident Investigation Report
**Social Media OSINT, Credential Harvest, and Corporate Email Exfiltration**

> **Analyst:** Muhammad Essam · **Handle:** 0xM4R7YR · **Severity:** HIGH · **Status:** CONTAINED

---

## ⚠️ Before Reading

This is a **structured DFIR overview** of the CloutHaus investigation, designed as a portfolio reference.  
For the full Q&A-style walkthrough with KQL queries, screenshots, and step-by-step reasoning, check the full investigation writeup on Medium.

---

## Overview

This repository contains the full incident investigation report for **IR-2026-002 (CloutHaus)** — a simulated multi-phase cyber operation conducted against CloutHaus and its employee, **Afomiya Storm** (Influencer Partner), within the **KC7 cybersecurity training platform**.

The scenario was built by former Microsoft DART analysts and mirrors real-world SOC investigation methodology. All company names, individuals, IP addresses, and artifacts are entirely simulated for training purposes.

---

## The Incident — TL;DR

A threat actor conducted a six-month OSINT campaign against a public-facing CloutHaus employee, harvested her corporate credentials via a luxury brand spearphish, then spent six days systematically exfiltrating 21 emails containing identity documents, financial records, and corporate intelligence — all while simultaneously compromising her Instagram to run a financial scam against her followers.

**No malware. No exploits. One disabled MFA toggle and a lifetime of oversharing.**

| Metric | Finding |
|---|---|
| OSINT pre-operation period | ~6 months (Oct 2024 – Apr 2025) |
| Active operation duration | April 3 – April 8, 2025 (6 days) |
| Initial access method | Credential harvest via spearphishing link |
| Accounts compromised | 2 — corporate email + Instagram |
| KBA bypass attempt | Executed — blocked by personal Gmail MFA |
| Phishing emails delivered | 1 (high-precision spearphish) |
| Time-to-click | 39 minutes after delivery |
| Time-to-auth (attacker) | 60 minutes after click |
| Emails exfiltrated | 21 forwarded to attacker-controlled inbox |
| Exfiltration destination | `noreply@influencer-deals.net` |
| Physical world escalation | Apartment located, key duplicated, mailbox accessed |
| Overall severity | **HIGH** |

---

## Attack Chain Summary

```
[Phase 1 — Extended Reconnaissance]
    └─ ~6 months of Instagram OSINT
         └─ Location (Washington D.C.), daily schedule, travel history
         └─ Personal email publicly listed for brand deals
         └─ Apartment view photographed → reverse image searched
         └─ Physical key visible in photo dump → duplicated
    └─ 27 suspicious HTTP queries logged from 182.45.67.89 against clouthaus.com
         └─ Home address, Venmo history, agent contact, apartment reverse-image method
         └─ "How to hack an influencer's location from their Instagram story"
         └─ "How much would it cost to hire Afomiya for a fake birthday party?"

[Phase 2 — KBA Bypass Attempt → BLOCKED]
    └─ KBA answers harvested via Instagram Q&A session (in-flight, bored, unguarded)
         └─ Pet name: Arsema | First school: Lalibela High
         └─ Mother's maiden name: Kidus | Hometown: Washington, DC
    └─ Password reset submitted against afomiya.storm@gmail.com
    └─ BLOCKED — OTP-based MFA on personal Gmail account stopped the reset

[Phase 3 — Spearphishing & Credential Harvest]
    └─ Fake Dior collaboration email → collabs@dior-partners.com
         └─ Subject: [EXTERNAL] Exclusive Partnership Opportunity with Dior
         └─ Link: https://super-brand-offer.com/login
         └─ Email gateway verdict: CLEAN
    └─ Afomiya clicked at 2025-04-03T11:20:00Z (src: 10.10.0.3)
    └─ Credentials entered 2 seconds later — username: afstorm

[Phase 4 — Account Compromise]
    └─ Corporate email (afomiya_storm@clouthaus.com)
         └─ MFA: DISABLED — no second factor to block attacker auth
         └─ Attacker authenticated at 2025-04-03T12:20:00Z from 182.45.67.89
         └─ User-Agent: MSIE 5.0 / Windows NT 5.2 / Trident/4.1 (anomalous)
         └─ Geolocation: Shandong, China — impossible travel vs. Afomiya's US location
    └─ Instagram
         └─ Password reused from harvested corporate credentials
         └─ MFA not enabled — attacker walked straight in

[Phase 5 — Post-Compromise Activity]
    └─ Auto-forwarding rule established → noreply@influencer-deals.net
    └─ 21 emails exfiltrated April 3–8 (identity docs, financial records, corporate IP)
    └─ Instagram used to run financial scam against Afomiya's followers
    └─ Brand partners contacted to redirect physical deliveries to her apartment
    └─ Physical mailbox accessed using duplicated key from Instagram photo
```

---

## Phase 1 — Reconnaissance

**Duration:** ~October 2024 – April 7, 2025

The threat actor treated Afomiya's Instagram as an open-source intelligence database, systematically building a complete digital — and physical — profile over six months. She hired at CloutHaus on `2024-10-10`. The attacker's first logged query appeared `2025-04-03` — approximately six months and seven days later. That gap was not idle time.

**Instagram OSINT surface exposed by Afomiya:**
- Location: Washington D.C., DC-based, named apartment building identifiable via rooftop pool reverse image search
- Travel history: France, Italy, Malta, Bahamas, Dubai, Puerto Rico (Story Highlights)
- Professional email publicly listed: `afomiya.storm@gmail.com`
- Daily content pattern: GRWM videos, Day-in-My-Life vlogs — predictable schedule
- Physical keys: visible in apartment photo dump → key duplication confirmed feasible
- Financial exposure: Venmo account existence and public transaction visibility probed

**KBA answers harvested via Instagram Q&A (in-flight session):**

| Security Question | Answer Exposed |
|---|---|
| Childhood pet name | Arsema |
| Name of first school | Lalibela High |
| Mother's maiden name | Kidus |
| Where did you grow up | Washington, DC |

**Website reconnaissance — 27 HTTP requests logged from `182.45.67.89`:**  
Queries ranged from address lookups and Venmo history to apartment reverse-image search methodology and a planned fake birthday party lure — indicating the attacker was preparing multiple follow-on attack vectors beyond the phishing campaign.

---

## Phase 2 — KBA Bypass Attempt

**Date:** Early April 2025

Armed with all four KBA answers, the attacker submitted a password reset against `afomiya.storm@gmail.com`. The attempt failed at the final step — Gmail required an OTP sent to Afomiya's registered phone number. Without physical access to the device, the reset could not complete.

**What saved the personal account:** OTP-based MFA on Gmail.  
**What did not save the corporate account:** MFA was disabled on `afomiya_storm@clouthaus.com`.

This asymmetry — personal account protected, corporate account exposed — became the decisive factor in Phase 4.

---

## Phase 3 — Spearphishing & Credential Harvest

**Date:** April 3, 2025

The attacker deployed a single, high-precision spearphishing email impersonating the luxury brand Dior — chosen deliberately to exploit Afomiya's professional context as an Influencer Partner actively seeking brand collaborations.

- **Sender:** `collabs@dior-partners.com`
- **PassiveDNS correlation:** `dior-partners.com` resolves to `182.45.67.89` — the same IP conducting recon since April 3rd. Domain registered before the email was sent.
- **Subject:** `[EXTERNAL] Exclusive Partnership Opportunity with Dior`
- **Payload:** `https://super-brand-offer.com/login` → credential harvesting page
- **Gateway verdict:** `CLEAN` — email filtering did not flag the message
- **Click confirmed:** `2025-04-03T11:20:00Z` via OutboundNetworkEvents from `10.10.0.3`
- **Credentials submitted:** `2025-04-03T11:20:02Z` — two seconds after click

The `[EXTERNAL]` tag in the subject line was present but insufficient. The email cleared all automated filters.

---

## Phase 4 — Account Compromise

**Date:** April 3, 2025

**Corporate Email**  
With credentials in hand, the attacker authenticated to `afomiya_storm@clouthaus.com` at `2025-04-03T12:20:00Z` — exactly 60 minutes after Afomiya clicked the link. MFA was disabled; there was no second factor to challenge the login.

Two details made this authentication event detectable in hindsight:
- **User-Agent:** `Mozilla/5.0 (compatible; MSIE 5.0; Windows NT 5.2; Trident/4.1)` — Internet Explorer 5 on Windows XP/Server 2003 era. No legitimate user in 2025 authenticates with this string.
- **Geolocation:** `182.45.67.89` resolves to Shandong, China (ChinaNet / AS4134). Afomiya was on a domestic US flight at the time — textbook impossible travel.

Neither signal triggered an automated alert.

**Instagram**  
Afomiya reused her corporate password across accounts. Instagram had no MFA enabled. The attacker authenticated without friction, gaining full control of her public-facing profile and DMs.

---

## Phase 5 — Post-Compromise Activity & Exfiltration

**Date:** April 3–8, 2025

**Corporate mailbox exfiltration**  
The attacker established an auto-forwarding rule routing all outbound email to `noreply@influencer-deals.net` — a domain pre-linked to the same attacker infrastructure via PassiveDNS. Over the following six days, 21 emails were silently exfiltrated.

Exfiltrated content covered three distinct threat categories:

| Category | Examples |
|---|---|
| **Identity** | Passport scan, personal phone number, health insurance details |
| **Financial** | Direct deposit form, bank statement, tax year-end summary |
| **Corporate intelligence** | Brand campaign insights, Influencer NDA, PR contact list, content calendar |
| **Persistence** | Gmail login credentials |

**Instagram abuse**  
- Followers targeted with financial scam DMs — soliciting money under the guise of "exclusive investment opportunities"
- Brand partners contacted to redirect physical package deliveries to Afomiya's apartment (location already confirmed via reverse image search)
- Physical mailbox accessed using a key duplicated from her Instagram photo

The operation crossed from digital compromise into physical world impact.

---

## What Made This Attack Succeed

No malware. No exploit. A sequence of policy gaps and human behaviour that an attacker methodically mapped over six months:

- MFA disabled on the corporate CloutHaus account — the single most consequential failure
- Password reused across corporate email and Instagram
- KBA left enabled as an authentication mechanism (deprecated by NIST SP 800-63B in 2017)
- Public-facing employee with no OPSEC guidance, exposing location, schedule, financial accounts, and physical keys via Instagram
- Email gateway returning `CLEAN` on a socially engineered spearphish from a days-old domain
- No behavioral alerting on: anomalous User-Agent strings, impossible travel, or newly established mailbox forwarding rules

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Reconnaissance | Search Open Websites / Domains — Social Media | T1593.001 |
| Reconnaissance | Gather Victim Identity Info — Credentials | T1589.001 |
| Reconnaissance | Search Open Technical Databases — Passive DNS | T1596.001 |
| Resource Development | Acquire Infrastructure — Domain | T1583.001 |
| Initial Access | Phishing — Spearphishing Link | T1566.002 |
| Credential Access | Steal Web Session Credentials | T1555.003 |
| Credential Access | Modify Authentication — Security Questions (KBA) | T1556 |
| Persistence | Valid Accounts | T1078 |
| Defense Evasion | Valid Accounts | T1078 |
| Collection | Email Collection — Remote Email Collection | T1114.002 |
| Exfiltration | Exfiltration Over Web Service | T1567 |
| Impact | Financial Theft | T1657 |

---

## Indicators of Compromise (IOCs)

| Indicator | Type | Context |
|---|---|---|
| `182.45.67.89` | IP | Primary attacker node — recon, authentication, phishing infrastructure (Shandong, CN) |
| `198.51.100.12` | IP | Secondary attacker infrastructure — linked via pDNS to phishing domains |
| `dior-partners.com` | Domain | Phishing sender domain — resolves to `182.45.67.89` |
| `super-brand-offer.com` | Domain | Credential harvesting page — resolves to `198.51.100.12` |
| `influencer-deals.net` | Domain | Exfiltration destination domain |
| `collabs@dior-partners.com` | Email | Phishing sender address |
| `noreply@influencer-deals.net` | Email | Auto-forward exfiltration recipient |
| `https://super-brand-offer.com/login` | URL | Credential harvesting endpoint |
| `afomiya_storm@clouthaus.com` | Email | Compromised corporate account |
| `afomiya.storm@gmail.com` | Email | Targeted personal account — KBA bypass attempted, MFA blocked |
| `10.10.0.3` | IP (internal) | Afomiya's workstation — click confirmed via OutboundNetworkEvents |
| `afstorm` | Username | Compromised corporate credential |
| `MSIE 5.0 / Windows NT 5.2 / Trident/4.1` | User-Agent | Anomalous browser fingerprint used during attacker authentication |

---

## Key Investigative Decisions

**Why start with InboundNetworkEvents instead of AuthenticationEvents?**  
The investigation was intel-led, anchored by a suspicious IP identified through recon-phase log analysis. At the point of flagging, the attacker had not yet touched the authentication layer. Defaulting to AuthenticationEvents — the standard alert-driven starting point — would have missed the entire six-month reconnaissance phase, the KBA bypass attempt, and the pre-phishing infrastructure setup. Log source selection must match the current phase of the attack chain.

**Why was the PassiveDNS pivot the critical unlock?**  
A single confirmed IOC — `182.45.67.89` — queried against PassiveDNS resolved to `dior-partners.com` and `influencer-deals.net`. That one pivot linked the phishing sender domain directly to the exfiltration destination, proving both were operated by the same actor on shared infrastructure. One IP. The entire operation mapped.

**Why did the two-IP infrastructure matter?**  
`182.45.67.89` was the noisy IP — 27 logged recon queries, attacker authentication, linked to phishing domains. `198.51.100.12` was quieter — only two InboundNetworkEvents, both appearing routine. Without PassiveDNS correlation, `198.51.100.12` would have been invisible. It hosted the credential harvesting page the victim actually typed her password into.

---

## Defensive Recommendations

**Immediate**
- Enforce MFA on all corporate accounts — mandatory, not optional. This single control would have blocked the entire Phase 4 attack.
- Audit and purge all mailbox forwarding rules across the organization immediately.
- Eliminate KBA as an authentication fallback. Replace with FIDO2/passkeys or hardware tokens. NIST deprecated KBA in 2017; this scenario shows why.

**Detection Engineering**
- Alert on authentication events with legacy or anomalous User-Agent strings (MSIE, Trident engine versions pre-IE11).
- Alert on impossible travel — same account authenticated from geographically separated locations within an implausible timeframe.
- Alert on new external mailbox forwarding rules created post-authentication, particularly to non-corporate domains.
- Alert on emails delivered from domains registered within the last 30 days — `dior-partners.com` would have been blocked at delivery.

**Organizational**
- Public-facing employees (influencers, executives, PR roles) require bespoke OPSEC briefings. Standard security awareness training does not account for the elevated OSINT surface these roles carry by design.
- Password reuse is a policy failure before it is a user failure. Enforce unique credentials via SSO and mandate a password manager.
- The attacker mapped Afomiya's physical location, duplicated her key, and accessed her mailbox — all from Instagram. OPSEC guidance for public figures must extend to physical world exposure, not only digital hygiene.

---

## KQL Query Reference

```kql
// Q1 — Identify attacker IP via recon activity
InboundNetworkEvents
| where src_ip == "182.45.67.89"
| order by timestamp asc

// Q2 — Confirm phishing email delivery and sender domain
Email
| where recipient == "afomiya_storm@clouthaus.com"
| where subject contains "dior" or links contains "dior"

// Q3 — PassiveDNS pivot — map attacker IP to domains
PassiveDns
| where ip == "182.45.67.89"

// Q4 — PassiveDNS pivot — map credential harvesting domain to IP
PassiveDns
| where domain contains "super-brand-offer.com"

// Q5 — Confirm victim clicked phishing link
OutboundNetworkEvents
| where url contains "super-brand-offer"
| where src_ip == "10.10.0.3"

// Q6 — Confirm attacker authentication (impossible travel)
AuthenticationEvents
| where username == "afstorm"
| where src_ip == "182.45.67.89"

// Q7 — Map full attacker infrastructure from secondary IP
PassiveDns
| where ip == "198.51.100.12"
| distinct domain

// Q8 — Identify exfiltrated emails via auto-forwarding rule
Email
| where sender contains "afomiya_storm@clouthaus.com"
| where recipient contains "dior-partners" or recipient contains "influencer-deals"
```

---

## About This Report

This investigation was completed on the **KC7 cybersecurity training platform** — a free, browser-based SOC simulation environment built by former Microsoft DART analysts. The scenario uses real-world log sources (KQL queries against InboundNetworkEvents, OutboundNetworkEvents, Email, PassiveDns, AuthenticationEvents, Employees) and mirrors production incident investigation methodology.

> Published as part of Muhammad Essam's public cybersecurity portfolio.

**Written by Muhammad Essam · 0xM4R7YR · IR-2026-002**
