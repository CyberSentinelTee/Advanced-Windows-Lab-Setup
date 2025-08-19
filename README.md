# Advanced Windows Lab Setup (Offline Pentest Lab)

## Overview
This project extends the **Offline Pentest Lab** by setting up a **realistic Windows Active Directory environment** for penetration testing.  
It includes a Windows Server 2012 R2 domain controller, a domain-joined Windows 10 workstation, and common services to simulate real-world enterprise networks.

This lab is fully offline and intended for **educational and security testing purposes only**.

## Lab Topology
[DC01.corp.local] 192.168.56.100 (Windows Server 2012 R2 - Domain Controller)  
[WIN10-PC] 192.168.56.102 (Windows 10 - Domain Workstation)  
[Metasploitable2] 192.168.56.128  
[Kali Linux Attacker] 192.168.56.129  

## PART 1: Windows Server 2012 R2 — Domain Controller Setup

### 1. Basic Configuration
1. Log in as **Administrator**.
2. Rename PC:  
   - Press `Win + R` → type `sysdm.cpl` → rename to `DC01`.
3. Set static IP:
   IP: 192.168.56.100  
   Subnet: 255.255.255.0  
   Gateway: 192.168.56.1 (optional)  
   DNS: 192.168.56.100
4. Restart VM.

### 2. Promote to Domain Controller
1. Open **Server Manager**.
2. Go to **Add Roles and Features**.
3. Choose **Role-based installation** → select `DC01`.
4. Enable roles:
    - Active Directory Domain Services
    - DNS Server
    - File and Storage Services
5. Finish installation.
6. Click **Promote this server to a domain controller**.
7. Create new forest: `corp.lab`.
8. Set **DSRM password**: `Password123!`.
9. Complete wizard & reboot.

![DC-Config](DC%20Setup/DC_Config.png)

### 3. DNS Configuration
1. Open DNS Manager.
2. Confirm Forward Lookup Zone: corp.lab (auto-created).
3. Create Reverse Lookup Zone: 192.168.56.0/24.
4. Add Records:
   - dc1.corp.lab → 192.168.56.100

![DNS-Config](DC%20Setup/DNS_Config.png)
  
### 4. Create Realistic Users and Shares
1. Open **Active Directory Users & Computers**.
2. Create users:
    - `alice.user` → Password: `noob1234!!`
    - `soma.hr` → Password: `user2@123`
    - `erina.dev` → Password: `letmein@2025`
3. Create groups:
    - **HR**
    - **Developers**
4. Add users to groups.
5. Create a shared folder:  
- `C:\Shares\HRDocs` → share with **HR** group only.

![User-Config](DC%20Setup/User_Creation.png)
  
6. Enable SMB file sharing:
    ```powershell
    netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes

### 5. Firewall Rules
Allow essential ports for AD + exploitation scenarios.
   #### Enable RDP
   ```bash
   Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
   ```

   #### Enable PSRemoting
   ```bash
   Enable-PSRemoting -Force
   ```

   #### Install IIS Web Server
   ```bash
   Install-WindowsFeature -name Web-Server -IncludeManagementTools
   ```

   #### Install FTP Server
   ```bash
   Install-WindowsFeature Web-Ftp-Server -IncludeAllSubFeature
   ```

   #### Restart WinRM Service
   ```bash
   Restart-Service WinRM
   ```

   #### LDAP (389 TCP/UDP)
   ```bash
   Enable-NetFirewallRule -DisplayGroup "Active Directory Domain Controller"
   ```
  
   #### Install Telnet Server
   ```bash
   dism /online /Enable-Feature /FeatureName:TelnetServer
   sc config TlntSvr start= auto
   net start TlntSvr
   ```

   #### DNS
   ```bash
   netsh advfirewall firewall add rule name="DNS" protocol=UDP dir=in localport=53 action=allow
   ```

   #### LDAP
   ```bash
   netsh advfirewall firewall add rule name="LDAP" protocol=TCP dir=in localport=389 action=allow
   ```

   #### Kerberos
   ```bash
   netsh advfirewall firewall add rule name="Kerberos" protocol=UDP dir=in localport=88 action=allow
   ```

   #### RDP
   ```bash
   netsh advfirewall firewall add rule name="RDP" protocol=TCP dir=in localport=3389 action=allow
   ```

   #### WinRM
   ```bash
   netsh advfirewall firewall add rule name="WinRM" protocol=TCP dir=in localport=5985 action=allow
   ```

   #### SMB
   ```bash
   netsh advfirewall firewall add rule name="SMB" protocol=TCP dir=in localport=445 action=allow
   ```

   #### FTP
   ```bash
   netsh advfirewall firewall add rule name="FTP" protocol=TCP dir=in localport=21 action=allow
   ```

   #### Telnet
   ```bash
   netsh advfirewall firewall add rule name="Telnet" protocol=TCP dir=in localport=23 action=allow
   ```

Enabled Services 

| Service | Port  | Use Case |
|---------|-------|----------|
| SMB     | 445   | Enumerate shares, relay attacks |
| RDP     | 3389  | Brute force, screenshots |
| WinRM   | 5985  | Remote PowerShell access |
| DNS     | 53    | DNS zone transfers |
| HTTP    | 80    | Test vulnerabilities like XSS, file upload |
| TELNET  | 23    | Test telnet vulnerabilities |
| FTP     | 21    | Insecure credentials, anonymous access |
| LDAP    | 389   | LDAP enumeration |

## PART 2: Windows 10 — Domain-Joined Workstation

### 1. Rename + Static IP
1. Rename PC to WIN10-PC.
2. Set static IP:  
   IP: 192.168.56.102  
   Subnet: 255.255.255.0  
   DNS: 192.168.56.103  # Server IP  

### 2. Join to Domain
1. Go to System → Rename PC or Join Domain.
2. Enter: corp.lab.
3. Use:  
   Username: Administrator  
   Password: Password123!  
4. Restart after joining.

### 3. Log in as Domain Users
At login:
  - Click Other User.
  - Enter: alice.user@corp.lab
  - Password: Password123!

## Skills Demonstrated
- Active Directory configuration
- Windows Server role deployment
- Domain joining and authentication
- Network service enumeration
- Simulated attack surface creation
- Penetration testing lab design

## References
- Glen D. Singh, *Learn Kali Linux 2019: Perform Powerful Penetration Testing Using Kali Linux, Metasploit, Nessus, Nmap, and Wireshark*, Packt Publishing, 2019. ISBN: 978-1789611806.
- EC-Council. *Certified Ethical Hacker (CEH) v13 Official Courseware*. Albuquerque, NM: EC-Council, 2023.

## Author
Tafadzwa Nemukuyu(www.linkedin.com/in/tafadzwa-nemukuyu)
