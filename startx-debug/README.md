Problem:
On login, running `startx` sometimes appears to hang until I press Ctrl+C several times. After that, DWM starts normally.

Goal:
Identify what is blocking startup and fix it without hiding the underlying issue.

---Diagnose
1 check the .xintrc file for any instructions that could hang it up
    ----there is basically a while loop that sets up my status bar for dmw with date/time and it is backgrounded so this should not cause a hang up
2 check the xsession-errors logs in my home directory
