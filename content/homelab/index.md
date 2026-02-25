---
title: "Homelab"
layout: "single"
---

## Projects

**Ubuntu Server**  
The foundation of my homelab, running as the primary host for all services and containers.

**Docker Containers**  
Containerized services for easy deployment, management, and isolation of applications.

- **Wazuh** — Full 3-container stack:
  - Wazuh Manager — agent coordination and alerting
  - Wazuh Indexer (OpenSearch) — log storage and search
  - Wazuh Dashboard — visualization and analysis

- **NetBox** — Network documentation and IPAM tool:
  - PostgreSQL — dedicated database
  - Redis (Valkey) — caching for performance and reliability

- **Tailscale** — Zero-config VPN for secure remote access to all homelab services from anywhere.

**Wazuh User Agents**  
Lightweight agents deployed on homelab endpoints for log collection and security event forwarding back to the Wazuh manager.
