# Firewall 2

Okay, so a couple of things were bothering me with the [firewall](https://erroronsecurity.net/posts/2026-07-28/firewall/).
Namely:
* IPv6 kept re-enabling.
* SSH needed to be kicked on every reboot.
* I have 2 ISPs.

So, let's address these.


## IPv6
IPv6 is an annoying little protocol that has services and all kinds of magic wrapped into the protocol.
This is usually left unconfigured and represents massive risk. I'm deliberately choosing to configure an IPv4 network here. IPv6 shenanigans come later.

There are two common ways of disabling IPv6.
* The first, we tried. It's doing so via sysctl. Systemd-networkd unfortunately overrides this.
* The second is via systemd-networkd, which will blatantly ignore settings.

There is a secret third way we decided to employ, grub:
As you know, grub is the default bootloader for many a linux distribution. It can also pass a CMDLINE to the kernel to enable or disable specific behavior.

Edit /etc/default/grub to contain the following:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet ipv6.disable=1"
```

Then `update-grub` and `systemctl reboot` or `shutdown -r now`.

Great, that's taken care of.


## SSH
Alright, so you'll remember from the previous post that we had SSH listen on only internal address. Unfortunately, those addresses may or may not come up prior to SSH. That's annoying and locks me out more often than not.

So, what are we to do.
Change the line in /etc/ssh/sshd_config back to:
```
ListenAddress 0.0.0.0
```

So it binds to everything, including addresses yet to come up.

Unfortunately, this exposes ssh to the outside world... Let's add some stuff to /etc/nftables.conf:
```
table inet filter {
    chain input {
        type filter hook input priority filter;

        iifname enp7s0 tcp dport 22 drop
        iifname enp0s20f2 tcp dport 22 drop
        iifname external2.201 tcp dport 22 drop
    }
...
```

Kindof spoiling some stuff for later there, but that prevents ssh connections from my external interfaces.


## 2 ISPs
Okay, let's configure the external2 physical interface.
In /etc/systemd/network/external2.network, let's write:
```
[Match]
Name=enp0s20f2


[Network]
VLAN=external2.201
```

What's that do? It tells the system to use that as the physical interface for vlan 201, yet to be created (netdev files do come first...).

Then, in /etc/systemd/network/external2.201.netdev:
```
[NetDev]
Name=external2.201
Kind=vlan
MACAddress=aa:aa:aa:bb:bb:bb


[VLAN]
Id=201
```

This creates the actual VLAN interface.

Now, we can write to /etc/systemd/network/external2.201.network:
```
[Match]
Name=external2.201


[Network]
DHCP=ipv4
IPMasquerade=ipv4


[Route]
MultipathRouteWeight=1
```

We also need to add those last 2 lines to our external1.network file to enable some sweet automatic egress loadbalancing.


From there, `systemctl restart systemd-networkd` and boom. Should work like a charm, might require some fenangling.


And that's all 3 problems solved.
