# Dragonflybsd-nvmm host laptop using Ethernet

Guest with Ethernet host networking.
Provisioning is simple when using wired Ethernet on the host as Ethernet can support multiple IP addresses.

A. This Ethernet dragonfly6.5 snapshot laptop host install tested with a dragonfly6.5 snapshot guest.
	
  - Dragonfly guest uses dhclient for Ethernet configuration.

  - Host and guest use hammer2 filesystem.

  - Be sure to install qemu: ```pkg install qemu```.

## **_Step-by step procedure, assuming your dragonfly host Ethernet is re0:_**

1)  Modify /boot/loader.conf to add the line (or use "kldload nvmm" to load the kernel module):
```
nvmm_load="YES"
```

2) Create the bridge interface.
```
ifconfig bridge0 create
ifconfig bridge0 up
```

3) Add the following network tap on the host:
```
ifconfig tap0 create
ifconfig tap0 up
```

4) Allow non root user to use tap0.
```
chown $USER /dev/tap0
```

5) Add the tap0 interface and host Ethernet interface (re0 here) to the bridge.
```
ifconfig bridge0 addm re0
ifconfig bridge0 addm tap0
```

5.1) Modify sysctl: this must be done after creating the tap0 interface.
```
sysctl net.link.tap.up_on_open=1
sysctl net.link.tap.user_open=1
```
(add permanently in /etc/sysctl.conf -> net.link.tap.up_on_open=1, net.link.tap.user_open=1)

>[!Caution]
>Security note: This bridged network topology is a critical vulnerability. By adding re0 to the bridge promiscuous mode is enabled.
>This exposes the virtual machine to the local network making it accessible to all. Use this configuration with caution, or if the guest machine must be secure and require Internet access consider using network address translation (NAT), e.g. set up pf.

6) Ensure Tiger VNC Viewer (```pkg install tigervnc-viewer```) or GTK-VNC Viewer (```pkg install gtk-vnc```) is installed.

7) To create dfly.qcow2 disk of size 20G:
```
qemu-img create -f qcow2 dfly.qcow2 20G
```

8) Start your guest virtual mavhine. Be sure tap0 is up. Run:
```
ifconfig tap0 up
```
first.

```
qemu-system-x86_64 \
	-accel nvmm \
        -cpu max -smp cpus=4 -m 4G \
	-hda /vms/dfly.qcow2 \
	-boot menu=on \
	-netdev user,id=net0,hostfwd=tcp:127.0.0.1:6022-:22 \
	-device virtio-net-pci,netdev=net0 \
  	-object rng-random,id=rng0,filename=/dev/urandom \
  	-device virtio-rng-pci,rng=rng0 \
	-display curses \
	-vga qxl \
	-spice addr=127.0.0.1,port=5900,ipv4=on,disable-ticketing=on,seamless-migration=on \
	-display vnc=unix:/home/$USER/.qemu-myvm-vnc -vga vmware \
	-usb \
	-device usb-tablet \
	-device usb-mouse,bus=usb-bus.0 \
	-cdrom /vms/DragonFly-x86_64-LATEST-ISO.iso \
	-boot d
```

This will provide a boot menu. Boot from the cdrom and follow the directions to install from the iso file (option 1) to the disk (dfly.qcow2). The virtual console provides a login shell. If one plans on running Xorg in the guest see #10.

>[!Note]
>Choose the MBR boot method during installation to disk, not UEFI (otherwise your new virtual machine will not boot because no UEFI partition was created and provisioned). Once you have installed on disk, reboot into the newly installed disk image (option 3).

9) Configure dhcp networking on the guest virtual machine.
```
dhclient vtnet0
```

Add ```ifconfig_vtnet0="DHCP"``` to /etc/rc.conf to make it permanent.

Check by running ```netstat -nr```  and ```ifconfig -a```

```
Routing tables                                  

Internet:           
Destination        Gateway            Flags    Refs      Use  Netif Expire
default            10.0.2.2           UGSc        4        0 vtnet0
10.0.2/24          link#1             UC          2        0 vtnet0
10.0.2.2           52:55:0a:00:02:02  UHLW        5        0 vtnet0    977
10.0.2.3           52:55:0a:00:02:03  UHLW        0        0 vtnet0    971
127.0.0.1          127.0.0.1          UH          0        0    lo0
```

```
vtnet0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=28<VLAN_MTU,JUMBO_MTU>
        ether 52:54:00:12:34:56
        inet6 fe80::5054:ff:fe12:3456qvtnet0 prefixlen 64 scopeid 0x1
        inet 10.0.2.15 netmask 0xffffff00 broadcast 10.0.2.255
        media: Ethernet 1000baseT <full-duplex>
        status: active
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> metric 0 mtu 16384
        options=43<RXCSUM,TXCSUM,RSS>
        inet 127.0.0.1 netmask 0xff000000
        inet6 ::1 prefixlen 128
        inet6 fe80::1qlo0 prefixlen 64 scopeid 0x2
        groups: lo
```

10) Connecting to the virtual machine from the host. TBD: ssh in /etc/ssh must be setup first.
```
spicy -p 5900
```
```
ssh -p 8719 $USER@127.0.0.1
```

Tigervnc TBD.


>[!Note]
> This network configuration, bridged re0, also works for FreeBSD 14-STABLE, 14.1, and current where the guest is bridged to a hardwired Ethernet interface.




