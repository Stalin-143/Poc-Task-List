# Automated Security Auditing & Scripting

## 📌 Introduction
This report outlines an automated security auditing script written in Bash. The script performs the following tasks:

✅ Checks user login attempts using `last` and `/var/log/auth.log`  
✅ Detects running services using `systemctl list-units --type=service`  
✅ Monitors disk usage using `df -h`

Additionally, the script identifies weak configurations, demonstrates how attackers can exploit them, and implements mitigation techniques using `cron` jobs and security alerts.

---

## 🔧 Setup: Bash Script
The following Bash script performs automated security checks:

```bash
#!/bin/bash

# Log file for storing results
LOG_FILE="security_audit.log"

# Function to check user login attempts
echo "Checking user login attempts..." | tee -a $LOG_FILE
last | head -n 10 | tee -a $LOG_FILE

echo "Recent authentication failures (if any):" | tee -a $LOG_FILE
grep "Failed password" /var/log/auth.log | tail -n 10 | tee -a $LOG_FILE

# Function to check running services
echo "\nDetecting running services..." | tee -a $LOG_FILE
systemctl list-units --type=service --state=running | tee -a $LOG_FILE

# Function to monitor disk usage
echo "\nChecking disk usage..." | tee -a $LOG_FILE
df -h | tee -a $LOG_FILE

# Print completion message
echo "\nSecurity audit completed. Results saved in $LOG_FILE"
```

### 📌 Execution
1. Save the script as `security_audit.sh`.
2. Give execution permission:
   ```bash
   chmod +x security_audit.sh
   ```
3. Run the script:
   ```bash
   ./security_audit.sh
   ```

![Image](https://github.com/user-attachments/assets/c67eb4ea-617f-44d2-94a3-02321ccbbdb0)

   
4. View the results in `security_audit.log`:
   ```bash
   cat security_audit.log
   ```

![Image](https://github.com/user-attachments/assets/26e6ef21-c199-424b-8027-9554cc63c9e0)

---

## ⚠️ Exploiting Weak Configurations

### 1. **Weak User Accounts**
- If old or inactive user accounts exist, attackers can exploit them.
- Example command to list old user accounts:
  ```bash
  awk -F: '$3 < 1000 {print $1}' /etc/passwd
  ```
![Image](https://github.com/user-attachments/assets/58766ee1-5844-4473-9d06-8349a41bf2a2)
  
- Attackers may attempt brute force attacks on these accounts.

### 2. **Unnecessary Running Services**
- Services like `telnet`, `ftp`, or outdated web servers can be exploited.
- Example command to find such services:
  ```bash
  systemctl list-units --type=service --state=running
  ```
![Image](https://github.com/user-attachments/assets/ebe10d49-6880-42b7-9546-439431bba23b)
  
- If unnecessary services are running, disable them:
  ```bash
  sudo systemctl disable [service_name]
  ```

### 3. **Low Disk Space Issues**
- Attackers may exploit full disk conditions to disrupt system performance.
- Example check:
  ```bash
  df -h
  ```

  ![Image](https://github.com/user-attachments/assets/3cf48b8e-2624-4e23-90b6-ac5388043e5a)

---

## 🔒 Mitigation Strategies

### 1. **Automate System Monitoring with `cron`**
Add a `cron` job to run the script every day:
```bash
crontab -e
```
Add the following line at the end:
```bash
0 0 * * * /path/to/security_audit.sh
```

![Image](https://github.com/user-attachments/assets/19fb9432-9703-45f6-99f4-23615efe0d39)



### 2. **Email Notifications for Unauthorized SSH Logins**
To send an email alert when unauthorized SSH login attempts occur, modify the script:
```bash
#!/bin/bash
EMAIL="admin@example.com"
FAILED_LOGINS=$(grep "Failed password" /var/log/auth.log | tail -n 5)

if [[ ! -z "$FAILED_LOGINS" ]]; then
    echo -e "Unauthorized SSH login attempts detected:\n$FAILED_LOGINS" | mail -s "Security Alert" $EMAIL
fi
```

Install `mailutils` if not installed:
```bash
sudo apt install mailutils
```

---

## 📌 Conclusion
- The Bash script automates security auditing.
- Identifies weak configurations and possible exploits.
- Implements mitigation strategies using `cron` and email alerts.
- Regular monitoring ensures a secure system.

**Always update security measures to prevent attacks!** 🔐
