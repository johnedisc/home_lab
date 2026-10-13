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

### 1. Inspect `.xinitrc`


### 2. Look for X session logs


### 3. Try to capture `startx` output


## Result


## What I Learned

