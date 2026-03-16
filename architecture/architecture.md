```

\# Architecture \& Design Decisions



\## Network Architecture

```

┌─────────────────────────────────────────────────────────────┐

│                        Internet                              │

└──────────────────────────┬──────────────────────────────────┘

&#x20;                          │

&#x20;             ┌────────────▼───────────┐

&#x20;             │    Vultr Cloud VM       │

&#x20;             │    104.x.x.x (Public)   │

&#x20;             │    100.x.x.x (Tailscale)│

&#x20;             │                         │

&#x20;             │  ┌─────────────────┐   │

&#x20;             │  │ UFW Firewall    │   │

&#x20;             │  │ + Vultr FW      │   │

&#x20;             │  └────────┬────────┘   │

&#x20;             │           │             │

&#x20;             │  ┌────────▼────────┐   │

&#x20;             │  │Splunk Enterprise│   │

&#x20;             │  │ :8000 Web UI    │   │

&#x20;             │  │ :8089 API       │   │

&#x20;             │  └─────────────────┘   │

&#x20;             │                         │

&#x20;             │  ┌─────────────────┐   │

&#x20;             │  │   Tailscale     │   │

&#x20;             │  │   tailscale0    │   │

&#x20;             │  └─────────────────┘   │

&#x20;             └─────────────────────────┘

&#x20;                   ▲           ▲

&#x20;         Direct IP │           │ Tailscale VPN

&#x20;         (Home)    │           │ (Any Network)

&#x20;                   │           │

&#x20;        ┌──────────┴──┐  ┌────┴──────────┐

&#x20;        │   Desktop   │  │    Laptop     │

&#x20;        │  (Home)     │  │  (Anywhere)  │

&#x20;        │             │  │              │

&#x20;        │ Claude      │  │ Claude       │

&#x20;        │ Desktop     │  │ Desktop      │

&#x20;        │ MCP Server  │  │ MCP Server   │

&#x20;        └─────────────┘  └──────────────┘

```



\## Component Descriptions



\### Vultr Cloud VM

\- OS: Ubuntu 22.04 LTS

\- Specs: 2 vCPU, 8 GB RAM, 160 GB NVMe

\- Cost: \~$48/month

\- Purpose: Hosts Splunk and provides always-on log collection



\### Splunk Enterprise

\- Version: 9.3.2

\- License: Free (500 MB/day)

\- User: Runs as dedicated splunk system user (not root)

\- Data Sources: auth.log, syslog, ufw.log



\### Model Context Protocol (MCP) Server

\- Project: chalithah/splunk-claude-mcp-agent

\- Language: Python

\- Mode: stdio (Claude Desktop spawns as child process)

\- Auth: Dedicated read-only mcp\_agent Splunk user



\### Tailscale VPN

\- Purpose: Secure remote access from laptop on any network

\- Benefit: Works on restrictive networks via port 443 fallback

\- Replaces: Manual WireGuard (NAT traversal issues)



\## Security Design Decisions



\### Why a Dedicated MCP User?

The mcp\_agent Splunk user has read-only user role. If credentials

are ever compromised, an attacker can only read data, not modify

Splunk configuration or delete indexes.



\### Why Tailscale Over WireGuard?

Manual WireGuard failed due to NAT traversal issues on mobile networks

and school networks. Tailscale handles this automatically and works

even when UDP is blocked (falls back to TCP 443).



\### Why Not Open Splunk to the Internet?

Splunk is not hardened for direct internet exposure. Keeping it

behind a firewall/VPN significantly reduces attack surface.



\### Why Separate Vultr Cloud Firewall + UFW?

Defense in depth — two independent firewall layers. If one

misconfiguration occurs, the other still provides protection.



\## Data Flow



1\. Log events generated on Vultr VM (SSH attempts, syslog, UFW blocks)

2\. Splunk monitors /var/log/\* files

3\. Events indexed in Splunk main index

4\. Analyst asks Claude a question in natural language

5\. Claude calls MCP server tool (search\_splunk)

6\. MCP server authenticates to Splunk API as mcp\_agent

7\. SPL query executed against Splunk index

8\. Results returned to MCP server

9\. Claude analyzes results and responds in natural language



\## Lessons Learned



1\. Vultr Cloud Firewall must be attached to VM — rules do not apply until linked

2\. Microsoft Store Claude Desktop uses different config path than direct install

3\. Two VPNs conflict — Surfshark and Tailscale cannot run simultaneously

4\. Splunk force-password-change must be disabled for service accounts

5\. splunk user needs adm group membership to read system log files

