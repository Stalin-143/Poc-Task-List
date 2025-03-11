# Create the Markdown content for Task 2: Remote Access & SSH Hardening

# 🔐 Linux Security - Remote Access & SSH Hardening (PoC)

This Proof of Concept (PoC) demonstrates **SSH misconfigurations**, how they can be exploited using brute-force attacks, and the necessary security hardening techniques.

---

## 📌 **Task 2: Remote Access & SSH Hardening**

### ✅ **Setup: Enabling SSH & Weak Security Configurations**

1. **Ensure SSH is installed and running:**
   ```bash
   sudo apt update && sudo apt install -y openssh-server
   sudo systemctl enable ssh
   sudo systemctl start ssh

![Image](https://github.com/user-attachments/assets/2f105a47-3971-4311-b5b3-236c3e909920)


### Allow root login and password authentication: Open the SSH configuration file:

      sudo nano /etc/ssh/sshd_config

  #### Modify or add the following lines:

      PermitRootLogin yes
      PasswordAuthentication yes

  #### Save and exit, then restart SSH:

      sudo systemctl restart ssh

 #### Verify SSH access:

   - Use a second machine or terminal to connect via SSH:


          ssh root@<server-ip> 

     
### ✅ Exploit: Brute-Force Attack on SSH


  #### Using Hydra to brute-force SSH credentials:
  
        hydra -l root -P passwords.txt <server-ip> ssh

  ![Image](https://github.com/user-attachments/assets/9572506e-8217-4018-af4c-a533b847e082)

  ![Image](https://github.com/user-attachments/assets/bda2e27a-6614-4e35-b216-e7aa2e8e949c)

- -l root: Specifies the username.
- -P passwords.txt: List of common passwords for brute-force attack.
- server-ip : Replace with the target machine's IP.

  #### Impact Analysis:

  - If the root account uses a weak password, an attacker can gain full control over the system.
  - Automated brute-force tools make SSH a common attack vector.
 

### ✅ Mitigation: Hardening SSH Security

   #### 1. Disable root login and password authentication:

                 sudo nano /etc/ssh/sshd_config

   ##### Update the following:

                PermitRootLogin no
                PasswordAuthentication no


  ![Image](https://github.com/user-attachments/assets/361d2eb3-93b7-43a6-add0-4b6a2c647547)


  ##### Restart SSH:

                sudo systemctl restart ssh
                
 #### 2. Install and configure Fail2Ban to prevent brute-force attempts:


                 sudo apt install fail2ban -y
                 sudo nano /etc/fail2ban/jail.local

  ##### Add the following:

                  [sshd]
                  enabled = true
                  maxretry = 5
                  bantime = 600
                  findtime = 600

  ![Image](https://github.com/user-attachments/assets/1f49b3e7-708f-4342-a01f-d7eb56c87ab4)


 ##### Restart Fail2Ban:

                 sudo systemctl restart fail2ban




## 🔒 Conclusion

This PoC highlights the importance of disabling root login, enforcing key-based authentication, and using Fail2Ban to prevent unauthorized access.

🚀 Secure Your SSH & Stay Protected!




