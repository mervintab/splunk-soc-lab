\# UFW Firewall Rules



\## Default Policy



sudo ufw default deny incoming

sudo ufw default allow outgoing

sudo ufw default allow routed



\## Rules



Port  | Protocol | Source           | Purpose

22    | TCP      | Anywhere         | SSH access

51820 | UDP      | Anywhere         | WireGuard VPN (optional)

8000  | TCP      | YOUR\_HOME\_IP/32  | Splunk Web UI

8089  | TCP      | YOUR\_HOME\_IP/32  | Splunk Management API

9997  | TCP      | YOUR\_HOME\_IP/32  | Splunk Universal Forwarder



\## Commands



\# Set defaults

sudo ufw default deny incoming

sudo ufw default allow outgoing



\# Allow SSH

sudo ufw allow 22/tcp



\# Allow WireGuard (optional)

sudo ufw allow 51820/udp



\# Allow Splunk from home IP only

sudo ufw allow from YOUR\_HOME\_IP to any port 8000 proto tcp

sudo ufw allow from YOUR\_HOME\_IP to any port 8089 proto tcp

sudo ufw allow from YOUR\_HOME\_IP to any port 9997 proto tcp



\# Allow Tailscale interface

sudo ufw allow in on tailscale0



\# Enable

sudo ufw enable

sudo ufw status verbose



\## Vultr Cloud Firewall Rules



In addition to UFW, configure Vultr cloud firewall:



Protocol | Port  | Source          | Purpose

TCP      | 22    | 0.0.0.0/0       | SSH

UDP      | 51820 | 0.0.0.0/0       | WireGuard

TCP      | 8000  | YOUR\_HOME\_IP/32 | Splunk UI

TCP      | 8089  | YOUR\_HOME\_IP/32 | Splunk API



IMPORTANT: Make sure the firewall group is attached to your VM

in the Vultr dashboard under VM Settings > Firewall.



\## Notes



\- Replace YOUR\_HOME\_IP with your actual public IP (check at whatismyip.com)

\- If using Tailscale, Splunk ports do not need to be open to any public IP

\- If using Surfshark as a consistent exit IP, whitelist the Surfshark exit IP

\- Your home IP may change if your ISP uses dynamic IP addresses

\- Run: curl https://api.ipify.org to check your current public IP

