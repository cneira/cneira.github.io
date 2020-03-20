I thought of leveraging zvols to expose rootfs to firecracker, so this is really simple.  
* First download a template image from OpenVZ
[http://download.openvz.org/template/precreated/centos-7-x86_64-minimal.tar.gz](http://download.openvz.org/template/precreated/centos-7-x86_64-minimal.tar.gz)

* Create a ZVOL to host this tarball 
```bash
# zfs create -V 1G  zpool/disk1 
# mkfs.ext4 -E /dev/zvol/zpool/disk1
# mount -t ext4 /mnt /dev/zvol/zpool/disk1
# tar xfvz centos-7-x86_64-minimal.tar.gz -C /mnt
# zfs snapshot zpool/disk1@final 
# zfs send zpool/disk1@final > disk1.img  
```
On your other system just receive it
```bash
# zfs receive zpool/disk1 < disk1.img
```
Now just run firecracker and point rootfs to that zvol :
```bash
alpine-build:~/firecracker/firecracker$ cat runvm.sh 
 ./firectl 
   --kernel=hello-vmlinux.bin 
   --root-drive=/dev/zvol/zpool/disk1 
     --firecracker-binary=./firecracker
alpine-build:~/firecracker/firecracker$ sudo sh runvm.sh 
 INFO[0000] Called startVMM(), setting up a VMM on /root/.firecracker.sock-8764-81 
 INFO[0000] VMM logging and metrics disabled.            
 INFO[0000] refreshMachineConfiguration: [GET /machine-config][200] getMachineConfigurationOK  &{CPUTemplate:Uninitialized HtEnabled:0xc000436053 MemSizeMib:0xc000436048 VcpuCount:0xc000436040} 
 INFO[0000] PutGuestBootSource: [PUT /boot-source][204] putGuestBootSourceNoContent  
 INFO[0000] Attaching drive /dev/zvol/zpool/vms, slot 1, root true. 
 INFO[0000] Attached drive /dev/zvol/zpool/vms: [PUT /drives/{drive_id}][204] putGuestDriveByIdNoContent  
 INFO[0000] startInstance successful: [PUT /actions][204] createSyncActionNoContent  
 [    0.000000] Linux version 4.14.55-84.37.amzn2.x86_64 (mockbuild@ip-10-0-1-79) (gcc version 7.3.1 20180303 (Red Hat 7.3.1-5) (GCC)) #1 SMP Wed Jul 25 18:47:15 UTC 2018
 [    0.000000] Command line: ro console=ttyS0 noapic reboot=k panic=1 pci=off nomodules  root=/dev/vda virtio_mmio.device=4K@0xd0000000:5
 [    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
 [    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
 [    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
 [    0.000000] x86/fpu: Supporting XSAVE feature 0x008: 'MPX bounds registers'
 [    0.000000] x86/fpu: Supporting XSAVE feature 0x010: 'MPX CSR'
 [    0.000000] x86/fpu: Enabled xstate features 0x1f, context size is 960 bytes, using 'compacted' format.
 [    0.000000] e820: BIOS-provided physical RAM map:
 [    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
 [    0.000000] BIOS-e820: [mem 0x0000000000100000-0x000000001fffffff] usable
 [    0.000000] NX (Execute Disable) protection: active
 [    0.000000] DMI not present or invalid.
 [    0.000000] Hypervisor detected: KVM
 [    0.000000] tsc: Using PIT calibration value
 [    0.000000] e820: last_pfn = 0x20000 max_arch_pfn = 0x400000000
 [    0.000000] MTRR: Disabled
 [    0.000000] x86/PAT: MTRRs disabled, skipping PAT initialization too.
 [    0.000000] CPU MTRRs all blank - virtualized system.
 [    0.000000] x86/PAT: Configuration [0-7]: WB  WT  UC- UC  WB  WT  UC- UC  
 [    0.000000] found SMP MP-table at [mem 0x0009fc00-0x0009fc0f] mapped at [ffffffffff200c00]
 [    0.000000] Scanning 1 areas for low memory corruption
 [    0.000000] No NUMA configuration found
 [    0.000000] Faking a node at [mem 0x0000000000000000-0x000000001fffffff]
 [    0.000000] NODE_DATA(0) allocated [mem 0x1ffde000-0x1fffffff]
 [    0.000000] kvm-clock: Using msrs 4b564d01 and 4b564d00
 [    0.000000] kvm-clock: cpu 0, msr 0:1ffdc001, primary cpu clock
 [    0.000000] kvm-clock: using sched offset of 299586726 cycles
 [    0.000000] clocksource: kvm-clock: mask: 0xffffffffffffffff max_cycles: 0x1cd42e4dffb, max_idle_ns: 881590591483 ns
 [    0.000000] Zone ranges:
 [    0.000000]   DMA      [mem 0x0000000000001000-0x0000000000ffffff]
 [    0.000000]   DMA32    [mem 0x0000000001000000-0x000000001fffffff]
 [    0.000000]   Normal   empty
 [    0.000000] Movable zone start for each node
 [    0.000000] Early memory node ranges
 [    0.000000]   node   0: [mem 0x0000000000001000-0x000000000009efff]
 [    0.000000]   node   0: [mem 0x0000000000100000-0x000000001fffffff]
 [    0.000000] Initmem setup node 0 [mem 0x0000000000001000-0x000000001fffffff]
 [    0.000000] Intel MultiProcessor Specification v1.4
 [    0.000000] MPTABLE: OEM ID: FC      
 [    0.000000] MPTABLE: Product ID: 000000000000
 [    0.000000] MPTABLE: APIC at: 0xFEE00000
 [    0.000000] Processor #0 (Bootup-CPU)
 [    0.000000] IOAPIC[0]: apic_id 2, version 17, address 0xfec00000, GSI 0-23
 [    0.000000] Processors: 1
 [    0.000000] smpboot: Allowing 1 CPUs, 0 hotplug CPUs
 [    0.000000] PM: Registered nosave memory: [mem 0x00000000-0x00000fff]
 [    0.000000] PM: Registered nosave memory: [mem 0x0009f000-0x000fffff]
 [    0.000000] e820: [mem 0x20000000-0xffffffff] available for PCI devices
 [    0.000000] Booting paravirtualized kernel on KVM
 [    0.000000] clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
 [    0.000000] random: get_random_bytes called from start_kernel+0x94/0x486 with crng_init=0
 [    0.000000] setup_percpu: NR_CPUS:128 nr_cpumask_bits:128 nr_cpu_ids:1 nr_node_ids:1
 [    0.000000] percpu: Embedded 41 pages/cpu @ffff88001fc00000 s128728 r8192 d31016 u2097152
 [    0.000000] KVM setup async PF for cpu 0
 [    0.000000] kvm-stealtime: cpu 0, msr 1fc15040
 [    0.000000] PV qspinlock hash table entries: 256 (order: 0, 4096 bytes)
 [    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 128905
 [    0.000000] Policy zone: DMA32
 [    0.000000] Kernel command line: ro console=ttyS0 noapic reboot=k panic=1 pci=off nomodules  root=/dev/vda virtio_mmio.device=4K@0xd0000000:5
 [    0.000000] PID hash table entries: 2048 (order: 2, 16384 bytes)
 [    0.000000] Memory: 498120K/523896K available (8204K kernel code, 622K rwdata, 1464K rodata, 1268K init, 2820K bss, 25776K reserved, 0K cma-reserved)
 ```
If we need more space, just resize the zvol  
  
  
```bash  

alpine-build:~/firecracker/firecracker$ sudo zfs set volsize=2G zpool/test2
alpine-build:~/firecracker/firecracker$ sudo resize2fs /dev/zvol/zpool/test2
 resize2fs 1.44.5 (15-Dec-2018)
 Please run 'e2fsck -f /dev/zvol/zpool/test2' first.
alpine-build:~/firecracker/firecracker$ sudo e2fsck -f  /dev/zvol/zpool/test2

e2fsck 1.44.5 (15-Dec-2018)

Pass 1: Checking inodes, blocks, and sizes

Pass 2: Checking directory structure

Pass 3: Checking directory connectivity

Pass 4: Checking reference counts

Pass 5: Checking group summary information

/dev/zvol/zpool/test2: 15275/65536 files (1.5% non-contiguous), 128736/262144 blocks

alpine-build:~/firecracker/firecracker$ sudo resize2fs /dev/zvol/zpool/test2

resize2fs 1.44.5 (15-Dec-2018)

Resizing the filesystem on /dev/zvol/zpool/test2 to 524288 (4k) blocks.

The filesystem on /dev/zvol/zpool/test2 is now 524288 (4k) blocks long.
```
# Ope4nVZ (Virtuozzo) 7 Templates

Here are pre-made openvz images that are useful for firecracker.  
```bash
wget   http://mirror.whatuptime.com/sc72wyn2/openvz7/template/ct/ubuntu-18.04-x86_64.tar.gz  -O /vz/template/cache/ubuntu-18.04-x86_64.tar.gz 
```
