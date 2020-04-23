A couple of days ago, I was reading the pksrc build documents https://github.com/joyent/pkgsrc/wiki/pkgdev:setup
and thought that would be really useful to import the pkgsrc image d001d2b4-3157-11ea-832d-df421d070030 into a zone with
Omniosce, asking on irc andyf on #omnios help me right away and explained that was a 'illumos' branded zone, that 
could be use to host other illumos distros, after trying that  it failed but he found out that some customizations
must be done, here are the steps provided by andyf.
First you need to get the joyent image with 
```
% curl -o /data/iso/pkgsrc.zss.gz https://images.joyent.com/images/d001d2b4-3157-11ea-832d-df421d070030/file
```

```
zonename: tt
zonepath: /data/zone/tt
brand: illumos
autoboot: false
bootargs:
pool:
limitpriv: default
scheduling-class:
ip-type: exclusive
hostid:
fs-allowed:
fs:
        dir: /usr
        special: /usr
        type: lofs
        options: [ro]
fs:
        dir: /sbin
        special: /sbin
        type: lofs
        options: [ro]
fs:
        dir: /lib
        special: /lib
        type: lofs
        options: [ro]
net:
        global-nic: switch10
        physical: tt0
```

```
# zonecfg -z tt export
create -b
set zonepath=/data/zone/tt
set brand=illumos
set autoboot=false
set limitpriv=default
set ip-type=exclusive
add fs
set dir="/usr"
set special="/usr"
set type="lofs"
add options ro
end
add fs
set dir="/sbin"
set special="/sbin"
set type="lofs"
add options ro
end
add fs
set dir="/lib"
set special="/lib"
set type="lofs"
add options ro
end
add net
set physical="tt0"
set global-nic="switch10"
end
```

```
# zoneadm -z tt install -s /data/iso/pkgsrc.zss.gz
A ZFS file system has been created for this zone.
Installing zone from ZFS stream /data/iso/pkgsrc.zss.gz
```

```
reaper# cd /data/zone/tt/root
reaper# rm -rf ccs config cores dev local site lastbooted
reaper# mv root l
reaper# mv l/* .
reaper# rm -rf l
reaper# chmod 755 .
reaper#
reaper# zoneadm -z tt boot
reaper# zlogin tt
[Connected to zone 'tt' pts/12]
   __        .                   .
 _|  |_      | .-. .  . .-. :--. |-
|_    _|     ;|   ||  |(.-' |  | |
  |__|   `--'  `-' `;-| `-' '  ' `-'
                   /  ; Instance (pkgbuild 19.4.0)
                   `-'  https://docs.joyent.com/images/smartos/pkgbuild

[root@tt ~]#
```
