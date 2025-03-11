

# 🛡️ Linux Security - Exploitation & Hardening (PoC)

This Proof of Concept (PoC) demonstrates **user and permission misconfigurations** in Linux, their exploitation, and mitigation techniques to enhance system security.

---

## 📌 **Task: User & Permission Misconfigurations**

### ✅ **Setup: Creating Users & Assigning Incorrect Permissions**

#### 1. **Create multiple users:**
   ```bash
   sudo useradd attacker
   sudo useradd victim
   echo "attacker:password" | sudo chpasswd
   echo "victim:password" | sudo chpasswd


   ```

   ![Image](https://github.com/user-attachments/assets/5e1a8098-5026-4195-8faf-df167f1cd583)
   
- The attacker user will be used to exploit the misconfiguration.
- The victim user is a normal system user.

#### 2. **Assign incorrect permissions to sensitive files:**
    
      sudo chmod 777 /etc/shadow
      sudo chmod 777 /etc/passwd


  ![Image](https://github.com/user-attachments/assets/6ff869a6-e015-4149-85be-ab1b99001ee8)


- chmod 777 makes the files readable, writable, and executable by all users, which is a major security risk.
- /etc/shadow contains hashed passwords, while /etc/passwd stores user account details.

### ✅ Exploit: Accessing Sensitive Files as a Low-Privileged User

   #### 1. Switch to the attacker user:

              su attacker

  #### 2. Attempt to read sensitive files:

              cat /etc/shadow
              cat /etc/passwd

  ![Image](https://github.com/user-attachments/assets/1485c4e0-4174-4a77-ba81-97690e6cc156)

 #### 3. Impact Analysis:

  - If successful, the attacker can steal password hashes and attempt to crack them using tools like john or hashcat.
  - Access to /etc/passwd reveals user information, which can be useful for privilege escalation or other attacks.

### ✅ Mitigation: Fixing Permission Issues & Securing sudo Usage

 #### 1. Restore correct permissions:

            sudo chmod 640 /etc/shadow
            sudo chmod 644 /etc/passwd

  ![Image](https://github.com/user-attachments/assets/950a71c7-5cc0-4cb9-abfd-be3582a64971)

 #### 2. Ensure proper file ownership:

            sudo chown root:shadow /etc/shadow
            sudo chown root:root /etc/passwd

  ![Image](https://github.com/user-attachments/assets/97b38306-a8b0-45d1-883e-b0024598fab7)

#### 3. Secure sudo privileges using visudo:

            sudo /etc/passwd

  ![Image](https://github.com/user-attachments/assets/8f0f7409-e3b4-4304-98da-1f333899bef1)
            
- Limit sudo access only to trusted users.
- Remove unnecessary NOPASSWD entries.


## 🔒 Conclusion

This PoC highlights the dangers of misconfigured file permissions and how an attacker can exploit them. Proper file permissions, ownership settings, and sudo configuration are crucial for Linux system security.

🚀 Stay Secure & Harden Your System!







## 💰 
 [![BuyMeACoffee](https://img.shields.io/badge/Buy%20Me%20a%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/stalin143) [![PayPal](https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/stalinS143) 






