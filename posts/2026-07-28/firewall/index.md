# Firewall
Creating a firewall from a basic Debian install.


## Requirements
* IPv6 disabled.
* DHCPv4 on external interface.
* Static IP on internal interface.
* IP forwarding enabled.
* NAT / masquerade setup between interfaces.
* Related, Established connection state rules.
* OpenSSH on internal interface.


### IPv6 disabled (via sysctl)
Create or append to /etc/sysctl.conf:
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Then run:
```
sysctl -p
```


### DHCPv4 on external interface
Create a systemd-networkd network file such as /etc/systemd/network/external.network:
```
[Match]
Name=enp7s0

[Network]
DHCP=yes
```
Enable systemd-networkd with `systemctl enable systemd-networkd`.

Now install systemd-resolved with `apt install -y systemd-resolved`.
Then edit /etc/systemd/resolved.conf:
```
DNS=8.8.8.8
DNS=8.8.4.4
LLMNR=no
```
Then restart systemd-resolved with `systemctl restart systemd-resolved`.



### Static IP on internal interface
Create another systemd-networkd file such as /etc/systemd/network/internal.network:
```
[Match]
Name=enp8s0

[Network]
Address=172.16.0.1/24
```

Restart systemd-networkd again: `systemctl restart systemd-networkd`


### IP Forwarding
Back to /etc/sysctl.conf, append:
```
net.ipv4.ip_forward = 1
net.ipv4.conf.all.forwarding = 1
```

Then again, `sysctl -p`.


### NAT / Masquerade setup between interfaces (with nftables)
Add the nat table to /etc/nftables.conf:
```
table inet nat {
    chain postrouting {
        type nat hook postrouting priority srcnat
        policy drop

        oifname enp7s0 masquerade
    }
}
```


### Related, Established connection state rules
Add the forwarding filter to /etc/nftables.conf:
(the `table inet filter` and `chain forward` blocks should already exist)
```
chain forward {
    type filter hook forward priority filter
    policy drop

    ct state related,established accept
    iifname enp8s0 oifname enp7s0 accept
}
```
Note: This allows anything out and anything related or established back in. Not very secure.

Now, enable and restart nftables: `systemctl enable nftables; systemctl restart nftables`


### OpenSSH on internal interface
Install the server: `apt install -y openssh-server`
Edit /etc/ssh/sshd_config and set:
```
ListenAddress=172.16.0.1
```
Then restart it with `systemctl restart sshd`


And just like that, you've got yourself a firewall...
