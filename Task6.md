# Log Analysis & Intrusion Detection 

## **1. Setup**

### **Enabling System Logging **

Kali Linux uses `systemd` for logging instead of `/var/log/auth.log` by default. Ensure logging is enabled:

```bash
sudo systemctl status systemd-journald
```

If it’s not running, enable and start it:

```bash
sudo systemctl enable --now systemd-journald
```

To persist logs across reboots, modify `/etc/systemd/journald.conf`:

```bash
sudo nano /etc/systemd/journald.conf
```

Ensure the following lines are set:

```
Storage=persistent
```

Then restart the service:

```bash
sudo systemctl restart systemd-journald
```
![Image](https://github.com/user-attachments/assets/2ed3161d-aa84-4f29-8408-7439499915d4)


#### **Checking SSH Logs on Kali Linux**

To monitor SSH authentication logs:

```bash
sudo journalctl -u ssh --no-pager
```

For real-time monitoring:

```bash
sudo journalctl -u ssh -f
```

If `/var/log/auth.log` exists, verify SSH logs using:

```bash
sudo tail -f /var/log/auth.log
```
![Image](https://github.com/user-attachments/assets/1cc74db8-1aac-41e7-b959-e4aa0f67bc72)

---

## **2. Exploit - Simulating Multiple Failed SSH Login Attempts**

To simulate a brute-force attack:

```bash
for i in {1..10}; do ssh invaliduser@localhost; done
```

This will generate failed SSH attempts recorded in the system logs.

### **Analyzing Logs for Failed SSH Attempts**

If logs are stored in `journalctl`, analyze failures using:

```bash
sudo journalctl -u ssh | grep "Failed password"
```

Or if `/var/log/auth.log` exists:

```bash
grep "Failed password" /var/log/auth.log
```

Example output:

```
Mar 25 12:34:56 kali sshd[12345]: Failed password for invaliduser from 192.168.1.100 port 54721 ssh2
Mar 25 12:34:57 kali sshd[12346]: Failed password for invaliduser from 192.168.1.100 port 54722 ssh2
```

![Image](https://github.com/user-attachments/assets/2d05eaef-f1a7-467e-a003-4c66ff2fa0f8)



#### **Detecting Brute-force Attacks**

To count failed attempts per IP:

```bash
sudo journalctl -u ssh | grep "Failed password" | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
```

Example output:

```
10 192.168.1.100
5  192.168.1.101
```

![Image](https://github.com/user-attachments/assets/b2f239f6-747a-4e33-bef8-6f41615d5f2e)

This helps identify brute-force attempts.

---

## **3. Mitigation Measures**

### **Installing and Configuring Fail2Ban on Kali Linux**

Fail2Ban prevents brute-force attacks by banning IPs after repeated failures.

#### **Install Fail2Ban**

```bash
sudo apt update && sudo apt install fail2ban -y
```

#### **Configure SSH Protection**

Edit the jail configuration file:

```bash
sudo nano /etc/fail2ban/jail.local
```

Add the following:

```
[sshd]
enabled = true
port = ssh
maxretry = 3
findtime = 600
bantime = 3600
logpath = %(syslog_auth)s
```

Restart fail2ban:

```bash
sudo systemctl restart fail2ban
```
![Image](https://github.com/user-attachments/assets/979406a4-d656-4e50-a02c-ee19ce564f38)


Check banned IPs:

```bash
sudo fail2ban-client status sshd
```

### **Setting up Log Monitoring with Logwatch**

Logwatch provides automated log analysis.

```bash
sudo apt install logwatch -y
```

Generate a report:

```bash
sudo logwatch --detail high --mailto admin@example.com
```

### **Setting up Rsyslog for Remote Logging**

Enable Rsyslog:

```bash
sudo systemctl enable --now rsyslog
```

Edit `/etc/rsyslog.conf` to enable remote logging:

```
*.* @192.168.1.200:514
```

Restart Rsyslog:

```bash
sudo systemctl restart rsyslog
```

---

## **4. Conclusion**

- Logs were successfully analyzed for failed SSH attempts.
- Brute-force attacks were detected and mitigated.
- Fail2Ban was deployed to block repeated login failures.
- Logwatch and Rsyslog were set up for automated log monitoring.
- `journalctl` was used in place of missing traditional log files.

By implementing these measures, system security is strengthened against intrusion attempts.

