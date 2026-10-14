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

### I need to figure out what's handling wireless connectivity and DHCP/IP assignment.

I will check for running services for what is managing the wireless device and network services


```bash
systemctl --type=service --state=running | grep -Ei 'iwd|wpa|network|dhcp|connman'
ip addr show wlan0
```
#### Output

```bash
dhcpcd.service           loaded active running DHCP/ IPv4LL/ IPv6RA/ DHCPv6 client on all interfaces
iwd.service              loaded active running Wireless service
systemd-networkd.service loaded active running Network Management
systemd-resolved.service loaded active running Network Name Resolution
```

```bash
4: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether e4:a7:a0:d1:4c:9b brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.29/24 brd 10.0.0.255 scope global dynamic noprefixroute wlan0
       valid_lft 170427sec preferred_lft 148827sec
    inet6 2601:1c2:301:cd4e::651d/128 scope global dynamic noprefixroute 
       valid_lft 46930sec preferred_lft 46930sec
    inet6 2601:1c2:301:cd4e:172f:40e:3129:7fb4/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 49307sec preferred_lft 49307sec
    inet6 fe80::718f:e89c:418d:4c7c/64 scope link 
       valid_lft forever preferred_lft forever
```

IWD is up and handling wifi connectivity. DHCPCD and NETWORKD however can both handle IP assigning and the OS recommends having only one service per interface


### 2. is networkd actually managing the wireless card?

    1. run ``` networkctl status wlan0 ```
    2. output ```
        Network File: n/a
        State: routable (unmanaged)
        ```
    Result. both dhcpcd and networkd are active, and I was worried there was a management conflict. but networkd is not dealing with network config for wlan0.
    thus, iwd = connects wlan0 to wifi network
    systemd-networkd = not managing wlan0
    dhcpcd = *probably* gives wlan0 its ip
    systemd-resolved = *probably* deals with dns

### 3. is dhcpcd managing wlan0's ip and route?

    1. run ``` journalctl -u dhcpcd -b | grep wlan0 | tail -n 20 ``` to see logs for what dhcpcd has been up to
    2. output ```
        Oct 13 23:11:32 machine dhcpcd[384]: wlan0: carrier acquired
        Oct 13 23:11:32 machine dhcpcd[384]: wlan0: IAID ...
        Oct 13 23:11:32 machine dhcpcd[384]: wlan0: adding address 
        Oct 13 23:11:34 machine dhcpcd[384]: wlan0: rebinding lease of 10.0.0.29
        Oct 13 23:11:34 machine dhcpcd[384]: wlan0: soliciting an IPv6 router
        Oct 13 23:11:34 machine dhcpcd[384]: wlan0: Router Advertisement from id
        Oct 13 23:11:34 machine dhcpcd[384]: wlan0: adding address id
        Oct 13 23:11:34 machine dhcpcd[384]: wlan0: adding route to id
        Oct 13 23:11:34 machine dhcpcd[384]: wlan0: adding default route via id
    ```
    3. thus dhcpcd is handling the ip/route
    4. AND the theory that dchpcd and networkd were fighting was wrong



### 3. why do i have to manually connect to wifi then if iwd is enabled and running at boot?

    1. does it remember connecting to wifi networks in the past?
        ```
        Known Networks
          BedsideManor                      open                    Jan  1,  4:45 PM     
          Starbucks WiFi                    open                    Aug  5, 10:13 PM     
        ```
    Answer. Yes.
    2. is autoconnect on?
        ``` iwctl known-networks network show ```
        ``` autoconnect yes ``` 
    Answer. Yes.
    So why do i have to reconect after every time i open my screen?
    3. What does the iwd log say?
    ``` journalctl -u iwd -b --no-pager ```
    Answer. It says it is autoconnecting already with some failed attempts.





## What I Learned

