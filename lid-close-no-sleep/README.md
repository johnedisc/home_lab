# Lid Close Problem

## Problem

Closing my Dell's lid doesn't put machine to sleep

## Goal

When I close the laptop lid, I want Arch Linux to suspend the computer. When I reopen it, I want the existing session to resume normally.

## Diagnosis

### 1. does Linux detect the lid state?

**Why I checked this:**

Before checking any behavior settings, we need to know if the kernal recognizes the lid switch.

**Command:**

```bash
cat /proc/acpi/button/lid/*/state
```

**Result:**

```text
state:      open
```

### What did this show me?

The kernel is aware that the lid is currently open.

### What does this mean?

We need now to log repeatedly whether the lid is open and then close the lid for 10 seconds and read the log.

**Command:**

```bash
for i in {1..15}; do
    date '+%H:%M:%S'
    cat /proc/acpi/button/lid/*/state
    sleep 1
done > ~/lid-test.txt
```

**Result:**

```text
23:24:59
state:      open
23:25:00
state:      open
23:25:01
state:      open
23:25:02
state:      open
23:27:11
state:      open
23:27:12
state:      open
23:27:13
state:      open
23:27:14
state:      open
```

### What did this show me?

It is unclear still what it shows. It was supposed to record every second and it did and then stopped for 2 minutes before resuming.

### What does this mean?

This suggests that machine may have stopped recording data because it was asleep, but we need more data.

---

### 2. Check system logs for evidence of suspension

**Why I checked this:**

This gives definitely evidence of whether Arch thinks it was suspended when I closed the lid

**Command:**

```bash
journalctl -b | grep -Ei 'suspend|sleep|resume' | tail n -20
```

**Result:**

```text
Oct 15 23:25:03 <user> systemd-logind[372]: Suspending...
Oct 15 23:25:03 <user> systemd[1]: Reached target Sleep.
Oct 15 23:25:03 <user> systemd[1]: Starting System Suspend...
Oct 15 23:25:03 <user> systemd-sleep[36980]: Successfully froze unit 'user.slice'.
Oct 15 23:25:03 <user> systemd-sleep[36980]: Performing sleep operation 'suspend'...
Oct 15 23:25:03 <user> kernel: PM: suspend entry (deep)
Oct 15 23:27:11 <user> kernel: printk: Suspending console(s) (use no_console_suspend to debug)
Oct 15 23:27:11 <user> kernel: ACPI: PM: Preparing to enter system sleep state S3
Oct 15 23:27:11 <user> kernel: ACPI: PM: Low-level resume complete
Oct 15 23:27:11 <user> kernel: ACPI: PM: Waking up from system sleep state S3
Oct 15 23:27:11 <user> kernel: xhci_hcd 0000:00:14.0: xHC error in resume, USBSTS 0x411, Reinit
Oct 15 23:27:11 <user> systemd-sleep[36980]: System returned from sleep operation 'suspend'.
Oct 15 23:27:11 <user> kernel: PM: suspend exit
Oct 15 23:27:11 <user> systemd-sleep[36980]: Successfully thawed unit 'user.slice'.
Oct 15 23:27:11 <user> systemd[1]: systemd-suspend.service: Deactivated successfully.
Oct 15 23:27:11 <user> systemd[1]: Finished System Suspend.
Oct 15 23:27:11 <user> systemd[1]: Stopped target Sleep.
Oct 15 23:27:11 <user> systemd[1]: Reached target Suspend.
Oct 15 23:27:11 <user> systemd[1]: Stopped target Suspend.
Oct 15 23:27:11 <user> systemd-logind[372]: Operation 'suspend' finished.
```

### What did this show me?

The log shows suspension at 25:03 and ending suspension at 27:11. These correspond when exactly when I closed and opened the lid.

### What does this mean?

The system is suspending when I close the lid, and so the behavior of the session resume when I open the lid with no password prompt was not evidence that it wasn't sleeping. I have to investigate further.

---

## Fix

[Describe what I changed and why.]

```bash
[commands/configuration changes]
```

## Verification

[Explain how I tested whether the fix actually worked.]

```text
[test results]
```

## Result

[State whether the problem was solved and what changed.]

## What I Learned

- [Technical concept]
- [Troubleshooting lesson]
- [Something I now understand about how the system works]
