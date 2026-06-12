# Lab 02 — Splunk Universal Forwarder: Deploy & Troubleshoot

## Objective
Deploy Splunk Universal Forwarder on a Linux client, configure it to 
forward logs to the Splunk indexer, and practice real-world 
troubleshooting scenarios.

## Architecture

```
[linux-client: 192.168.119.132]
|
| Splunk UF (port 9997)
|
[splunk-server: 192.168.119.131]
|
| Splunk Web (port 8000)
|
[Browser: Search & Reporting]

```
## Environment
- Splunk Universal Forwarder 10.4.0 on Ubuntu 22.04 (linux-client)
- Splunk Enterprise 10.4.0 on Ubuntu 22.04 (splunk-server)
- Both VMs on VMware NAT network (192.168.119.0/24)

## What I Did

### 1. Built the Linux Client VM
- Ubuntu Server 22.04, 2GB RAM, 20GB disk
- Installed OpenSSH for remote access via VS Code

### 2. Installed Universal Forwarder
- Downloaded and installed Splunk UF 10.4.0 (.deb package)
- Created admin user via user-seed.conf
- Pointed forwarder at Splunk indexer: 192.168.119.131:9997
- Configured boot-start for automatic startup on reboot

### 3. Configured Log Inputs
Added two monitored paths:
- `/var/log/auth.log` — sourcetype: linux_auth
- `/var/log/syslog` — sourcetype: syslog

### 4. Verified Forwarding in Splunk
```spl
index=main host=linux-client | head 20
```
Confirmed 20 events from linux-client appearing in Splunk with 
correct host, source, and sourcetype fields.

### 5. Verified Forwarder Connectivity
```spl
index=_internal source=*metrics.log group=tcpin_connections
| stats count by hostname
```
Confirmed linux-client appearing in Splunk's internal telemetry 
with correct IP, port, and forwarder type (fwdType=uf).

## Troubleshooting Exercises

### Exercise 1 — Simulate Forwarder Outage
**Action:** Stopped the UF with `splunk stop`
**Detection:** linux-client disappeared from tcpin_connections metrics
**Log Evidence:** splunkd.log showed clean sequential shutdown of all 
pipelines ending with "Shutdown complete"
**Recovery:** Started UF, confirmed linux-client reappeared in metrics
**Key Lesson:** Clean shutdown shows ordered pipeline stops. A crash 
shows ERROR lines or abrupt end with no shutdown sequence.

### Exercise 2 — Bad inputs.conf Path
**Action:** Added a monitor stanza pointing to a non-existent file
**Detection:** grep for the path in splunkd.log showed:
`Adding watch on path: /var/log/this_file_doesnt_exist.log`
**Key Lesson:** Splunk silently watches bad paths without erroring. 
If a log source is missing from Splunk, always check inputs.conf 
for typos — Splunk will not alert you to the problem.

## Key Config Files
| File | Location | Purpose |
|------|----------|---------|
| inputs.conf | /opt/splunkforwarder/etc/system/local/ | What logs to monitor |
| outputs.conf | /opt/splunkforwarder/etc/system/local/ | Where to send logs |
| splunkd.log | /opt/splunkforwarder/var/log/splunk/ | UF internal diagnostics |
| user-seed.conf | /opt/splunkforwarder/etc/system/local/ | Initial admin credentials |

## Key Commands
```bash
# Start/stop/status
sudo /opt/splunkforwarder/bin/splunk start --run-as-root
sudo /opt/splunkforwarder/bin/splunk stop --run-as-root
sudo /opt/splunkforwarder/bin/splunk status --run-as-root

# Add forwarding destination
sudo /opt/splunkforwarder/bin/splunk add forward-server [IP]:9997 --run-as-root

# Add log monitor
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -index main -sourcetype linux_auth --run-as-root

# Check forwarder connectivity from Splunk
index=_internal source=*metrics.log group=tcpin_connections
| stats count by hostname

# Read UF diagnostic logs
sudo grep -i "warn\|error" /opt/splunkforwarder/var/log/splunk/splunkd.log | tail -15
```

## Screenshots
- lab02-uf-linux-forwarding.png — linux-client logs appearing in Splunk
- lab02-uf-recovery.png — forwarder reconnection confirmed in metrics

## Lessons Learned
- UF requires a local admin user before it can accept CLI commands
- Splunk silently monitors bad file paths — no error thrown
- splunkd.log is the first place to look when diagnosing UF issues
- Clean shutdown vs crash can be distinguished by reading shutdown 
  sequence in splunkd.log