6.1-6.5 only
#  **6.1 Network Reconnaissance (Nmap)**

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

- **Enable Windows Firewall Logging (not enabled by default):**  
    `wf.msc → Firewall Properties → Logging → Log dropped packets = Yes, Log successful connections = Yes`
    
- **Ensure Sysmon is installed and configured** (Sysmon logs are not default on Windows)
    
- **Optional: Run active services** (SMB/RDP/HTTP) to generate outbound responses that produce Event ID 3
    
- **For network-level visibility:**
    
    - Configure **ESXi Port Mirroring** to mirror LAN traffic to OPNsense LAN NIC
        
    - Set OPNsense LAN interface **Promiscuous Mode = Accept**
        
    - Suricata on OPNsense will then detect internal scans
        

## **Expected Detection**

- **Windows POS / DC:**
    
    - Sysmon Event ID **3** (NetworkConnect) _only if a service responds to the scan_
        
    - Windows Firewall logs show inbound probe attempts
        
- **Wazuh:**
    
    - “Port scanning activity” (requires firewall logs or Suricata logs)
        
- **Splunk:**
    
    - Spike in inbound connection attempts
        
    - Search by source IP from Kali
        

## **Mind the Rabbit Hole**

- No Sysmon Event ID 3 appears for **inbound** Nmap scans unless the host responds.
    
- Windows Firewall Logging is **off by default**, resulting in zero visibility.
    
- Suricata on OPNsense will **not** see internal scans unless ESXi port mirroring is enabled.
    
- If Sysmon config excludes NetworkConnect events, nothing will be logged.
    
- If POS/DC has no active services, Nmap probes produce no loggable responses.

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

- **Enable Advanced Audit Policy → Account Logon → Kerberos Authentication (Success & Failure)**  
    _(Not fully enabled by default on Windows Server)_
    
- **Ensure Sysmon is installed** (Sysmon logs are not default)
    
- **Create predictable usernames** (e.g., `tommy`, `henry`, `staff1`, `posuser`)
    
- **Verify DC time sync** (Kerberos is time-sensitive; skew prevents event generation)
    
- **Ensure Wazuh Agent running on DC** to forward Event IDs 4768/4771
    

## **Expected Detection**

- **Windows Security Log (Domain Controller):**
    
    - Event ID **4768** (Kerberos AS-REQ)
        
    - Event ID **4771** (Kerberos pre-auth failed)
        
- **Wazuh:**
    
    - MITRE ATT&CK **T1087 – Account Discovery**
        
    - Kerberos enumeration alerts
        
- **Splunk:**
    
    - Spike in Kerberos failures from Kali source IP
        

## **Mind the Rabbit Hole**

- Kerberos failures **won’t log** unless **Advanced Audit Policy** is actually applied.
    
- If DC and Kali clocks differ by >5 minutes, Kerberos requests fail silently.
    
- If usernames list contains invalid domain format, Kerbrute sends no Kerberos traffic.
    
- If Wazuh Agent on DC is misconfigured, **no Kerberos events reach Splunk**.
    
- Event ID 4768/4771 do not appear if network issues block UDP/88 (Kerberos).

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

- **Create SMB share:** `C:\PublicShare`
    
- **Share Permissions:** `Everyone: Read`
    
- **Enable “Audit Object Access” (Success & Failure)** in Advanced Audit Policy  
    _(Required for Event ID 5140 — not enabled by default)_
    
- **Ensure Sysmon is installed** to record network activity
    
- **Confirm SMBv2/3 enabled** (default in modern Windows) so scans generate authentication attempts
    
- **Ensure Wazuh Agent is forwarding Security logs** from the DC
    

## **Expected Detection**

- **Event ID 4624** — Anonymous or guest-type logons from enumeration
    
- **Event ID 5140** — “A network share was accessed” (requires Object Access auditing enabled)
    
- **Wazuh Alerts:**
    
    - SMB enumeration attempt
        
    - Excessive network share probing
        
    - MITRE mapping under Discovery (T1135: Network Share Discovery)
        

## **Mind the Rabbit Hole**

- Event ID **5140** will not appear unless **Object Access auditing** is explicitly enabled.
    
- If the SMB share has restrictive permissions, enumeration may fail with no logs.
    
- If SMB signing or firewall blocks SMB, tools return errors but DC logs nothing.
    
- If Wazuh is not collecting the **Security** log channel, SMB activity never reaches Splunk.
    
- Using `enum4linux` against hardened AD may produce limited output, making detection harder.

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

- **Disable Account Lockout Policy temporarily** to prevent lockouts during testing
    
- **Set weak passwords** to ensure valid authentication attempts
    
- **Enable “Audit Logon” (Failures)** under Advanced Audit Policy  
    _(4625 Failure Audit is enabled by default on DC, but confirm via GPO)_
    
- **Ensure Sysmon installed** for supplemental network telemetry
    
- **Verify Wazuh Agent is forwarding the Security log channel** (SMB brute force appears only in Security logs)
    

## **Expected Detection**

- **Event ID 4625** — Failed logon attempts (rapid spike from Kali)
    
- **Event ID 4624** (optional) — If a password matches
    
- **Wazuh Alerts:**
    
    - SMB brute-force
        
    - Excessive authentication failures
        
    - MITRE ATT&CK **T1110 – Brute Force**
        

## **Mind the Rabbit Hole**

- If “Audit Logon” Failure is disabled, **4625 will not be generated**.
    
- Account lockout policy left enabled results in premature lockouts, stopping the simulation.
    
- If SMB is blocked by firewall, tools return errors but Windows logs nothing.
    
- Kerberos may intercept some SMB auth attempts—ensure RC4/NTLM allowed for clear visibility.
    
- Missing Security event forwarding in Wazuh causes SMB brute-force attempts to be invisible in Splunk.
---

# **6.5 RDP Brute Force (Hydra)**

## **Tools**

`hydra`

## **Attacker Commands (Kali)**

```
hydra -L users.txt -P passwords.txt rdp://192.168.10.50
```

## **Target Prep**

- **Enable Remote Desktop** on POS (`System → Remote Desktop`)
    
- **Allow inbound TCP/3389** in Windows Firewall (RDP not allowed by default)
    
- **Enable “Audit Logon” Failure** under Advanced Audit Policy  
    _(Event ID 4625 relies on this — typically enabled on DC but not always on clients)_
    
- **Disable Account Lockout Policy temporarily** to avoid locking test users
    
- **Ensure Sysmon is installed** for additional context on RDP-handling processes
    
- **Verify Wazuh Agent is collecting Security logs** from the POS
    

## **Expected Detection**

- **Event ID 4625** — Failed RDP logon attempts
    
- **Event ID 4624** (optional) — If a password is correct during brute-force
    
- **Wazuh Alerts:**
    
    - RDP brute-force
        
    - Excessive authentication failures
        
    - MITRE ATT&CK **T1110 – Brute Force**
        
- **Splunk:**
    
    - High failure spike from Kali source IP
        

## **Mind the Rabbit Hole**

- If RDP is disabled or firewall blocks 3389, **Hydra errors but Windows logs nothing**.
    
- If “Audit Logon” Failure is off, **no 4625 events appear**.
    
- If NLA (Network Level Authentication) is misconfigured, RDP may reject attempts before logging.
    
- Excessive lockouts interrupt brute-force unless policy is relaxed.
    
- If Wazuh agent on POS is not forwarding the Security channel, Splunk shows no RDP failures.













