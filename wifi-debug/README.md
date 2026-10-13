## Problem

After logging into the console, I often have to reconnect to wifi with the following commands:

```bash
iwctl
station wlan0 connect <ssid>
station wlan0 show
```

## Goal

On boot, I want 
1. Wifi connects automaticall
2. IP address is obtained automatically
3. iwctl is not needed

## Diagnosis

### We need to figure out what's handling wireless connectivity and DHCP/IP assignment.

### 1. Run commands 

```bash
systemctl status iwd --no-pager
systemctl status dhcpcd --no-pager
systemctl status systemd-networkd --no-pager
ip addr show wlan0
```
### Output

```bash
● iwd.service - Wireless service
     Loaded: loaded (/usr/lib/systemd/system/iwd.service; enabled; preset: disabled)
     Active: active (running) since Sun 2026-10-11 23:19:32 PDT; 21h ago
 Invocation: 33a4227687144239bb6b7868f9773204
       Docs: man:iwd(8)
             man:iwd.config(5)
             man:iwd.network(5)
             man:iwd.ap(5)
   Main PID: 380 (iwd)
      Tasks: 1 (limit: 9356)
     Memory: 10.4M (peak: 11.1M)
        CPU: 1.984s
     CGroup: /system.slice/iwd.service
             └─380 /usr/lib/iwd/iwd

Oct 12 20:12:45 arlobot iwd[380]: invalid HE capabilities for f8:79:0a:4b:15:e5
Oct 12 20:12:45 arlobot iwd[380]: invalid HE capabilities for e6:79:0a:4b:15:e5
Oct 12 20:13:45 arlobot iwd[380]: event: roam-scan,
Oct 12 20:13:45 arlobot iwd[380]: invalid HE capabilities for f8:79:0a:4b:15:e5
Oct 12 20:13:45 arlobot iwd[380]: invalid HE capabilities for e6:79:0a:4b:15:e5
Oct 12 20:13:45 arlobot iwd[380]: event: roam-scan,
Oct 12 20:14:47 arlobot iwd[380]: event: roam-scan,
Oct 12 20:14:47 arlobot iwd[380]: invalid HE capabilities for e6:79:0a:4b:15:e5
Oct 12 20:14:47 arlobot iwd[380]: invalid HE capabilities for f6:79:0a:4b:15:e5
Oct 12 20:14:47 arlobot iwd[380]: event: roam-scan,
```
```bash
● dhcpcd.service - DHCP/ IPv4LL/ IPv6RA/ DHCPv6 client on all interfaces
     Loaded: loaded (/usr/lib/systemd/system/dhcpcd.service; enabled; preset: disabled)
     Active: active (running) since Sun 2026-10-11 23:19:32 PDT; 21h ago
 Invocation: 215bc2ff746b4dc8b606a04794aa4cda
   Main PID: 379 (dhcpcd)
      Tasks: 5 (limit: 9356)
     Memory: 4.2M (peak: 6.3M)
        CPU: 49.456s
     CGroup: /system.slice/dhcpcd.service
             ├─ 379 "dhcpcd: [manager] [ip4] [ip6]"
             ├─ 384 "dhcpcd: [privileged proxy]"
             ├─ 385 "dhcpcd: [network proxy]"
             ├─ 387 "dhcpcd: [control proxy]"
             └─3279 "dhcpcd: [BPF ARP] wlan0 10.0.0.29"

Oct 12 20:04:24 arlobot dhcpcd[384]: wlan0: adding default route via fe80::fad0:eff:fe29:da67
Oct 12 20:04:24 arlobot dhcpcd[384]: wlan0: soliciting a DHCPv6 lease
Oct 12 20:04:26 arlobot dhcpcd[384]: wlan0: ADV 2601:1c2:301:cd4e::651d/128 from fe80::fad0:eff:fe29:da67 (255)
Oct 12 20:04:26 arlobot dhcpcd[384]: wlan0: accepted reconfigure key
Oct 12 20:04:26 arlobot dhcpcd[384]: wlan0: REPLY6 received from fe80::fad0:eff:fe29:da67
Oct 12 20:04:26 arlobot dhcpcd[384]: wlan0: adding address 2601:1c2:301:cd4e::651d/128
Oct 12 20:04:26 arlobot dhcpcd[384]: wlan0: renew in 24653, rebind in 39445, expire in 49307 seconds
Oct 12 20:04:29 arlobot dhcpcd[384]: wlan0: leased 10.0.0.29 for 172800 seconds
Oct 12 20:04:29 arlobot dhcpcd[384]: wlan0: adding route to 10.0.0.0/24
Oct 12 20:04:29 arlobot dhcpcd[384]: wlan0: adding default route via 10.0.0.1


```


### 2. Look for X session logs


### 3. Try to capture `startx` output


## Result


## What I Learned

