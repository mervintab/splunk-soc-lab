```

\# Tailscale Setup Guide



Tailscale provides zero-config VPN access to your Splunk server from any network,

including restrictive school and corporate networks that block traditional VPN protocols.



\## Why Tailscale Over Manual WireGuard



Feature          | Manual WireGuard | Tailscale

NAT traversal    | Manual/unreliable | Automatic

Key management   | Manual           | Automatic

New device setup | Edit config files | Install + sign in

Restricted nets  | Sometimes        | Yes (uses port 443 fallback)

Same network     | Conflicts        | Handled automatically

Cost             | Free             | Free (up to 100 devices)



\## Installation



\### On Vultr Server (Linux)



curl -fsSL https://tailscale.com/install.sh | sh

sudo tailscale up

tailscale ip



Note the IP shown - use it in your Claude Desktop config.



\### On Windows (Desktop/Laptop)



1\. Download from tailscale.com/download

2\. Install with default options

3\. Sign in with the SAME Google/GitHub account used on the server

4\. Verify connection at tailscale.com/admin



\## Allow Tailscale Through UFW



sudo ufw allow in on tailscale0

sudo ufw reload



\## Claude Desktop Config (Laptop)



Use your Tailscale IP instead of the public Vultr IP:



{

&#x20; "mcpServers": {

&#x20;   "splunk": {

&#x20;     "env": {

&#x20;       "SPLUNK\_HOST": "100.x.x.x"

&#x20;     }

&#x20;   }

&#x20; }

}



\## Usage at School or Away from Home



1\. Turn off Surfshark (conflicts with Tailscale)

2\. Tailscale connects automatically on startup

3\. Access Splunk at http://100.x.x.x:8000

4\. Claude Desktop MCP connects automatically



\## Troubleshooting



Problem                          | Solution

Won't connect                    | Turn off other VPNs (Surfshark etc.)

Connected but can't reach Splunk | Run: sudo ufw allow in on tailscale0

Tailscale IP changed             | Check tailscale.com/admin for current IP

School network blocking          | Tailscale auto falls back to TCP 443

