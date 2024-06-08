# Feature 2 - Tailscale VPN

To access your PiKVM for anywhere you can use Tailscale VPN.

With Tailscale you dont need nat, portforwaring etc... 


Here is a copy of Tailscale guide

Enable read-write mode.
```
rw
```


Install the tailscale-pikvm package.

```
[root@pikvm ~]# pacman -Sy tailscale-pikvm
```

Enable and start the Tailscale service.

```
[root@pikvm ~]# systemctl enable --now tailscaled
Created symlink /etc/systemd/system/multi-user.target.wants/tailscaled.service -> /usr/lib/systemd/system/tailscaled.service.
```

Log in to Tailscale using the link provided by the command

```
[root@pikvm ~]# tailscale up

To authenticate, visit:

        https://login.tailscale.com/a/a6b12<snip>>1f80c


To approve your machine, visit (as admin):

        https://login.tailscale.com/admin

Success.
Some peers are advertising routes but --accept-routes is false
```

Enable SSH

```
[root@pikvm ~]# tailscale set --ssh
```

Verify that you have a address an test from another device connected to the tailscale network


```
[root@pikvm ~]# ip add
<snip>
5: tailscale0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1280 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet <snip> scope global tailscale0
       valid_lft forever preferred_lft forever
    inet6 <snip> scope global 
       valid_lft forever preferred_lft forever
    inet6 <snip> scope link stable-privacy proto kernel_ll 
       valid_lft forever preferred_lft forever
```
    