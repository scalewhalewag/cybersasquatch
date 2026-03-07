---
title: "Homelab"
layout: "single"
---

## Infrastructure

**Ubuntu Server**
- Primary host for all homelab services
- All services run as Docker containers for easy deployment, management, and isolation

**Ubuntu Desktop**
- Local workstation used for development and homelab management

**MacBook**
- Primary machine for all homelab management
- Used extensively for SSH access into the server and desktop
- Runs Zed editor with local LLMs (Gemma3 12B and Qwen2.5-Coder 7B) for offline AI-assisted development, keeping code and queries secure and off third-party servers
- Uses Claude Code in the terminal for agentic task execution across the homelab

**Tailscale**
- Zero-config VPN meshing all devices (server, desktop, Mac, and Android) into a private tailnet
- Custom DNS configured for `.homelab` domains across all devices

## Networking

**AdGuard Home**
- DNS server handling all `.homelab` domain resolution
- Network-wide ad blocking
- Configured as the Tailscale DNS resolver so all tailnet devices can reach internal services by hostname

**Nginx Proxy Manager**
- Reverse proxy handling SSL termination for all internal services
- Enables valid HTTPS for every `.homelab` subdomain

**Internal Certificate Authority**
- Self-hosted CA issuing certificates with SANs for all internal IPs, Tailscale hostnames, and `.homelab` domains
- Trusted on all homelab devices

## Threat Intelligence Pipeline

The homelab runs a fully integrated threat intelligence pipeline connecting MISP, Wazuh, and TheHive.

**MISP**
- Threat intelligence platform aggregating IOCs from 16 curated external feeds (CIRCL, Emerging Threats, abuse.ch, and others)
- Feeds refresh daily via cron, keeping the IOC database current
- Four containers: core app, MISP modules (enrichment), MySQL, Redis

**MISP → Wazuh (Batch Sync)**
- Daily cron job pulls all IOCs from MISP and writes them into Wazuh CDB lookup lists
- Populates three lists: malicious IPs, malicious domains, and malware file hashes
- Runs at 4 AM so the latest threat intel is always available for rule evaluation

**MISP → Wazuh (Real-Time)**
- Custom integration script fires on every Wazuh alert
- Extracts IPs, domains, and file hashes from alert fields and queries MISP live
- On a match, writes enriched JSON hits to a log file that Wazuh re-ingests as a new alert
- Custom rules (level 10–13) fire on these hits with MITRE ATT&CK tags
- Sightings are automatically posted back to MISP, recording which IOCs were actually observed on the network

**Wazuh → TheHive (Automated Case Creation)**
- Custom integration script fires on high-severity alerts (level 12+) and all MISP threat intel hits
- Automatically creates TheHive cases with full alert context: agent, rule, timestamp, and raw alert JSON
- MISP hit cases include IOC type and value in the case title for immediate triage

**TheHive**
- Incident investigation and case management platform
- Receives cases automatically from Wazuh for high-severity events and threat intel matches
- Wired to Cortex for automated IOC analysis on every case
- Two containers: TheHive 5 and a dedicated Elasticsearch 8 instance

**Cortex**
- Observable analysis engine that runs automated analyzers against IOCs in TheHive cases
- Runs analyzers as isolated Docker containers for each job
- Configured with AbuseIPDB, VirusTotal, and Shodan analyzers for IP/domain/hash enrichment
- Two containers: Cortex and a dedicated Elasticsearch 7 instance (required by Cortex 3.x)

## Security Monitoring

**Wazuh**
- SIEM platform for log aggregation, threat detection, and security event alerting
- Three containers: Wazuh Manager, Wazuh Indexer/OpenSearch, Wazuh Dashboard
- Agents deployed on all homelab devices for endpoint log collection and security event forwarding

**Suricata**
- Network IDS/IPS monitoring traffic for threats and suspicious activity
- Rules updated weekly from Emerging Threats Open ruleset via suricata-update
- MISP IOCs exported daily as native Suricata rules, enabling network-level detection of known-bad IPs and domains in raw traffic before any other rule fires
- Noisy low-value rules suppressed to reduce alert fatigue; logs rotated daily with 7-day retention
- Integrated with Wazuh for alert forwarding and MISP for sighting feedback

**Cowrie**
- SSH honeypot capturing and logging unauthorized access attempts on decoy ports

## Observability

**Grafana**
- Visualization platform for homelab metrics and log data
- Dashboards over Wazuh alerts and MISP threat intel hits via Loki

**Loki**
- Log aggregation backend with 30-day retention
- Ingests Wazuh alerts (level 7+) and MISP hit logs for long-term querying

**Promtail**
- Log shipping agent collecting and forwarding logs from Wazuh and MISP integrations into Loki

## Documentation and Management

**NetBox**
- Network documentation and IPAM tool for tracking infrastructure, IPs, and devices
- Backed by PostgreSQL and two Valkey instances (caching and task queue)

## Password Management

**KeePass**
- Self-hosted password manager storing all homelab credentials in an encrypted database

**+**

**Syncthing**
- Open-source peer-to-peer file sync tool
- Syncs the KeePass database across all devices (server, desktop, Mac, and Android) with no cloud provider required
