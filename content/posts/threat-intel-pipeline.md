---
title: "Building a Threat Intel Pipeline: MISP + Wazuh + TheHive"
date: 2026-03-03
draft: false
tags: ["homelab", "security", "misp", "wazuh", "thehive", "threat-intel"]
---

Built out a full threat intelligence pipeline in the homelab today. The goal was to go from raw log data to enriched, actionable alerts with real IOC context — not just signature matching.

## The Problem

Wazuh out of the box is good at detecting known attack patterns — brute force, privilege escalation, file tampering. But it has no context about *who* is attacking. An SSH brute force from a random IP is an alert. An SSH brute force from a known Feodo C2 node is an incident.

## What I Built

### MISP Feeds

Loaded 72 default threat intelligence feeds into MISP and enabled 16 high-quality free ones:

- **abuse.ch** — Feodo C2 IPs, URLhaus malware URLs, MalwareBazaar samples, Threatfox IOCs, SSL blacklist
- **CIRCL OSINT Feed** — curated threat events with full context
- **Emerging Threats** — compromised IP blocklist
- **DigitalSide Threat-Intel** — community OSINT feed
- **Botvrij.eu, firehol_level1, blocklist.de, IPsum** — aggregated bad IP lists

Feeds auto-refresh daily at 2:30 AM via MISP's built-in cron.

### Two-Track Wazuh Integration

Rather than picking one approach, I wired up two:

**Track 1 — Daily batch sync**

A Python script runs at 4 AM, pulls all published IOCs from MISP (IPs, domains, SHA256 hashes), and writes them to Wazuh's CDB lists. Currently sitting at ~4,900 IPs, ~5,200 domains, and ~2,900 hashes. Wazuh rules 99900-99920 match these lists against SSH, web, Suricata, FIM, and Windows logs in real time.

**Track 2 — Real-time enrichment**

Every Wazuh alert at level 3+ triggers a custom integration script that queries MISP's REST API for any IOC fields in the alert (source IP, destination IP, DNS query, HTTP hostname, file hash). If MISP has a record, it writes a hit to a log file with the full event context — campaign name, threat level, tags, category. Wazuh then picks that up and fires a new enriched alert at level 10-13.

The result: when Suricata sees traffic to a known C2 IP, Wazuh fires two alerts — one from the CDB list match (fast, no API call) and one from the real-time lookup with the MISP event details attached.

### TheHive

Added TheHive 5 as the incident investigation layer. When something interesting shows up in Wazuh, I now have somewhere to open a case, track findings, and document the investigation. Backed by its own Elasticsearch instance, separate from Wazuh's OpenSearch.

## The Stack

```
Threat feeds (72 sources)
    -> daily
MISP — structured IOC database
    -> two tracks
Wazuh CDB lists          Real-time API lookup
(fast matching)          (enriched context)
    ->
Wazuh alerts (rules 99900-99920, 100200-100204)
    ->
Loki + Grafana (30-day retention)
    ->
TheHive (incident investigation)
```

## What This Actually Gets You

Before: "SSH brute force from 185.220.101.45 — level 5 alert"

After: "SSH brute force from 185.220.101.45 — MISP HIGH CONFIDENCE — Feodo tracker, Tor exit node — level 13 alert"

The difference matters when you're triaging ten alerts and need to know which one to actually care about.

## What's Next

- Wazuh to MISP sightings (report back when a known IOC is observed in the environment)
- Active response to auto-block confirmed C2 IPs at the firewall
- Cortex integration with TheHive for automated enrichment and response
