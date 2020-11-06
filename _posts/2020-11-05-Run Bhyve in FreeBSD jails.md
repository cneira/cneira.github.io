# Table of Contents

1.  [How to run bhyve in a jail](#orgf2f80ce)
    1.  [Install bastillebsd to create and manage jails.](#orgeced498)
    2.  [Setup bastillebsd](#org11aee24)
    3.  [Issues](#orgd919e99)
    4.  [References](#org29ec288)

<a id="orgf2f80ce"></a>

# How to run bhyve in a jail

I&rsquo;ll setup a jail dedicated to run bhyve vms , for jail creation I&rsquo;ll use bastillebsd

Steps:

<a id="orgeced498"></a>

## Install bastillebsd to create and manage jails.

    # pkg install bastillebsd

<a id="org11aee24"></a>

## Setup bastillebsd

Follow the getting started guide at <https://bastillebsd.org/getting-started/>
I&rsquo;m using zfs so /usr/local/etc/bastille/bastille.conf I must edit bastille.conf (this must be done before bootstraping a release).

I used this on bastille.conf.

    bastille_zfs_enable="YES"
    bastille_zfs_zpool="zroot"

Create a set of rules to allow to run bhyve inside a jail edit /etc/devfs.rules, create it if does not exists.

    [devfs_rules_bhyve_jail=25]
    add include $devfsrules_jail
    add path vmm unhide
    add path vmm/* unhide
    add path tap* unhide
    add path nmdm* unhide

Create a new jail that will be use these rules.

    sudo bastille create --vnet test-bhyve 12.2-RELEASE 192.168.1.225 em0

Modify test-bhyve jail.conf for this jail:

    sudo bastille edit test-bhyve

Now add

    allow.vmm;

So the jail.conf will look like:

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

Load the required modules:

    kldload vmm
    kldload nmdm

Start test-bhyve jail

    sudo bastille start test-bhyve

Go inside the new jail

    sudo bastille console test-bhyve

Now install vm-bhyve inside the jail follow the vm-bhyve setup at <https://github.com/churchers/vm-bhyve>

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

by default a vm-bhyve vm has 256mb, if you need more you will need to run

    vm config test

And configure how much RAM do you need.

<a id="orgd919e99"></a>

## Issues

If you are going to use dhcp on the vm, you will need to configure the interface to
use SYNCDHCP.
According to this post using SYNCDHCP works, but we need to reboot the vm first.

> This gives me an interface named vnet0 in my jails that i can then configure through the jail&rsquo;s rc.conf. for some reason SYNCDHCP works but not DHCP in the guest rc.conf.

<a id="org29ec288"></a>

## References

- <https://github.com/lattera/articles/blob/master/freebsd/2018-10-27_jailed_bhyve/article.md>

- <https://github.com/churchers/vm-bhyve/issues/267>

- <https://bastillebsd.org/getting-started/>

- <https://github.com/churchers/vm-bhyve>
