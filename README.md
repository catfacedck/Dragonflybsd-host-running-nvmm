# Dragonflybsd-nvmm host laptop using Ethernet

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
(add permanently in /etc/sysctl.conf -> net.link.tap.up_on_open=1)

3) Create the bridge interface.
   ```
   ifconfig bridge0 create
   ifconfig bridge0 up
   ```

4) 
