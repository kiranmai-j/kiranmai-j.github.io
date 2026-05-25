---
layout: post
title: "Building a Phishing Email Analyzer — A SOC Tool from Scratch"
date: 2026-05-26
categories: python security soc
---

Phishing is responsible for the majority of cyberattacks, and one of the core tasks of a SOC analyst is triaging suspicious emails quickly and accurately. I wanted to understand that process from the inside — so I built my own phishing analyzer tool in Python.

---

## The Problem

When a suspicious email lands in a SOC queue, an analyst needs to answer several questions fast:

- Is the sender who they claim to be?
- Did the email pass SPF, DKIM, and DMARC checks?
- Is the sender's IP known for abuse?
- Do any links in the email lead to malicious sites?

Doing this manually across multiple tools is slow. I wanted a single CLI tool that does all of this automatically and produces a clean report.

---

## What I Built

A Python CLI tool that takes a `.eml` email file as input and produces a detailed HTML threat report. It checks for:

- **SPF / DKIM / DMARC failures** — authentication failures that indicate spoofing
- **Reply-To mismatch** — when the Reply-To address differs from the From address (a classic phishing trick)
- **Sender IP reputation** — checks every sender IP against AbuseIPDB for abuse score, country, ISP, and whether it's a Tor exit node
- **URL threat scanning** — submits every URL in the email to VirusTotal for a malicious/suspicious verdict
- **Risk score (0–100)** — a calculated score with a final verdict: Low, Medium, or High Risk

---

## How the Risk Score Works

Each indicator adds points to the risk score:

| Indicator | Points Added |
|---|---|
| SPF fail | +25 |
| DKIM fail | +20 |
| DMARC fail | +20 |
| Reply-To mismatch | +15 |
| High abuse IP (≥75 score) | +30 |
| Tor exit node | +20 |
| Malicious URL | +25 |
| Suspicious URL | +10 |

A score of 70–100 means HIGH RISK, 35–69 is MEDIUM, and 0–34 is LOW.

---

## The Workflow

```
.eml file
    │
    ├── Extract Headers → SPF/DKIM/DMARC + Reply-To mismatch check
    ├── Extract IPs     → AbuseIPDB reputation lookup
    ├── Extract URLs    → VirusTotal scan
    │
    └── Risk Score (0–100) → HTML Report
```

The whole thing runs with one command:

```bash
python analyzer.py sample_phishing.eml
```

---

## Testing It

I included a sample phishing email in the repo that contains all the classic red flags:

- SPF / DKIM / DMARC all failing
- From address spoofed as PayPal, but Reply-To pointing to a Russian domain
- Suspicious URLs embedded in the body
- Sender IP traced to a Tor exit node

Running the analyzer on it produces a full HTML report flagging every indicator with a HIGH RISK verdict.

---

## What I Used

- **Python standard library only** — no pip installs needed
- **AbuseIPDB API** — free tier (1000 checks/day)
- **VirusTotal API** — free tier (4 requests/min)
- Python's built-in `email` module for parsing `.eml` files

The fact that it uses zero external libraries (besides the APIs) means it's easy to run anywhere without setup headaches.

---

## What I Learned

- How email authentication works — SPF, DKIM, and DMARC are the three layers that verify a sender's identity, and most phishing emails fail at least one
- How SOC analysts triage phishing emails in real workflows (L1/L2 triage process)
- How to work with threat intelligence APIs (AbuseIPDB, VirusTotal)
- How to parse raw `.eml` files using Python's email module

---

## What's Next

- Add support for analyzing email attachments (check for malicious file hashes)
- Integrate with Shodan for deeper IP context
- Build a simple web interface so it doesn't require the command line

---

## Try It Yourself

The full code and sample phishing email are on GitHub:
[github.com/kiranmai-j/phishing-analyzer](https://github.com/kiranmai-j/phishing-analyzer)

> **Note:** Only analyze emails you own or have explicit permission to analyze.
