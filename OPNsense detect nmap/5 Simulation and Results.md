## 5.1 Ping sweeps

Standard ping sweep
```
sudo nmap -sn 192.168.10.0/24 -T4
```
![[Pasted image 20251129163527.png]]

ARP ping sweep (-PR)
```
sudo nmap -PR -sn 192.168.10.0/24 -T4
```
![[Pasted image 20251126150322.png|425]]


ARP Scan was not detected by Suricata IPS. Checked there is no scanning rule relating to -PR and ARP.
**Reason:** 
**ARP (layer2) is NOT IP traffic (layer 3), they are not meant to be detected by L3 Suricata**
See [[6 Extras about ARP scan & detection]]

## 5.2 Service detection
-sV on the whole subnet
(Not recommended, service detection should aim at a few selected host)
```bash
sudo nmap -sV -sC -O 192.168.10.0/24 -oX scan.xml && xsltproc /usr/share/nmap/nmap.xsl scan.xml > scan.html
```

```bash
sudo nmap -sV -sC -O 192.168.10.0/24 -T4
```

![[Pasted image 20251128002741.png]]
Result is in the attachment [[scan.html]]

As an attacker, we are interested to find out more about the DC-SERVER.phoanhhai.local (192.168.10.20)

```
sudo nmap -A -T4 192.168.10.20

# which is equivant to sudo nmap -sV -sC -O --traceroute -T4 192.168.10.20
# -sC = same as --script=default, default NSE script
# --traceroute = path to the host
# -sV = enumerate service banners
# -O = OS fingerprinting
```

### Scan result
```bash
┌──(kali㉿attack-kali)-[~]
└─$ sudo nmap -A 192.168.10.20 -T4
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-29 16:58 AEDT
Nmap scan report for DC-SERVER.phoanhhai.local (192.168.10.20)
Host is up (0.00056s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-29 05:59:10Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: phoanhhai.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: phoanhhai.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server
| ssl-cert: Subject: commonName=DC-SERVER.phoanhhai.local
| Not valid before: 2025-11-15T18:48:56
|_Not valid after:  2026-05-17T18:48:56
| rdp-ntlm-info:
|   Target_Name: PHOANHHAI
|   NetBIOS_Domain_Name: PHOANHHAI
|   NetBIOS_Computer_Name: DC-SERVER
|   DNS_Domain_Name: phoanhhai.local
|   DNS_Computer_Name: DC-SERVER.phoanhhai.local
|   DNS_Tree_Name: phoanhhai.local
|   Product_Version: 10.0.26100
|_  System_Time: 2025-11-29T05:59:20+00:00
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.95%I=7%D=11/29%Time=692A8BB3%P=x86_64-pc-linux-gnu%r(T
SF:erminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02
SF:\0\0\0");
MAC Address: 00:0C:29:9F:0D:2F (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2016|11 (96%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_11
Aggressive OS guesses: Microsoft Windows Server 2022 (96%), Microsoft Windows Server 2016 (91%), Microsoft Windows 11 21H2 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: Host: DC-SERVER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
|_nbstat: NetBIOS name: DC-SERVER, NetBIOS user: <unknown>, NetBIOS MAC: 00:0c:29:9f:0d:2f (VMware)
| smb2-time:
|   date: 2025-11-29T05:59:20
|_  start_date: N/A

TRACEROUTE
HOP RTT     ADDRESS
1   0.56 ms DC-SERVER.phoanhhai.local (192.168.10.20)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.56 seconds

```

Suricata alert
![[Pasted image 20251129172414.png]]

Wazuh alert
![[Pasted image 20251129172649.png]]

Splunk
