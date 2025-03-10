# Proof of Concept (PoC): Linux Security - Exploitation & Hardening

## Task 1: User & Permission Misconfigurations

### 1. Setup
#### Creating Multiple Users
```bash
sudo useradd attacker
sudo passwd attacker
sudo useradd victim
sudo passwd victim
```


  ![Image](https://github.com/user-attachments/assets/e3344a06-4f68-474e-a246-57e4689ab660)

#### Assigning Incorrect Permissions to Sensitive Files
```bash
sudo chmod 777 /etc/shadow
sudo chmod 777 /etc/passwd
```


### 2. Exploitation
#### Accessing Sensitive Files as a Low-Privilege User
```bash
su - attacker
cat /etc/shadow
cat /etc/passwd
```





*Expected Outcome:* The `attacker` user is able to read and potentially modify sensitive system files due to incorrect permissions.

### 3. Mitigation
#### Fixing Permission Issues
```bash
sudo chmod 640 /etc/shadow
sudo chmod 644 /etc/passwd
sudo chown root:shadow /etc/shadow
sudo chown root:root /etc/passwd
```

#### Using Proper sudo Privileges
```bash
sudo visudo
```
_Add the following line to restrict unnecessary sudo access:_
```bash
attacker ALL=(ALL) ALL
```
_Or create a specific rule:_
```bash
attacker ALL=(ALL) !/bin/cat /etc/shadow
```

### 4. Deliverables
#### Report
1. Commands used for exploitation and mitigation.
2. Screenshots of exploitation attempts and successful mitigation.


