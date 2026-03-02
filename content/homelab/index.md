---
title: "Homelab"
layout: "single"
---

## Infrastructure

**Ubuntu Server**
The foundation of my homelab, serving as the primary host for all containerized services. Accessible remotely via Tailscale.

**Ubuntu Desktop**
Local workstation used for development and homelab management.

**Tailscale**
Zero-config VPN meshing all devices — server, desktop, work Mac, and Android — into a private tailnet with custom DNS for `.homelab` domains.

## Networking

**AdGuard Home**
DNS server handling all `.homelab` domain resolution and network-wide ad blocking. Configured as the Tailscale DNS resolver so all devices on the tailnet can reach internal services by hostname.

**Nginx Proxy Manager**
Reverse proxy handling SSL termination for all internal services. Backed by a self-signed internal CA with certificates trusted across all devices, enabling valid HTTPS for every `.homelab` subdomain.

**Internal Certificate Authority**
Self-hosted CA issuing certificates with SANs for all internal IPs, Tailscale hostnames, and `.homelab` domains. Trusted on all homelab devices.

## Security Monitoring

**Wazuh**
SIEM platform running as a single-node stack for log aggregation, threat detection, and security event alerting across all homelab endpoints. Backed by OpenSearch for log storage and search, and the Wazuh Dashboard for visualization.

**Suricata**
Network intrusion detection system monitoring traffic for threats and suspicious activity.

**Cowrie**
SSH honeypot capturing and logging unauthorized access attempts on decoy ports.

**MISP**
Threat intelligence platform for aggregating and correlating IOCs from external feeds. Backed by MySQL for data storage and Redis for caching.

## Observability

**Grafana**
Visualization platform for homelab metrics and log data.

**Loki**
Log aggregation backend feeding into Grafana dashboards.

**Promtail**
Log shipping agent collecting and forwarding logs from services into Loki.

## Documentation & Management

**NetBox**
Network documentation and IPAM tool for tracking infrastructure, IPs, and devices. Backed by PostgreSQL for data storage and Valkey for caching.

## Credential Management

**KeePass**
Self-hosted password manager storing all homelab credentials in an encrypted database. The KeePass database is synced across all devices — server, desktop, Mac, and Android — using Syncthing, ensuring every device always has access to the latest credentials without relying on any cloud provider.
