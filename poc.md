# Linux & Networking Lab Handout - Arch Linux
**Date:** 2025-07-04  
**Instructor:** [

## Objective
This lab aims to familiarize students with essential Linux commands, networking tools, and scripting techniques through hands-on tasks. By completing these tasks, students will gain foundational skills for system administration and automation on Arch Linux.

---

## Task 1: Linux Essentials & File Permissions

### Instructions:
1. Create a new user called 'studentuser'
2. Create the following directory structure: `/home/studentuser/projectX/logs` and `/home/studentuser/projectX/scripts`
3. Create a file 'welcome.txt' in 'projectX' with the content 'Welcome to Linux'
4. Set permissions so only 'studentuser' can read/write the file
5. Create a script 'backup.sh' in 'scripts' that copies 'welcome.txt' to 'logs' with a timestamp

### Solution:
```bash
# Create new user (run as root/sudo)
sudo useradd -m -s /bin/bash studentuser
sudo passwd studentuser

# Create directory structure as root first
sudo mkdir -p /home/studentuser/projectX/{logs,scripts}

# Create welcome.txt file as root
sudo bash -c 'echo "Welcome to Linux" > /home/studentuser/projectX/welcome.txt'

# Change ownership to studentuser
sudo chown -R studentuser:studentuser /home/studentuser/projectX

# Set permissions (owner read/write only)
sudo chmod 600 /home/studentuser/projectX/welcome.txt

# Create backup script as root
sudo tee /home/studentuser/projectX/scripts/backup.sh > /dev/null << 'EOF'
#!/bin/bash
timestamp=$(date +"%Y%m%d_%H%M%S")
cp /home/studentuser/projectX/welcome.txt /home/studentuser/projectX/logs/welcome_${timestamp}.txt
echo "Backup completed: welcome_${timestamp}.txt"
EOF

# Make script executable and set proper ownership
sudo chmod +x /home/studentuser/projectX/scripts/backup.sh
sudo chown studentuser:studentuser /home/studentuser/projectX/scripts/backup.sh

# Test the backup script
sudo -u studentuser /home/studentuser/projectX/scripts/backup.sh

# Alternative: Switch to studentuser after setup
# sudo su - studentuser
# ./projectX/scripts/backup.sh
```
<img width="920" height="446" alt="Image" src="https://github.com/user-attachments/assets/d38f193e-0fb5-4c7d-adff-ffa821b80eca" />

<img width="950" height="422" alt="Image" src="https://github.com/user-attachments/assets/5d16f2e0-0014-4475-a3a9-e42c62418d3a" />

### Alternative Solution (Working as studentuser):
```bash
# After creating user, switch to studentuser
sudo su - studentuser

# Now create directories (this will work since we're in the user's home)
mkdir -p ~/projectX/{logs,scripts}

# Create welcome.txt file
echo "Welcome to Linux" > ~/projectX/welcome.txt

# Set permissions (owner read/write only)
chmod 600 ~/projectX/welcome.txt

# Create backup script
cat > ~/projectX/scripts/backup.sh << 'EOF'
#!/bin/bash
timestamp=$(date +"%Y%m%d_%H%M%S")
cp ~/projectX/welcome.txt ~/projectX/logs/welcome_${timestamp}.txt
echo "Backup completed: welcome_${timestamp}.txt"
EOF

# Make script executable
chmod +x ~/projectX/scripts/backup.sh

# Test the backup script
~/projectX/scripts/backup.sh
```
<img width="786" height="332" alt="Image" src="https://github.com/user-attachments/assets/3d482b6f-cff9-4282-ac67-5b9ff79ebdcb" />

---

## Task 2: Networking Toolkit PoC

### Instructions:
Write a script 'netinfo.sh' that:
- Displays IP address, subnet mask, and default gateway
- Lists open ports using 'ss' or 'netstat'
- Pings google.com and logs the response
- Resolves the IP address of 'openai.com'
- Store the output in 'network_report.txt'

### Solution:
```bash
# Create netinfo.sh script
cat > /home/studentuser/projectX/scripts/netinfo.sh << 'EOF'
#!/bin/bash

echo "=== Network Information Report ===" > network_report.txt
echo "Generated on: $(date)" >> network_report.txt
echo "" >> network_report.txt

echo "=== IP Configuration ===" >> network_report.txt
ip addr show | grep -E "inet.*scope global" >> network_report.txt
echo "" >> network_report.txt

echo "=== Default Gateway ===" >> network_report.txt
ip route | grep default >> network_report.txt
echo "" >> network_report.txt

echo "=== Open Ports ===" >> network_report.txt
ss -tuln >> network_report.txt
echo "" >> network_report.txt

echo "=== Ping Google.com ===" >> network_report.txt
ping -c 4 google.com >> network_report.txt 2>&1
echo "" >> network_report.txt

echo "=== DNS Lookup for openai.com ===" >> network_report.txt
nslookup openai.com >> network_report.txt 2>&1

echo "Network report saved to network_report.txt"
EOF

# Make executable and run
chmod +x /home/studentuser/projectX/scripts/netinfo.sh
cd /home/studentuser/projectX/scripts
./netinfo.sh
```
<img width="886" height="726" alt="Image" src="https://github.com/user-attachments/assets/dba4e5c0-d23f-49d2-83d7-18ab8734460b" />
<img width="901" height="970" alt="Image" src="https://github.com/user-attachments/assets/2da9a9b2-51ff-4f58-951d-06ac5c836fc1" />
<img width="845" height="378" alt="Image" src="https://github.com/user-attachments/assets/06614da7-aff7-4fc5-a7cc-5b887bb6de4d" />



---

## Task 3: Mini Server Monitor Script

### Instructions:
Create a script 'monitor.sh' that:
- Checks if 'nginx' is running and starts it if not
- Displays memory usage, CPU load, and disk usage
- Logs all activities with timestamps
- Schedule it via cron to run every 5 minutes

### Solution:
```bash
# Install nginx if not present
sudo pacman -S nginx

# Create monitor script
cat > /home/studentuser/projectX/scripts/monitor.sh << 'EOF'
#!/bin/bash

LOGFILE="/home/studentuser/projectX/logs/monitor.log"
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

echo "[$TIMESTAMP] === System Monitor Report ===" >> $LOGFILE

# Check nginx status
if systemctl is-active --quiet nginx; then
    echo "[$TIMESTAMP] nginx is running" >> $LOGFILE
else
    echo "[$TIMESTAMP] nginx is not running, attempting to start..." >> $LOGFILE
    sudo systemctl start nginx
    if systemctl is-active --quiet nginx; then
        echo "[$TIMESTAMP] nginx started successfully" >> $LOGFILE
    else
        echo "[$TIMESTAMP] Failed to start nginx" >> $LOGFILE
    fi
fi

# Memory usage
echo "[$TIMESTAMP] Memory Usage:" >> $LOGFILE
free -h >> $LOGFILE

# CPU load
echo "[$TIMESTAMP] CPU Load:" >> $LOGFILE
uptime >> $LOGFILE

# Disk usage
echo "[$TIMESTAMP] Disk Usage:" >> $LOGFILE
df -h >> $LOGFILE

echo "[$TIMESTAMP] === End Report ===" >> $LOGFILE
echo "" >> $LOGFILE
EOF

# Make executable
chmod +x /home/studentuser/projectX/scripts/monitor.sh

# Add to crontab (run every 5 minutes)
(crontab -l 2>/dev/null; echo "*/5 * * * * /home/studentuser/projectX/scripts/monitor.sh") | crontab -
```

<img width="1896" height="584" alt="Image" src="https://github.com/user-attachments/assets/bf6dfcd4-d57b-4ee7-a4f8-e10e2e3fddb3" />

<img width="881" height="654" alt="Image" src="https://github.com/user-attachments/assets/abfa90a5-d28c-40da-9378-34eb90ed8f42" />

<img width="1892" height="623" alt="Image" src="https://github.com/user-attachments/assets/5f8378e7-394b-4212-ac40-b1719c081a5b" />

---

## Task 4: File Watcher Script

### Instructions:
Write a script 'watch_dir.sh' that watches '/home/studentuser/projectX/logs' for new '.txt' files.
When a new file is created, echo its name with a timestamp to 'log_monitor.txt'.

### Solution:
```bash
# Install inotify-tools if not present
sudo pacman -S inotify-tools

# Create file watcher script
cat > /home/studentuser/projectX/scripts/watch_dir.sh << 'EOF'
#!/bin/bash

WATCH_DIR="/home/studentuser/projectX/logs"
LOG_FILE="/home/studentuser/projectX/log_monitor.txt"

echo "Starting file watcher for $WATCH_DIR"
echo "File watcher started at $(date)" >> $LOG_FILE

inotifywait -m -e create --format '%w%f' $WATCH_DIR |
while read file; do
    if [[ $file == *.txt ]]; then
        timestamp=$(date "+%Y-%m-%d %H:%M:%S")
        echo "[$timestamp] New file created: $(basename $file)" >> $LOG_FILE
        echo "[$timestamp] New file detected: $(basename $file)"
    fi
done
EOF

# Make executable
chmod +x /home/studentuser/projectX/scripts/watch_dir.sh

# Run in background
# /home/studentuser/projectX/scripts/watch_dir.sh &
```
<img width="1905" height="429" alt="Image" src="https://github.com/user-attachments/assets/477ddbcc-5d5b-4272-aba4-2639bfce417a" />

<img width="890" height="447" alt="Image" src="https://github.com/user-attachments/assets/cbe3f620-5bf1-4e00-9f8d-dda2a818a087" />

---

## Task 5: SSH Login Audit

### Instructions:
Parse '/var/log/auth.log' or use 'journalctl' to:
- Show last 5 successful logins
- Show last 5 failed login attempts
- Save results to 'ssh_audit.txt'

### Solution:
```bash
# Create SSH audit script
cat > /home/studentuser/projectX/scripts/ssh_audit.sh << 'EOF'
#!/bin/bash

AUDIT_FILE="/home/studentuser/projectX/ssh_audit.txt"

echo "=== SSH Login Audit Report ===" > $AUDIT_FILE
echo "Generated on: $(date)" >> $AUDIT_FILE
echo "" >> $AUDIT_FILE

echo "=== Last 5 Successful SSH Logins ===" >> $AUDIT_FILE
journalctl -u sshd | grep "Accepted" | tail -5 >> $AUDIT_FILE 2>/dev/null
echo "" >> $AUDIT_FILE

echo "=== Last 5 Failed SSH Login Attempts ===" >> $AUDIT_FILE
journalctl -u sshd | grep "Failed" | tail -5 >> $AUDIT_FILE 2>/dev/null
echo "" >> $AUDIT_FILE

echo "=== Alternative: Check auth logs ===" >> $AUDIT_FILE
if [ -f /var/log/auth.log ]; then
    grep "sshd.*Accepted" /var/log/auth.log | tail -5 >> $AUDIT_FILE
    grep "sshd.*Failed" /var/log/auth.log | tail -5 >> $AUDIT_FILE
else
    echo "Note: /var/log/auth.log not found, using journalctl instead" >> $AUDIT_FILE
fi

echo "SSH audit completed and saved to ssh_audit.txt"
EOF

# Make executable and run
chmod +x /home/studentuser/projectX/scripts/ssh_audit.sh
/home/studentuser/projectX/scripts/ssh_audit.sh
```
<img width="904" height="478" alt="Image" src="https://github.com/user-attachments/assets/124aa51c-2dc4-4d58-97ee-71c87afdf18d" />

---

## Task 6: Crontab Practice

### Instructions:
Create three cron jobs:
- Print 'Good morning!' every day at 8 AM
- Backup '/home/studentuser/projectX/' every Sunday
- Delete '.log' files older than 7 days every Friday at midnight

### Solution:
```bash
# Create scripts for cron jobs

# 1. Good morning script
cat > /home/studentuser/projectX/scripts/good_morning.sh << 'EOF'
#!/bin/bash
echo "$(date): Good morning!" >> /home/studentuser/projectX/logs/morning.log
EOF

# 2. Weekly backup script
cat > /home/studentuser/projectX/scripts/weekly_backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/home/studentuser/backups"
mkdir -p $BACKUP_DIR
DATE=$(date +%Y%m%d)
tar -czf $BACKUP_DIR/projectX_backup_$DATE.tar.gz /home/studentuser/projectX/
echo "$(date): Backup completed - projectX_backup_$DATE.tar.gz" >> /home/studentuser/projectX/logs/backup.log
EOF

# 3. Log cleanup script
cat > /home/studentuser/projectX/scripts/cleanup_logs.sh << 'EOF'
#!/bin/bash
find /home/studentuser/projectX/logs -name "*.log" -type f -mtime +7 -delete
echo "$(date): Old log files cleaned up" >> /home/studentuser/projectX/logs/maintenance.log
EOF

# Make all scripts executable
chmod +x /home/studentuser/projectX/scripts/good_morning.sh
chmod +x /home/studentuser/projectX/scripts/weekly_backup.sh
chmod +x /home/studentuser/projectX/scripts/cleanup_logs.sh

# Add to crontab
(crontab -l 2>/dev/null; cat << 'EOF'
# Good morning message at 8 AM daily
0 8 * * * /home/studentuser/projectX/scripts/good_morning.sh
# Weekly backup every Sunday at 2 AM
0 2 * * 0 /home/studentuser/projectX/scripts/weekly_backup.sh
# Cleanup old logs every Friday at midnight
0 0 * * 5 /home/studentuser/projectX/scripts/cleanup_logs.sh
EOF
) | crontab -
```
<img width="1031" height="371" alt="Image" src="https://github.com/user-attachments/assets/7d3db03e-2a7f-43c3-9e78-15fbe8f2a9aa" />

<img width="1033" height="241" alt="Image" src="https://github.com/user-attachments/assets/b955922d-8284-4fa5-9584-7aa7b4b3aa43" />

---

## Task 7: Port Scanner Script

### Instructions:
Write a script to scan ports 20-25 on a user-supplied IP using 'nc' or 'timeout'.

### Solution:
```bash
# Create port scanner script
cat > /home/studentuser/projectX/scripts/port_scanner.sh << 'EOF'
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: $0 <IP_ADDRESS>"
    exit 1
fi

TARGET_IP=$1
START_PORT=20
END_PORT=25

echo "Scanning ports $START_PORT-$END_PORT on $TARGET_IP"
echo "=================================="

for port in $(seq $START_PORT $END_PORT); do
    timeout 3 bash -c "echo >/dev/tcp/$TARGET_IP/$port" 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "Port $port: OPEN"
    else
        echo "Port $port: CLOSED"
    fi
done

echo "Scan completed."
EOF

# Make executable
chmod +x /home/studentuser/projectX/scripts/port_scanner.sh

# Usage example:
# ./port_scanner.sh 192.168.1.1
```
<img width="911" height="670" alt="Image" src="https://github.com/user-attachments/assets/e7a2bded-74a1-4280-9789-b596546748bc" />

---

## Task 8: Website Availability Checker

### Instructions:
Use a file 'sites.txt' containing a list of URLs.
Check if each site is up using 'curl' or 'ping'.
Log results to 'site_status.log'.

### Solution:
```bash
# Create sites.txt with sample URLs
cat > /home/studentuser/projectX/sites.txt << 'EOF'
https://google.com
https://github.com
https://archlinux.org
https://stackoverflow.com
https://nonexistentsite12345.com
EOF

# Create website checker script
cat > /home/studentuser/projectX/scripts/site_checker.sh << 'EOF'
#!/bin/bash

SITES_FILE="/home/studentuser/projectX/sites.txt"
LOG_FILE="/home/studentuser/projectX/logs/site_status.log"

if [ ! -f "$SITES_FILE" ]; then
    echo "Error: $SITES_FILE not found!"
    exit 1
fi

echo "=== Website Availability Check ===" >> $LOG_FILE
echo "Check performed on: $(date)" >> $LOG_FILE
echo "" >> $LOG_FILE

while IFS= read -r site; do
    if [ -n "$site" ]; then
        echo "Checking $site..." 
        if curl -s --connect-timeout 10 --max-time 30 "$site" > /dev/null 2>&1; then
            status="UP"
        else
            status="DOWN"
        fi
        timestamp=$(date "+%Y-%m-%d %H:%M:%S")
        echo "[$timestamp] $site - $status" >> $LOG_FILE
        echo "$site - $status"
    fi
done < "$SITES_FILE"

echo "" >> $LOG_FILE
echo "Website availability check completed."
EOF

# Make executable and run
chmod +x /home/studentuser/projectX/scripts/site_checker.sh
/home/studentuser/projectX/scripts/site_checker.sh
```
<img width="948" height="725" alt="Image" src="https://github.com/user-attachments/assets/fd2425bf-2756-417f-ba0e-c7dad180d427" />

<img width="763" height="217" alt="Image" src="https://github.com/user-attachments/assets/d85bf8ec-7f20-47f3-a5a7-a5f33bd519f5" />

---

## Task 9: Environment and Disk Report

### Instructions:
Generate a report with:
- Current user, hostname, uptime
- Mounted filesystems and usage
- Path and shell-related environment variables

### Solution:
```bash
# Create environment report script
cat > /home/studentuser/projectX/scripts/env_report.sh << 'EOF'
#!/bin/bash

REPORT_FILE="/home/studentuser/projectX/environment_report.txt"

echo "=== System Environment Report ===" > $REPORT_FILE
echo "Generated on: $(date)" >> $REPORT_FILE
echo "" >> $REPORT_FILE

echo "=== User Information ===" >> $REPORT_FILE
echo "Current User: $(whoami)" >> $REPORT_FILE
echo "Hostname: $(hostname)" >> $REPORT_FILE
echo "Uptime: $(uptime)" >> $REPORT_FILE
echo "" >> $REPORT_FILE

echo "=== Mounted Filesystems ===" >> $REPORT_FILE
df -h >> $REPORT_FILE
echo "" >> $REPORT_FILE

echo "=== Filesystem Usage Details ===" >> $REPORT_FILE
mount | column -t >> $REPORT_FILE
echo "" >> $REPORT_FILE

echo "=== Environment Variables (PATH and Shell) ===" >> $REPORT_FILE
echo "PATH: $PATH" >> $REPORT_FILE
echo "SHELL: $SHELL" >> $REPORT_FILE
echo "HOME: $HOME" >> $REPORT_FILE
echo "USER: $USER" >> $REPORT_FILE
echo "PWD: $PWD" >> $REPORT_FILE
echo "TERM: $TERM" >> $REPORT_FILE
echo "" >> $REPORT_FILE

echo "=== All Environment Variables ===" >> $REPORT_FILE
env | sort >> $REPORT_FILE

echo "Environment report saved to $REPORT_FILE"
EOF

# Make executable and run
chmod +x /home/studentuser/projectX/scripts/env_report.sh
/home/studentuser/projectX/scripts/env_report.sh
```
<img width="873" height="667" alt="Image" src="https://github.com/user-attachments/assets/5ca3e59e-0b5a-4493-879f-7a21aaba50fc" />

<img width="854" height="96" alt="Image" src="https://github.com/user-attachments/assets/fe8bf48d-1617-4d54-9f8f-dc010f6bef69" />


---

## Task 10: Compress & Archive Automation

### Instructions:
Find all '.log' files >10MB in '/home/studentuser/projectX/logs'.
Compress them into 'archive_<timestamp>.tar.gz'.
Move the archive to '/home/studentuser/projectX/backup/'.

### Solution:
```bash
# Create archive automation script
cat > /home/studentuser/projectX/scripts/archive_logs.sh << 'EOF'
#!/bin/bash

LOGS_DIR="/home/studentuser/projectX/logs"
BACKUP_DIR="/home/studentuser/projectX/backup"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
ARCHIVE_NAME="archive_${TIMESTAMP}.tar.gz"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Find log files larger than 10MB
echo "Searching for .log files larger than 10MB in $LOGS_DIR..."

# Create temporary file list
TEMP_LIST="/tmp/large_logs_$$"

find "$LOGS_DIR" -name "*.log" -type f -size +10M > "$TEMP_LIST"

if [ -s "$TEMP_LIST" ]; then
    echo "Found large log files:"
    cat "$TEMP_LIST"
    
    # Create archive
    echo "Creating archive: $ARCHIVE_NAME"
    tar -czf "/tmp/$ARCHIVE_NAME" -T "$TEMP_LIST"
    
    # Move archive to backup directory
    mv "/tmp/$ARCHIVE_NAME" "$BACKUP_DIR/"
    
    echo "Archive created and moved to: $BACKUP_DIR/$ARCHIVE_NAME"
    
    # Optionally remove original files after archiving
    # read -p "Remove original files? (y/N): " -n 1 -r
    # if [[ $REPLY =~ ^[Yy]$ ]]; then
    #     while read -r file; do
    #         rm "$file"
    #         echo "Removed: $file"
    #     done < "$TEMP_LIST"
    # fi
else
    echo "No .log files larger than 10MB found in $LOGS_DIR"
fi

# Cleanup
rm -f "$TEMP_LIST"

echo "Archive operation completed."
EOF

# Make executable
chmod +x /home/studentuser/projectX/scripts/archive_logs.sh

# Create some test log files for demonstration
mkdir -p /home/studentuser/projectX/logs
dd if=/dev/zero of=/home/studentuser/projectX/logs/large_test.log bs=1M count=15 2>/dev/null
dd if=/dev/zero of=/home/studentuser/projectX/logs/small_test.log bs=1K count=50 2>/dev/null

# Run the script
/home/studentuser/projectX/scripts/archive_logs.sh
```
<img width="921" height="893" alt="Image" src="https://github.com/user-attachments/assets/e19ba21b-07b4-4166-ad3b-008429a971e3" />

<img width="1154" height="239" alt="Image" src="https://github.com/user-attachments/assets/466aa044-8294-4328-a0af-b9954e09a1fa" />

---



### Security Considerations
- Always run scripts as appropriate users
- Use sudo only when necessary
- Set proper file permissions (chmod/chown)
- Regularly update the system with `pacman -Syu`

---

## Lab Completion Checklist

- [ ] Task 1: User creation and file permissions ✓
- [ ] Task 2: Network information gathering ✓
- [ ] Task 3: System monitoring with cron ✓
- [ ] Task 4: File system monitoring ✓
- [ ] Task 5: SSH login auditing ✓
- [ ] Task 6: Crontab scheduling ✓
- [ ] Task 7: Port scanning ✓
- [ ] Task 8: Website availability checking ✓
- [ ] Task 9: Environment reporting ✓
- [ ] Task 10: Log archiving automation ✓

**Lab completed successfully!**
