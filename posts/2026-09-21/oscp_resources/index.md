# Some OSCP Resources
This post was written when I was prepping for the OSCP. Preserving the resources here.

Good OSCP guidance: https://johnjhacking.com/blog/oscp-reborn-2023

This, in turn links to several other good resources.

NMAPAutomator: https://github.com/21y4d/nmapAutomator/

GTFO and LOL bins:

    https://gtfobins.github.io

    https://lolbas-project.github.io

Reverse Shell generator (you shouldn’t need this, but it can be a good reference for those pesky one-liners): https://www.revshells.com/

Playing with Active Directory: https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md

PrivEsc classes:

    https://academy.tcm-sec.com/p/windows-privilege-escalation-for-beginners

    https://academy.tcm-sec.com/p/linux-privilege-escalation

Advice:

    always try default credentials.
    with web servers, virtual hosts are a pain in the ass, always check if the response you get from getting the IP is different from getting the hostname.
    note / != /index.html != /index.php
    always have a statically compiled busybox binary: https://busybox.net
    always have a windows vm ready in case you need to compile or test anything.
    always check both tcp and udp.
    searchsploit for parts of an app name, they often have multiple names in the database.
    dirbust subdirectories, you never know what you’ll find.
    check AppData in post exploitation.
    with hashes, when you can’t crack it, pass it.
    write your own exploits. 100% reference other people’s code, but write it yourself so you understand it.

SQL Injection cheat sheets:

    https://portswigger.net/web-security/sql-injection/cheat-sheet
    https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet/

Hack The Box Active Directory Practice Machines:

* Active
* Forest
* Sauna
* Monteverde
* Timelapse
* Flight
* Return
* Blackfield
* Cicada
* Escape

Windows Registry cheat sheet: https://github.com/d3fenderz/windows_reg

Query autologin creds with:

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\winlogon"
```

Kernel exploits:

    https://github.com/SecWiki/windows-kernel-exploits
    https://github.com/SecWiki/linux-kernel-exploits
