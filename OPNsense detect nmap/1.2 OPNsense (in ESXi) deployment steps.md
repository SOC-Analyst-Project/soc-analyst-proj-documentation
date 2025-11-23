![[Pasted image 20251123150657.png]]

Download dvd
Extract ISO file
upload iso to ESXi

Create a New vSwitch and new Port group for WAN

Create VM, attach iso, make sure disk is large enough (minimum >20gb for thin provisioned), attach NIC (WAN port group), and give mac addresses:
![[Pasted image 20251123150940.png]]
![[Pasted image 20251123151003.png]]

**Important:** When booted, it automatically runs in live CD mode.
After the configuration, we need to login as installer (password: opnsense), and then the actual installation will start.

Otherwise, in the web UI, it will prompt "you are currently in live CD mode, config will not be saved"

