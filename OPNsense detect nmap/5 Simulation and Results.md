
![[Pasted image 20251126150322.png]]
=======
![[Pasted image 20251126150322.png|425]]





ARP Scan was not detected by Suricata IPS. Checked there is no scanning rule relating to -PR and ARP.

Reason: 
**ARP is NOT IP traffic**
ARP frames:
- Are **Layer 2**, not Layer 3
- Do not use IPv4/IPv6
- Do not use TCP/UDP/ICMP
- Never pass through NAT, routing, or L3 firewall rules
- Are not inspected by Suricata (which is an **L3+ IDS/IPS engine**)

Lesson learned: ARP Sweep more stealthy as it's layer 2 and cannot be detected by Layer 3 firewalls. But Palo Alto and Fortinet+FortiSwitch can still detect them.
![[Pasted image 20251126150916.png]]
=======
**ARP (layer2) is NOT IP traffic (layer 3), they are not meant to be detected by L3 Suricata**

See [[6 Extras about ARP scan & detection]]


```bash
sudo nmap -sV -sC -O 192.168.10.0/24 -oX scan.xml && xsltproc /usr/share/nmap/nmap.xsl scan.xml > scan.html
```

```bash
sudo nmap -sV -sC -O 192.168.10.0/24 -T4
```


![[Pasted image 20251128002741.png]]


>>>>>>> origin/main
![[Pasted image 20251126150322.png|425]]

ARP Scan was not detected by Suricata IPS. Checked there is no scanning rule relating to -PR and ARP.

Reason: 
**ARP (layer2) is NOT IP traffic (layer 3), they are not meant to be detected by L3 Suricata**

See [[6 Extras about ARP scan & detection]]


```bash
sudo nmap -sV -sC -O 192.168.10.0/24 -oX scan.xml && xsltproc /usr/share/nmap/nmap.xsl scan.xml > scan.html
```

```bash
sudo nmap -sV -sC -O 192.168.10.0/24 -T4
```


![[Pasted image 20251128002741.png]]


