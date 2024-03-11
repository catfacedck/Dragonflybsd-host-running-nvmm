## Dragonflybsd-nvmm host laptop using Ethernet

Guest with Ethernet host networking.
Provisioning is simple when using wired Ethernet on the host as Ethernet can support multiple IP addresses.

A. This Ethernet dragonfly6.5 snapshot laptop host install tested with a dragonfly6.5 snapshot guest.
	
  - Dragonfly guest uses dhclient for Ethernet configuration.

  - Host and guest use hammer2 filesystem.

**_Step-by step procedure, assuming your dragonfly host Ethernet is re0:_**

1)  Modify /boot/loader.conf to add the line (or use "kldload nvmm" to load the kernel module):
```
nvmm_load="YES"
```

2) Modify sysctl:
```
sysctl net.link.tap.up_on_open=1
sysctl net.link.tap.user_open=1
```
(add permanently in /etc/sysctl.conf -> net.link.tap.up_on_open=1, net.link.tap.user_open=1)

3) Create the bridge interface.
```
ifconfig bridge0 create
ifconfig bridge0 up
```

4) Add the following network tap on the host:
```
ifconfig tap0 create
ifconfig tap0 up
```

5) Allow non root user to use tap0.
```
chown $USER /dev/tap0
```

6) Add the tap0 interface and host Ethernet interface (re0 here) to the bridge.
```
ifconfig bridge0 addm re0
ifconfig bridge0 addm tap0
```

**Security notes: by adding re0 to the bridge will enable promiscuous mode on it (e.g. it can be sniffed).**

**This bridge networking method exposes your virtual machine to the local area network and makes it accessible to all other networked machines, in addition to the host. This creates two risks: for the guest virtual machine itself, and exposes information from the guest machine to the local network and the Internet.**

**If the guest machines must be secure and require Internet access consider using network address reanslation (NAT).**

