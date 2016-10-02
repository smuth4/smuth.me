+++
date = "2016-10-02T15:58:01Z"
tags = ["networking", "proxmox"]
title = "Enabling jumbo frames on Proxmox 4"

+++

While turning on [jumbo frames](https://en.wikipedia.org/wiki/Jumbo_frame) on a basic interface is straightforward (`ip link set eth0 mtu 9000`), [Proxmox's](https://www.proxmox.com/) use of bridges to connect VMs makes things much more interesting. To start with, all interfaces connected to the bridge must have their MTUs upgraded first, otherwise it will give you an unhelpful error. Note that interfaces connected to the bridge include those of running containers/VMs.[^1]

```
# ip a | grep mtu
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master vmbr0 state UP group default qlen 1000
3: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
4: veth102i0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master vmbr0 state UP group default qlen 1000
# ip link set dev vmbr0 mtu 9000
RTNETLINK answers: Invalid argument
# ip link set dev eth0 mtu 9000
# ip link set dev veth102i0 mtu 9000
# ip link set dev vmbr0 mtu 9000
```

Note that setting the MTU on vmbr0 is technically unnecessary, since bridges inherit the smallest MTU of the slaved devices[^3].

However, this only enables it temporarily. The jumbo frames will be lost at the next reboot. To fix this, ideally we could simply define the MTU via the `/etc/network/interfaces` file. However, due to a series of bugs, I was not able to get this working, even with the `post-up` option.[^2][^4] My (hacky) solution was to write a simple script that goes through and raises the MTU for each interface assigned to a bridge, and run it every 5 minutes via cron.

``` bash
#!/bin/bash
intf="$1"
mtu="$2"

if [[ -z "$intf" || -z "$mtu" ]]; then
  echo "Usage: $0 interface mtu"
  exit 3
fi
        
for lower in /sys/devices/virtual/net/"${intf}"/lower_*; do
  child_intf="$(basename "$lower" | sed 's/lower_//')"
  if [[ "$(cat "$lower"/mtu)" -lt "$mtu" ]]; then
    ip link set "$child_intf" mtu 9000
  fi
done
```

Saving this to /usr/local/bin/br_mtu and running it as `/usr/local/bin/br_mtu vmbr0 9000` takes care of the host, and we can test that out with ping:

```
$ ping -c 5 -M do -s 8000 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 8000(8028) bytes of data.
8008 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=1.42 ms
8008 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=1.42 ms
8008 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=1.43 ms
8008 bytes from 192.168.1.1: icmp_seq=4 ttl=64 time=1.51 ms
8008 bytes from 192.168.1.1: icmp_seq=5 ttl=64 time=1.45 ms

--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4057ms
rtt min/avg/max/mdev = 1.425/1.449/1.512/0.058 ms
```

(Note that the network stack all the way to the target host will need to have jumbo frames enabling for this to work)

However, the containers/VMs will still need to be told to use jumbo frames on their internal interfaces. As long as it's enabled on the host, you should be able to simply follow your OS's instructions for doing so.

A word of warning: On one of my hosts, after a reboot while testing some of these changes, containers with "Start at boot" set to yes were unreachable, and I had to delete and re-add the interfaces for them to work correctly again. This is probably due to all the changes I was making, and not a direct result of changing the MTU.

### Sources
[^1]: https://joshua.hoblitt.com/rtfm/2014/05/dynamically_changing_the_mtu_of_a_linux_bridge_interface/
[^2]: http://askubuntu.com/questions/279362/how-do-i-set-a-network-bridge-to-have-an-mtu-of-9000
[^3]: https://lists.linuxfoundation.org/pipermail/bridge/2007-August/005488.html
[^4]: https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1399064
