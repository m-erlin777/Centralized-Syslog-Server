# Centralized-Syslog-Server
This project configures a Raspberry Pi 5 as a centralized syslog server to receive Windows Event Logs forwarded by NXLog over TCP. Logs are stored dynamically by hostname to facilitate organized log management. The setup highlights networking, Linux administration, Windows log forwarding, security hardening, and troubleshooting skills.

---

## Prerequisites

- Raspberry Pi 5 running Raspberry Pi OS (64-bit)
- Windows client machine with admin rights
- Local network connectivity on the same subnet (e.g., 192.168.X.XX/22)
- Basic familiarity with Linux shell and Windows PowerShell

---

## Step 1: Network Setup and SSH Configuration on Raspberry Pi

- Configured a static IP on the Pi: `192.168.X.XX`
- Enabled and hardened SSH:
- Disabled password authentication (`PasswordAuthentication no`)
- Allowed SSH access for user `username` only (`AllowUsers username`)
- Generated and configured SSH key authentication (`ssh-keygen`)
- Verified network connectivity from Windows client:
  ```
  ping 192.168.X.XX
  ssh username@192.168.X.XX

## Step 2: Installing and Configuring rsyslog on Raspberry Pi

- Installed rsyslog:
  `sudo apt update && sudo apt install rsyslog -y`
- Created TCP input configuration `/etc/rsyslog.d/windows-tcp.conf`:
  ```
  module(load="imtcp")
  input(type="imtcp" port="514")

  template(name="WindowsLogs" type="string" string="/var/log/windows_logs/%HOSTNAME%.log")

  ruleset(name="windows") {
    action(type="omfile" dynaFile"WindowsLog")
  }

  input(type="imtcp" port="514" ruleset="windows")
- Created log directory and set ownership to username:
  ```
  sudo mkdir -p /var/log/windows_logs
  sudo chown username:username /var/log/windows_logs
- Restarted rsyslog service and verified it listened on TCP port 514:
  ```
  sudo systemctl restart rsyslog
  sudo ss -tulnp | grep 514

## Step 3: Configuring NXLog on Windows Client

- Installed NXLog CE
- Edited `nxlog.conf` to forward logs via TCP to Pi server
  ```
  <Output out>
    Module  om_tcp
    Host    192.168.X.XX
    Port    514
    Exec    to_syslog_snare();
  </Output>

  <Route 1>
    Path    in => out
  </Route>
- Opened Windows Firewall outbound TCP port 514 and ensured Pi firewall allowed inbound connections
- Restarted NXLog service via PowerShell `Restart-Service nxlog`

## Step 4: Vertification and Troubleshooting

- Diagnosed network issues including:
  - VPN interface on Windows client, allowed local network sharing
  - Subnet mask and multiple IP addresses on Pi corrected to ensure communication
- Corrected file permission issues (chmod + chown) for username
- Verified SSH, ping, and TCP connectivity between Windows client and Pi server
- Verified log reception on Pi server
  `sudo tail -f /var/log/windows_logs/*.log`

## Step 5: Log Management and Automation

- Cleared logs without deleting file
  `sudo find /var/log/windows_logs/ -type f -name "*.log" -exec truncate -s 0 {} \;`
- Setup weekly cron job to automate log truncation every Sunday at 3 AM
  ```
  sudo crontab -e
  0 3 * * 0 /usr/bin/find /var/log/windows_logs/ -type f -name "*.log" -exec truncate -s 0 {} \;
- Monitored system resource usage for stability

---

# Skills Demonstrated

- Network configuration and troubleshooting
- Secure SSH hardening
- Rsyslog configuration and multi-host log collection
- Windows Event forwarding with NXLog
- Permissions and file system management
- Log rotation automation with cron
- Cross-platform troubleshooting and system administration
  




















  
  
