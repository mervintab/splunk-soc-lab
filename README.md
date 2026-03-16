# AI-Powered SOC Lab: Splunk + Claude MCP Integration

> A home lab project demonstrating how to build a cloud-hosted Security Operations Center (SOC) with AI-assisted threat hunting using Splunk Enterprise, Claude Desktop, and the Model Context Protocol (MCP).

---

## Project Overview

This project builds a fully functional, cloud-hosted SOC lab that allows a security analyst to query and investigate real security events using natural language through Claude AI. Instead of writing complex SPL queries manually, analysts can ask questions like:

- *"Show me the top 10 IPs brute-forcing my SSH server in the last 24 hours"*
- *"Are there any signs of a coordinated attack campaign in my logs?"*
- *"Give me a SOC analyst daily briefing based on today's events"*

And receive structured, actionable intelligence instantly.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Your Devices                         │
│                                                         │
│  ┌──────────────┐          ┌──────────────────────────┐ │
│  │   Desktop    │          │        Laptop            │ │
│  │  (Home)      │          │   (Home / School / Go)   │ │
│  │              │          │                          │ │
│  │ Claude       │          │ Claude Desktop           │ │
│  │ Desktop      │          │ + Tailscale VPN          │ │
│  │ + MCP Server │          │ + MCP Server             │ │
│  └──────┬───────┘          └────────────┬─────────────┘ │
└─────────┼──────────────────────────────┼───────────────┘
          │ Direct IP                    │ Tailscale VPN
          │ (Home network)               │ (Any network)
          ▼                              ▼
┌─────────────────────────────────────────────────────────┐
│                   Vultr Cloud VM                         │
│                  Ubuntu 22.04 LTS                        │
│                                                         │
│  ┌─────────────────────┐   ┌────────────────────────┐  │
│  │  Splunk Enterprise  │   │      Tailscale VPN     │  │
│  │  - Port 8000 (UI)   │   │   (Secure Remote       │  │
│  │  - Port 8089 (API)  │   │    Access)             │  │
│  └─────────────────────┘   └────────────────────────┘  │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              UFW Firewall                        │   │
│  │  + Vultr Cloud Firewall                         │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Component | Technology |
|---|---|
| Cloud Provider | Vultr |
| OS | Ubuntu 22.04 LTS |
| SIEM | Splunk Enterprise (Free License) |
| AI Assistant | Claude Desktop (Pro) |
| AI Integration | Model Context Protocol (MCP) |
| MCP Server | chalithah/splunk-claude-mcp-agent |
| VPN (Remote) | Tailscale |
| Firewall | UFW + Vultr Cloud Firewall |

---

## Features

- **Natural Language Threat Hunting** — Query Splunk using plain English through Claude Desktop
- **Real Attack Data** — Ingests live SSH brute force, firewall blocks, and system logs
- **Multi-Device Access** — Desktop (direct IP) and laptop (Tailscale VPN) support
- **Secure by Design** — No Splunk ports exposed to internet; all access via VPN or IP whitelist
- **AI-Powered Analysis** — Claude identifies attack patterns, correlates events, and generates SOC briefings

---

## Prerequisites

- Vultr account (or any cloud provider)
- Claude Desktop with Pro subscription (required for MCP support)
- Python 3.10+ on local machines
- Windows 10/11 on local machines
- Tailscale account (free tier sufficient)

---

## Quick Start

### 1. Deploy the Vultr VM

- Ubuntu 22.04 LTS
- 2 vCPU / 8 GB RAM / 160 GB NVMe
- New York region (or closest to you)

### 2. Harden the Server

```bash
# Create non-root user
adduser yourname
usermod -aG sudo yourname

# Configure UFW firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 51820/udp
sudo ufw enable
```

### 3. Install Splunk

```bash
# Create splunk user
sudo useradd -m -r -s /bin/bash splunk

# Download and install
cd /opt
sudo wget -O splunk.deb "https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498-linux-amd64.deb"
sudo dpkg -i splunk.deb
sudo chown -R splunk:splunk /opt/splunk

# Start Splunk
sudo -u splunk /opt/splunk/bin/splunk start --accept-license
sudo /opt/splunk/bin/splunk enable boot-start -user splunk -systemd-managed 1
```

### 4. Configure Data Inputs

```bash
sudo -u splunk /opt/splunk/bin/splunk add monitor /var/log/auth.log -index main -sourcetype linux_secure
sudo -u splunk /opt/splunk/bin/splunk add monitor /var/log/syslog -index main -sourcetype syslog
sudo -u splunk /opt/splunk/bin/splunk add monitor /var/log/ufw.log -index main -sourcetype ufw
```

### 5. Set Up Tailscale (Remote Access)

```bash
# On Vultr server
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
tailscale ip  # Note this IP for client config
```

Install Tailscale on your laptop from **tailscale.com/download** and sign in with the same account.

### 6. Set Up MCP Server (Windows)

```powershell
# Clone the MCP server
mkdir C:\Users\$env:USERNAME\Projects
cd C:\Users\$env:USERNAME\Projects
git clone https://github.com/chalithah/splunk-claude-mcp-agent.git
cd splunk-claude-mcp-agent
pip install -r requirements.txt
```

### 7. Configure Claude Desktop

Create/edit the config file at:
```
%LOCALAPPDATA%\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\claude_desktop_config.json
```

Use the template from `mcp-server/claude_desktop_config.example.json` and fill in your credentials.

---

## Sample Threat Hunting Queries

Once connected, try these in Claude Desktop:

```
Show me the top 10 IPs attempting to brute force SSH in the last 24 hours
```

```
Are there any coordinated attack campaigns visible in the auth logs?
```

```
Give me a SOC analyst daily briefing based on today's security events
```

```
Check if any of the attacking IPs successfully logged in after their failed attempts
```

```
What ports are being targeted most by the UFW firewall blocks?
```

---

## Real-World Results

Within 24 hours of deployment this lab detected:

- **354 SSH brute force attempts** from 42 unique IP addresses
- **Coordinated campaign** from `80.94.92.0/24` subnet (5 IPs, 239 attempts)
- **Secondary campaign** from `2.57.122.0/24` subnet (3 IPs)
- Both subnets blocked via UFW after AI-assisted analysis

---

## Security Considerations

- Never commit real credentials, IPs, or private keys to this repository
- Use environment variables for all sensitive configuration
- Create a dedicated read-only `mcp_agent` Splunk user for MCP access
- Keep Tailscale and Surfshark VPNs from running simultaneously
- Regularly update Splunk and system packages

---

## Repository Structure

```
splunk-soc-lab/
├── README.md
├── architecture/
│   └── architecture.md
├── splunk/
│   └── setup.md
├── tailscale/
│   └── setup.md
├── mcp-server/
│   └── claude_desktop_config.example.json
├── firewall/
│   └── ufw-rules.md
└── screenshots/
```

---

## Credits

- MCP Server: [chalithah/splunk-claude-mcp-agent](https://github.com/chalithah/splunk-claude-mcp-agent)
- Splunk Enterprise: [splunk.com](https://www.splunk.com)
- Claude Desktop: [claude.ai](https://claude.ai)
- Tailscale: [tailscale.com](https://tailscale.com)

---

## License

MIT License — feel free to use this as a reference for your own SOC lab.

---

## Author

Built as a portfolio project for an aspiring cybersecurity analyst.
Demonstrates practical skills in: cloud infrastructure, SIEM deployment, AI integration, network security, and threat hunting.
