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



