notepad C:\\Users\\mervi\\Projects\\splunk-soc-lab\\splunk\\setup.md

```

Click \*\*Yes\*\* to create new file.



\*\*Step 2 — Copy and paste this entire text into Notepad:\*\*

```

\# Splunk Enterprise Setup Guide



\## Installation



\# Create dedicated splunk user

sudo useradd -m -r -s /bin/bash splunk



\# Download Splunk Enterprise

cd /opt

sudo wget -O splunk.deb "https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498-linux-amd64.deb"



\# Install

sudo dpkg -i splunk.deb

sudo chown -R splunk:splunk /opt/splunk



\# Start and accept license

sudo -u splunk /opt/splunk/bin/splunk start --accept-license



\# Enable boot start

sudo -u splunk /opt/splunk/bin/splunk stop

sudo /opt/splunk/bin/splunk enable boot-start -user splunk -systemd-managed 1

sudo systemctl start Splunkd

sudo systemctl status Splunkd



\## Add Log Permissions



\# Add splunk user to adm group for log access

sudo usermod -aG adm splunk

sudo systemctl restart Splunkd



\## Configure Data Inputs



sudo -u splunk /opt/splunk/bin/splunk add monitor /var/log/auth.log -index main -sourcetype linux\_secure

sudo -u splunk /opt/splunk/bin/splunk add monitor /var/log/syslog -index main -sourcetype syslog

sudo -u splunk /opt/splunk/bin/splunk add monitor /var/log/ufw.log -index main -sourcetype ufw



\## Create MCP Agent User



1\. Log into Splunk UI at http://YOUR\_IP:8000

2\. Go to Settings > Users > New User

3\. Username: mcp\_agent

4\. Role: user (read-only)

5\. Uncheck Require password change on next login

6\. Click Save



\## Verify Data Ingestion



In Splunk Search and Reporting:



index=main | stats count by sourcetype



\## Useful SPL Queries



\### SSH Brute Force Detection

index=main sourcetype=linux\_secure "Failed password"

| rex "from (?P<src\_ip>\\d+\\.\\d+\\.\\d+\\.\\d+)"

| stats count by src\_ip

| sort -count



\### UFW Blocked Traffic

index=main sourcetype=ufw

| stats count by dest\_port

| sort -count



\### Successful Logins After Failed Attempts

index=main sourcetype=linux\_secure "Accepted password"

| rex "from (?P<src\_ip>\\d+\\.\\d+\\.\\d+\\.\\d+)"

| table \_time, src\_ip, user



\## Ports Used



Port 8000 - Splunk Web UI

Port 8089 - Splunk Management API (used by MCP)

Port 9997 - Universal Forwarder receiver



\## Free License Limits



\- 500 MB/day data ingestion

\- No expiration

\- All core features included

\- Sufficient for 1-5 device home lab

