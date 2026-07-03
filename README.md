# SOC & Honeypot Lab

**Author:** Domenico Pisicoli

**Project Status:** 🟢 Active

**Last Updated:** 2026-07-01

**Stack:** Cowrie SSH/Telnet Honeypot (Oracle Cloud) → Wazuh SIEM (local, self-hosted) → Evidence-based incident analysis

## About This Project

This repository documents a self-built, ongoing home lab combining a **Cowrie SSH/Telnet honeypot** (exposed on Oracle Cloud Infrastructure) with a **Wazuh SIEM stack** (running locally, connected over a Tailscale encrypted overlay network) to capture, detect, and analyze real, unsolicited attack traffic from the internet — no simulated or synthetic data.

The goal is twofold:

1. Build hands-on SOC / detection-engineering experience end-to-end — deployment, log ingestion, custom detection rules, and analysis — not just a checklist of tools.
2. Produce evidence-based incident reporting that clearly separates raw observation from interpretation from assessed confidence, the way a working SOC analyst would.

## Responsible Use & Ethics

This honeypot runs in an isolated Docker environment on infrastructure I own and control (Oracle Cloud Always Free tier), exposed solely to capture unsolicited, real-world attack traffic for defensive research and learning. No systems outside this isolated lab are targeted, scanned, or interacted with. All source IPs analyzed in this repository belong to inbound connections initiated by external parties against the honeypot — none were contacted by the lab. One source IP identified as the analyst's own testing traffic has been redacted from published reports as a matter of OPSEC practice.

## 📁 Incident Reports

Each dated report below documents one analysis session over the honeypot's captured traffic — statistics, detection engineering notes, and individually evidenced incidents with MITRE ATT&CK mapping and a confidence level attached to every conclusion.

- **2026-06-29 — First analysis batch (5 incidents):** credential-guessing compromises, an SSH tunneling attempt toward Cloudflare infrastructure, automated scanning, and a self-identified false positive → [read the report](./incident-reports/2026-06-29-analysis.md)
- **2026-07-01 — Second analysis batch (6 incidents, 6-day window):** a replayed multi-architecture malware-dropper campaign with SSH persistence, a 25-second automated backdoor deployment chain, and two high-volume automated credential-stuffing bots → [read the report](./incident-reports/2026-07-01-analysis.md)

*(Future reports will be added here as new traffic is captured and analyzed.)*

## Architecture

Infrastructure follows a Zero-Trust isolation principle, decoupling the public-facing attack surface from the analysis engine via an encrypted tunnel.

**Cloud Environment (Exposed Segment)**

- Oracle Cloud Infrastructure (OCI) VPS
- Ubuntu Server + Docker
- Cowrie SSH/Telnet Honeypot

**Monitoring Environment (Local/On-Premise)**

- Local Hypervisor (UTM on macOS)
- Ubuntu Server VM (wazuh-docker / single-node stack: manager, indexer, dashboard)

```
   Internet
       │
       ▼
+-----------------------+
| Oracle Cloud VPS       |
| (Public IP: Exposed)   |
| 1. Docker              |
| 2. Cowrie Honeypot     |
| 3. Wazuh Agent (007)   |
+-----------------------+
       │
   Tailscale overlay tunnel
       │
       ▼
+-----------------------+
| Local Ubuntu VM        |
| (Private IP: NAT)      |
| 1. Wazuh Manager       |
| 2. Wazuh Indexer       |
| 3. Wazuh Dashboard     |
+-----------------------+
```

Agent traffic between the honeypot and the local Wazuh manager is carried over a Tailscale encrypted overlay network, keeping the SIEM itself off the public internet.

## Hands-On Experience

This is what I actually did building and analyzing this lab — described as concrete actions rather than skill labels, since I'm still early in my career and would rather be precise than oversell:

- Deployed and administered a Wazuh SIEM stack (manager, indexer, dashboard) via Docker Compose, including agent enrollment and troubleshooting log ingestion issues along the way
- Wrote custom XML decoders and detection rules for Cowrie's JSON telemetry, tagged with MITRE ATT&CK technique IDs
- Manually correlated raw alert data at the field level (session IDs, SSH client fingerprints, embedded timestamps) to reconstruct what happened in each attacker session
- Structured incident write-ups that separate what the logs actually show from my interpretation of it, with an honest confidence level attached to each conclusion
- Mapped observed attacker behavior to MITRE ATT&CK techniques and tactics, only where the evidence directly supported it
- Caught and down-graded a likely false positive in my own data instead of reporting it as a real compromise, and redacted personal source IPs before publishing
- Set up the underlying infrastructure: OCI free-tier provisioning, Docker, UTM virtualization, Tailscale networking

Feedback and detection-engineering suggestions are welcome via GitHub Issues.
