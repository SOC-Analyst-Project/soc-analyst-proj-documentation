issue: chrony running, but massive time drift
This is very likely caused by nested virtualization
Your VM’s virtual hardware clock jumped/drifted heavily (ESXi timekeeping issue), AND chrony could not step-correct it because NTP servers did not form a clear majority.
![Pasted image 20251129173859](OPNsense%20detect%20nmap/resources/Pasted%20image%2020251129173859.png)

The solution: 
sync guest time with host is no good because it will compete with chrony
the better fix is to change chrony config

```
sudo nano /etc/chrony/chrony.conf
```

```
makestep 1.0 -1 # makestep 1 3
# make -1 (infinite times of corrections whenever the time is wrong by more than 1.0 second)
```

fixed
![Pasted image 20251129175546](resources/resources/Pasted%20image%2020251129175546.png)