## Problem:
On login, running `startx` sometimes appears to hang until I press Ctrl+C several times. After that, DWM starts normally.

## Goal:
Identify what is blocking startup and fix it without hiding the underlying issue.

### Diagnose
1 check the .xinitrc file for any instructions that could hang it up
'bash
#!/bin/sh

while true; do
    xsetroot -name "$(date '+%Y-%m-%d %H:%M')"
    sleep 60
done &

exec dwm'
#### there is basically a while loop that sets up my status bar for dwm with date/time and it is backgrounded so this should not cause a hang up
2 check the xsession-errors logs in my home directory
#### xsession-errors not present. check var/log/Xorg.0.log instead. lots of information there.
3 try loading startx and logging it to file instead of stdout while in an open X session
#### xsession wont load while currently in an existing x session. furthermore .xauthority seems to have permissions issues. here is the stdout
'bash
-rw------- 1 root root 675 Dec 15  2025 .Xauthority
-rw------- 2 krys krys   0 Oct 11 22:39 .Xauthority-c
-rw------- 2 krys krys   0 Oct 11 22:39 .Xauthority-l
'
'bash
stat ~/.Xauthority
  File: .Xauthority
  Size: 675       	Blocks: 8          IO Block: 4096   regular file
Device: 8,3	Inode: 1048875     Links: 1
Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2026-10-11 22:33:45.838031098 -0700
Modify: 2025-12-15 06:56:05.983989365 -0800
Change: 2025-12-15 06:56:05.983989365 -0800
 Birth: 2025-12-15 06:56:05.983006088 -0800
  File: .Xauthority-c
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: 8,3	Inode: 1048582     Links: 2
Access: (0600/-rw-------)  Uid: ( 1000/    krys)   Gid: ( 1000/    krys)
Access: 2026-10-11 22:39:12.834044787 -0700
Modify: 2026-10-11 22:39:12.834044787 -0700
Change: 2026-10-11 22:39:12.835359952 -0700
 Birth: 2026-10-11 22:39:12.834044787 -0700
  File: .Xauthority-l
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: 8,3	Inode: 1048582     Links: 2
Access: (0600/-rw-------)  Uid: ( 1000/    krys)   Gid: ( 1000/    krys)
Access: 2026-10-11 22:39:12.834044787 -0700
Modify: 2026-10-11 22:39:12.834044787 -0700
Change: 2026-10-11 22:39:12.835359952 -0700
 Birth: 2026-10-11 22:39:12.834044787 -0700
'
'bash
who
username     tty1         date 11:01
'
'bash
tty
/dev/pts/4
'
'bash
echo "$DISPLAY"
:0
'

