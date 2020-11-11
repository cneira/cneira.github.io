---
author: Carlos Neira
title: Run Bhyve in a FreeBSD Jail
---

# How to run bhyve in a jail

I\'ll setup a jail dedicated to run bhyve vms , for jail creation I\'ll
use bastillebsd

Steps:

## Install bastillebsd to create and manage jails.

```{.bash org-language="sh"}
# pkg install bastillebsd
```

## Setup bastillebsd

Follow the getting started guide at
<https://bastillebsd.org/getting-started/> I\'m using zfs so
/usr/local/etc/bastille/bastille.conf I must edit bastille.conf (this
must be done before bootstraping a release).

I used this on bastille.conf.

```{.text}
bastille_zfs_enable="YES"
bastille_zfs_zpool="zroot"
```

Create a set of rules to allow to run bhyve inside a jail edit
/etc/devfs.rules, create it if does not exists.

```{.text}
[devfs_rules_bhyve_jail=25]
add include $devfsrules_jail
add path vmm unhide
add path vmm/* unhide
add path tap* unhide
add path mem unhide
add path kmem unhide
add path nmdm* unhide
add path pci unhide
add path io unhide

```

Create a new jail that will be use these rules.

```{.shell}
sudo bastille create --vnet test-bhyve 12.2-RELEASE 192.168.1.225 em0
```

Modify test-bhyve jail.conf for this jail:

```{.shell}
sudo bastille edit test-bhyve
```

Now add

```{.text}
allow.vmm;
```

So the jail.conf will look like:

```{.text}
test-bhyve {
  devfs_ruleset =25;
  enforce_statfs = 2;
  exec.clean;
  exec.consolelog = /var/log/bastille/test-bhyve_console.log;
  exec.start = '/bin/sh /etc/rc';
  exec.stop = '/bin/sh /etc/rc.shutdown';
  host.hostname = test-bhyve;
  mount.devfs;
  mount.fstab = /usr/local/bastille/jails/test-bhyve/fstab;
  path = /usr/local/bastille/jails/test-bhyve/root;
  securelevel = 2;
  allow.vmm;
  allow.raw_sockets;
  vnet;
  vnet.interface = e0b_bastille0;
  exec.prestart += "jib addm bastille0 em0";
  exec.poststop += "jib destroy bastille0";
}
~
```

Load the required modules:

```{.shell}
kldload vmm
kldload nmdm
```

Start test-bhyve jail

```{.shell}
sudo bastille start test-bhyve
```

Go inside the new jail

```{.shell}
sudo bastille console test-bhyve
```

Now install vm-bhyve inside the jail follow the vm-bhyve setup at
<https://github.com/churchers/vm-bhyve>

```{.shell}
pkg install vm-bhyve
sysrc vm_enable="YES"
mkdir /vms
sysrc vm_dir="/vms"
vm init
cp /usr/local/share/examples/vm-bhyve/* /vms/.templates/
vm switch create public
vm switch add public vtnet0
vm iso https://download.freebsd.org/ftp/releases/ISO-IMAGES/12.2/FreeBSD-12.2-RELEASE-amd64-bootonly.iso
vm create test
vm install -f test FreeBSD-12.2-RELEASE-amd64-bootonly.iso
```

by default a vm-bhyve vm has 256mb, if you need more you will need to
run

```{.shell}
vm config test
```

And configure how much RAM do you need.

## Issues

If you are going to use dhcp on the vm, you will need to configure the
interface to use SYNCDHCP. According to this post using SYNCDHCP works,
but we need to reboot the vm first.

> This gives me an interface named vnet0 in my jails that i can then
> configure through the jail\'s rc.conf. for some reason SYNCDHCP works
> but not DHCP in the guest rc.conf.

## PCI passthrough

We need to mask the device which will be passed to the guest, to do that
we need to add an entry to /boot/loader.conf. First let\'s identify the
device, you could use

```{.shell}
root@test-bhyve:~ pciconf -lv
subclass   = ethernet
igb0@pci0:1:0:0:        class=0x020000 card=0x12a28086 chip=0x150e8086 rev=0x01 hdr=0x00
    vendor     = 'Intel Corporation'
    device     = '82580 Gigabit Network Connection'
    class      = network

```

or just use vm-bhyve passthru command

```{.shell}
root@test-bhyve:~ # vm passthru
DEVICE     BHYVE ID     READY        DESCRIPTION
hostb0     0/0/0        No           Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor Host Bridge/DRAM Registers
pcib1      0/1/0        No           Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor PCIe Controller (x16)
xhci0      0/20/0       No           100 Series/C230 Series Chipset Family USB 3.0 xHCI Controller
none0      0/22/0       No           100 Series/C230 Series Chipset Family MEI Controller
none1      0/22/3       No           100 Series/C230 Series Chipset Family KT Redirection
ahci0      0/23/0       No           Q170/Q150/B150/H170/H110/Z170/CM236 Chipset SATA Controller [AHCI Mode]
pcib2      0/29/0       No           100 Series/C230 Series Chipset Family PCI Express Root Port
isab0      0/31/0       No           C236 Chipset LPC/eSPI Controller
none2      0/31/2       No           100 Series/C230 Series Chipset Family Power Management Controller
hdac1      0/31/3       No           100 Series/C230 Series Chipset Family HD Audio Controller
none3      0/31/4       No           100 Series/C230 Series Chipset Family SMBus
em0        0/31/6       No           Ethernet Connection (2) I219-LM
igb0       1/0/0        No           82580 Gigabit Network Connection
igb1       1/0/1        No           82580 Gigabit Network Connection
igb2       1/0/2        No           82580 Gigabit Network Connection
igb3       1/0/3        No           82580 Gigabit Network Connection
vgapci0    2/0/0        No           GM107GL [Quadro K620]

```

I\'ll pass igb0 to the guest and is identified as 1/0/0, so I\'ll add
this

```{.text}
pptdevs="1/0/0"
```

to my /boot/loader.conf, to apply then reboot. The device should be
available to the guest vm as ppt0.

```{.shell}
ppt0@pci0:1:0:0:    class=0x020000 card=0x12a28086 chip=0x150e8086 rev=0x01 hdr=0x00
    vendor     = 'Intel Corporation'
    device     = '82580 Gigabit Network Connection'
    class      = network
    subclass   = ethernet
```

Now we need to edit the vm configuration to allow to passthru the device
Here is mine, the ones we need to add are **wired_memory** and
**passthru0**.

```{.shell}
loader="bhyveload"
cpu=2
memory=4G
wired_memory="yes"
network0_type="virtio-net"
network0_switch="public14"
disk0_type="virtio-blk"
disk0_name="disk0.img"
uuid="a179833c-204e-11eb-97b6-90e2ba4673cc"
network0_mac="58:9c:fc:0c:fa:98"
passthru0="1/0/0"
debug="yes"
```

## Errors

```{.shell}
bhyve: failed to open /dev/pci: Operation not permitted
device emulation initialization error: No such file or directory
```

This is because our jail has a securelevel \> 0 so it cannot access the devices.  
Modify the jail\'s jail.conf file to include

```{.shell}
securelevel = -1;
```

But at the end it won\'t work as jails\' currently don\'t have
_PRIV_IO_ nor _KMEM_WRITE_ access, but this simple patch will
allow you to run bhyve with passthrough in jails. I have reported the
bug as : <https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=251046>  
If you need passthru now you could use the following patch for
12.2-RELEASE.

```{.shell}
cneira@cl-west-00:/usr/src $ cat ~/bhyve-in-jail.patch
Index: sys/kern/kern_jail.c
===================================================================
--- sys/kern/kern_jail.c    (revision 367580)
+++ sys/kern/kern_jail.c    (working copy)
@@ -3319,7 +3319,9 @@
         * able to read kernel/phyiscal memory (provided /dev/[k]mem
         * exists in the jail and they have permission to access it).
         */
+   case PRIV_IO:
    case PRIV_KMEM_READ:
+   case PRIV_KMEM_WRITE:
        return (0);

        /*
```

After that booting will show igb0 in your guest vm.

```{.text}
/boot/kernel/kernel text=0x16bdcc4 data=0x140 data=0x75fe80 syms=[0x8+0x17e098+0x8+0x19bdd3]
Loading configured modules...
/boot/kernel/zfs.ko size 0x3bad38 at 0x247b000
loading required module 'opensolaris'
/boot/kernel/opensolaris.ko size 0xa448 at 0x2836000
/etc/hostid size=0x25
/boot/entropy size=0x1000
---<<BOOT>>---
Copyright (c) 1992-2020 The FreeBSD Project.
Copyright (c) 1979, 1980, 1983, 1986, 1988, 1989, 1991, 1992, 1993, 1994
    The Regents of the University of California. All rights reserved.
FreeBSD is a registered trademark of The FreeBSD Foundation.
FreeBSD 12.2-RELEASE r366954 GENERIC amd64
FreeBSD clang version 10.0.1 (git@github.com:llvm/llvm-project.git llvmorg-10.0.1-0-gef32c611aa2)
VT: init without driver.
CPU: Intel(R) Xeon(R) CPU E3-1225 v5 @ 3.30GHz (3312.19-MHz K8-class CPU)
  Origin="GenuineIntel"  Id=0x506e3  Family=0x6  Model=0x5e  Stepping=3
  Features=0x9f83fbff<FPU,VME,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,CMOV,PAT,PSE36,MMX,FXSR,SSE,SSE2,SS,HTT,PBE>
  Features2=0xfeda7a17<SSE3,PCLMULQDQ,DTES64,DS_CPL,SSSE3,SDBG,FMA,CX16,xTPR,PCID,SSE4.1,SSE4.2,MOVBE,POPCNT,AESNI,XSAVE,OSXSAVE,AVX,F16C,RDRAND,HV>
  AMD Features=0x24100800<SYSCALL,NX,Page1GB,LM>
  AMD Features2=0x121<LAHF,ABM,Prefetch>
  Structured Extended Features=0x40f39<FSGSBASE,BMI1,HLE,AVX2,BMI2,ERMS,INVPCID,RTM,RDSEED>
  XSAVE Features=0x1<XSAVEOPT>
  TSC: P-state invariant
Hypervisor: Origin = "bhyve bhyve "
real memory  = 5368709120 (5120 MB)
avail memory = 4096118784 (3906 MB)
Event timer "LAPIC" quality 600
ACPI APIC Table: <BHYVE  BVMADT  >
FreeBSD/SMP: Multiprocessor System Detected: 2 CPUs
FreeBSD/SMP: 2 package(s) x 1 core(s)
random: unblocking device.
ioapic0 <Version 1.1> irqs 0-31 on motherboard
Launching APs: 1
random: entropy device external interface
kbd1 at kbdmux0
000.000023 [4336] netmap_init               netmap: loaded module
[ath_hal] loaded
module_register_init: MOD_LOAD (vesa, 0xffffffff81115e40, 0) error 19
random: registering fast source Intel Secure Key RNG
random: fast provider: "Intel Secure Key RNG"
nexus0
cryptosoft0: <software crypto> on motherboard
acpi0: <BHYVE BVXSDT> on motherboard
acpi0: Power Button (fixed)
atrtc0: <AT realtime clock> port 0x70-0x71 irq 8 on acpi0
atrtc0: registered as a time-of-day clock, resolution 1.000000s
Event timer "RTC" frequency 32768 Hz quality 0
attimer0: <AT timer> port 0x40-0x43 irq 0 on acpi0
Timecounter "i8254" frequency 1193182 Hz quality 0
Event timer "i8254" frequency 1193182 Hz quality 100
hpet0: <High Precision Event Timer> iomem 0xfed00000-0xfed003ff on acpi0
Timecounter "HPET" frequency 16777216 Hz quality 950
Event timer "HPET" frequency 16777216 Hz quality 550
Event timer "HPET1" frequency 16777216 Hz quality 450
Event timer "HPET2" frequency 16777216 Hz quality 450
Event timer "HPET3" frequency 16777216 Hz quality 450
Event timer "HPET4" frequency 16777216 Hz quality 450
Event timer "HPET5" frequency 16777216 Hz quality 450
Event timer "HPET6" frequency 16777216 Hz quality 450
Timecounter "ACPI-fast" frequency 3579545 Hz quality 900
acpi_timer0: <32-bit timer at 3.579545MHz> port 0x408-0x40b on acpi0
pcib0: <ACPI Host-PCI bridge> port 0xcf8-0xcff on acpi0
pcib0: could not evaluate _ADR - AE_NOT_FOUND
pci0: <ACPI PCI bus> on pcib0
pcib0: no PRT entry for 0.6.INTA
virtio_pci0: <VirtIO PCI Block adapter> port 0x2000-0x207f mem 0xc0000000-0xc0001fff irq 16 at device 4.0 on pci0
vtblk0: <VirtIO Block Adapter> on virtio_pci0
vtblk0: 20480MB (41943040 512 byte sectors)
virtio_pci1: <VirtIO PCI Network adapter> port 0x2080-0x209f mem 0xc0002000-0xc0003fff irq 17 at device 5.0 on pci0
vtnet0: <VirtIO Networking Adapter> on virtio_pci1
vtnet0: Ethernet address: 58:9c:fc:0c:fa:98
vtnet0: netmap queues/slots: TX 1/1024, RX 1/512
000.000781 [ 445] vtnet_netmap_attach       vtnet attached txq=1, txd=1024 rxq=1, rxd=512
igb0: <Intel(R) PRO/1000 PCI-Express Network Driver> mem 0xc0080000-0xc00fffff,0xc0100000-0xc0103fff irq 16 at device 6.0 on pci0
igb0: Using 1024 TX descriptors and 1024 RX descriptors
igb0: Using 2 RX queues 2 TX queues
igb0: Using MSI-X interrupts with 3 vectors
igb0: Ethernet address: 90:e2:ba:46:73:cc
igb0: netmap queues/slots: TX 2/1024, RX 2/1024
isab0: <PCI-ISA bridge> at device 31.0 on pci0
isa0: <ISA bus> on isab0
atkbdc0: <Keyboard controller (i8042)> port 0x60,0x64 irq 1 on acpi0
atkbd0: <AT Keyboard> irq 1 on atkbdc0
kbd0 at atkbd0
atkbd0: [GIANT-LOCKED]
driver bug: Unable to set devclass (class: atkbdc devname: (unknown))
psm0: <PS/2 Mouse> irq 12 on atkbdc0
psm0: [GIANT-LOCKED]
psm0: model Generic PS/2 mouse, device ID 0
uart0: <16550 or compatible> port 0x3f8-0x3ff irq 4 flags 0x10 on acpi0
uart0: console (9600,n,8,1)
uart1: <16550 or compatible> port 0x2f8-0x2ff irq 3 on acpi0
vga0: <Generic ISA VGA> at port 0x3b0-0x3bb iomem 0xb0000-0xb7fff pnpid PNP0900 on isa0
ZFS NOTICE: Prefetch is disabled by default if less than 4GB of RAM is present;
            to enable, add "vfs.zfs.prefetch_disable=0" to /boot/loader.conf.
ZFS filesystem version: 5
ZFS storage pool version: features support
```

## References

- <https://github.com/lattera/articles/blob/master/freebsd/2018-10-27_jailed_bhyve/article.md>

- <https://github.com/churchers/vm-bhyve/issues/267>

- <https://bastillebsd.org/getting-started/>

- <https://github.com/churchers/vm-bhyve>

- <https://www.freebsd.org/cgi/man.cgi?query=security(7)&sektion=&manpath=freebsd-release-ports>
