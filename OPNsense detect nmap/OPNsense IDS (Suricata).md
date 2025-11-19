see also [[(obsolete) OPNsense syslog forwarding to Wazuh]]

![[Pasted image 20251118142052.png]]

![[Pasted image 20251119160936.png]]


![[Pasted image 20251119160436.png]]
# Guide to configure OPNsense

# ✅ 1. Confirm VMware Workstation Settings (CRITICAL)

**VMware Workstation → Edit → Virtual Network Editor → vmnet9 → “Change Adapter Settings”**

Enable:

✔ **Promiscuous mode: Allow on "VM Network" port group**  
✔ **Status: Connected**  
✔ **Bridged to: VMnet9

**Pitfall:**  
If Promiscuous Mode is _not_ set to **Allow All**, OPNsense will see zero traffic.

---

# ✅ 2. Prepare OPNsense VM NICs

Inside VMware Workstation:

- OPNsense NIC1 → WAN → e.g., NAT or vmnet8
    
- **OPNsense NIC2 → LAN → vmnet9 (monitor target)**
    

**Pitfall:**  
If OPNsense is **not** on vmnet9, it cannot see ESXi internal traffic.

---

# ✅ 3. Assign Interfaces (OPNsense Web UI)

Go to:

**Interfaces → Assignments**

You should see something like:

- **WAN → em0**
    
- **LAN/MONITOR → em1 (vmnet9)**
    

Click LAN/MONITOR:

- **Enable interface** (checked)
    
- **Static IPv4** (optional, but recommended)
    
- You _can_ leave IP blank if using STRICT passive sniffing.
    
- **Do NOT enable DHCP on this interface** (unless you want it)
    

**Pitfall:**  
If the LAN interface is DOWN or not assigned, Suricata won’t start.

---

# ✅ 4. Enable Suricata IDS (Passive Mode)

Go to:

**Services → Intrusion Detection → Administration → Settings**

Set:

✔ **Enabled**  
✔ **IPS Mode = OFF** (must be off for passive sniffing)  
✔ **Promiscuous = ON**  
✔ **Interfaces: select LAN/MONITOR (vmnet9)**  
✔ Pattern Matcher: **Hyperscan** (Intel) or Aho-Corasick (AMD)

Click **Apply**.

**Pitfall:**  
If **IPS Mode = ON**, traffic _must flow through_ the interface → NOT POSSIBLE for mirrored/sniffed traffic.

---

# ✅ 5. Enable Rulesets

Go to:

**Services → Intrusion Detection → Administration → Download**

Enable:

- **ET Open – Emerging Threats Open**
    
- (Optional) Abuse.ch, SSLBL, Spamhaus
    

Download updates.

Then go to:

**Services → Intrusion Detection → Policies**

Enable:

- **emerging-scan.rules**
    
- **reconnaissance categories**
    
- All **ET SCAN** signatures
    

**Pitfall:**  
If rules are downloaded but NOT applied via a Policy, Suricata logs nothing.

---

# ✅ 6. Start Suricata

Same menu:

**Start/Restart Suricata**

Verify:

- Status = **Running**
    
- Interface = **LAN/MONITOR**
    
- Kernel mode = **IDS** (NOT IPS)
    

---

# ✅ 7. Test Nmap from ESXi VM to ESXi VM

Example:

```
nmap -sS 192.168.10.20
```

Then go to:

**Services → Intrusion Detection → Alerts**

You should see:

- **ET SCAN Nmap Synchronous FIN Scan**
    
- **ET SCAN Nmap Null Scan**
    
- **ET SCAN Potential SSH Scan** (if port 22 hit)
    

**Pitfall:**  
If you see no alerts, it means **vmnet9 is not in promiscuous mode** or  
the **Suricata interface is incorrect**.

---

# 🚨 Common Pitfalls (Very Important)

### ❌ VMware Workstation Promiscuous Mode not allowed

→ OPNsense sees **only broadcast & traffic to its own MAC**.

### ❌ OPNsense monitoring NIC not on vmnet9

→ Sees nothing from ESXi.

### ❌ Suricata interface assigned wrong (WAN instead of LAN/MONITOR)

→ Alerts empty.

### ❌ IPS Mode = ON

→ Suricata ignores sniffed/mirrored traffic.

### ❌ ET rules downloaded but not assigned via a Policy

→ No signatures loaded, no alerts.
