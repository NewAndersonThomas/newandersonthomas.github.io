---
layout: default
title: WSL
parent: VM
nav_order: 2
last_modified_date:   2024-12-02 23:41:18
---

# WSL
---
## commonly use command
1. Shutdown all wsl system: `wsl --shutdown`

## network model
1. NAT:
   - Linux and Windows have separate IP address
   - Accessing Linux networking apps from Windows(localhost)  
   - Accessing Windows networking apps from Linux(host ip)
     - Obtain the IP address of your host machine by running this command from your Linux distribution: `ip route show | grep -i default | awk '{ print $3}'`
1. mirror
   - Linux and Windows both have the some IP address
   - some Linux apps likes k8s cannot directly access with `localhost` from Windows 
## wslconfig
```bash
[wsl2]
autoMemoryReclaim=gradual  # gradual  | dropcache | disabled
networkingMode=NAT
dnsTunneling=true
firewall=false
# autoProxy=true
localhostForwarding=true
hostAddressLoopback=true
nestedVirtualization=false
debugConsole=false
dnsProxy=true
```