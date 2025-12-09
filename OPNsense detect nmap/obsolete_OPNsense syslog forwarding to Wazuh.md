Why obsolete: See [3 OPNsense log forward to Wazuh via agent](3%20OPNsense%20log%20forward%20to%20Wazuh%20via%20agent.md) 


![Pasted image 20251119202221](../../resources/Pasted%20image%2020251119202221.png)
![Pasted image 20251119211429](resources/Pasted%20image%2020251119211429.png)



# **To enable syslog over UDP 514, you must add this block:**

In:

```
/var/ossec/etc/ossec.conf
```

Add:

```xml
<localfile>
  <location>udp://0.0.0.0:514</location>
  <log_format>syslog</log_format>
</localfile>
```

Then restart:

```
sudo systemctl restart wazuh-manager
```

Now Wazuh will:

- Open **UDP/514**
    
- Receive Suricata / OPNsense syslog
    
- Parse them with the syslog decoder
    
