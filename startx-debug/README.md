````markdown
## Problem

After logging into the console, running `startx` sometimes appeared to hang until I pressed `Ctrl+C` several times. Afterward, DWM would eventually start.

## Goal

Identify what was blocking X startup and fix the underlying problem rather than working around it.

## Diagnosis

### 1. Inspect `.xinitrc`

My `.xinitrc` contains a loop that updates the DWM status bar with the current date and time:

```bash
#!/bin/sh

while true; do
    xsetroot -name "$(date '+%Y-%m-%d %H:%M')"
    sleep 60
done &

exec dwm
```

Because the loop is backgrounded with `&`, it should not block execution from reaching:

```bash
exec dwm
```

**Result:** No obvious blocking command was found in `.xinitrc`.

### 2. Look for X session logs

There was no `~/.xsession-errors` file in my home directory.

I instead inspected the available Xorg logs, including:

```bash
/var/log/Xorg.0.log
```

### 3. Try to capture `startx` output

While already inside an active X/DWM session, I attempted to run `startx` again while redirecting its output to a file.

This failed because an X server was already running on display `:0`, and the new command was being executed from a pseudo-terminal rather than the original console.

That test did not reproduce the original startup problem, but it exposed another issue:

```bash
-rw------- 1 root root 675 Dec 15 2025 .Xauthority
-rw------- 2 user user   0 Oct 11 22:39 .Xauthority-c
-rw------- 2 user user   0 Oct 11 22:39 .Xauthority-l
```

Inspecting the main authority file showed:

```bash
Access: (0600/-rw-------)
Uid: (0/root)
Gid: (0/root)
```

The file belonged to `root`, even though X was normally being started as my regular user.

Because the file permissions were `0600`, my user could not modify it.

The failed startup attempt reported errors including:

```text
.Xauthority not writable
timeout in locking authority file
```

The `.Xauthority-c` and `.Xauthority-l` files were temporary lock files created during the failed attempt.

Additional session information confirmed that an X session was already active:

```bash
who
username tty1 date 11:01
```

```bash
tty
/dev/pts/4
```

```bash
echo "$DISPLAY"
:0
```

## Hypothesis

The root-owned `.Xauthority` file was preventing my normal user account from properly updating X authentication information during startup.

This could explain the intermittent delay and the unusual behavior when starting X.

## Fix

I changed ownership of `.Xauthority` back to my normal user:

```bash
sudo chown user:user ~/.Xauthority
```

After verifying that no `xauth` process was using the temporary lock files, I removed the stale locks:

```bash
rm -f ~/.Xauthority-c ~/.Xauthority-l
```

I then restarted the machine and launched X normally from `tty1`.

## Result

`startx` launched immediately.

The previous delay and repeated `Ctrl+C` behavior did not occur.

The root-owned `.Xauthority` file appears to have been the cause of the startup problem.

## What I Learned

- `.xinitrc` was not the source of the problem.
- Running `startx` from inside an existing X session is not a valid reproduction of console startup.
- File ownership can cause application failures even when the configuration itself is correct.
- Inspecting permissions and ownership is an important troubleshooting step when programs report that files are not writable.
````
