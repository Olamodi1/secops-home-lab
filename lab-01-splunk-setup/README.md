# Lab 01 — Splunk Enterprise Installation & Log Ingestion

## Objective
Install Splunk Enterprise on Ubuntu Server 22.04, configure initial 
log ingestion from Linux system logs, and verify data is searchable 
in the SIEM.

## Environment
- Splunk Enterprise 10.4.0 on Ubuntu Server 22.04 LTS
- VMware Workstation Player
- Host IP: 192.168.119.131

## What I Did

### 1. Built the Ubuntu Server VM
- 50GB disk, 4GB RAM, 2 vCPUs
- Installed OpenSSH server during setup for remote access
- Connected VS Code via Remote-SSH extension

### 2. Installed Splunk Enterprise
- Downloaded Splunk Enterprise 10.4.0 (.deb package)
- Installed via dpkg
- Started Splunk and configured boot-start so it survives reboots

### 3. Configured Log Inputs
Added two monitored log sources:
- `/var/log/auth.log` — captures authentication, sudo usage, session events
- `/var/log/syslog` — captures general system events

### 4. Configured Receiving Port
- Enabled TCP port 9997 to receive forwarded data from Universal Forwarders
- This prepares Splunk to accept logs from Windows and Linux clients in Lab 02

### 5. Verified Log Ingestion
Ran first SPL query and confirmed 20 events ingested from auth.log

## Key Commands Used

```bash
# Install Splunk
sudo dpkg -i splunk-10.4.0-f798d4d49089-linux-amd64.deb

# Start Splunk
sudo /opt/splunk/bin/splunk start --accept-license --run-as-root

# Enable boot start
sudo /opt/splunk/bin/splunk enable boot-start --run-as-root

# Add log monitors
sudo /opt/splunk/bin/splunk add monitor /var/log/auth.log -index main -sourcetype linux_auth --run-as-root
sudo /opt/splunk/bin/splunk add monitor /var/log/syslog -index main -sourcetype syslog --run-as-root

# Enable receiving port for Universal Forwarders
sudo /opt/splunk/bin/splunk enable listen 9997 --run-as-root
```

## First SPL Query
```splunk
index=main sourcetype=linux_auth | head 20
```

## Results
- 20 auth events ingested and searchable
- Splunk automatically extracted fields: COMMAND, TTY, PWD, USER, host
- Sudo commands, session opens/closes all captured with timestamps

## Screenshots
- lab01-splunk-homepage.png — Splunk web UI after first login
- lab01-splunk-first-search.png — First successful SPL search showing auth events

## Lessons Learned
- Splunk 10.x requires --run-as-root flag when running as sudo
- Auth.log captures extremely detailed activity including exact commands run via sudo
- Field extraction happens automatically — Splunk parsed linux_auth sourcetype 
  without any manual configuration