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

## Security Monitoring

**Wazuh**
- SIEM platform for log aggregation, threat detection, and security event alerting
- Three containers: Wazuh Manager (agent coordination and alerting), Wazuh Indexer/OpenSearch (log storage and search), Wazuh Dashboard (visualization)
- Agents deployed on all homelab devices for endpoint log collection and security event forwarding

**Suricata**
- Network IDS/IPS (Intrusion Detection/Prevention System) monitoring traffic for threats and suspicious activity

**Cowrie**
- SSH honeypot capturing and logging unauthorized access attempts on decoy ports

**MISP**
- Threat intelligence platform for aggregating and correlating IOCs from external feeds
- Four containers: core MISP app, MISP modules (enrichment and import/export), MySQL (data storage), Redis (caching)

**TheHive**
- Incident investigation and case management platform
- Integrated with MISP and Wazuh for enriched alert triage and response tracking
- Two containers: TheHive 5 and a dedicated Elasticsearch instance

## Observability

**Grafana**
- Visualization platform for homelab metrics and log data

**Loki**
- Log aggregation backend feeding into Grafana dashboards

**Promtail**
- Log shipping agent collecting and forwarding logs from services into Loki

## Documentation and Management

**NetBox**
- Network documentation and IPAM tool for tracking infrastructure, IPs, and devices
- Backed by PostgreSQL for data storage and two Valkey instances (one for caching, one for the background task queue)

## Password Management

**KeePass**
- Self-hosted password manager storing all homelab credentials in an encrypted database
- Database synced across all devices (server, desktop, Mac, and Android) using Syncthing, an open-source peer-to-peer file sync tool that requires no cloud provider
