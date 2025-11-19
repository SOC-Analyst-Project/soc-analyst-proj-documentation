# **6.1 Scenario 1: Network Reconnaissance (Scanning)**

## **Goal**

Identify active hosts and open services inside the SOC LAN.

## **Technique**

- Reconnaissance
    
- Service discovery
    
- Network scanning
    

## **Attacker Commands (Kali)**

```
nmap -sn 192.168.10.0/24
nmap -sV -p 1-1000 192.168.10.20
nmap -sV 192.168.10.50
```

## **Expected Windows Logs**

**Windows 11 POS** (Sysmon):

- Event ID **3** only if a probed service responds (Sysmon does not log inbound SYNs)
    

**Domain Controller**:

- Same as above
    
- DNS logs may appear if hostname probes occur
    

## **Expected Wazuh Alerts**

- Network scan / port sweep alerts (if Windows Firewall logging is enabled and forwarded)
    
- Suricata scan alerts (only if ESXi vSwitch mirrors internal LAN traffic to OPNsense)
    

## **Expected Splunk Queries**

```
index=wazuh* src_ip=<Kali_IP> ("scan" OR "port" OR event.data.dest_port=*)
```

### **Additional Visibility Requirements (Summary)**

- Enable **Windows Firewall logging** (allows detection of inbound scan attempts)
    
- Ensure **ESXi port mirroring** is configured if Suricata should detect internal scans
    
- Allow **Promiscuous Mode** on mirrored port groups
    
- Ensure endpoints run **Sysmon + Wazuh Agent** to forward any network-related events
    

### **Screenshot Placeholder**

> _(Screenshot: Kali Nmap scan output)_  
> _(Screenshot: Wazuh alert for scanning)_  
> _(Screenshot: Splunk timeline of scan events)_

---

# **6.2 Scenario 2: RDP Brute-Force Attempt**

## **Goal**

Test how repeated failed RDP logins appear in logs and how defensive tools respond.

## **Technique**

- Credential Access
    
- Brute-force
    
- Repeated authentication attempts
    

## **Attacker Commands (Kali)**

```
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.10.50
```

## **Expected Windows Logs**

### **POS Endpoint (Windows 11)**

- **4625** — Failed logon attempt
    
    - Will always appear for RDP brute-force failures
        

### **Domain Controller**

- **4625** — Failed domain authentication (if the user is domain-based)
    
- **4768** — Kerberos TGT request (only if RDP triggers Kerberos; depends on NLA and domain config)
    
- **4771** — Kerberos pre-authentication failure (common during incorrect password attempts)
    

_Note:_  
Event ID **4776** applies only when **NTLM** is used. RDP with NLA normally uses **Kerberos**, so **4776 appears only if the system falls back to NTLM**.

## **Expected Wazuh Alerts**

- Detection of repeated authentication failures
    
- Identification of brute-force pattern from Kali source IP
    

## **Expected Splunk Queries**

```
index=wazuh* src_ip=<Kali_IP> (EventID=4625 OR EventID=4771 OR EventID=4768)
| stats count by Account_Name, src_ip
```

### **Screenshot Placeholder**

> _(Screenshot: Hydra brute-force running)_  
> _(Screenshot: Wazuh brute-force alert)_  
> _(Screenshot: Splunk query output)_

---

# **6.3 Scenario 3: Password Spray Attack**

## **Goal**

Attempt one password across many domain accounts to avoid lockouts.

## **Technique**

- Credential Access
- Password spraying

## **Attacker Commands (Kali)**

Using Kerbrute:

```
kerbrute passwordspray -d phoanhhai.local /usr/share/wordlists/usernames.txt "Password123!" 
```

## **Expected Windows Logs**

**Domain Controller**:

- Event ID **4768** / **4771** with multiple different usernames
- Repeated failure patterns with same source IP

## **Expected Wazuh Alerts**

- “Password spraying detected”
- “High number of authentication failures across multiple accounts”

## **Expected Splunk Queries**

```
index=wazuh* EventID=4768 OR EventID=4771 src_ip=<Kali_IP>| stats count by user, src_ip
```

### **Screenshot Placeholder**

> _(Screenshot: Kerbrute output)_  
> _(Screenshot: Wazuh spray detection alert)_  
> _(Screenshot: Splunk timeline of failures)_

---

# **6.4 Scenario 4: Suspicious PowerShell Execution**

## **Goal**

Detect malicious PowerShell usage on the POS endpoint.

## **Technique**

- Execution
- Living-off-the-land (LOLBins)
- PowerShell abuse

## **Attacker Commands (Kali via Evil-WinRM or remote execution attempt)**

Attempt execution:

```
crackmapexec winrm 192.168.10.50 -u user -p password -x "powershell -enc UwB5AHMAdABlAG0ALgBlAGMAaABvAA==" 
```

Or via local PowerShell (if compromised):

```
powershell -enc UwB5AHMAdABlAG0ALgBlAGMAaABvAA==
```

## **Expected Windows Logs**

**Sysmon**:

- Event ID **1**: Process creation (PowerShell.exe)
- Event ID **4104**: PowerShell script block logging (if enabled)

## **Expected Wazuh Alerts**

- “Encoded PowerShell command detected”
- “Suspicious process creation”
- “Potential obfuscated execution”

## **Expected Splunk Queries**

```
index=wazuh* (EventID=1 OR EventID=4104) CommandLine="*enc*"
```

### **Screenshot Placeholder**

> _(Screenshot: Sysmon EventID 1)_  
> _(Screenshot: Wazuh alert – PowerShell encoded)_  
> _(Screenshot: Splunk search results)_

---

# **6.5 Scenario 5: Suspicious File Drop (Executable)**

## **Goal**

Detect new executable placement in sensitive directories.

## **Technique**

- Persistence
- File-based malware simulation

## **Attacker Commands (Kali)**

Simulated file transfer:

```
smbclient //192.168.10.50/C$ -U Administratorput test.exe C:\\Users\\Public\\
```

Or:

```
crackmapexec smb 192.168.10.50 -u user -p pass --put-file test.exe C:\Windows\Temp\test.exe
```

## **Expected Windows Logs**

**Sysmon**:

- Event ID **11**: File creation
- Shows path, filename, and process responsible

## **Expected Wazuh Alerts**

- “New executable file created”
- “Suspicious file drop”

## **Expected Splunk Queries**

```
index=wazuh* EventID=11 Image="*.exe" 
```

### **Screenshot Placeholder**

> _(Screenshot: Sysmon EventID 11)_  
> _(Screenshot: Wazuh file integrity alert)_  
> _(Screenshot: Splunk query results)_

---

# **6.6 Scenario 6: Lateral Movement Attempt (SMB / WinRM)**

## **Goal**

Simulate attempted lateral movement from the POS endpoint or Kali.

## **Technique**

- Lateral Movement
- SMB authentication attempts
- WinRM probing

## **Attacker Commands (Kali)**

### SMB:

```
crackmapexec smb 192.168.10.20 -u user -p password
```

### WinRM:

```
crackmapexec winrm 192.168.10.20 -u user -p password
```

## **Expected Windows Logs**

**Domain Controller**:

- Event ID **4624**: Successful logon (if creds work)
- Event ID **4625**: Failed logon
- Event ID **4648**: Logon with explicit credentials

**Sysmon** (if on POS):

- Event ID **3**: Connection to DC
- Event ID **1**: New process attempts

## **Expected Wazuh Alerts**

- “Lateral movement attempt detected”
- “Suspicious authentication pattern”

## **Expected Splunk Queries**

```
index=wazuh* src_ip=<Kali_IP> (EventID=4625 OR EventID=4648 OR EventID=4624)
```

### **Screenshot Placeholder**

> _(Screenshot: CrackMapExec lateral movement attempt)_  
> _(Screenshot: Wazuh alert – Lateral movement)_  
> _(Screenshot: Splunk correlation search)_