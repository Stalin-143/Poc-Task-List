# 🔹 Task 3: Firewall & Network Security (PoC)

## ✅ Setup: Install & Configure Web Server

1. **Install Apache Web Server**
   ```bash
   sudo apt update && sudo apt install -y apache2
   sudo systemctl enable apache2
   sudo systemctl start apache2
   sudo apt install ufw

  ![Image](https://github.com/user-attachments/assets/fafe61e1-846d-4ccc-b4da-fa3c10b203eb)

  2. **Disable UFW (Uncomplicated Firewall)**

     ```bash
        sudo ufw disable

 ![Image](https://github.com/user-attachments/assets/878b8064-98e4-4263-a324-9c789a3ce77e)

 3. **Verify Apache is Running**

        systemctl status apache2

 4. **Check Open Ports**

        sudo netstat -tulnp | grep LISTEN



  #### Expected output:
  
  
  ![Image](https://github.com/user-attachments/assets/b2ff26f4-76b2-450a-b1f9-dba758053171)


  ### 🚨 Exploit: Scanning for Open Ports & Services

   #### 1. Use Nmap to Find Open Ports


       nmap -sV -p- <Your-IP>

   ![Image](https://github.com/user-attachments/assets/b8be4e1a-bbce-455a-a5ef-20f7f33d08fa)

       
  - -sV: Service detection
  - -p-: Scan all 65,535 ports

    
#### 2. Use Netcat to Interact with Open Ports



      nc -vz <YOUR-IP>


#### Expected output:

    Connection to <YOUR_IP> 80 port [tcp/http] succeeded!


 #### 3. Impact Analysis

   - Without a firewall, attackers can discover all open services and attempt brute-force attacks or exploits.


## 🔒 Mitigation: Firewall Hardening

### Enable UFW & Allow Only Necessary Services


    sudo ufw enable
    sudo ufw allow 22/tcp
    sudo ufw allow 80/tcp
    sudo ufw deny 23/tcp   # Block Telnet
    sudo ufw deny 3306/tcp # Block MySQL


![Image](https://github.com/user-attachments/assets/0a0936d2-a34c-4a42-9dde-6ab690d69ac6)


#### Verify UFW Rules



    sudo ufw status verbose


![Image](https://github.com/user-attachments/assets/be85d7eb-4b4e-469a-8351-16676ed27f1e)


#### Implement IPTables for Extra Protection


     sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
     sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
     sudo iptables -A INPUT -p tcp --dport 3306 -j DROP
     sudo iptables -A INPUT -p tcp --dport 23 -j DROP
     
#### Re-scan Using Nmap to Verify


      nmap -sV -p- <YOUR-IP>

  ![Image](https://github.com/user-attachments/assets/70ecd786-f99f-47b1-8331-c22e810f522b)



## 🔥 Conclusion

- Before Hardening: Open services exposed to attacks.
- After Hardening: Only SSH (22) & HTTP (80) remain accessible.
- Firewall rules significantly reduce the attack surface.
  
🚀 Secure Your Network & Stay Protected! 🔐


